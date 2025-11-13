# Integração com API Feegow

## Configuração

**Base URL:** `https://api.feegow.com/v1/api`

**Autenticação:** Token JWT via header `x-access-token`

**Localização:** `src/feegow/client.ts`

---

## Endpoints Utilizados

### 1. POST `/reports/generate`

**Método:** `postGenerationReport()`

Gera relatórios de pacientes agendados ou atendidos.

**Parâmetros:**

- `report`: `"schedule-appointments"` ou `"duration-of-service"`
- `DATA_INICIO`: Data inicial (dd/MM/yyyy)
- `DATA_FIM`: Data final (dd/MM/yyyy)

**Retorna:** Lista de agendamentos (`Schedule`) ou atendimentos (`ScheduleServed`)

---

### 2. GET `/appoints/search`

**Método:** `getAppoints()`

Busca agendamentos de um paciente específico.

**Parâmetros:**

- `paciente_id`: ID do paciente
- `data_start`: Data inicial (dd-MM-yyyy)
- `data_end`: Data final (dd-MM-yyyy)

**Retorna:** Array de `ResponseAppointment`

---

### 3. GET `/patient/list`

**Método:** `patientByCpf()`

Busca dados do paciente pelo CPF.

**Parâmetros:**

- `cpf`: CPF (somente números)

**Retorna:** Dados do paciente

---

### 4. GET `/appoints/search` (variante)

**Método:** `schedulesPatient()`

Busca agendamentos de um paciente em período específico.

**Parâmetros:**

- `paciente_id`: ID do paciente
- `data_start`: Data inicial
- `data_end`: Data final

**Retorna:** Lista de agendamentos

---

## Camada de Serviço

**Localização:** `src/feegow/service.ts`

### Métodos Principais

#### `postPatientsServed(initialDate, finishDate)`

Busca pacientes **atendidos** em um período.

- Usa relatório `"duration-of-service"`
- Filtra "Visita Representante"
- Retorna `ScheduleServed[]`

#### `postScheduledPatient(initialDate, finishDate)`

Busca pacientes **agendados** em um período.

- Usa relatório `"schedule-appointments"`
- Filtra "Visita Representante"
- Retorna `Schedule[]`
- **Usado em:** confirmação, lembretes, no-show, CSAT

#### `getAppoints(patient_id, data_start, data_end)`

Busca agendamentos específicos de um paciente.

- **Usado em:** `schedulesNoShow.ts` para verificar histórico

#### `searchPatientByCpf(cpf)`

Busca paciente pelo CPF (normaliza automaticamente).

#### `patientAppointmentsInTheLast15Days(patient_id)`

Busca agendamentos dos últimos 15 dias de um paciente.

---

## Onde é Usado

### schedulesConfirmation.ts

```typescript
feegowService.postScheduledPatient(tomorrow, inFiveDays)
```

- Busca agendamentos de D+1 até D+5
- Filtra status "Marcado - não confirmado"
- Envia confirmações via Escallo

### schedulesReminder.ts

```typescript
feegowService.postScheduledPatient(tomorrow, tomorrow)
```

- Busca agendamentos do dia seguinte
- Envia lembretes

### schedulesNoShow.ts

```typescript
feegowService.postScheduledPatient(yesterday, yesterday)
feegowService.getAppoints({ patient_id, data_end, data_start })
```

- Busca agendamentos do dia anterior
- Verifica histórico de comparecimento
- Identifica faltas (no-show)

### schedulesCsat.ts

```typescript
feegowService.postScheduledPatient(twoDaysAgo, threeDaysAgo)
```

- Busca agendamentos de 2-3 dias atrás
- Envia pesquisa de satisfação

---

## DTOs (Tipos de Dados)

**Localização:** `src/dtos/feegow/`

### Schedule

Representa um **agendamento** com principais campos:

- `AgendamentoID`, `PacienteID`, `UnidadeID`
- `NomePaciente`, `Cel1`, `CPF`, `Email1`
- `Data`, `Hora`, `StaConsulta`
- `NomeProcedimento`, `NomeEspecialidade`
- `NomeProfissional`, `NomeUnidade`

### ScheduleServed

Representa um **atendimento realizado** com campos:

- `id`, `AgendamentoID`, `PacienteID`
- `Data`, `HoraInicio`, `HoraFim`
- `TempoPermanencia`, `TempoEspera`
- `NomeProcedimento`, `NomeProfissional`

### ResponseAppointment

Resposta da busca de agendamentos:

- `agendamento_id`, `paciente_id`, `data`, `horario`
- `status_id`, `profissional_id`, `procedimento_id`
- `telemedicina`, `primeiro_agendamento`

---

## Formatos de Data

| Uso | Formato | Exemplo |
|-----|---------|---------|
| Relatórios (query) | dd/MM/yyyy | 13/11/2025 |
| Agendamentos (query) | dd-MM-yyyy | 13-11-2025 |
| Retorno API | dd-MM-yyyy | 13-11-2025 |
| DateTime | yyyy-MM-dd HH:mm:ss | 2025-11-13 14:30:00 |

---

## Regras de Negócio

### Filtros Aplicados

- **Procedimento excluído:** "Visita Representante"
- **Unidade excluída:** "Serviços Remotos - Ni"
- **Status monitorados:** "Marcado - não confirmado", "Atendido", "Aguardando"

### Transformação de Dados

Dados da Feegow são transformados para formato Escallo (`CampaignData`):

```typescript
{
    chaveExterna: getUnityName(NomeUnidade),
    contato: normalizePhoneNumber(Cel1),
    paciente_id: PacienteID,
    paciente_nome: NomePaciente,
    empresa_id: UnidadeID,
    atendimento_id: AgendamentoID,
    atendimento_dataHora: `${DataMensal} ${Hora}`
}
```

---

## Fluxo de Integração

```text
Feegow API → FeegowClient → FeegowService → Schedules → SQS → Workers → Escallo
```

1. **FeegowClient**: Faz requisições HTTP
2. **FeegowService**: Aplica regras de negócio e filtros
3. **Schedules**: Seleciona pacientes para campanha
4. **SQS**: Enfileira mensagens
5. **Workers**: Processa assincronamente
6. **Escallo**: Dispara campanhas

---

## Dependências

- `axios`: Cliente HTTP
- `date-fns`: Manipulação de datas

---

## ⚠️ Importante

- Token JWT está hardcoded em `client.ts` (considerar usar env vars)
- Todas as respostas são parseadas de JSON string
- Status HTTP < 400 = sucesso
