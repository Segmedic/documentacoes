# Relação com a API Feegow

## Visão Geral

Este sistema integra-se com a API da Feegow para gerenciar agendamentos médicos, pacientes, profissionais, unidades e outros recursos relacionados à gestão de clínicas.

---

## Configuração

### Arquivos de Configuração

#### `src/config/feegowRoutes.ts`

Define todas as rotas e endpoints da API Feegow:

- **Base URLs:**
  - v1: `https://api.feegow.com/v1/api/`
  - v2: `https://api.feegow.com/v1/api/v2/`

#### `src/config/feegowXAccessToken.ts`

Armazena o token de autenticação JWT para requisições à API.

---

## Instâncias Axios

### `feegowApi` (v1)

Localização: `src/services/feegowApi.ts`

Cliente HTTP configurado para:

- Base URL: API v1
- Header de autenticação: `x-access-token`

### `feegowApiSchedule` (v2)

Localização: `src/services/feegowApiSchedule.ts`

Cliente HTTP para endpoints v2 (agendamentos avançados).

---

## Endpoints Utilizados

### 1. **Agendamentos**

#### Buscar Horários Disponíveis

- **Endpoint:** `appoints/available-schedule`
- **Método:** GET
- **Uso:**
  - `Schedule.index()` - Busca horários disponíveis por procedimento/especialidade
- **Parâmetros:**
  - `data_start`, `data_end`
  - `tipo` (E=Especialidade, P=Procedimento)
  - `procedimento_id`, `especialidade_id`
  - `profissional_id`, `unidade_id`
  - `convenio_id`, `convenio_plano_id`

#### Listar Agendamentos

- **Endpoint:** `appoints/search`
- **Método:** GET
- **Uso:**
  - `Schedule.get()` via função `getSchedules()`
- **Parâmetros:**
  - `paciente_id`
  - `data_start`, `data_end`
- **Retorno:** Lista de agendamentos com dados do procedimento

#### Criar Agendamento

- **Endpoint:** `appoints/new-appoint`
- **Método:** POST
- **Uso:**
  - `Schedule.store()`
- **Parâmetros:**
  - `paciente_id`, `data`, `horario`
  - `procedimento_id`, `especialidade_id`
  - `local_id`, `profissional_id`
  - `valor`, `convenio_id`, `convenio_plano_id`
  - `tabela_id`, `plano` (0=sem convênio, 1=com convênio)
  - `notas`, `sys_user`

#### Cancelar Agendamento

- **Endpoint:** `appoints/cancel-appoint`
- **Método:** POST
- **Uso:**
  - `Schedule.cancel()`
- **Parâmetros:**
  - `agendamento_id`
  - `motivo_id` (1=Paciente, 2=Profissional, 3=Clínica)
  - `obs`

#### Atualizar Status do Agendamento

- **Endpoint:** `appoints/statusUpdate`
- **Método:** POST
- **Uso:**
  - `Schedule.update()`
- **Parâmetros:**
  - `AgendamentoID`
  - `StatusID`
  - `Obs`

---

### 2. **Pacientes**

#### Buscar Paciente

- **Endpoint:** `patient/search`
- **Método:** GET
- **Uso:**
  - `Patient.fetch()`
- **Parâmetros:**
  - `paciente_cpf`

#### Criar Paciente

- **Endpoint:** `patient/create`
- **Método:** POST
- **Uso:**
  - `Patient.store()`
- **Parâmetros:**
  - `nome_completo`, `cpf`, `email`
  - `data_nascimento`, `sexo`, `telefone`

---

### 3. **Profissionais**

#### Listar Profissionais

- **Endpoint:** `professional/list`
- **Método:** GET
- **Uso:**
  - `Professionals.index()`
- **Parâmetros:**
  - `especialidade_id` (opcional)
  - `procedimento_id` (opcional)
  - `local_id` (opcional)
  - `convenio_id` (opcional)
- **Processamento:** Remove profissionais com `permite_agendamento_online = 0`

---

### 4. **Unidades**

#### Listar Unidades

- **Endpoint:** `company/list-unity`
- **Método:** GET
- **Uso:**
  - `Units.index()`
- **Filtros:**
  - Por convênio (via `feegowGetUnitsByInsurance`)
  - Apenas unidades com `agenda_online = 1`

#### Gerar Relatórios

- **Endpoint:** `reports/generate`
- **Método:** POST
- **Uso:**
  - `Units.generateReport()`
- **Parâmetros:**
  - `tipo_relatorio`
  - `data_start`, `data_end`
  - Outros parâmetros específicos do relatório

---

### 5. **Convênios**

#### Listar Convênios

- **Endpoint:** `insurance/list`
- **Método:** GET
- **Uso:**
  - `Insurance.index()`
- **Filtros:**
  - Apenas convênios com `exibir_agendamento_online = 1`

---

### 6. **Especialidades**

#### Listar Especialidades

- **Endpoint:** `specialties/list`
- **Método:** GET
- **Uso:**
  - `Specialities.index()`

---

### 7. **Procedimentos**

#### Listar Procedimentos

- **Endpoint:** `procedures/list`
- **Método:** GET
- **Uso:**
  - `Procedures.index()`
- **Parâmetros:**
  - `tipo_procedimento` (ex: 2=Consulta Médica)
  - `procedimento_id` (opcional)
  - `tabela_id` (ex: 37=Clubflex)
- **Filtros:**
  - Por convênio (via arquivo `InsuranceProceduresRelation.json`)

---

## Fluxo de Dados

### Fluxo de Agendamento Completo

1. **Cliente busca convênios** → `Insurance.index()`
2. **Cliente seleciona especialidade/procedimento** → `Specialities.index()` ou `Procedures.index()`
3. **Cliente busca unidades disponíveis** → `Units.index()`
4. **Sistema verifica se paciente existe** → `Patient.fetch()`
5. **Se não existe, cria paciente** → `Patient.store()`
6. **Cliente busca horários disponíveis** → `Schedule.index()`
7. **Cliente confirma agendamento** → `Schedule.store()`
8. **Cliente pode consultar agendamentos** → `Schedule.get()`
9. **Cliente pode cancelar** → `Schedule.cancel()`
10. **Sistema pode atualizar status** → `Schedule.update()`

---

## Funções Auxiliares

### `getSchedules(pacient_id)`
Localização: `src/services/feegowApi.ts`

Busca agendamentos de um paciente e enriquece com nome do procedimento.

### `getAvailableSchedules(pacient_id)`
Localização: `src/services/feegowApi.ts`

Busca horários disponíveis futuros (próximo mês).

### `getUnitsByInsurance(id, data)`
Localização: `src/inc/service/feegowGetUnitsByInsurance.ts`

Filtra unidades que atendem determinado convênio.

### `getOnlyProfessionals(schedule, total)`
Localização: `src/inc/service/getOnlyProfessionals.ts`

Extrai lista única de profissionais dos horários disponíveis.

### `getOnlyUnits(schedule, procedureId, planoId)`
Localização: `src/inc/service/getOnlyUnits.ts`

Extrai lista única de unidades dos horários disponíveis.

---

## Tratamento de Erros

Todas as requisições implementam tratamento de erros consistente:

```typescript
.catch((error: Error | AxiosError) => {
  if (axios.isAxiosError(error)) {
    res.json({
      success: false,
      content: "Mensagem de erro",
      err: "feegow",
      details: error.response?.data
    })
  }
})
```

---

## Mapeamento de Dados

### Enums Utilizados

#### Tipo de Agendamento

- `E` - Por Especialidade
- `P` - Por Procedimento

#### Plano

- `0` - Sem Convênio (Particular)
- `1` - Com Convênio

#### Motivo de Cancelamento

- `1` - Cancelado pelo Paciente
- `2` - Cancelado pelo Profissional
- `3` - Cancelado pela Clínica

### IDs Especiais

- `tipo_procedimento: 2` - Consulta Médica
- `tabela_id: 37` - Tabela Clubflex
- `sys_user: 0` - Agendamento via sistema

---

## Arquivos de Dados Estáticos

### `src/databases/Procedures.json`
Lista de procedimentos com mapeamento local.

### `src/databases/InsuranceProceduresRelation.json`
Relacionamento entre convênios e procedimentos aceitos.

---

## Utilitários

### `urlQueryBuilder`

Localização: `src/inc/urlQueryBuilder.ts`

Constrói URLs com query strings para requisições GET.

### `parseDate`

Localização: `src/utils.ts`

Formata datas no padrão esperado pela API Feegow.

---

## Schemas de Validação

### `scheduleSchema`

Valida dados de busca de horários disponíveis.

### `storeScheduleSchema`

Valida dados para criação de agendamento.

### `patientSchema`

Valida dados para criação de paciente.

---

## Observações Importantes

1. **Autenticação:** Todas as requisições requerem `x-access-token` no header
2. **Versões da API:** Sistema usa tanto v1 quanto v2 dos endpoints
3. **Filtros Locais:** Alguns filtros são aplicados após receber dados da Feegow (ex: `agenda_online`, `permite_agendamento_online`)
4. **Enriquecimento de Dados:** Sistema adiciona informações locais aos dados da Feegow (ex: nomes de procedimentos)
5. **Cache:** Não há implementação de cache, todas as consultas são feitas em tempo real

---

## Dependências

- **axios** - Cliente HTTP
- **TypeScript** - Tipagem e interfaces
- **Zod** - Validação de schemas
