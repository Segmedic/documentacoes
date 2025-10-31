# Automação: Annual Renewal

## 📋 Visão Geral

A automação **Annual Renewal** é responsável por identificar e processar assinaturas de planos anuais que precisam ser renovados, criando ou atualizando negócios (deals) no RD Station CRM para permitir o acompanhamento e gestão comercial da renovação anual.

## 🎯 Objetivo

Automatizar o processo de identificação de clientes com planos anuais cujo último pagamento ocorreu no mesmo mês do ano anterior, criando registros no CRM para ação comercial de renovação antes do vencimento do plano.

## ⚙️ Configuração

### Serverless Framework

```yaml
# Schedule Function
functions:
  annual:
    timeout: 900
    memorySize: 168
    handler: src/schedules/annualRennewal.handler
    events:
      - schedule: cron(0 3 1 * ? *)  # Todo dia 1º de cada mês às 3h UTC

# Worker Queue
constructs:
  annual:
    type: queue
    delay: 30
    batchSize: 1
    maxConcurrency: 2
    worker:
      reservedConcurrency: 2
      timeout: 180
      memorySize: 168
      handler: src/workers/annualRennewal.handler
```

### Agendamento

- **Frequência**: Todo dia 1º de cada mês às 3h UTC
- **Cron Expression**: `cron(0 3 1 * ? *)`
- **Timezone**: UTC (Universal Time Coordinated)
- **Lógica**: Identifica planos que foram pagos no mesmo mês do ano anterior

## 🔄 Fluxo de Execução

### 1. Schedule (Agendador)

**Arquivo**: `src/schedules/annualRennewal.ts`

- Conecta ao banco de dados DNA
- Busca planos anuais com último pagamento até o ano anterior via `listAnnualPlan(lastYear)`
- Filtra planos onde:
  - O último pagamento ocorreu no **mesmo mês atual** do **ano anterior**
  - Status **diferente** de `CANCELED`
- Envia cada plano elegível para a fila SQS

### 2. Worker (Processador)

**Arquivo**: `src/workers/annualRennewal.ts`

- Consome mensagens da fila SQS
- Para cada plano:
  - Busca dados completos (titular, plano, faturas, cartão)
  - Processa e cria/atualiza deal no RD Station
  - Registra logs de sucesso ou erro
  - Aguarda 5 segundos entre processamentos

### 3. Use Case (Lógica de Negócio)

**Arquivo**: `src/use-cases/annualRennewal-worker.ts`

- Monta o objeto DealDTO com todas as informações necessárias
- Calcula dias de inadimplência (se houver faturas abertas)
- Define stage inicial do funil específico para renovação anual
- Adiciona campos customizados (CPF, forma de pagamento, score, etc.)
- Inclui informações de códigos de retorno (quando disponível)

## 📊 Critérios de Seleção

### Query SQL - Principais Filtros

```sql
WHERE 
  p.name LIKE '%ANUAL%'                         -- Apenas planos anuais
  AND i.status <> 'CANCELED'                    -- Exclui faturas canceladas
HAVING 
  YEAR(MAX(i.payment_date)) <= lastYear         -- Último pagamento até ano anterior
```

### Filtro Adicional (Código)

- **Verificação de mês**: `monthOfLastPayament === actualyMonth`
- **Status da assinatura**: `status !== "CANCELED"`
- **Lógica combinada**: Último pagamento no mesmo mês do ano anterior + assinatura ativa

### Exemplo Prático

Se estamos em **outubro de 2025**:

- Busca planos com último pagamento em 2024 ou antes
- Filtra apenas os que pagaram em **outubro de 2024**
- Esses são os candidatos à renovação anual

## 🏗️ Estrutura de Dados

### Input (Annual Plan)

```typescript
{
  idholder: number;
  idplan: number;
  idsubscription: number;
  status: 'OK' | 'CANCELED' | 'BLOCKED';
  date_last_competence: number | null;  // Formato YYYYMM
  payment_type: string;
  price_holder: string;
  data_begin: string;
  data_begin_fatura: string;
  data_end_fatura: string;
  name: string;
  cellphone: string;
  ult_pg: Date;                         // Data do último pagamento
  transact_nsu: string | null;
  faturas_pg: number;                   // Quantidade de faturas pagas
  parcelas: number | null;
}
```

### Output (Deal no RD Station)

```typescript
{
  assinatura: string;           // Ex: "12345-anual"
  titular: {
    email: string;
    nome: string;
    celular: string;
    telefone: string;
  };
  tipo_pagamento: string;
  stage: string;               // Stage inicial do funil de renovação
  classificacao: string;       // Baseado em score e tipo de pagamento
  custom_fields: CustomField[];
  anotacoes: string[];         // Links e informações de faturas
  openedInvoices: Invoice[];   // Faturas em aberto (se houver)
}
```

## 🔍 Campos Customizados (RD Station)

| Campo | ID | Descrição |
|-------|-----|-----------|
| Tempo Inadimplente | `OP_TEMPO_INADIMPLENTE` | Dias de atraso (se houver fatura aberta) |
| CPF | `OP_CPF` | CPF do titular |
| Forma de Pagamento | `OP_FORMA_PAGAMENTO` | Tipo de pagamento |
| Plano | `OP_PLANO` | Nome do plano anual |
| Status Assinatura | `OP_STATUS_ASSINATURA` | Status atual |
| Código Retorno | `OP_CODIGO_RETORNO` | Código ABECS (se aplicável) |
| Score | `OP_SCORE` | Score do cartão (se disponível) |
| Mensagem Erro | `MSG_ERRO` | Descrição do erro ABECS |

## 📝 Anotações Criadas

1. **Link Backoffice**: URL direta para assinatura no sistema
   - Formato: `https://backoffice.clubflex.com.br/assinatura/{idsubscription}`

2. **Lista de Faturas**: IDs das faturas abertas (se houver)
   - Formato: `Fatura - {idinvoice}`

## 🚨 Tratamento de Erros

### Erros Esperados

- **Titular ou Plano não encontrado**: Log + continue para próximo
- **Falha ao processar lead**: Log + continue para próximo
- **Erros gerais**: Log do erro + fecha conexão + aguarda 5s

### Estratégia de Retry

- SQS gerencia automaticamente retries
- `maxConcurrency: 2` evita sobrecarga
- Delay de 30s entre tentativas
- 5s de espera entre processamentos

## 📈 Métricas e Monitoramento

### Logs Importantes

```typescript
// Sucesso
console.log(`Deal processado: ${id}`);

// Avisos
console.log(`Falta dados para a assinatura ${idsubscription}.`);
console.log(`Falha ao processar lead para a assinatura ${idsubscription}.`);

// Erros
console.error(`Erro ao processar assinatura ${idsubscription}:`, err);
```

### Recursos AWS

- **Lambda Schedule**: 900s timeout, 168MB RAM
- **Lambda Worker**: 180s timeout, 168MB RAM
- **SQS Queue**: Delay 30s, batchSize 1

## 🔗 Dependências

### Internas

- `DnaRepository`: Acesso aos dados do banco
- `RdService`: Integração com RD Station CRM
- `AnnuaRennewallUseCase`: Lógica de negócio
- `date.ts`: Utilitários de data (lastYear, actualyMonth)

### Externas

- `mysql2/promise`: Conexão com banco de dados
- `date-fns`: Manipulação de datas
- `aws-sdk`: SQS para filas

### Recursos

- **Banco de Dados**: MySQL (DNA)
- **CRM**: RD Station
- **Fila**: AWS SQS
- **Execução**: AWS Lambda

## 🎨 Stages do Funil

A constante `FIRST_STAGE_ANNUAL_RENNEWAL` define o estágio inicial no funil do RD Station onde os deals de renovação anual são criados.

## 🔄 Classificação de Deals

A classificação é determinada pela função `classification()` que considera:

- Tipo de pagamento do plano anual
- Score do cartão (quando disponível)

## 📅 Lógica de Renovação

### Identificação de Candidatos

A automação usa uma lógica inteligente para identificar planos que devem ser renovados:

1. **Busca Ampla**: Recupera todos os planos anuais com último pagamento até o ano anterior
2. **Filtro Mensal**: Identifica apenas os planos cujo último pagamento foi no mesmo mês atual do ano anterior
3. **Exemplo**:
   - **Hoje**: 1º de outubro de 2025
   - **Busca**: Planos com último pagamento ≤ 2024
   - **Filtra**: Apenas os que pagaram em outubro de 2024
   - **Resultado**: Planos que vencem em outubro de 2025

### Vantagens desta Abordagem

- ⏰ **Antecipação**: Identifica renovações no início do mês
- 🎯 **Precisão**: Foca apenas nos planos do mês corrente
- 📊 **Previsibilidade**: Execução mensal consistente
- 🔄 **Renovação Proativa**: Permite ação comercial antes do vencimento

## 📋 Diferenças em Relação ao Tickets Renewal

| Aspecto | Annual Renewal | Tickets Renewal |
|---------|----------------|-----------------|
| **Frequência** | Mensal (dia 1º) | Semanal (segundas) |
| **Identificador** | `{id}-anual` | `{id}-ticket` |
| **Critério Principal** | Mesmo mês do ano anterior | Últimos 4 meses |
| **Stage** | `FIRST_STAGE_ANNUAL_RENNEWAL` | `FIRST_STAGE_TICKET_GENERATE` |
| **Objetivo** | Renovação anual proativa | Cobrança de carnês atrasados |
| **Tipo de Plano** | LIKE '%ANUAL%' | LIKE '%CARNE%' |

## 📋 Próximos Passos

Após a criação/atualização do deal no RD Station:

1. Equipe comercial visualiza no pipeline
2. Ações de renovação são realizadas proativamente
3. Deal avança pelos estágios do funil conforme interações
4. Cliente é contatado para renovação antes do vencimento

## 🛠️ Manutenção

### Pontos de Atenção

- Validar query SQL periodicamente
- Monitorar taxa de sucesso/erro
- Verificar se a lógica de mês está funcionando corretamente
- Acompanhar limites de API do RD Station
- Verificar custos de Lambda e SQS

### Melhorias Futuras

- Adicionar métricas detalhadas (CloudWatch)
- Implementar DLQ (Dead Letter Queue)
- Criar dashboard de monitoramento
- Adicionar alertas automáticos
- Considerar antecipação para 15 dias antes do vencimento
- Implementar notificações automáticas para equipe comercial

## 🔍 Troubleshooting

### Plano não apareceu no CRM

**Verificações**:

1. Plano tem "ANUAL" no nome?
2. Último pagamento foi no mesmo mês do ano anterior?
3. Status é diferente de CANCELED?
4. Logs indicam processamento?

### Deal criado no mês errado

**Possível causa**: Timezone UTC vs horário local
**Solução**: Verificar se execução ocorre após meia-noite UTC

### Muitos ou poucos deals criados

**Diagnóstico**:

- Revisar query SQL e filtros de data
- Validar função `actualyMonth` e `lastYear`
- Verificar dados de payment_date no banco

## 💡 Observações Importantes

- A automação é executada sempre no **dia 1º de cada mês**
- Foca em **renovações proativas**, não em cobranças atrasadas
- Ideal para manter relacionamento comercial antes do vencimento
- Permite preparar propostas de renovação com antecedência
