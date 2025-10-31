# Automação: Tickets Renewal

## 📋 Visão Geral

A automação **Tickets Renewal** é responsável por identificar e processar assinaturas com pagamento via carnê (tickets) que possuem faturas em aberto, criando ou atualizando negócios (deals) no RD Station CRM para permitir o acompanhamento e gestão da cobrança.

## 🎯 Objetivo

Automatizar o processo de identificação de clientes com pagamento via carnê que estão com faturas abertas há até 4 meses, criando registros no CRM para ação comercial de renovação e regularização.

## ⚙️ Configuração

### Serverless Framework

```yaml
# Schedule Function
functions:
  ticketsRennewal:
    timeout: 900
    memorySize: 168
    handler: src/schedules/ticketsRennewal.handler
    events:
      - schedule: cron(0 3 ? * 2 *)  # Toda segunda-feira às 3h UTC

# Worker Queue
constructs:
  ticketsRennewal:
    type: queue
    delay: 30
    batchSize: 1
    maxConcurrency: 2
    worker:
      reservedConcurrency: 2
      timeout: 180
      memorySize: 168
      handler: src/workers/ticketsRennewal.handler
```

### Agendamento

- **Frequência**: Toda segunda-feira às 3h UTC
- **Cron Expression**: `cron(0 3 ? * 2 *)`
- **Timezone**: UTC (Universal Time Coordinated)

## 🔄 Fluxo de Execução

### 1. Schedule (Agendador)

**Arquivo**: `src/schedules/ticketsRennewal.ts`

- Conecta ao banco de dados DNA
- Busca todas as assinaturas elegíveis via `listTicketToRennewal()`
- Filtra apenas assinaturas com última data de pagamento há 4 meses ou menos
- Envia cada assinatura para a fila SQS

### 2. Worker (Processador)

**Arquivo**: `src/workers/ticketsRennewal.ts`

- Consome mensagens da fila SQS
- Para cada assinatura:
  - Busca dados completos (titular, plano, faturas, cartão)
  - Processa e cria/atualiza deal no RD Station
  - Registra logs de sucesso ou erro
  - Aguarda 5 segundos entre processamentos

### 3. Use Case (Lógica de Negócio)

**Arquivo**: `src/use-cases/ticketRennewal-worker.ts`

- Monta o objeto DealDTO com todas as informações necessárias
- Calcula dias de inadimplência
- Define stage inicial do funil
- Adiciona campos customizados (CPF, forma de pagamento, score, etc.)
- Inclui informações de códigos de retorno (quando disponível)

## 📊 Critérios de Seleção

### Query SQL - Principais Filtros

```sql
WHERE 
  s.payment_type IN ('TICKET', 'TICKETS')      -- Apenas carnês
  AND s.status <> 'CANCELED'                    -- Exclui canceladas
  AND p.name LIKE '%CARNE%'                     -- Planos de carnê
HAVING 
  qtd_aberto <= 1                               -- Máximo 1 fatura aberta
```

### Filtro Adicional (Código)

- **Período de análise**: Últimos 4 meses desde o último pagamento
- **Lógica**: `differenceInMonths(new Date(), ult_pg) <= 4`

## 🏗️ Estrutura de Dados

### Input (Subscription)

```typescript
{
  idholder: number;
  idplan: number;
  idsubscription: number;
  status: string;
  payment_type: string;
  data_begin: Date;
  ult_pg: Date;          // Última data de pagamento
  qtd_aberto: number;    // Quantidade de faturas abertas
  // ... dados do titular
}
```

### Output (Deal no RD Station)

```typescript
{
  assinatura: string;           // Ex: "12345-ticket"
  titular: {
    email: string;
    nome: string;
    celular: string;
    telefone: string;
  };
  tipo_pagamento: string;
  stage: string;               // Stage inicial do funil
  classificacao: string;       // Baseado em score e tipo de pagamento
  custom_fields: CustomField[];
  anotacoes: string[];         // Links e informações de faturas
  openedInvoices: Invoice[];
}
```

## 🔍 Campos Customizados (RD Station)

| Campo | ID | Descrição |
|-------|-----|-----------|
| Tempo Inadimplente | `OP_TEMPO_INADIMPLENTE` | Dias de atraso da fatura |
| CPF | `OP_CPF` | CPF do titular |
| Forma de Pagamento | `OP_FORMA_PAGAMENTO` | Tipo de pagamento |
| Plano | `OP_PLANO` | Nome do plano |
| Status Assinatura | `OP_STATUS_ASSINATURA` | Status atual |
| Código Retorno | `OP_CODIGO_RETORNO` | Código ABECS (se aplicável) |
| Score | `OP_SCORE` | Score do cartão (se disponível) |
| Mensagem Erro | `MSG_ERRO` | Descrição do erro ABECS |

## 📝 Anotações Criadas

1. **Link Backoffice**: URL direta para assinatura no sistema
   - Formato: `https://backoffice.clubflex.com.br/assinatura/{idsubscription}`

2. **Lista de Faturas**: IDs das faturas abertas
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
- `TicketsRennewalUseCase`: Lógica de negócio

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

A constante `FIRST_STAGE_TICKET_GENERATE` define o estágio inicial no funil do RD Station onde os deals de tickets são criados.

## 🔄 Classificação de Deals

A classificação é determinada pela função `classification()` que considera:

- Tipo de pagamento (TICKET/TICKETS)
- Score do cartão (quando disponível)

## 📋 Próximos Passos

Após a criação/atualização do deal no RD Station:

1. Equipe comercial visualiza no pipeline
2. Ações de cobrança são realizadas
3. Deal avança pelos estágios do funil conforme interações
4. Atualizações são sincronizadas via webhook ou outras automações

## 🛠️ Manutenção

### Pontos de Atenção

- Validar query SQL periodicamente
- Monitorar taxa de sucesso/erro
- Ajustar período de 4 meses se necessário
- Verificar limites de API do RD Station
- Acompanhar custos de Lambda e SQS

### Melhorias Futuras

- Adicionar métricas detalhadas (CloudWatch)
- Implementar DLQ (Dead Letter Queue)
- Criar dashboard de monitoramento
- Adicionar alertas automáticos
