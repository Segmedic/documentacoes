# Documentação: Integração com API Feegow

## Visão Geral

Este projeto utiliza a API da Feegow para gerenciar o sistema de agendamento de exames médicos. A integração abrange funcionalidades de consulta, criação e gerenciamento de agendamentos, pacientes, convênios e unidades.

---

## Configuração

### Variáveis de Ambiente

```env
FEEGOW_ACCESS_TOKEN=<seu-token-aqui>
```

### Funções Auxiliares

- **`getFeegowAccessToken()`** → `src/app/lib/functions/getFeegowAccessToken.ts`
  - Retorna o token de acesso da API

- **`getFeegowUrl()`** → `src/app/lib/functions/getFeegowUrl.ts`
  - Retorna a URL base: `https://api.feegow.com/v1/api`

---

## Endpoints Utilizados

### 1. 📅 Agendamentos (Appointments)

#### 1.1 Buscar Horários Disponíveis

**Endpoint Feegow:**
```
GET /v2/appoints/available-schedule
```

**Implementação Backend:**
- Arquivo: `src/app/lib/services/Feegow/index.ts`
- Função: `ProcedureAvailableSchedule(ProcID, convenioId?)`

**Rota API Local:**
```
POST /api/feegow/schedule/list
```

**Parâmetros:**
- `tipo=P` (Procedimento)
- `procedimento_id` - ID do procedimento/exame
- `data_start` - Data inicial (formato ISO8601)
- `data_end` - Data final (60 dias à frente)
- `convenio_id` - (Opcional) ID do convênio

**Função Frontend:**
- Função: `getSchedule(ProcID)`
- Arquivo: `src/app/lib/services/Feegow/index.ts`
- Aplica filtros por convênio e regras de negócio específicas

**Uso:**
```typescript
const scheduleData = await getSchedule(procedureId)
```

---

#### 1.2 Criar Novo Agendamento

**Endpoint Feegow:**
```
POST /appoints/new-appoint
```

**Implementação Backend:**
- Arquivo: `src/app/lib/services/Schedule/index.ts`
- Função: `InsertScheduleFX(InsertSchedule)`

**Rota API Local:**
```
POST /api/feegow/schedule/post
```

**Dados Enviados:**
```typescript
{
  data: string,              // Data do agendamento
  horario: string,           // Horário (HH:MM:SS)
  local_id: number,          // ID do local
  notas: string,             // Observações e dados do paciente
  paciente_id: number,       // ID do paciente
  procedimento_id: number,   // ID do procedimento
  profissional_id: number,   // ID do profissional
  valor: number,             // Valor em centavos
  convenio_id: number,       // ID do convênio (0 se particular)
  plano: number,             // 0 = sem convênio, 1 = com convênio
  convenio_plano_id: number  // ID do plano do convênio
}
```

**Função Frontend:**
- Função: `HandleInsertSchedule(ScheduleState)`
- Arquivo: `src/app/lib/services/Schedule/index.ts`
- Prepara dados de triagem e formata requisição

**Uso:**
```typescript
const { success, content } = await HandleInsertSchedule(scheduleData)
```

---

#### 1.3 Cancelar Agendamento

**Endpoint Feegow:**
```
POST /appoints/statusUpdate
```

**Implementação Backend:**
- Arquivo: `src/app/lib/services/Schedule/index.ts`
- Função: `deleteScheduleFX(schedule)`

**Rota API Local:**
```
DELETE /api/feegow/schedule/delete
```

**Dados Enviados:**
```typescript
{
  AgendamentoID: number,
  StatusID: 11,  // Status de cancelado
  Obs: "Desmarcado pelo paciente"
}
```

---

#### 1.4 Buscar Agendamentos por Paciente

**Implementação:**
- Arquivo: `src/app/lib/services/Feegow/index.ts`
- Função: `getSchedulesByPatientId(pacienteID)`

**Nota:** Utiliza backend legado (NEXT_PUBLIC_OLD_BACKEND_URL)

**Filtros Aplicados:**
- Retorna apenas agendamentos com `status_id` = 1 (Confirmado) ou 7 (Pendente)

**Uso:**
```typescript
const agendamentos = await getSchedulesByPatientId(pacienteId)
```

---

### 2. 👤 Pacientes (Patients)

#### 2.1 Buscar Paciente por CPF

**Endpoint Feegow:**
```
GET /patient/search?paciente_cpf={cpf}
```

**Rota API Local:**
```
POST /api/patient
```

**Arquivo:** `src/app/api/patient/route.ts`

**Dados Enviados:**
```typescript
{
  pacienteCPF: string  // CPF com ou sem máscara
}
```

---

#### 2.2 Criar Novo Paciente

**Endpoint Feegow:**
```
POST /patient/create
```

**Implementação Backend:**
- Arquivo: `src/app/lib/services/Patient/index.ts`
- Função: `CreatePatient(params)`

**Rota API Local:**
```
POST /api/patient/create
```

**Dados Enviados:**
```typescript
{
  ...params,
  cpf: string,
  origemId: 1,     // ID de origem fixo
  tabela_id: 5     // ID de tabela fixo
}
```

**Função Frontend:**
- Função: `CreatePatientHandler(params)`
- Arquivo: `src/app/lib/services/Patient/index.ts`

---

### 3. 🏥 Convênios (Insurance)

#### 3.1 Listar Convênios

**Endpoint Feegow:**
```
GET /insurance/list
```

**Rota API Local:**
```
POST /api/feegow/insurance/list
```

**Arquivo:** `src/app/api/feegow/insurance/list/route.ts`

**Processamento:**
1. Busca todos os convênios da API Feegow
2. Filtra apenas convênios com `exibir_agendamento_online = 1`
3. Filtra por IDs específicos que possuem logos no sistema
4. Transforma formato Feegow para formato Frontend

**IDs de Convênios Permitidos:**
```typescript
[2, 5, 6, 8, 9, 10, 11, 13, 14, 16, 448, 836, 840, 1094, 
 1378, 1421, 2463, 2467, 2466, 2573, 2576, 2582, 2580, 2619, 2620]
```

**Formato de Resposta:**
```typescript
{
  success: boolean,
  content: InsuranceFrontend[],
  count: number
}

// InsuranceFrontend
{
  ID: number,
  nome: string,
  show: boolean,
  telemedicina: boolean,
  planos: [
    { id: number, name: string }
  ]
}
```

---

### 4. 🏢 Unidades (Unity)

#### 4.1 Listar Unidades

**Endpoint Feegow:**
```
GET /company/list-unity
```

**Implementação Backend:**
- Arquivo: `src/app/lib/services/Feegow/index.ts`
- Função: `getUnity()`

**Rota API Local:**
```
GET /api/feegow/unity/list
```

**Funções Frontend:**

**`listUnity()`**
- Lista todas as unidades disponíveis

**`getAvailableUnity(scheduleData)`**
- Retorna unidades disponíveis para os horários agendados
- Relaciona `unidade_id` com dados completos da unidade
- Trata unidade matriz (`unidade_id = 0`)

**`getScheduleByUnity(schedule, unityId)`**
- Filtra agendamentos por unidade específica
- Retorna apenas horários disponíveis

**`getAdditionalScheduleInfo(schedule, Unity, Date)`**
- Busca informações adicionais de um agendamento específico
- Filtra por unidade e data

---

#### 4.2 Listar Unidades por Convênio

**Implementação:**
- Arquivo: `src/app/lib/services/Feegow/index.ts`
- Função: `getUnitsByConvenio(convenio)`

**Nota:** Utiliza backend legado (NEXT_PUBLIC_OLD_BACKEND_URL)

**Uso:**
```typescript
const units = await getUnitsByConvenio(convenioId)
```

---

## Fluxo de Dados

### 1. Fluxo de Agendamento

```
1. Usuário seleciona convênio → GET /insurance/list
2. Escolhe procedimento → Dados locais (procedures.json)
3. Busca horários → GET /v2/appoints/available-schedule
4. Seleciona unidade → GET /company/list-unity
5. Seleciona data/hora → Dados filtrados localmente
6. Confirma agendamento → POST /appoints/new-appoint
```

### 2. Fluxo de Cadastro de Paciente

```
1. Preenche CPF → GET /patient/search
2. Se não existir → POST /patient/create
3. Salva ID do paciente → SessionStorage
4. Prossegue para agendamento
```

---

## Estrutura de Arquivos

### Services (Camada de Serviço)

```
src/app/lib/services/
├── Feegow/
│   └── index.ts          # Funções de agendamento, unidades e consulta
├── Schedule/
│   └── index.ts          # CRUD de agendamentos
└── Patient/
    └── index.ts          # CRUD de pacientes
```

### API Routes (Next.js API)

```
src/app/api/
├── feegow/
│   ├── schedule/
│   │   ├── list/route.ts    # POST - Lista horários disponíveis
│   │   ├── post/route.ts    # POST - Cria agendamento
│   │   └── delete/route.ts  # DELETE - Cancela agendamento
│   ├── insurance/
│   │   └── list/route.ts    # POST - Lista convênios
│   └── unity/
│       └── list/route.ts    # GET - Lista unidades
└── patient/
    ├── route.ts             # POST - Busca paciente por CPF
    └── create/route.ts      # POST - Cria novo paciente
```

### Types (TypeScript)

```
src/app/lib/types/
├── Schedule.d.ts        # Tipos de agendamento
├── Insurance.d.ts       # Tipos de convênio
└── Patient.d.ts         # Tipos de paciente
```

---

## Componentes que Utilizam a API

### Frontend Components

1. **FormConvenio** (`src/app/components/FormConvenio/index.tsx`)
   - Lista convênios disponíveis
   - Busca unidades por convênio

2. **FormProcedimento** (`src/app/components/client/Procedimento/index.tsx`)
   - Busca agendamentos do paciente
   - Exibe consultas agendadas

3. **Agendamento** (`src/app/components/client/Agendamento/index.tsx`)
   - Coordena todo o fluxo de agendamento
   - Busca horários disponíveis
   - Confirma agendamento

4. **ProcedureUnity** (`src/app/components/client/Agendamento/ProcedureContent/ProcedureUnity/index.tsx`)
   - Seleciona unidade de atendimento
   - Filtra unidades disponíveis

---

## Regras de Negócio

### Filtros Aplicados

1. **Convênios:**
   - Apenas convênios com `exibir_agendamento_online = 1`
   - Lista específica de IDs permitidos (logos disponíveis)

2. **Agendamentos:**
   - Busca sempre 60 dias à frente
   - Filtra por status: apenas confirmados (1) e pendentes (7)
   - Procedimento ID 216 + plano específico → Remove profissional 18246

3. **Profissionais:**
   - Filtro adicional por convênio utilizando backend legado
   - Verifica limites de profissionais por convênio

### Valores e Formatação

- **Valores monetários:** Enviados em centavos (`valor * 100`)
- **Horários:** Formato `HH:MM:SS`
- **Datas:** Formato ISO8601
- **CPF:** Remove máscara antes de enviar

---

## Autenticação

Todas as requisições para a API Feegow incluem:

```typescript
headers: {
  'x-access-token': FEEGOW_ACCESS_TOKEN,
  'Content-Type': 'application/json'
}
```

---

## Tratamento de Erros

### Estratégias Implementadas

1. **Validação de Resposta:**
   - Verifica `response.ok`
   - Valida Content-Type: `application/json`

2. **Fallback:**
   - Retorna arrays vazios em caso de erro
   - `getSchedulesByPatientId` retorna `[]` em falha

3. **Status de Resposta:**
   - Success: `200`
   - Error: `400` com mensagem descritiva

---

## Dados Armazenados (SessionStorage)

O sistema utiliza SessionStorage para manter estado entre páginas:

- `procedimentos` - Procedimentos selecionados
- `dados_triagem` - Dados de saúde do paciente
- `schedule` - Dados do agendamento em andamento
- `patient` - ID do paciente
- `atendimento` - Tipo de atendimento selecionado
- `client_atendimento` - ID do convênio
- `client_plan` - Plano do convênio selecionado
- `clubflex` - Flag para agendamentos ClubFlex

---

## Observações Importantes

1. **Backend Legado:** Algumas funcionalidades ainda dependem do backend antigo:
   - Busca de agendamentos por paciente
   - Filtro de profissionais por convênio
   - Unidades por convênio

2. **Dados Locais:** Lista de procedimentos é mantida em arquivo local:
   - `src/app/lib/static/procedures.json`

3. **Notas de Agendamento:** Inclui automaticamente:
   - Dados de saúde do paciente (triagem)
   - Pedido médico
   - Tag `#agWebExames`

4. **Valores Fixos:**
   - `origemId: 1` - Origem dos pacientes
   - `tabela_id: 5` - Tabela de preços
   - `StatusID: 11` - Status de cancelamento

---

## Melhorias Futuras Sugeridas

1. Migrar funcionalidades do backend legado para a API Feegow
2. Implementar cache para lista de convênios e unidades
3. Adicionar retry logic para requisições falhadas
4. Implementar logs estruturados de requisições
5. Adicionar testes automatizados para integração com API

---

**Última atualização:** Novembro 2025
