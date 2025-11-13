# Documentação: Relação com a API Feegow

## 📋 Visão Geral

Este documento descreve como o sistema interage com a API do Feegow, incluindo configurações, endpoints utilizados e casos de uso.

---

## 🔐 Configuração de Acesso

**Base URL:** `https://api.feegow.com/v1/api`

**Autenticação:** Token JWT via header `x-access-token`

**Credenciais:** Armazenadas em variáveis de ambiente (ver `pass.txt`)

---

## 🔧 Cliente Feegow

### Localização

`src/feegow/client.ts` - Classe `FeegowClient`

### Configuração

```typescript
baseURL: "https://api.feegow.com/v1/api"
headers: {
  "x-access-token": process.env.FEEGOW_TOKEN,
  "Content-Type": "application/json"
}
```

---

## 📡 Endpoints e Métodos

### 👤 Pacientes

#### `patientByCpf(cpf: Number)`

Busca paciente por CPF.

- **Endpoint:** `GET /patient/search?paciente_cpf={cpf}`
- **Retorno:** `PatientByCpfResponse`
- **Uso:** Localizar paciente específico pelo CPF

#### `patientById(paciente_id: Number)`

Busca paciente por ID.

- **Endpoint:** `GET /patient/search?paciente_id={id}`
- **Uso:** Recuperar dados completos do paciente

#### `getPatientByPhone(phone: string)`

Busca paciente por telefone.

- **Endpoint:** `GET /patient/list?telefone={phone}`
- **Retorno:** Primeiro paciente da lista
- **Uso:** Identificar paciente através do número de telefone

#### `getPatientByPhoneOrUndefined(phone: string)`

Busca paciente por telefone com tratamento de erro.

- **Retorno:** Paciente ou `undefined`
- **Uso:** Busca segura sem throw de exceção

---

### 📅 Agendamentos

#### `getSchedules(dateBegin: Date, dateEnd: Date)`

Lista agendamentos em um período.

- **Endpoint:** `POST /reports/generate?report=schedule-appointments&DATA_INICIO={dd/MM/yyyy}&DATA_FIM={dd/MM/yyyy}`
- **Retorno:** `Agendamento[]`
- **Formato de data:** `dd/MM/yyyy`
- **Uso:** Recuperar todos os agendamentos do período

#### `listSchedulesByPatientId(patientId: string, data_start: Date, data_end: Date)`

Lista agendamentos de um paciente específico.

- **Endpoint:** `GET /appoints/search?patient_id={id}&data_start={yyyy/MM/dd}&data_end={yyyy/MM/dd}`
- **Retorno:** `AgendamentoResponse`
- **Formato de data:** `yyyy/MM/dd`
- **Uso:** Histórico de agendamentos de um paciente

---

### 🏥 Recursos

#### `getSpecialties()`

Lista todas as especialidades médicas.

- **Endpoint:** `GET /specialties/list`
- **Retorno:** `EspecialidadesResponse`
- **Uso:** Mapear especialidades disponíveis

#### `listProfissionals()`

Lista profissionais ativos.

- **Endpoint:** `GET /professional/list?ativo=1`
- **Retorno:** `Profissionals[]`
- **Uso:** Listar médicos e profissionais disponíveis

#### `listUnits()`

Lista todas as unidades de atendimento.

- **Endpoint:** `GET /company/list-unity`
- **Retorno:** `Unit[]` (matriz + unidades combinadas)
- **Uso:** Mapear locais de atendimento

---

### 📊 Relatórios

#### `generateReport(dateBegin: string, dateEnd: string, report: string)`

Gera relatório customizado.

- **Endpoint:** `POST /reports/generate?report={tipo}&DATA_INICIO={inicio}&DATA_FIM={fim}`
- **Uso:** Gerar relatórios personalizados

---

## 📦 Tipos de Dados

### Paciente (`Content`)

**Arquivo:** `src/feegow/types/patient.ts`

Principais campos:

- `id`: ID do paciente
- `nome`: Nome completo
- `cpf`: CPF
- `nascimento`: Data de nascimento
- `sexo`: M/F
- `telefones[]`: Lista de telefones
- `celulares[]`: Lista de celulares
- `email[]`: Lista de emails
- `convenios[]`: Convênios do paciente
- Endereço completo (rua, número, bairro, cidade, estado, CEP)

### Agendamento (`Agendamento`)

**Arquivo:** `src/feegow/types/agendamentos.ts`

Principais campos:

- `AgendamentoID`: ID do agendamento
- `PacienteID`: ID do paciente
- `Data`: Data do agendamento
- `Hora`: Horário
- `NomePaciente`: Nome do paciente
- `NomeProfissional`: Nome do profissional
- `NomeEspecialidade`: Especialidade
- `NomeUnidade`: Unidade de atendimento
- `CPF`: CPF do paciente
- `Cel1`: Celular do paciente
- `StaConsulta`: Status da consulta
- `ConvenioID` e `NomeConvenio`: Convênio
- `NomeTabela`: Tabela de preço

### Agendamento de Paciente (`AgendamentoItem`)

**Arquivo:** `src/feegow/types/agendamentoPatient.ts`

Formato retornado ao buscar agendamentos por paciente.

### Especialidade (`Especialidade`)

**Arquivo:** `src/feegow/types/speciality.ts`

Campos:

- `especialidade_id`: ID
- `nome`: Nome da especialidade
- `codigo_tiss`: Código TISS (opcional)

### Profissional (`Profissionals`)

**Arquivo:** `src/feegow/types/profissionals.ts`

Campos:

- `profissional_id`: ID
- `nome`: Nome completo
- `ativo`: Status
- `especialidades[]`: Array de especialidades
- `cpf`, `email`, etc.

### Unidade (`Unit`)

**Arquivo:** `src/feegow/types/units.ts`

Campos:

- `unidade_id`: ID
- `nome_fantasia`: Nome da unidade
- `cnpj`: CNPJ
- Endereço completo
- `telefone_1`, `telefone_2`
- `email_1`, `email_2`

---

## 🎯 Casos de Uso

### 1. **Leads de Amanhã** (`src/schedules/leads_amanha.ts`)

- Busca agendamentos do dia seguinte
- Coleta telefones e CPFs dos pacientes
- Envia dados para fila SQS para processamento no CRM

**Métodos usados:**

- `getSchedules(start, end)`

### 2. **Recuperação de Agendamento** (`src/schedules/recuperacao_agendamento.ts`)

- Busca agendamentos em um período de 15 dias
- Busca especialidades para categorização
- Compara com leads no banco de dados
- Envia conversões para RD Station

**Métodos usados:**

- `getSpecialties()`
- `getSchedules(yesterday, fifteenDaysAhead)`

### 3. **Parser de Callbacks do Escallo** (`src/rd/callbackParser.ts`)

- Recebe dados de ligações/chats do Escallo
- Busca paciente por telefone no Feegow
- Enriquece dados para envio ao RD Station CRM

**Métodos usados:**

- `getPatientByPhoneOrUndefined(number)`

### 4. **Serviço de Agendamento** (`src/schedule/scheduleService.ts`)

- Gerencia recuperação de carrinho abandonado
- Busca pacientes e agendamentos

**Métodos usados:**

- `FeegowClient` é instanciado no construtor

---

## 🔄 Integrações

### RD Station

O Feegow é usado para enriquecer dados antes de enviar para o RD Station:

- Identificar pacientes existentes
- Validar agendamentos
- Mapear especialidades e unidades

### Medula (Data Warehouse)

Dados do Feegow são cruzados com o banco Medula para:

- Verificar histórico de atendimentos
- Validar convênios
- Calcular métricas de recorrência

### ClubFlex

Integração para:

- Validar dependentes removidos
- Processar retenção B2B

---

## 🗺️ Mapeamentos Auxiliares

### Unidades (`src/utils/constantes.ts`)

```typescript
unityAtendimentMap: {
  "Unidade Campo Grande" → "Campo Grande"
  "Unidade Meier" → "Meier"
  "Centro Medico Matriz" → "Nova Iguaçu"
  // etc...
}
```

### Especialidades (`src/utils/determinaEspecialidade.ts`)

Mapeia especialidades do Feegow para categorias padronizadas do RD Station.

### Faixa Etária (`src/utils/determinaFaixaEtaria.ts`)

Calcula faixa etária com base na data de nascimento do paciente.

---

## ✅ Tabelas Válidas

Convênios aceitos para processamento (`src/utils/constantes.ts`):

- Particular
- Interclinica
- Interclinica 500
- Interclinicas Coleta Domiciliar
- Interclinicas Faturado
- Sindicato Dos Rodoviários 2024.3
- Sindicato dos Rodoviários
- Sindicato dos Rodoviários - Faturado

---

## 📝 Observações

1. **Formato de Data:** Atenção aos formatos diferentes:
   - Relatórios de agendamento: `dd/MM/yyyy`
   - Busca por paciente: `yyyy/MM/dd`

2. **Normalização de Telefone:** Telefones são normalizados antes de buscar no Feegow

3. **Tratamento de Erro:** `getPatientByPhoneOrUndefined` não lança exceção, retorna `undefined`

4. **Unidades:** O método `listUnits()` combina matriz e unidades em um único array

5. **Validação de Status:** Apenas profissionais ativos (`ativo=1`) são listados
