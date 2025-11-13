# Documentação - Integração com API Feegow

## Visão Geral

Este projeto integra com a **API Feegow** (sistema de gestão para clínicas médicas) para automatizar processos relacionados a agendamentos, propostas e dados de pacientes.

## Estrutura da Integração

### 1. Cliente HTTP (`src/feegow/client.ts`)

Classe `FeegowClient` responsável por todas as comunicações com a API.

**Configuração:**

- Base URL: `https://api.feegow.com/v1/api`
- Autenticação: Header `x-access-token` (JWT Token)
- Protocolo: HTTPS com Axios

### 2. Endpoints Utilizados

#### 2.1 Buscar Paciente

```typescript
GET /patients/{patient_id}
```

- **Método:** `getClient(patient_id: number)`
- **Uso:** Obter dados detalhados de um paciente específico
- **Retorno:** Dados do paciente

#### 2.2 Buscar Dados do Paciente (Busca)

```typescript
GET /patient/search?paciente_id={id}
```

- **Método:** `getPatient(paciente_id: number)`
- **Uso:** Buscar informações do paciente via parâmetro de query
- **Retorno:** Objeto `Patient` com dados completos (nome, nascimento, endereço, CPF, telefones, etc.)

#### 2.3 Gerar Relatório de Propostas (Coleta)

```typescript
POST /reports/generate
Body: {
  DATA_INICIO: string,
  DATA_FIM: string,
  report: "proposal",
  TIPO_DATA_PROPOSTA: ["CRIACAO"],
  AGRUPAR_POR_PROPOSTA: ["S"],
  PROCEDIMENTO_IDS: array,
  PROPOSTA_STATUS_IDS: ["1"]
}
```

- **Método:** `proposals(date: string)`
- **Uso:** Buscar propostas de exames de coleta criadas em uma data específica
- **Filtros:** Apenas propostas com status ID = 1 (ativas)
- **Procedimentos:** IDs definidos em `groupExamColeta`

#### 2.4 Gerar Relatório de Propostas (RC)

```typescript
POST /reports/generate
```

- **Método:** `proposalsRc(date: string)`
- **Uso:** Buscar propostas de exames RC (Ressonância/Tomografia)
- **Diferença:** Usa `groupExamRc` como filtro de procedimentos

#### 2.5 Gerar Relatório de Agendamentos

```typescript
POST /reports/generate
Body: {
  DATA_INICIO: string,
  DATA_FIM: string,
  report: "schedule-appointments"
}
```

- **Método:** `schedules(date: string)`
- **Uso:** Listar todos os agendamentos do dia
- **Retorno:** Array de `Agendamento[]`

#### 2.6 Gerar Relatório de Procedimentos

```typescript
POST /reports/generate
Body: {
  report: "procedures"
}
```

- **Método:** `procedures()`
- **Uso:** Listar todos os procedimentos cadastrados no sistema

#### 2.7 Buscar Agendamentos por Paciente

```typescript
GET /appoints/search?paciente_id={id}&data_start={date}&data_end={date}
```

- **Método:** `schedulesByPatientId(paciente_id: number)`
- **Uso:** Verificar agendamentos futuros de um paciente específico
- **Período:** 1 dia anterior até 30 dias futuros
- **Retorno:** `AppointmentsResponse` com lista de agendamentos

### 3. Constantes (`src/feegow/constants.ts`)

- **Token de Autenticação:** Credencial JWT para acesso à API
- **Status de Agendamentos:** Mapeamento de IDs para descrições (Atendido, Aguardando, Marcado, etc.)
- **Procedimentos:** Dicionário com IDs e nomes de especialidades médicas
- **Grupos de Exames:**
  - `groupExamColeta`: IDs de procedimentos de exames laboratoriais
  - `groupExamRc`: IDs de procedimentos de imagem (RC)

### 4. Tipos TypeScript

#### `Agendamento`

Representa um agendamento completo com:
- Dados do paciente (nome, CPF, telefone, endereço)
- Dados da consulta (data, hora, profissional, especialidade)
- Status e informações de faturamento
- Origem e canal de agendamento

#### `AppointmentsResponse`

Resposta da busca de agendamentos contendo:
- Lista de agendamentos futuros
- IDs de agendamento, procedimento, status, local, profissional

#### `Patient`

Dados completos do paciente:
- Informações pessoais (nome, nascimento, sexo)
- Contatos (celulares, emails)
- Endereço completo
- Documentos (RG, CPF)

## Fluxos de Uso

### Fluxo 1: Processamento de Consultas

**Arquivo:** `src/schedules/consultas.ts`

1. Busca leads no banco de dados local (sem agendamento confirmado)
2. Para cada lead com `pacienteId`:
   - Consulta API Feegow: `schedulesByPatientId()`
   - Verifica se existem agendamentos válidos no período
3. Filtra leads sem agendamentos recentes
4. Envia para fila SQS para processamento

### Fluxo 2: Processamento de Propostas (Exames Coleta)

**Arquivo:** `src/schedules/proposals.ts`

1. Busca propostas do dia anterior via `proposals()`
2. Filtra apenas propostas ativas (status = 1)
3. Agrupa propostas por paciente
4. Distribui entre atendentes
5. Envia para fila SQS

### Fluxo 3: Processamento de Propostas RC

**Arquivo:** `src/schedules/proposals-rc.ts`

1. Busca propostas RC do dia anterior via `proposalsRc()`
2. Filtra propostas ativas
3. Agrupa por paciente
4. Define agente como "ATIVOS"
5. Envia para fila SQS

### Fluxo 4: Processamento de Exames

**Arquivo:** `src/schedules/exam.ts`

1. Busca leads de exames no banco local
2. Para cada lead:
   - Consulta agendamentos via `schedulesByPatientId()`
   - Valida se não há agendamentos no período
3. Remove duplicatas por nome
4. Envia leads qualificados para fila SQS

## Validações e Regras de Negócio

### Validação de Agendamentos Válidos

- Agendamentos fora do próximo mês são considerados inválidos
- Agendamentos com mais de 1 mês desde a criação são considerados inválidos
- Pacientes sem agendamentos ou com agendamentos inválidos são priorizados

### Filtros de Status

- **Propostas:** Apenas status ID = 1 (propostas ativas/pendentes)
- **Agendamentos:** Múltiplos status mapeados (Marcado, Atendido, Não compareceu, etc.)

### Períodos de Busca

- **Propostas:** Dia anterior (D-1)
- **Agendamentos por paciente:** 1 dia anterior até 30 dias futuros
- **Leads de consultas/exames:** Dia anterior (D-1)

## Utilitários

### Link para Paciente

```typescript
FeegowClient.linkPaciente(pacienteId: number)
```

Gera URL direta para o cadastro do paciente na interface web do Feegow:
`https://app.feegow.com/pre-v8/?P=pacientes&I={pacienteId}&Pers=1`

## Observações Importantes

- Todas as requisições utilizam autenticação via token JWT no header
- Datas são formatadas no padrão `dd/MM/yyyy` ou `dd-MM-yyyy` conforme o endpoint
- Tratamento de erros implementado com logs detalhados
- Suporte a múltiplas unidades e profissionais
- Integração com AWS SQS para processamento assíncrono
