# Documentação Técnica - Automação de Consultas

## Visão Geral

A automação de consultas é responsável por identificar leads de agendamento que não concluíram o processo e enviá-los para o RD Station CRM para recuperação. O sistema opera em duas etapas principais: coleta de leads e processamento de cada lead individual.

## Arquitetura

### Componentes Principais

1. **Schedule Function** (`src/schedules/consultas.ts`)
   - Função Lambda agendada via EventBridge
   - Execução diária às 04:00 UTC (01:00 Brasília)
   - Timeout: 900 segundos (15 minutos)
   - Memória: 256 MB

2. **Worker Function** (`src/workers/consultas.ts`)
   - Função Lambda processadora de mensagens SQS
   - Timeout: 180 segundos (3 minutos)
   - Memória: 128 MB
   - Batch size: 1 mensagem por execução
   - Delay: 15 segundos entre mensagens

3. **Repository** (`src/repositories/consulta.repository.ts`)
   - Acesso ao banco de dados MySQL
   - Integração com API Feegow

4. **SQS Queue**
   - Delay de 15 segundos
   - Processamento sequencial (batchSize = 1)

## Fluxo de Execução

### Etapa 1: Coleta de Leads (Schedule)

**Handler:** `src/schedules/consultas.ts`

```typescript
async function handler(event, context)
```

**Processo:**

1. Instancia cliente Feegow
2. Cria conexão com banco MySQL
3. Instancia repositório de consultas
4. Busca leads através do método `listLeads()`
5. Envia cada lead para a fila SQS

**Configuração:**

- **Cron:** `cron(0 4 ? * * *)` - Executa todo dia às 04:00 UTC
- **Environment:** `QUEUE_URL` - URL da fila SQS

### Etapa 2: Processamento de Leads (Worker)

**Handler:** `src/workers/consultas.ts`

```typescript
async function handler(event: Event, context)
```

**Processo:**

1. Itera sobre cada mensagem SQS recebida
2. Faz parse do corpo da mensagem
3. Converte dados em Deal do RD Station
4. Processa o lead através do serviço RD

## Lógica de Negócio

### Critérios de Seleção de Leads

O método `ConsultaRepository.listLeads()` seleciona leads com os seguintes critérios:

```sql
WHERE 
    FEEGOW_ID_AGENDAMENTO is null AND 
    COMPLETE_DATE is null AND
    CLIENT_PHONE is not null AND
    BEGIN_DATE like '{data_ontem}%'
```

**Filtros Adicionais:**

1. **Validação de Agendamentos:**
   - Verifica se o paciente possui agendamentos na Feegow
   - Se não houver agendamentos (total = 0), o lead é incluído
   - Se houver agendamentos, valida se:
     - Data do agendamento > próximo mês OU
     - Diferença entre hoje e data de criação > 1 mês

### Conversão de Lead para Deal

**Função:** `parseDeal(data, feegowClient)`

**Validações:**

1. **Verificação de Especialidade:**
   - Se o lead possui especialidade preenchida
   - Verifica se já existe agendamento para aquela especialidade
   - Se existir, o lead é descartado (retorna `null`)

2. **Identificação da Última Tela:**
   - Determina em qual etapa do funil o usuário parou
   - Baseado nos dados preenchidos:
     - Tela 6: Data de interesse ou valor preenchidos
     - Tela 5: Profissional selecionado
     - Tela 4: Unidade selecionada
     - Tela 3: Especialidade ou idade preenchidos
     - Tela 2: Nome preenchido
     - Tela 1: Cliente Clubflex identificado
     - Tela 0: Apenas CPF

**Custom Fields enviados ao RD:**

- Convênio
- Especialidade
- Profissional selecionado
- Valor
- Última tela
- Cliente Clubflex (Sim/Não)
- Unidade de interesse
- Data e hora de acesso
- Data e hora de interesse
- UTM Source, Medium, Content, Campaign, Term

**Stage e Source:**

- **Stage:** `STAGE_RECUPERACAO` - Funil de recuperação
- **Source:** `SOURCE_AGENDAMENTO` - Origem: site de agendamento

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
   - Define stage e source

3. **Adiciona Anotações:**
   - Cria anotação formatada com:
     - Dados do paciente (nome, idade)
     - Dados de interesse (data/hora acesso, especialidade, profissional, unidade, valor)
     - Última tela acessada
     - Link direto para o paciente na Feegow

## Dependências

### Integrações Externas

1. **Feegow API:**
   - `schedulesByPatientId(patientId)` - Busca agendamentos do paciente
   - `linkPaciente(patientId)` - Gera link para visualização

2. **RD Station CRM API:**
   - Criação de organizações
   - Criação de deals
   - Criação de anotações

3. **Banco de Dados MySQL:**
   - Tabela: `site_leads`
   - Campos principais: `FEEGOW_PATIENT_ID`, `CLIENT_NAME`, `CLIENT_PHONE`, `BEGIN_DATE`

### Bibliotecas

- `mysql2/promise` - Conexão MySQL
- `date-fns` - Manipulação de datas
- `aws-sdk` - Integração com SQS

## Variáveis de Ambiente

- `QUEUE_URL` - URL da fila SQS de consultas
- Credenciais MySQL (definidas em `src/repositories/setings.ts`)
- Credenciais Feegow API
- Credenciais RD Station API

## Tratamento de Erros

1. **Erro ao verificar agendamentos:**
   - Log no console com ID do paciente
   - Lead é ignorado e processamento continua

2. **Lead sem ID válido:**
   - Warning no console
   - Lead é ignorado

## Configurações de Performance

- **Schedule Function:**
  - Timeout: 900s para processar todos os leads
  - Memória: 256 MB para operações de banco

- **Worker Function:**
  - Timeout: 180s por lead
  - Memória: 128 MB suficiente para processamento
  - Batch size 1: Processamento sequencial evita sobrecarga
  - Delay 15s: Respeita rate limits da API

## Monitoramento

### Métricas Importantes

1. Número de leads coletados diariamente
2. Taxa de sucesso no processamento
3. Tempo de execução da schedule function
4. Mensagens na fila SQS
5. Dead Letter Queue (se configurada)

### Logs

- Logs de conexão com MySQL
- Logs de chamadas à API Feegow
- Logs de erros ao verificar agendamentos
- Logs de leads ignorados

## Manutenção

### Pontos de Atenção

1. **Data de Execução:**
   - Schedule busca leads do dia anterior
   - Ajustar se necessário alterar janela de tempo

2. **Limites de API:**
   - Feegow e RD Station possuem rate limits
   - Delay de 15s ajuda a respeitar limites

3. **Duplicação:**
   - RD Station busca organização existente antes de criar
   - Evita duplicação de leads

4. **Timeout:**
   - Schedule function tem 15 minutos
   - Se volume de leads ultrapassar capacidade, considerar paginação
