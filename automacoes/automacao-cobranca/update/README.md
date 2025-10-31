# Automação: Update

## 📋 Visão Geral

A automação **Update** é responsável por **sincronizar o status dos deals no RD Station CRM** com a situação real das assinaturas no banco de dados. Ela identifica deals que foram regularizados (sem faturas abertas) ou estão no último dia do mês, marcando-os como perdidos (lost) no CRM para manter o funil limpo e atualizado.

## 🎯 Objetivo

Automatizar o processo de limpeza e atualização do CRM, removendo do funil ativo deals de assinaturas que:

1. **Foram regularizadas**: Não possuem mais faturas em aberto
2. **Último dia do mês**: Marcadas para limpeza mensal do funil

Esta automação garante que o CRM reflita apenas inadimplências ativas, evitando trabalho desnecessário da equipe de cobrança.

## ⚙️ Configuração

### Serverless Framework

```yaml
# Schedule Function
functions:
  update:
    timeout: 900
    memorySize: 416
    handler: src/schedules/update.handler
    events:
      - schedule: cron(0 22 ? * * *)  # Todos os dias às 22h UTC

# Worker Queue
constructs:
  update:
    type: queue
    delay: 30
    batchSize: 1
    maxConcurrency: 2
    worker:
      reservedConcurrency: 2
      timeout: 120
      memorySize: 256
      handler: src/workers/update.handler
```

### Agendamento

- **Frequência**: Diariamente às 22h UTC (19h horário de Brasília)
- **Cron Expression**: `cron(0 22 ? * * *)`
- **Timezone**: UTC (Universal Time Coordinated)
- **Horário estratégico**: Fim do dia para processar regularizações

## 🔄 Fluxo de Execução

### 1. Schedule (Agendador)

**Arquivo**: `src/schedules/update.ts`

**Processo complexo de coleta de deals**:

1. Conecta à API do RD Station
2. **Busca deals de todos os 4 pipelines/stages**:
   - Pipeline 1-60 dias
   - Pipeline 61-180 dias
   - Pipeline 181-365 dias
   - Pipeline 366+ dias
3. Para cada pipeline/stage:
   - Pagina resultados (50 deals por página)
   - Aguarda 2s entre páginas (rate limiting)
   - Limite de segurança: 49 páginas por stage
4. **Extrai subscription ID** do nome da organização
5. Cria payload `UpdateDTO` com dealId e subscriptionId
6. Envia para fila SQS

**Características**:

- Processo **inverso** das outras automações (CRM → Banco)
- Coleta massiva de deals do CRM
- Aguarda 3s entre pipelines diferentes
- Proteção contra limite de API (49 páginas)

### 2. Worker (Processador)

**Arquivo**: `src/workers/update.ts`

- Consome mensagens da fila SQS
- Valida payload (subscription + dealId)
- Para cada deal:
  - Busca faturas abertas no banco
  - Verifica se é último dia do mês
  - Busca status da assinatura
  - Decide se marca como lost
  - Fecha conexão sempre

### 3. Use Case (Lógica de Negócio)

**Arquivo**: `src/use-cases/update-worker.ts`

**Duas condições para marcar deal como perdido**:

1. **Último dia do mês** → Marca como lost (limpeza mensal)
2. **Sem faturas abertas** → Marca como lost (regularizado)

**Processo `lostLead`**:

- Busca deal atual no CRM
- Preserva todos os custom fields
- Atualiza campo `OP_STATUS_ASSINATURA` com status atual
- Marca `win: false` (deal perdido)
- Mantém user_id original

## 📊 Critérios de Atualização

### Deals Coletados do CRM

```typescript
// Busca de todos os pipelines
PIPE_1_60_DIAS      // Inadimplência recente
PIPE_61_180_DIAS    // Inadimplência média
PIPE_181_365_DIAS   // Inadimplência longa
PIPE_366_DIAS       // Inadimplência crítica
```

### Condições para Marcar como Lost

| Condição | Lógica | Ação |
|----------|--------|------|
| **Sem faturas abertas** | `openedInvoices.length == 0` | Marca lost (regularizado) |
| **Último dia do mês** | `today.getDate() === lastDayOfMonth` | Marca lost (limpeza) |

### Extração do Subscription ID

```typescript
// Extrai números do nome da organização
const subscription = deal.organization?.name
  ?.match(/[0-9]/g)
  ?.join("") as string;

// Valida se é número válido
if (!isNaN(data.subscription)) {
  // Processa
}
```

## 🏗️ Estrutura de Dados

### Input (UpdateDTO)

```typescript
{
  subscription: number;     // ID da assinatura extraído do CRM
  dealId: string;          // ID do deal no RD Station
}
```

### Processo de Lost Lead

```typescript
// Atualiza deal no CRM
{
  deal: {
    deal_custom_fields: [...customFields, updatedStatus],
    user_id: originalUserId,
    win: false              // Marca como perdido
  }
}
```

## 🔍 Campos Atualizados

### Atualização no CRM

| Campo | Ação | Descrição |
|-------|------|-----------|
| `deal_custom_fields` | Mantém todos + atualiza status | Preserva histórico |
| `OP_STATUS_ASSINATURA` | Atualiza com status atual | OK, CANCELED, BLOCKED |
| `user_id` | Mantém original | Responsável não muda |
| `win` | Define como `false` | Marca deal como perdido |

## 🚨 Tratamento de Erros

### Erros Esperados

- **Payload inválido**: Log + exceção
- **Subscription não é número**: Pula deal
- **DealId ausente**: Pula deal
- **Erro no processamento**: Log + exceção com subscription ID

### Estratégia de Retry

- SQS gerencia automaticamente retries
- `maxConcurrency: 2` evita sobrecarga
- Delay de 30s entre tentativas
- Conexão fechada sempre no finally
- Exceções re-lançadas para DLQ

## 📈 Métricas e Monitoramento

### Logs Importantes

```typescript
// Avisos
console.info(data);  // Payload inválido

// Proteção de API
console.log("Perto do limite da api, estamos parando.");
console.log('Fim das páginas. Não há mais deals.');

// Erros
console.error("falha assinatura: " + subscription);
throw new Error("payload invalido");
```

### Recursos AWS

- **Lambda Schedule**: 900s timeout, 416MB RAM (coleta massiva)
- **Lambda Worker**: 120s timeout, 256MB RAM
- **SQS Queue**: Delay 30s, batchSize 1

## 🔗 Dependências

### Internas

- `DnaRepository`: Acesso ao banco de dados
- `RdService`: Integração com RD Station CRM
- `RdClient`: Cliente direto da API do RD
- `UpdateUseCase`: Lógica de negócio
- `lostLead()`: Marca deal como perdido

### Externas

- `mysql2/promise`: Conexão com banco de dados
- `date-fns`: Manipulação de datas (lastDayOfMonth)
- `aws-sdk`: SQS para filas

### Recursos

- **Banco de Dados**: MySQL (DNA)
- **CRM**: RD Station (leitura massiva + atualização)
- **Fila**: AWS SQS
- **Execução**: AWS Lambda

## 🔄 Diferenças em Relação às Outras Automações

| Aspecto | Update | Sync/Tickets/Annual |
|---------|--------|---------------------|
| **Direção** | CRM → Banco | Banco → CRM |
| **Objetivo** | Limpar funil | Criar/atualizar deals |
| **Ação** | Marcar lost | Criar/mover deals |
| **Horário** | 22h (fim do dia) | 3h (início do dia) |
| **Fonte** | API RD Station | Banco DNA |
| **Volume** | Alto (todos deals) | Variável |
| **Operação** | Leitura massiva + update | Leitura banco + create/update |

## 💡 Regras Especiais

### 1. Limpeza no Último Dia do Mês

**Lógica**:

```typescript
const today = new Date()
const dayOfToday = today.getDate()
const lastDayMonth = lastDayOfMonth(today).getDate()
const isLastDayOfMonth = dayOfToday === lastDayMonth
```

**Motivo**: Limpeza mensal do funil, independente do status real

**Exemplo**:

- 28/fev → Marca lost
- 30/abr → Marca lost
- 31/out → Marca lost

### 2. Regularização (Sem Faturas Abertas)

**Lógica**:

```typescript
if (openedInvoices.length == 0) {
  await rdService.lostLead(dealId, status);
}
```

**Motivo**: Cliente regularizou pagamento, não precisa mais estar no funil

### 3. Preservação de Dados

Ao marcar como lost:

- ✅ Mantém todos os custom fields
- ✅ Atualiza apenas status da assinatura
- ✅ Preserva user_id (responsável)
- ✅ Mantém histórico completo

### 4. Proteção de Rate Limiting

```typescript
// Entre páginas do mesmo stage
await new Promise((resolve) => setTimeout(resolve, 2000));

// Entre diferentes pipelines
await new Promise((resolve) => setTimeout(resolve, 3000));

// Limite de segurança
if(page === 49) {
  console.log("Perto do limite da api, estamos parando.");
  break;
}
```

## 📋 Próximos Passos

Após marcar deal como lost:

1. Deal sai do funil ativo
2. Equipe não visualiza mais no pipeline
3. Relatórios mostram como lost
4. Status preservado para análise histórica
5. Pode ser reaberto manualmente se necessário

## 🛠️ Manutenção

### Pontos de Atenção

- ✅ Monitorar tempo de coleta de deals (pode ser longo)
- ✅ Validar rate limiting da API RD Station
- ✅ Acompanhar taxa de deals marcados como lost
- ✅ Verificar extração correta do subscription ID
- ✅ Monitorar limite de 49 páginas por stage
- ✅ Validar lógica de último dia do mês
- ✅ Acompanhar custos AWS (muitos deals)

### Melhorias Futuras

- [ ] Cache de deals já processados (evitar duplicados)
- [ ] Métricas de quantidade de deals por pipeline
- [ ] Dashboard de taxa de regularização
- [ ] Alertas para volume anormal de lost
- [ ] Otimizar paginação (paralelizar stages)
- [ ] Adicionar DLQ (Dead Letter Queue)
- [ ] Implementar idempotência no processamento
- [ ] Logs estruturados com mais detalhes
- [ ] Considerar horário diferente para último dia do mês

## 🔍 Troubleshooting

### Problema: Deals não sendo marcados como lost

**Verifique**:

1. ✅ Schedule está executando às 22h?
2. ✅ Coleta de deals está funcionando?
3. ✅ Extração do subscription ID está correta?
4. ✅ Faturas realmente não existem no banco?
5. ✅ É realmente o último dia do mês?
6. ✅ API do RD Station está respondendo?

### Problema: Timeout no schedule

**Causas**:

- Muitos deals para coletar (> 10.000)
- API do RD Station lenta
- Rate limiting ativado

**Soluções**:

- Aumentar timeout da Lambda schedule
- Reduzir delay entre páginas (cuidado!)
- Processar pipelines em paralelo
- Dividir em múltiplas execuções

### Problema: Subscription ID não extraído

**Causa**: Nome da organização sem números ou formato diferente

**Exemplo válido**: `Assinatura 12345` → `12345`

**Exemplo inválido**: `Cliente João` → `null`

**Solução**: Padronizar nomenclatura no CRM

### Problema: Deals regularizados voltando ao funil

**Causa**: Sync cria deal após Update marcar lost

**Motivo**: Sync roda às 3h, Update às 22h

**Solução Normal**: Sync detecta que não há faturas e não recria

### Problema: Rate limiting da API

**Sintomas**:

- Timeouts frequentes
- Erros 429 (Too Many Requests)
- Demora excessiva na coleta

**Ações**:

- Aumentar delays entre requisições
- Reduzir maxConcurrency
- Contatar suporte RD Station para limites

## 📊 Análise de Volume

### Estimativa de Processamento

**Coleta (Schedule)**:

- 4 pipelines × até 49 páginas = até 196 requisições
- 50 deals/página = até 9.800 deals
- Delays: 2s entre páginas + 3s entre pipelines
- **Tempo estimado**: 6-8 minutos para coleta completa

**Processamento (Worker)**:

- Concorrência: 2 workers
- Delay: 0s (interno no finally)
- **Throughput**: ~120 deals/minuto
- **Capacidade**: Processa 9.800 deals em ~80 minutos

### Recomendações por Volume

| Total de Deals | Ação |
|----------------|------|
| < 1.000 | ✅ Configuração OK |
| 1.000-5.000 | ⚠️ Monitorar tempos |
| 5.000-10.000 | 🔶 Considerar otimizações |
| > 10.000 | 🔴 Reavaliar estratégia |

## 🎯 Importância da Automação

O **Update** é crucial porque:

1. 🧹 **Limpeza do funil**: Remove deals regularizados
2. 📊 **Dados confiáveis**: CRM reflete situação real
3. ⚡ **Eficiência da equipe**: Foco em inadimplências reais
4. 📈 **Métricas precisas**: Relatórios mostram situação verdadeira
5. 🔄 **Complementa Sync**: Trabalham juntos (criar + limpar)
6. 📅 **Limpeza mensal**: Último dia do mês reorganiza funil

## 🔄 Relação com Sync

**Ciclo Completo**:

``` bash
Sync (3h)          Update (22h)
   ↓                   ↓
Cria deals    →   Limpa deals regularizados
   ↓                   ↓
Atualiza      →   Marca lost se sem faturas
   ↓                   ↓
Organiza      →   Remove do funil ativo
```

**Complementaridade**:

- **Sync**: Banco → CRM (criação/atualização)
- **Update**: CRM → Banco (validação/limpeza)
- Juntos mantém CRM sincronizado e limpo

---

**💡 Dica**: O Update é a "faxina" do CRM! Ele garante que apenas inadimplências ativas permaneçam no funil, evitando que a equipe perca tempo com deals já regularizados.

**⏰ Estratégia de Horário**: Roda às 22h (fim do dia) para processar todas as regularizações do dia, garantindo que o Sync da manhã seguinte (3h) trabalhe com um funil limpo.
