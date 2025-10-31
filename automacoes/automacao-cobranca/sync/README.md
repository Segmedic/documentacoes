# Automação: Sync

## 📋 Visão Geral

A automação **Sync** é responsável por sincronizar **todas as assinaturas inadimplentes** do sistema com o RD Station CRM, criando ou atualizando deals organizados por faixas de inadimplência (1-60 dias, 61-180 dias, 181-365 dias, 366+ dias). Esta é a automação principal de cobrança do sistema.

## 🎯 Objetivo

Automatizar o processo de identificação e sincronização diária de todas as assinaturas com faturas vencidas, mantendo o CRM atualizado com informações completas para ação da equipe de cobrança, organizando os deals em stages específicos baseados no tempo de inadimplência.

## ⚙️ Configuração

### Serverless Framework

```yaml
# Schedule Function
functions:
  sync:
    timeout: 900
    memorySize: 416
    handler: src/schedules/sync.handler
    events:
      - schedule: cron(0 3 ? * * *)  # Todos os dias às 3h UTC

# Worker Queue
constructs:
  sync:
    type: queue
    delay: 30
    batchSize: 1
    maxConcurrency: 2
    worker:
      reservedConcurrency: 2
      timeout: 180
      memorySize: 256
      handler: src/workers/sync.handler
```

### Agendamento

- **Frequência**: Diariamente às 3h UTC
- **Cron Expression**: `cron(0 3 ? * * *)`
- **Timezone**: UTC (Universal Time Coordinated)
- **Memória**: 416MB (maior que outras automações devido ao volume)

## 🔄 Fluxo de Execução

### 1. Schedule (Agendador)

**Arquivo**: `src/schedules/sync.ts`

- Conecta ao banco de dados DNA
- Busca **todas as assinaturas** com faturas vencidas via `listSubscription()`
- Envia cada assinatura para a fila SQS
- Processamento simples e direto (sem filtros adicionais)

### 2. Worker (Processador)

**Arquivo**: `src/workers/sync.ts`

- Consome mensagens da fila SQS
- Para cada assinatura:
  - Busca dados completos (titular, plano, faturas, cartão)
  - Valida se possui faturas abertas
  - Valida regra especial para tickets (mínimo 2 dias de atraso)
  - Processa e cria/atualiza deal no RD Station
  - Registra logs de sucesso ou erro
  - Aguarda 5 segundos entre processamentos

### 3. Use Case (Lógica de Negócio)

**Arquivo**: `src/use-cases/sync-worker.ts`

- Valida existência de faturas abertas (retorna vazio se não houver)
- Aplica regra especial: tickets/carnês com menos de 2 dias não são processados
- Calcula dias de inadimplência baseado na fatura mais antiga
- Define stage automaticamente por faixa de dias:
  - **1-60 dias**: `STAGES_1_60_DIAS`
  - **61-180 dias**: `STAGES_61_180_DIAS`
  - **181-365 dias**: `STAGES_181_365_DIAS`
  - **366+ dias**: `STAGES_366_DIAS`
- Adiciona campos customizados completos
- Inclui informações de códigos de retorno ABECS

## 📊 Critérios de Seleção

### Query SQL - Principais Filtros

```sql
WHERE 
  i.status = 'OPENED'                           -- Apenas faturas abertas
  AND i.due_date < CURRENT_DATE()               -- Vencidas (passado)
  AND s.waiting_first_pay = '0'                 -- Não aguardando primeiro pagamento
  AND s.status <> 'CANCELED'                    -- Assinatura ativa
  AND s.type_sub = 'PF'                         -- Pessoa Física
HAVING 
  DATEDIFF(CURRENT_DATE(), MIN(i.due_date)) <= 720  -- Até 2 anos de atraso
ORDER BY
  DATEDIFF(CURRENT_DATE(), MIN(i.due_date)) ASC     -- Mais recentes primeiro
```

### Validação Especial (Código)

**Regra para Tickets/Carnês**:

```typescript
// Não processa tickets/carnês DEFAULT com menos de 2 dias de atraso
if (payment_type IN ['TICKET', 'TICKETS'] && 
    invoice_type === 'DEFAULT' && 
    days <= 2) {
  throw Error("Cliente ainda não deve há 2 dias");
}
```

### Validação de Faturas Abertas

```typescript
// Não processa se não houver faturas abertas
if (invoices.length == 0) {
  return; // Pula assinatura
}
```

## 🏗️ Estrutura de Dados

### Input (Subscription)

```typescript
{
  idsubscription: number;
  idholder: number;
  idplan: number;
  payment_type: string;     // CREDIT_CARD, TICKET, TICKETS, etc.
  status: string;           // OK, BLOCKED, etc.
}
```

### Output (Deal no RD Station)

```typescript
{
  assinatura: string;           // Ex: "12345" (apenas ID)
  titular: {
    email: string;
    nome: string;
    celular: string;
    telefone: string;
  };
  tipo_pagamento: string;
  stage: string;               // Stage baseado em dias de inadimplência
  classificacao: string;       // Baseado em score e tipo de pagamento
  custom_fields: CustomField[];
  anotacoes: string[];         // Links e informações de faturas
  openedInvoices: Invoice[];
}
```

## 🔍 Campos Customizados (RD Station)

| Campo | ID | Descrição |
|-------|-----|-----------|
| Tempo Inadimplente | `OP_TEMPO_INADIMPLENTE` | Dias de atraso da fatura mais antiga |
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

2. **Lista de Faturas**: IDs de todas as faturas abertas
   - Formato: `Fatura - {idinvoice}` (uma por linha)

## 🎯 Stages Automáticos por Inadimplência

### Organização do Funil

A automação sync organiza automaticamente os deals em diferentes stages baseado no tempo de inadimplência:

| Dias de Atraso | Stage | Estratégia |
|----------------|-------|------------|
| **1-60 dias** | `STAGES_1_60_DIAS[0]` | Cobrança inicial/amigável |
| **61-180 dias** | `STAGES_61_180_DIAS[0]` | Cobrança intensificada |
| **181-365 dias** | `STAGES_181_365_DIAS[0]` | Cobrança crítica |
| **366+ dias** | `STAGES_366_DIAS[0]` | Cobrança judicial/encerramento |

### Lógica de Atribuição

```typescript
let stage = STAGES_1_60_DIAS;      // Padrão: 1-60 dias

if (days > 60) { stage = STAGES_61_180_DIAS }
if (days > 180) { stage = STAGES_181_365_DIAS }
if (days > 365) { stage = STAGES_366_DIAS }
```

## 🚨 Tratamento de Erros

### Erros Esperados

- **Titular ou Plano não encontrado**: Exceção + log + continue
- **Sem faturas abertas**: Log informativo + continue
- **Tickets com menos de 2 dias**: Exceção + continue
- **Falha ao processar lead**: Log + continue

### Estratégia de Retry

- SQS gerencia automaticamente retries
- `maxConcurrency: 2` evita sobrecarga
- Delay de 30s entre tentativas
- 5s de espera entre processamentos
- Conexão fechada sempre no finally

## 📈 Métricas e Monitoramento

### Logs Importantes

```typescript
// Sucesso implícito (ID retornado)

// Avisos
console.log(`Assinatura ${idsubscription} não possui faturas abertas ou não é válida.`);
console.log(`Falha ao processar lead para a assinatura ${idsubscription}.`);

// Erros
console.error(`Erro ao processar assinatura ${idsubscription}:`, err);

// Regra especial
throw Error("Cliente tem pagamento tipo ticket ou tickets por padrão mas ainda não deve há 2 dias");
```

### Recursos AWS

- **Lambda Schedule**: 900s timeout, 416MB RAM (maior memória)
- **Lambda Worker**: 180s timeout, 256MB RAM
- **SQS Queue**: Delay 30s, batchSize 1

## 🔗 Dependências

### Internas

- `DnaRepository`: Acesso aos dados do banco
- `RdService`: Integração com RD Station CRM
- `SyncUseCase`: Lógica de negócio e validações
- `classification()`: Classificação baseada em score
- `paymentTypeLabel()`: Tradução de tipo de pagamento

### Externas

- `mysql2/promise`: Conexão com banco de dados
- `date-fns`: Manipulação de datas (differenceInDays)
- `aws-sdk`: SQS para filas

### Recursos

- **Banco de Dados**: MySQL (DNA)
- **CRM**: RD Station
- **Fila**: AWS SQS
- **Execução**: AWS Lambda

## 🔄 Classificação de Deals

A classificação é determinada pela função `classification()` que considera:

- Tipo de pagamento (CREDIT_CARD, TICKET, TICKETS, etc.)
- Score do cartão (quando disponível)
- Combinação desses fatores para priorização

## 📋 Diferenças em Relação às Outras Automações

| Aspecto | Sync | Tickets Renewal | Annual Renewal |
|---------|------|-----------------|----------------|
| **Frequência** | Diária | Semanal (segundas) | Mensal (dia 1º) |
| **Escopo** | Todas inadimplências | Apenas carnês 4 meses | Renovações anuais |
| **Identificador** | `{id}` | `{id}-ticket` | `{id}-anual` |
| **Stage** | Dinâmico por dias | Fixo (tickets) | Fixo (renovação) |
| **Objetivo** | Cobrança geral | Cobrança carnês | Renovação proativa |
| **Memória** | 416MB / 256MB | 168MB / 168MB | 168MB / 168MB |
| **Volume** | Alto | Médio | Baixo |

## 💡 Regras Especiais

### 1. Limite de 2 Anos (720 dias)

Assinaturas com mais de 2 anos de inadimplência não são sincronizadas:

```sql
HAVING DATEDIFF(CURRENT_DATE(), MIN(i.due_date)) <= 720
```

### 2. Carência de 2 Dias para Tickets

Tickets/carnês DEFAULT precisam ter pelo menos 2 dias de atraso:

```typescript
if (typesPayment.includes(payment_type) && 
    invoice_type === 'DEFAULT' && 
    days <= 2) {
  // Não processa
}
```

**Motivo**: Evitar ação de cobrança prematura em carnês recém-emitidos.

### 3. Apenas Pessoa Física (PF)

```sql
WHERE s.type_sub = 'PF'
```

Pessoas jurídicas (PJ) não são processadas nesta automação.

### 4. Primeira Assinatura Aguardando Pagamento

```sql
WHERE s.waiting_first_pay = '0'
```

Assinaturas aguardando o primeiro pagamento não entram no fluxo.

## 📋 Próximos Passos

Após a criação/atualização do deal no RD Station:

1. Deal é posicionado no stage correto baseado em dias de atraso
2. Equipe de cobrança visualiza no pipeline específico
3. Ações de cobrança são realizadas conforme estágio
4. Deal pode avançar ou retroceder entre stages
5. Sistema atualiza diariamente a posição dos deals

## 🛠️ Manutenção

### Pontos de Atenção

- ✅ Monitorar volume diário processado
- ✅ Validar distribuição por stages
- ✅ Acompanhar taxa de sucesso/erro
- ✅ Verificar performance da query SQL (indexação)
- ✅ Monitorar uso de memória (maior alocação)
- ✅ Verificar limites de API do RD Station
- ✅ Acompanhar custos AWS (execução diária)

### Melhorias Futuras

- [ ] Reativar integração com DynamoDB (comentada no código)
- [ ] Adicionar métricas detalhadas por stage
- [ ] Implementar DLQ (Dead Letter Queue)
- [ ] Criar dashboard de monitoramento por faixa
- [ ] Adicionar alertas para volume anormal
- [ ] Otimizar query SQL com índices específicos
- [ ] Considerar processamento em lotes maiores
- [ ] Implementar cache para dados de planos/titulares

## 🔍 Troubleshooting

### Problema: Assinatura não apareceu no CRM

**Verifique**:

1. ✅ Possui faturas abertas?
2. ✅ Data de vencimento é anterior a hoje?
3. ✅ Status não é CANCELED?
4. ✅ Não está aguardando primeiro pagamento?
5. ✅ É pessoa física (PF)?
6. ✅ Atraso é menor que 720 dias?
7. ✅ Se for ticket/carnê DEFAULT, tem mais de 2 dias?

### Problema: Deal no stage errado

**Causa**: Dias de inadimplência mudaram

**Solução**: A próxima execução do sync atualizará o stage automaticamente baseado nos dias atuais.

### Problema: Volume muito alto

**Diagnóstico**:

- Verificar quantidade de assinaturas inadimplentes
- Analisar distribuição por faixas de dias
- Revisar limite de 720 dias se necessário

**Ações**:

- Considerar aumentar maxConcurrency
- Aumentar memória se necessário
- Implementar processamento em batches

### Problema: Timeout no schedule

**Causas possíveis**:

- Query SQL lenta
- Volume muito grande
- Problema de rede/banco

**Soluções**:

- Otimizar query com índices
- Aumentar timeout se necessário
- Verificar saúde do banco de dados

## 📊 Análise de Volume

### Estimativa de Processamento

Com configuração atual:

- **maxConcurrency**: 2
- **Delay entre registros**: 5s
- **Throughput**: ~24 assinaturas/minuto
- **Capacidade hora**: ~1.440 assinaturas

### Recomendações por Volume

| Assinaturas | Ação Recomendada |
|-------------|------------------|
| < 500 | Configuração atual OK |
| 500-1000 | Considerar aumentar concurrency para 3 |
| 1000-2000 | Aumentar para 4-5, revisar memória |
| > 2000 | Arquitetura de processamento em lote |

## 🎯 Importância da Automação

O **Sync** é a automação mais crítica do sistema porque:

1. 🔄 **Execução diária**: Mantém CRM sempre atualizado
2. 📊 **Cobertura total**: Processa todas as inadimplências
3. 🎯 **Organização automática**: Distribui por stages de cobrança
4. 💰 **Impacto financeiro direto**: Base para recuperação de receita
5. 📈 **Visão completa**: Permite análise de todo o funil de cobrança

---

**💡 Dica**: Esta é a automação central do sistema de cobrança. Monitore-a diariamente para garantir que todos os inadimplentes estejam sendo sincronizados corretamente!
