# Documentação Técnica - Automação de Propostas RC

## Visão Geral

A automação de propostas RC é uma variante da automação de propostas regular, focada especificamente em um segmento ou unidade diferenciada (RC). Compartilha a mesma estrutura base de agrupamento por paciente e criação de deals no RD Station, mas com diferenças importantes na distribuição de atendentes, filtros e stage de destino.

## Arquitetura

### Componentes Principais

1. **Schedule Function** (`src/schedules/proposals-rc.ts`)
   - Função Lambda agendada via EventBridge
   - Execução diária às 04:00 UTC (01:00 Brasília)
   - Timeout: 900 segundos (15 minutos)
   - Memória: 128 MB

2. **Worker Function** (`src/workers/proposals-rc.ts`)
   - Função Lambda processadora de mensagens SQS
   - Timeout: 180 segundos (3 minutos)
   - Memória: 128 MB
   - Batch size: 1 mensagem por execução
   - Reserved concurrency: 2
   - Max concurrency: 2

3. **SQS Queue**
   - Delay de 15 segundos
   - Processamento com concorrência limitada (max 2)

## Fluxo de Execução

### Etapa 1: Coleta e Agrupamento de Propostas RC (Schedule)

**Handler:** `src/schedules/proposals-rc.ts`

```typescript
async function handler(event, context)
```

**Processo:**

1. Instancia cliente Feegow
2. Busca propostas RC do dia anterior via `proposalsRc(date)` (endpoint específico)
3. Filtra apenas propostas com status "Em aberto" (PropostaStatusID = 1)
4. Agrupa propostas por paciente
5. Atribui atendente fixo "ATIVOS" para todas as propostas
6. Log do total de propostas encontradas
7. Envia cada proposta agrupada para a fila SQS

**Configuração:**

- **Cron:** `cron(0 4 ? * * *)` - Executa todo dia às 04:00 UTC
- **Environment:** `QUEUE_URL` - URL da fila SQS
- **Data:** Dia anterior formatado como dd/MM/yyyy
- **Endpoint:** `proposalsRc()` - Específico para RC

### Etapa 2: Processamento de Propostas RC (Worker)

**Handler:** `src/workers/proposals-rc.ts`

```typescript
async function handler(event: Event, context)
```

**Processo:**

1. Itera sobre cada mensagem SQS recebida
2. Busca dados completos do paciente na Feegow
3. Converte proposta em Deal do RD Station
4. **Busca agendamentos existentes do paciente**
5. **Filtra procedimentos já agendados** (diferencial!)
6. Se todos procedimentos já estão agendados, descarta proposta
7. Enriquece anotações com status de agendamentos
8. Processa lead no RD Station

## Lógica de Negócio

### Agrupamento de Propostas por Paciente

**Função:** `groupProposals(data: any[])`

**Processo:** Idêntico à automação de proposals regular

1. Cria objeto indexado por `PacienteID`
2. Para cada item da API:
   - Se paciente não existe no grupo, cria nova proposta
   - Adiciona procedimento ao array `items`
3. Retorna array de propostas agrupadas

### Diferenças em Relação a Proposals

#### 1. Endpoint Específico

```typescript
// Proposals regular
await feegow.proposals(date)

// Proposals RC
await feegow.proposalsRc(date)
```

**Razão:** Segmentação de dados, provavelmente por unidade ou tipo de proposta

#### 2. Atendente Fixo "ATIVOS"

```typescript
// Proposals regular
agente: ATENDENTES[indiceAtendente]  // Round-robin

// Proposals RC
agente: "ATIVOS"  // Fixo para todas
```

**Sem Round-Robin:**

- Código comentado: `//let indiceAtendente = 0`
- Todas propostas RC recebem atendente "ATIVOS"
- Provavelmente gerenciado por equipe ou pool específico

**Constante ATENDENTES_RC:**

```typescript
export const ATENDENTES_RC = [
  "Ana Carolina Pereira",
  "Ana Carolina Marcos"
]
```

**Nota:** Constante definida mas não utilizada no código atual

#### 3. Stage Diferente

```typescript
// Proposals regular
stage: RD.STAGE_PROPOSTA

// Proposals RC
stage: RD.FUNNEL_PROPOSTAS_GERAIS
```

**FUNNEL_PROPOSTAS_GERAIS:** ID 65391eec1e66020013a4a86a

**Motivo:** Funil separado permite:

- Segmentação de relatórios
- SLA diferenciado
- Processo de follow-up específico

#### 4. Filtro Inteligente de Procedimentos (Exclusivo!)

**Lógica única de Proposals RC:**

```typescript
const nonScheduledProcedures = deal.procedimentos.filter(({ id }) => {
    return !schedules.some((schedule) => schedule.procedimento_id === id)
})

if (nonScheduledProcedures.length === 0) {
    // Todos procedimentos já agendados - descarta proposta
    continue
}

deal.procedimentos = nonScheduledProcedures
```

**Comportamento:**

1. Busca agendamentos existentes do paciente
2. Filtra procedimentos da proposta que **não** têm agendamento
3. Se todos já estão agendados → **descarta proposta completa**
4. Se alguns não estão agendados → **mantém apenas os não agendados**

**Exemplo:**

``` bash
Proposta Original:
- Alergologia (R$ 150)
- Cardiologia (R$ 200)
- Dermatologia (R$ 120)

Agendamentos Existentes:
- Cardiologia (agendado para 15/11)

Resultado:
- Remove Cardiologia da proposta
- Mantém apenas: Alergologia + Dermatologia
- Deal criado com 2 procedimentos e valor R$ 270
```

#### 5. Sem Validação de Valor Mínimo

```typescript
// Proposals regular
if (totalValue >= 0.1 && totalValue <= 65) {
    continue; // Ignora
}

// Proposals RC
// SEM validação de valor mínimo
```

**Razão:** Todas propostas RC são processadas independente do valor

#### 6. Sem Validação de Telefone

```typescript
// Proposals regular
if (!deal) {
    console.log("Proposta ignorada, sem telefone");
    continue;
}

// Proposals RC
// SEM validação de telefone
```

**Razão:** Propostas RC processadas mesmo sem telefone cadastrado

### Conversão de Proposta para Deal

**Função:** `parseDeal(proposal: Proposal, clientFeegow: Patient)`

**Estrutura:** Praticamente idêntica a proposals regular

**Custom Fields (12 campos):**

1. FIELD_PRIORIDADE - Baseado no grupo
2. CLIENT_ANOTACAO - Observações
3. FIELD_UNIDADE - Unidade
4. CLIENT_CPF - CPF
5. CLIENT_CEP - CEP
6. CLIENT_CIDADE - Cidade
7. CLIENT_BAIRRO - Bairro
8. CLIENT_COMPLEMENTO - Complemento
9. CLIENT_NUMERO - Número
10. CLIENT_LOGRADOURO - Logradouro
11. CLIENT_TELEFONE - Telefone
12. AGENTE_PROPOSTA - **Sempre "ATIVOS"**

**Procedimentos:** Convertidos em produtos do deal

**Stage e Source:**

- **Stage:** `FUNNEL_PROPOSTAS_GERAIS`
- **Source:** `SOURCE_PROPOSTAS` (mesmo das proposals regulares)

### Enriquecimento com Status de Agendamentos

**Processo:** Idêntico a proposals regular

1. Busca agendamentos existentes
2. Para procedimentos que restaram após filtro:
   - Verifica se existe agendamento
   - Anota status se existir
3. Cria anotação formatada

## Dependências

### Integrações Externas

1. **Feegow API:**
   - `proposalsRc(date)` - **Endpoint específico** para propostas RC
   - `getPatient(patientId)` - Dados completos do paciente
   - `schedulesByPatientId(patientId)` - Agendamentos
   - `linkPaciente(patientId)` - Link de acesso

2. **RD Station CRM API:**
   - Criação de organizações
   - Criação de deals com produtos
   - Criação de anotações

### Bibliotecas

- `date-fns` - Manipulação de datas (format, sub)
- `aws-sdk` - Integração com SQS

## Comparação: Proposals RC vs Proposals Regular

| Aspecto | Proposals RC | Proposals Regular |
|---------|--------------|-------------------|
| **Endpoint API** | `proposalsRc(date)` | `proposals(date)` |
| **Atendente** | Fixo "ATIVOS" | Round-robin (5 atendentes) |
| **Filtro de Procedimentos** | ✅ Remove já agendados | ❌ Não filtra |
| **Descarte Total** | ✅ Se todos agendados | ❌ Não |
| **Validação de Valor** | ❌ Processa todos | ✅ Mínimo R$ 65 |
| **Validação de Telefone** | ❌ Não valida | ✅ Obrigatório |
| **Stage RD** | FUNNEL_PROPOSTAS_GERAIS | STAGE_PROPOSTA |
| **Source RD** | SOURCE_PROPOSTAS | SOURCE_PROPOSTAS |
| **Custom Fields** | 12 campos | 12 campos |
| **Produtos** | ✅ Sim | ✅ Sim |
| **Agrupamento** | ✅ Por paciente | ✅ Por paciente |
| **Status Agendamentos** | ✅ Anota | ✅ Anota |
| **Log de Total** | ✅ Sim | ❌ Não |

## Particularidades da Automação Proposals RC

### 1. Filtro Inteligente de Procedimentos (EXCLUSIVO!)

**Diferencial mais importante:**

- Remove procedimentos já agendados
- Evita duplicação de esforço comercial
- Foco apenas em procedimentos pendentes
- Descarta proposta se tudo já está agendado

**Cenários:**

**Cenário 1 - Descarte Total:**

``` bash
Proposta: Alergologia + Cardiologia
Agendamentos: Alergologia ✓, Cardiologia ✓
Resultado: Proposta DESCARTADA (nada a fazer)
```

**Cenário 2 - Filtro Parcial:**

``` bash
Proposta: Alergologia + Cardiologia + Dermatologia
Agendamentos: Cardiologia ✓
Resultado: Deal com Alergologia + Dermatologia apenas
```

**Cenário 3 - Sem Filtro:**

``` bash
Proposta: Alergologia + Cardiologia
Agendamentos: Nenhum
Resultado: Deal com todos os procedimentos
```

### 2. Atendente Fixo "ATIVOS"

**Sem distribuição automática:**

- Todas propostas RC vão para pool "ATIVOS"
- Provavelmente gerenciado por equipe específica
- Simplifica gestão de leads RC
- Permite especialização da equipe

### 3. Sem Validação de Valor

**Aceita qualquer valor:**

- Proposals regular descarta < R$ 65
- Proposals RC processa todos os valores
- Indica que RC pode ter procedimentos de menor valor
- Ou estratégia comercial diferente

### 4. Sem Validação de Telefone

**Mais permissivo:**

- Proposals regular exige telefone
- Proposals RC aceita sem telefone
- Permite outros canais de contato
- Ou processo de complementação posterior

### 5. Stage Específico

**Funil separado:**

- FUNNEL_PROPOSTAS_GERAIS vs STAGE_PROPOSTA
- Permite métricas isoladas
- SLA diferenciado
- Relatórios segmentados

### 6. Endpoint Específico

**API segregada:**

- `proposalsRc()` retorna subset específico
- Provavelmente filtrado por unidade ou categoria
- Dados já vêm segmentados da origem

### 7. Log de Total

**Visibilidade:**

```typescript
console.info("propostas: " + proposals.length);
```

- Único que loga total encontrado
- Facilita monitoramento
- Debug simplificado

## Variáveis de Ambiente

- `QUEUE_URL` - URL da fila SQS de proposals-rc
- Credenciais Feegow API
- Credenciais RD Station API

## Tratamento de Erros

1. **Todos procedimentos já agendados:**
   - Proposta é descartada silenciosamente
   - Continue para próxima mensagem
   - Código comentado tinha logs

2. **Erro ao buscar paciente ou agendamentos:**
   - Pode gerar exceção não tratada
   - Mensagem permanece na fila SQS para retry

## Configurações de Performance

### Schedule Function

- **Timeout:** 900s
- **Memória:** 128 MB
- **Execução:** 1x por dia (04:00 UTC)
- **Log:** Total de propostas encontradas

### Worker Function

- **Timeout:** 180s por proposta
- **Memória:** 128 MB
- **Batch size:** 1
- **Delay:** 15s
- **Concorrência:** Max 2 simultâneos
- **Reserved:** 2

### Otimizações

1. **Filtro de procedimentos:** Reduz dados enviados ao RD
2. **Descarte antecipado:** Se tudo agendado, não processa
3. **Agrupamento:** Reduz mensagens SQS
4. **Filtro de status:** Apenas propostas status 1

## Monitoramento

### Métricas Importantes

1. **Total de propostas RC por dia** (via log)
2. **Taxa de descarte** (todos procedimentos agendados)
3. **Taxa de filtro parcial** (alguns procedimentos agendados)
4. **Procedimentos removidos vs mantidos**
5. **Valor médio após filtro**
6. **Propostas sem telefone** (processadas mesmo assim)

### Logs Importantes

```typescript
console.info("propostas: " + proposals.length)
// Código comentado:
// console.log(`Paciente ${id} já possui agendamentos...`)
// console.log(`Proposta para paciente ${id} atualizada...`)
```

## Manutenção

### Pontos de Atenção

1. **Endpoint proposalsRc:**
   - Validar se retorna dados corretos
   - Confirmar filtros aplicados na API

2. **Atendente "ATIVOS":**
   - Verificar se deve ser alterado
   - Considerar ativar ATENDENTES_RC (já definido)

3. **Filtro de procedimentos:**
   - Lógica crítica e exclusiva
   - Testar cenários de agendamentos parciais

4. **Logs comentados:**
   - Considerar reativar para debugging
   - Úteis para entender comportamento

5. **Validações desativadas:**
   - Sem filtro de valor mínimo
   - Sem validação de telefone
   - Verificar se é intencional

### Troubleshooting

**Problema:** Propostas sendo descartadas em excesso

**Solução:**

- Verificar lógica de filtro de procedimentos
- Validar se agendamentos estão sendo buscados corretamente
- Conferir match por `procedimento_id`

**Problema:** Procedimentos duplicados no RD

**Solução:**

- Filtro deveria evitar isso
- Verificar se `schedulesByPatientId` está retornando todos agendamentos
- Validar comparação de IDs

**Problema:** Todas propostas com mesmo atendente

**Solução:**

- Comportamento esperado ("ATIVOS")
- Se precisar distribuir, ativar código comentado com ATENDENTES_RC

**Problema:** Propostas de baixo valor sendo processadas

**Solução:**

- Comportamento esperado (sem filtro de valor)
- Adicionar validação se necessário

## Código Comentado

**Distribuição de atendentes (desativada):**

```typescript
//let indiceAtendente = 0
//indiceAtendente = indiceAtendente == ATENDENTES_RC.length ? 0 : indiceAtendente += 1
```

**Logs de debug (desativados):**

```typescript
// console.log(`Paciente ${deal.paciente.id} já possui agendamentos...`);
// console.log(`Proposta para paciente ${deal.paciente.id} atualizada...`);
```

**Considerações:**

- Código mantido comentado sugere funcionalidade planejada
- Pode ser ativado se necessário redistribuir atendentes
- Logs úteis para troubleshooting
