# Documentação Técnica - Automação de Exames

## Visão Geral

A automação de exames é responsável por identificar leads de agendamento de exames médicos que não concluíram o processo e enviá-los para o RD Station CRM para recuperação. Similar à automação de consultas, mas focada especificamente em procedimentos de exames, com campos e validações adaptados para este contexto.

## Arquitetura

### Componentes Principais

1. **Schedule Function** (`src/schedules/exam.ts`)
   - Função Lambda agendada via EventBridge
   - Execução diária às 04:00 UTC (01:00 Brasília)
   - Timeout: 900 segundos (15 minutos)
   - Memória: 128 MB

2. **Worker Function** (`src/workers/exam.ts`)
   - Função Lambda processadora de mensagens SQS
   - Timeout: 180 segundos (3 minutos)
   - Memória: 128 MB
   - Batch size: 1 mensagem por execução
   - Delay: 15 segundos entre mensagens

3. **Repository** (`src/repositories/exame.repository.ts`)
   - Acesso ao banco de dados MySQL (tabela `Patient`)
   - Integração com API Feegow

4. **SQS Queue**
   - Delay de 15 segundos
   - Processamento sequencial (batchSize = 1)

## Fluxo de Execução

### Etapa 1: Coleta de Leads (Schedule)

**Handler:** `src/schedules/exam.ts`

```typescript
async function handler(event, context)
```

**Processo:**

1. Instancia cliente Feegow
2. Cria conexão com banco MySQL (configuração `agendamentoExameConnection`)
3. Instancia repositório de exames
4. Busca leads através do método `listLeads()`
5. Envia cada lead para a fila SQS
6. Log do total de leads encontrados

**Configuração:**

- **Cron:** `cron(0 4 ? * * *)` - Executa todo dia às 04:00 UTC
- **Environment:** `QUEUE_URL` - URL da fila SQS
- **Database:** Tabela `Patient` (banco de exames)

### Etapa 2: Processamento de Leads (Worker)

**Handler:** `src/workers/exam.ts`

```typescript
async function handler(event: Event, context)
```

**Processo:**

1. Itera sobre cada mensagem SQS recebida
2. Faz parse do corpo da mensagem como `LeadExame`
3. Converte dados em Deal do RD Station
4. Processa o lead através do serviço RD

## Lógica de Negócio

### Critérios de Seleção de Leads

O método `ExameRepository.listLeads()` seleciona leads com os seguintes critérios:

```sql
SELECT * FROM Patient 
WHERE 
    agendamentoId IS NULL
    AND pacienteNome IS NOT NULL
    AND pacienteId IS NOT NULL
    AND createdAt BETWEEN {início_dia_anterior} AND {fim_dia_anterior}
```

**Janela de Tempo:**

- Calcula início do dia anterior: `startOfDay(sub(new Date(), { days: 1 }))`
- Calcula fim do dia anterior: `endOfDay(sub(new Date(), { days: 1 }))`
- Formato ISO para query SQL

**Filtros Adicionais:**

1. **Deduplicação por Nome:**
   - Utiliza `Set<string>` para processar cada paciente apenas uma vez
   - Evita múltiplos leads do mesmo paciente

2. **Validação de Agendamentos:**
   - Verifica se o paciente possui agendamentos na Feegow
   - Se não houver agendamentos (total = 0), o lead é incluído
   - Se houver agendamentos, valida se:
     - Data do agendamento > próximo mês OU
     - Diferença entre hoje e data de criação > 1 mês

### Estrutura do Lead de Exame

**Interface LeadExame:**

```typescript
interface LeadExame {
    id: string;
    pacienteNome: string;
    pacienteId?: number;
    pacienteTelefone?: string;
    pacienteEmail?: string;
    pacienteNascimento?: string;
    pacienteSexo?: string;
    createdAt?: string;
    finishAt?: string;
    clubflexId?: string;           // ID Clubflex se for cliente
    convenio?: string;
    plano?: string;
    dataAgendamentos?: string[];   // Array de datas
    horaAgendamento?: string[];    // Array de horários
    especialidades?: string[];      // Array de especialidades
    procedimentos?: string[];       // Array de procedimentos
    profissionalsId?: string[];     // Array de IDs de profissionais
    unidadesId?: string[];         // Array de IDs de unidades
}
```

**Parsing de Arrays JSON:**

Diversos campos são armazenados como JSON no banco e precisam ser parseados:

- `dataAgendamentos` - Datas de interesse
- `horaAgendamento` - Horários selecionados
- `profissionalsId` - Profissionais escolhidos
- `unidadesId` - Unidades selecionadas
- `procedimentos` - Procedimentos desejados
- `especialidades` - Separadas por vírgula ou array

### Conversão de Lead para Deal

**Função:** `parseDeal(data: LeadExame, feegowClient)`

**Dados do Paciente:**

- **ID:** `pacienteId` ou string vazia
- **Document:** `clubflexId` ou string vazia (usado como documento)
- **Nome:** `pacienteNome`
- **Contato:** `pacienteTelefone` ou string vazia

**Custom Fields enviados ao RD:**

- `FIELD_CONVENIO`: Convênio médico (se disponível)
- `FIELD_CLIENTECLUB`: "Sim" se possui `clubflexId`, "Não" caso contrário

**Anotações Detalhadas:**

Cria uma anotação completa com:

**Seção Paciente:**

- Nome
- Data de nascimento
- Telefone de contato

**Seção Agendamentos:**

- Data e hora do acesso (formatada em padrão brasileiro)
- Datas de interesse (array formatado)
- Horários selecionados (array formatado)
- Especialidades de interesse (array formatado)
- Procedimentos desejados (array formatado)
- IDs dos profissionais selecionados
- IDs das unidades escolhidas
- Convênio
- Plano
- Link direto para o paciente na Feegow

**Stage e Source:**

- **Stage:** `FUNEL_RECUPERACAO_EX` (ID: 672e46159cdc880013450fb4)
- **Source:** `SOURCE_AGENDAMENTO`
- **Procedimentos:** Array vazio (não processa produtos)

### Processamento no RD Station

**Serviço:** `src/rd/service.ts`

**Função:** `processLead(deal: Deal)`

**Etapas:**

1. **Busca ou Cria Organização:**
   - Busca organização pelo nome do paciente
   - Se não existir, cria nova organização

2. **Cria Deal:**
   - Cria negócio vinculado à organização
   - Preenche custom fields
   - Define stage específico para exames (FUNEL_RECUPERACAO_EX)

3. **Adiciona Anotações:**
   - Cria anotação formatada com todos os detalhes
   - Inclui link Feegow para fácil acesso

## Dependências

### Integrações Externas

1. **Feegow API:**
   - `schedulesByPatientId(patientId)` - Busca agendamentos do paciente
   - Link do paciente gerado dinamicamente

2. **RD Station CRM API:**
   - Criação de organizações
   - Criação de deals
   - Criação de anotações

3. **Banco de Dados MySQL:**
   - Tabela: `Patient` (específica para exames)
   - Campos principais: `pacienteId`, `pacienteNome`, `agendamentoId`, `createdAt`
   - Campos JSON: arrays de dados do agendamento

### Bibliotecas

- `mysql2/promise` - Conexão MySQL
- `date-fns` - Manipulação de datas (sub, startOfDay, endOfDay, add, differenceInMonths)
- `aws-sdk` - Integração com SQS

## Diferenças em Relação à Automação de Consultas

| Aspecto | Exames | Consultas |
|---------|--------|-----------|
| **Tabela MySQL** | `Patient` | `site_leads` |
| **Conexão** | `agendamentoExameConnection` | `agendamentoConnection` |
| **Campos** | Arrays JSON complexos | Campos simples |
| **Stage RD** | `FUNEL_RECUPERACAO_EX` | `STAGE_RECUPERACAO` |
| **Custom Fields** | Convênio + Cliente Club | 10+ campos detalhados |
| **Deduplicação** | Por nome (Set) | Por agendamento |
| **Anotação** | Múltiplos arrays | Dados únicos |
| **Link Feegow** | Formato v8 com Pers=1 | Método estático |
| **Procedimentos** | Array vazio | Não usado |
| **Objetivo** | Recuperar leads de exames | Recuperar leads de consultas |

## Particularidades da Automação de Exames

### 1. Arrays de Dados

Diferente de consultas, exames permitem múltiplas seleções:

- **Múltiplas datas** de interesse
- **Múltiplos horários** disponíveis
- **Múltiplos procedimentos** (diferentes exames)
- **Múltiplas especialidades**
- **Múltiplos profissionais**
- **Múltiplas unidades**

### 2. Parsing JSON

Campos armazenados como JSON precisam ser parseados:

```typescript
dataAgendamentos: leadExames.dataAgendamentos 
    ? JSON.parse(leadExames.dataAgendamentos) 
    : undefined
```

### 3. Especialidades com Fallback

Tratamento especial para especialidades:

```typescript
especialidades: Array.isArray(leadExames.especialidades)
    ? leadExames.especialidades
    : leadExames.especialidades 
        ? leadExames.especialidades.split(",") 
        : undefined
```

### 4. Deduplicação por Nome

Usa `Set` para garantir um lead por paciente:

```typescript
const processedNames = new Set<string>();
if (processedNames.has(lead.pacienteNome)) {
    continue;
}
processedNames.add(lead.pacienteNome);
```

### 5. Formatação de Data Brasileira

Utiliza função específica `formatBrazilianDate()`:

```typescript
Data e Hora do Acesso: ${formatBrazilianDate(data.createdAt)}
```

## Variáveis de Ambiente

- `QUEUE_URL` - URL da fila SQS de exames
- Credenciais MySQL (definidas em `agendamentoExameConnection`)
- Credenciais Feegow API
- Credenciais RD Station API

## Tratamento de Erros

1. **Erro ao verificar agendamentos:**
   - Log no console com ID do paciente
   - Lead é ignorado e processamento continua

2. **Lead sem pacienteId válido:**
   - Warning no console
   - Lead é ignorado

3. **Erro no parsing JSON:**
   - Retorna `undefined` para campos opcionais
   - Processamento continua sem o campo

## Configurações de Performance

### Schedule Function

- **Timeout:** 900s para processar todos os leads
- **Memória:** 128 MB para operações de banco
- **Execução:** 1x por dia (04:00 UTC)

### Worker Function

- **Timeout:** 180s por lead
- **Memória:** 128 MB suficiente para processamento
- **Batch size:** 1 (processamento sequencial)
- **Delay:** 15s entre mensagens

### Otimizações

1. **Deduplicação em memória:** Set de nomes processados
2. **Query otimizada:** WHERE com índices em datas
3. **Processamento incremental:** Apenas dia anterior
4. **Parsing condicional:** JSON parseado apenas se existir

## Monitoramento

### Métricas Importantes

1. **Leads encontrados por dia**
2. **Taxa de deduplicação** (quantos nomes repetidos)
3. **Campos JSON com erro de parsing**
4. **Taxa de sucesso no processamento**
5. **Tempo de execução da schedule function**

### Logs Importantes

```typescript
console.log(lead)           // Cada lead processado
console.log(leads.length)   // Total de leads
console.error(`Erro ao verificar agendamentos...`)
console.warn(`Lead ${lead.id} não possui pacienteId válido...`)
```

## Manutenção

### Pontos de Atenção

1. **Estrutura JSON:**
   - Arrays devem estar no formato correto
   - Validar inserção no banco de dados

2. **Deduplicação:**
   - Baseada em nome, não em ID
   - Pode agrupar pessoas homônimas

3. **Tabela Patient:**
   - Diferente da tabela de consultas
   - Estrutura específica para exames

4. **Stage específico:**
   - `FUNEL_RECUPERACAO_EX` é diferente do funil de consultas
   - Permite segmentação no RD Station

5. **Link Feegow:**
   - Formato: `https://app.feegow.com/v8/?p=pacientes&i=${pacienteId}&Pers=1`
   - Parâmetro `Pers=1` importante para visualização

### Troubleshooting

**Problema:** Campos array vazios nas anotações

**Solução:**

- Verificar parsing JSON no repository
- Validar formato dos dados no banco
- Conferir se campos existem antes de parsear

**Problema:** Leads duplicados no RD

**Solução:**

- Verificar Set de deduplicação
- Considerar usar pacienteId ao invés de nome
- Validar se processLead busca organização existente

**Problema:** Erro ao formatar data brasileira

**Solução:**

- Verificar implementação de `formatBrazilianDate()`
- Validar formato do campo `createdAt`
- Tratar casos onde data é null/undefined
