# Documentação Técnica - Automação de Convênios (Atendimentos)

## Visão Geral

A automação de convênios monitora agendamentos realizados através de convênios médicos na Feegow e cria oportunidades (deals) no RD Station CRM para acompanhamento. O sistema verifica periodicamente os atendimentos do dia e identifica consultas realizadas via convênio que foram efetivamente atendidas.

## Arquitetura

### Componentes Principais

1. **Schedule Function** (`src/schedules/atendimentos.ts`)
   - Função Lambda agendada via EventBridge
   - Execução a cada 15 minutos das 10h às 23h UTC (07h às 20h Brasília)
   - Timeout: 900 segundos (15 minutos)
   - Memória: 128 MB

2. **Worker Function** (`src/workers/atendimentos.ts`)
   - Função Lambda processadora de mensagens SQS
   - Timeout: 180 segundos (3 minutos)
   - Memória: 256 MB
   - Batch size: 1 mensagem por execução
   - Max concurrency: 2 execuções simultâneas
   - Reserved concurrency: 2

3. **DynamoDB Table** (`ATENDIMENTOS_CONVENIOS`)
   - Armazena histórico de agendamentos processados
   - TTL configurado para 1 dia
   - Evita processamento duplicado

4. **SQS Queue**
   - Delay de 15 segundos
   - Processamento com concorrência limitada (max 2)

## Fluxo de Execução

### Etapa 1: Monitoramento de Atendimentos (Schedule)

**Handler:** `src/schedules/atendimentos.ts`

```typescript
async function handler()
```

**Processo:**

1. Instancia cliente Feegow
2. Obtém data atual formatada (dd/MM/yyyy)
3. Busca agendamentos do dia via API Feegow
4. Escaneia tabela DynamoDB para identificar agendamentos já processados
5. Para cada agendamento:
   - Verifica se é um lead válido
   - Verifica se já foi processado
   - Se válido e não processado:
     - Envia para fila SQS
     - Salva ID no DynamoDB com TTL de 1 dia

**Configuração:**

- **Cron:** `cron(0/15 10-23 ? * * *)` - Executa a cada 15 minutos das 10h às 23h UTC
- **Environment:** `QUEUE_URL` - URL da fila SQS
- **Tabela DynamoDB:** `ATENDIMENTOS_CONVENIOS`

### Etapa 2: Processamento de Leads (Worker)

**Handler:** `src/workers/atendimentos.ts`

```typescript
async function handler(event: Event)
```

**Processo:**

1. Itera sobre cada mensagem SQS recebida
2. Faz parse do agendamento
3. Converte em Deal do RD Station
4. Envia para RD Station CRM

## Lógica de Negócio

### Critérios para Identificar Lead

**Função:** `isLead(schedule: Agendamento)`

Um agendamento é considerado lead quando atende **TODOS** os critérios:

1. **Atendido:** `StaID == 3`
   - Status do agendamento indica que foi efetivamente atendido

2. **Com Convênio:** `NomeConvenio != ""`
   - Possui convênio médico associado

3. **É Consulta:** `NomeProcedimento.toLowerCase().includes("consulta")`
   - O procedimento contém a palavra "consulta"

**Exemplo:**

```typescript
const atendido = schedule.StaID == 3;
const convenio = schedule.NomeConvenio != "";
const consulta = schedule.NomeProcedimento.toLowerCase().includes("consulta");

return atendido && convenio && consulta;
```

### Prevenção de Duplicatas

O sistema utiliza DynamoDB para rastrear agendamentos processados:

**Estrutura da Row:**

```typescript
interface Row {
  id: number;           // AgendamentoID
  deleteAt: number;     // Timestamp Unix para TTL
}
```

**TTL (Time To Live):**

- Configurado para 1 dia após o processamento
- Calculado via função `deleteAtEpoch(date, days)`
- Limpa automaticamente registros antigos

### Conversão de Agendamento para Deal

**Função:** `parseDeal(agendamento: Agendamento)`

**Dados Extraídos:**

**Contato:**

- Nome: `NomePaciente`
- Telefone: `Cel1` (se disponível)
- Tipo: "paciente"

**Custom Fields enviados ao RD:**

- `FIELD_CONVENIO`: Nome do convênio
- `FIELD_ESPECIALIDADE`: Especialidade médica
- `FIELD_UNIDADE`: Unidade de atendimento

**Deal:**

- **Name:** Nome do paciente
- **Stage:** `STAGE_ATENDIMENTO_CONVENIOS` (ID: 65afc195b4b978000da55239)
- **User:** `DEFAULT_USER` - Usuário padrão do RD
- **Rating:** 0 (sem classificação inicial)

## Dependências

### Integrações Externas

1. **Feegow API:**
   - `schedules(dateString)` - Busca agendamentos por data
   - Formato de data: "dd/MM/yyyy"

2. **RD Station CRM API:**
   - `postDeal(deal)` - Criação de oportunidade
   - Não cria organização automaticamente (diferente da automação de consultas)

3. **DynamoDB:**
   - Tabela: `ATENDIMENTOS_CONVENIOS`
   - Operações: `scanAll()`, `save()`
   - TTL habilitado no campo `deleteAt`

4. **SQS:**
   - Fila de mensagens com delay
   - Processamento controlado por concorrência

### Bibliotecas

- `date-fns` - Manipulação de datas (add, format, getTime)
- `aws-sdk` - Integração com SQS e DynamoDB

## Diferenças em Relação à Automação de Consultas

| Aspecto | Convênios | Consultas |
|---------|-----------|-----------|
| **Trigger** | A cada 15 min (10h-23h UTC) | 1x ao dia (04:00 UTC) |
| **Fonte de Dados** | API Feegow (dia atual) | Banco MySQL (dia anterior) |
| **Cache** | DynamoDB com TTL | Sem cache |
| **Critério** | Atendidos + Convênio + Consulta | Leads não concluídos |
| **Stage RD** | ATENDIMENTO_CONVENIOS | RECUPERACAO |
| **Organização RD** | Não cria | Busca ou cria |
| **Concorrência** | Max 2 simultâneos | Sem limite |
| **Objetivo** | Monitorar atendimentos | Recuperar leads |

## Variáveis de Ambiente

- `QUEUE_URL` - URL da fila SQS de convênios
- Credenciais Feegow API
- Credenciais RD Station API
- Configurações AWS (DynamoDB, SQS)

## Tratamento de Erros

1. **Erro ao buscar agendamentos:**
   - Falha na API Feegow interrompe o processamento
   - Lambda retorna erro e pode ser retentado

2. **Erro no DynamoDB:**
   - Falha ao salvar pode resultar em processamento duplicado
   - TTL garante limpeza automática

3. **Erro no Worker:**
   - SQS mantém mensagem na fila
   - Retry automático após delay

## Configurações de Performance

### Schedule Function

- **Frequência:** A cada 15 minutos durante horário comercial
- **Timeout:** 900s para processar lote do dia
- **Memória:** 128 MB suficiente para scan DynamoDB + API

### Worker Function

- **Concorrência:** Máximo 2 simultâneos
- **Reserved Concurrency:** 2 (garante disponibilidade)
- **Timeout:** 180s por agendamento
- **Memória:** 256 MB (maior que consultas)
- **Batch size:** 1 (processamento individual)
- **Delay:** 15s entre mensagens

### Otimizações

1. **Scan DynamoDB:** Uma vez por execução, compartilhado entre loops
2. **TTL Automático:** Limpeza de dados sem intervenção manual
3. **Processamento Incremental:** Apenas novos atendimentos do dia

## Monitoramento

### Métricas Importantes

1. **Schedule Function:**
   - Número de agendamentos encontrados por execução
   - Taxa de leads válidos vs total de agendamentos
   - Tamanho da tabela DynamoDB

2. **Worker Function:**
   - Taxa de sucesso ao criar deals no RD
   - Tempo médio de processamento
   - Concorrência atual (0-2)

3. **DynamoDB:**
   - Read/Write capacity units
   - Items expirados por TTL
   - Tamanho da tabela

4. **SQS:**
   - Mensagens na fila
   - Age of oldest message
   - Dead Letter Queue (se configurada)

### Logs Importantes

```typescript
console.log(atendido, convenio, consulta) // Debug dos critérios
```

- Status de cada agendamento verificado
- Agendamentos enviados para fila
- Erros ao processar no RD Station

## Manutenção

### Pontos de Atenção

1. **Horário de Execução:**
   - Cron configurado para UTC (10h-23h)
   - Corresponde a 07h-20h em Brasília
   - Ajustar se necessário alterar janela de monitoramento

2. **TTL do DynamoDB:**
   - Configurado para 1 dia
   - Aumentar se necessário prevenir duplicatas por mais tempo
   - Diminuir para economizar armazenamento

3. **Critérios de Lead:**
   - StaID = 3 é específico para "atendido"
   - Verificar se outros status devem ser incluídos
   - Palavra "consulta" pode variar (case insensitive)

4. **Concorrência:**
   - Max 2 permite processamento paralelo sem sobrecarregar RD
   - Aumentar se volume crescer significativamente

5. **Custos:**
   - 84 execuções por dia (a cada 15min por 14h)
   - Considerar reduzir frequência se necessário

### Troubleshooting

**Problema:** Agendamentos duplicados no RD

**Solução:**

- Verificar se DynamoDB está salvando corretamente
- Verificar TTL configurado na tabela
- Checar logs de erros no save()

**Problema:** Leads não sendo capturados

**Solução:**

- Verificar critérios da função `isLead()`
- Validar se StaID = 3 corresponde a "atendido"
- Conferir formato do NomeProcedimento

**Problema:** Fila SQS crescendo

**Solução:**

- Verificar erros no Worker
- Aumentar concorrência (max de 2 para mais)
- Verificar timeout do Worker
