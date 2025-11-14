# Referência Rápida de Rotas

## Índice Visual de Endpoints

### 🔐 Autenticação

| Método | Rota | Descrição | Auth |
|--------|------|-----------|------|
| POST | `/api/sign_in` | Login e geração de token JWT | ❌ |

---

### 📋 Agendamentos

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/api/appointments` | Listar agendamentos do dia | ✅ | `paciente_id` |
| GET | `/api/appointments/:id` | Detalhes do agendamento | ✅ | - |
| POST | `/api/appointments` | Criar agendamento | ✅ | `local_id`, `paciente_id`, `profissional_id`, `data`, `horario` |
| DELETE | `/api/appointments/:id` | Cancelar agendamento | ✅ | `reason_id` |
| PATCH | `/api/appointments/update_status` | Atualizar status (check-in) | ✅ | `agendamento_id`, `status_id` |
| GET | `/api/appointments/available` | Horários disponíveis | ✅ | `profissional_id`, `especialidade_id` |
| GET | `/api/appointments/price_tables` | Tabelas de preço | ✅ | - |
| GET | `/api/appointments/reports` | Relatórios | ✅ | - |
| GET | `/api/appointments/invoices` | Notas fiscais | ✅ | `agendamento_id` |
| GET | `/api/appointments/coupon` | Gerar cupom PDF | ✅ | `ids`, `payment_form` |
| POST | `/api/appointments/create_password` | Gerar senha de atendimento | ✅ | `unidade_id`, `situacao` |
| GET | `/api/appointments/check_worklist` | Verificar worklist | ✅ | `especialidade_id` |

---

### 👥 Pacientes

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/api/patients` | Listar pacientes | ✅ | `offset`, `limit` |
| GET | `/api/patients/:id` | Buscar por ID | ✅ | - |
| GET | `/api/patients/search` | Buscar por CPF | ✅ | `cpf` |
| POST | `/api/patients` | Criar/atualizar paciente | ✅ | `patient[nome_completo]`, `patient[cpf]` |

---

### 👨‍⚕️ Profissionais

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/api/professionals` | Listar profissionais | ✅ | `especialidade_id`, `unidade_id` |
| GET | `/api/professionals/:id` | Detalhes do profissional | ✅ | - |
| GET | `/api/professionals/availability` | Verificar disponibilidade | ✅ | `profissional_id`, `data` |

---

### 🏥 Especialidades

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/api/specialties` | Listar especialidades | ✅ | - |
| GET | `/api/specialties/available` | Especialidades com horários | ✅ | `unidade_id`, `data` |

---

### 🏢 Unidades

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/api/units` | Listar unidades | ✅ | - |

---

### 📍 Locais

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/api/locals` | Listar consultórios | ✅ | - |

---

### 🔬 Procedimentos

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/api/procedures` | Listar procedimentos | ✅ | `especialidade_id` |
| GET | `/api/procedures/available` | Procedimentos disponíveis | ✅ | `especialidade_id`, `profissional_id` |

---

### 🏥 Convênios

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/api/insurances` | Listar convênios | ✅ | - |

---

### 📊 Tabelas de Preço

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/api/tables` | Listar tabelas | ✅ | - |
| GET | `/api/tables/available` | Tabelas disponíveis | ✅ | `convenio_id` |

---

### 💰 Financeiro

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| POST | `/api/financials` | Criar conta financeira | ✅ | `account[agendamento_id]` |
| POST | `/api/financials/create_invoice` | Gerar nota fiscal | ✅ | `agendamento_id` |
| POST | `/api/financials/pay` | Processar pagamento | ✅ | `payment[invoiceId]`, `payment[paymentMethod]` |
| POST | `/api/financials/create_pix` | Gerar QR Code PIX | ✅ | `value` |
| GET | `/api/financials/pix` | Consultar status PIX | ✅ | `txid` |
| GET | `/api/financials/payment_log` | Buscar log de pagamento | ✅ | `invoice_id` |

---

### 💳 ClubFlex

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/api/clubflex/check` | Verificar elegibilidade | ✅ | `cpf` |
| GET | `/api/clubflex/status` | Status do assinante | ✅ | `cpf` |

---

### 📱 Sessões de Usuário

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| POST | `/api/user_session_screens` | Registrar navegação | ✅ | `user_session_id`, `screen_name` |
| POST | `/api/user_session_actions` | Registrar ação | ✅ | `user_session_id`, `action` |

---

### 🔧 Utilidades

| Método | Rota | Descrição | Auth | Parâmetros Principais |
|--------|------|-----------|------|-----------------------|
| GET | `/static/healthcheck` | Verificar saúde da API | ❌ | - |
| GET | `/static/terms_of_services` | Termos de serviço | ❌ | - |
| GET | `/static/privacy_policies` | Políticas de privacidade | ❌ | - |

---

## Legenda

- ✅ = Autenticação obrigatória (Bearer token)
- ❌ = Endpoint público (sem autenticação)

---

## Grupos Funcionais

### 📋 Gestão de Agendamentos
```
/api/appointments              (CRUD completo)
/api/appointments/available    (Consulta de disponibilidade)
/api/appointments/update_status (Check-in)
/api/appointments/create_password (Senha)
```

### 👥 Gestão de Pacientes
```
/api/patients                  (CRUD)
/api/patients/search          (Busca por CPF)
```

### 💰 Financeiro
```
/api/financials               (Contas)
/api/financials/create_invoice (NF)
/api/financials/pay           (Pagamento)
/api/financials/create_pix    (PIX)
/api/financials/pix           (Status PIX)
```

### 🔍 Consultas
```
/api/professionals            (Médicos)
/api/specialties              (Especialidades)
/api/procedures               (Procedimentos)
/api/units                    (Unidades)
/api/locals                   (Consultórios)
/api/insurances               (Convênios)
/api/tables                   (Tabelas de preço)
```

### 💳 Convênio
```
/api/clubflex/check          (Elegibilidade)
/api/clubflex/status         (Status)
```

---

## Fluxos Comuns

### ➡️ Criar Agendamento Completo

```
1. GET  /api/patients/search?cpf=123
2. GET  /api/clubflex/check?cpf=123
3. GET  /api/specialties
4. GET  /api/professionals?especialidade_id=12
5. GET  /api/appointments/available
6. POST /api/appointments
7. POST /api/financials/create_invoice
8. POST /api/financials/create_pix
9. GET  /api/financials/pix?txid=...  (polling)
10. POST /api/financials/pay
11. POST /api/appointments/create_password
```

### ➡️ Check-in do Paciente

```
1. GET   /api/patients/search?cpf=123
2. GET   /api/appointments?paciente_id=678
3. PATCH /api/appointments/update_status
4. POST  /api/appointments/create_password
```

### ➡️ Cancelamento

```
1. GET    /api/patients/search?cpf=123
2. GET    /api/appointments?paciente_id=678
3. DELETE /api/appointments/123?reason_id=5
```

---

## Formatos de Data/Hora

| Campo | Formato | Exemplo |
|-------|---------|---------|
| `data` (Feegow) | DD-MM-YYYY | `15-11-2025` |
| `horario` | HH:MM | `14:30` |
| `data_nascimento` | DD-MM-YYYY | `15-05-1980` |
| Timestamps Rails | ISO8601 | `2025-11-15T14:30:00Z` |

---

## Códigos de Status Comuns

| Código | Significado | Quando Ocorre |
|--------|-------------|---------------|
| 200 | OK | Requisição bem-sucedida |
| 201 | Created | Recurso criado (PIX, registro) |
| 400 | Bad Request | Parâmetros inválidos |
| 401 | Unauthorized | Token ausente ou inválido |
| 404 | Not Found | Recurso não encontrado |
| 422 | Unprocessable Entity | Validação falhou |
| 500 | Internal Server Error | Erro no servidor |

---

## Rate Limiting

| Endpoint | Limite |
|----------|--------|
| Todos | 100 req/min por IP |

---

## Headers Padrão

### Request
```
Content-Type: application/json
Authorization: Bearer {seu_token_jwt}
```

### Response
```
Content-Type: application/json
X-RateLimit-Remaining: 95
```

---

## Paginação

Endpoints que suportam paginação:

| Endpoint | Parâmetros |
|----------|------------|
| `/api/patients` | `offset` (default: 0), `limit` (default: 10) |

---

## Filtros Comuns

| Parâmetro | Tipo | Endpoints que usam |
|-----------|------|-------------------|
| `paciente_id` | integer | appointments |
| `especialidade_id` | integer | professionals, procedures, specialties |
| `profissional_id` | integer | appointments/available |
| `unidade_id` | integer | specialties/available |
| `cpf` | string | patients/search, clubflex/check |
| `data` | string | appointments/available |

---

## Valores Especiais

### Status de Agendamento
| ID | Significado |
|----|-------------|
| 1 | Agendado |
| 2 | Chegou (Check-in) |
| 3 | Em Atendimento |
| 4 | Finalizado |
| 5 | Cancelado |

### Tipos de Senha (Feegow)
| ID | Situação |
|----|----------|
| 102 | Padrão |
| 19 | Clubflex |
| 103 | Exames |
| 107 | Orçamentos |
| 108 | Pagamento Particular |
| 109 | Convênio |
| 112 | Revisão |

### Formas de Pagamento
- `pix`
- `credit_card`
- `debit_card`
- `dinheiro` (cash)

---

Esta referência rápida foi gerada em: **14 de novembro de 2025**
