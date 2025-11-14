# Documentação Técnica da API - Segmedic Totem

## Sumário
1. [Visão Geral](#visão-geral)
2. [Autenticação](#autenticação)
3. [Endpoints](#endpoints)
4. [Códigos de Status](#códigos-de-status)
5. [Variáveis de Ambiente](#variáveis-de-ambiente)

---

## Visão Geral

API RESTful desenvolvida em Ruby on Rails para gerenciamento de agendamentos médicos através de totems de autoatendimento.

**Base URL:** `https://api.segmedic.com.br` (Produção)  
**Base URL:** `http://localhost:3000` (Desenvolvimento)

**Formato de Resposta:** JSON  
**Encoding:** UTF-8

---

## Autenticação

### POST /api/sign_in
Autentica um usuário e retorna um token JWT.

**Request Body:**
```json
{
  "user": {
    "email": "usuario@exemplo.com",
    "password": "senha123"
  }
}
```

**Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "user": {
    "id": 1,
    "email": "usuario@exemplo.com",
    "created_at": "2024-01-01T10:00:00.000Z"
  }
}
```

**Headers para requisições autenticadas:**
```
Authorization: Bearer {token}
Content-Type: application/json
```

---

## Endpoints

### 📋 Agendamentos (Appointments)

#### GET /api/appointments
Lista agendamentos do dia atual para um paciente.

**Query Parameters:**
- `paciente_id` (integer, obrigatório): ID do paciente

**Response (200 OK):**
```json
{
  "content": [
    {
      "agendamento_id": 12345,
      "paciente_nome": "João Silva",
      "profissional_nome": "Dr. Maria Santos",
      "especialidade_nome": "Cardiologia",
      "data": "15-11-2025",
      "horario": "14:30",
      "status": "Agendado",
      "unidade_nome": "Clínica Centro"
    }
  ]
}
```

---

#### GET /api/appointments/:id
Busca detalhes de um agendamento específico.

**URL Parameters:**
- `id` (integer): ID do agendamento

**Response (200 OK):**
```json
{
  "content": {
    "agendamento_id": 12345,
    "paciente_id": 678,
    "paciente_nome": "João Silva",
    "paciente_cpf": "123.456.789-00",
    "profissional_id": 45,
    "profissional_nome": "Dr. Maria Santos",
    "especialidade_id": 12,
    "especialidade_nome": "Cardiologia",
    "procedimento_id": 89,
    "procedimento_nome": "Consulta",
    "data": "15-11-2025",
    "horario": "14:30",
    "valor": "150.00",
    "status": "Agendado",
    "unidade_id": 3,
    "unidade_nome": "Clínica Centro",
    "unidade": {
      "endereco": "Rua Exemplo, 123",
      "telefone": "(11) 1234-5678"
    }
  }
}
```

---

#### POST /api/appointments
Cria um novo agendamento.

**Request Body:**
```json
{
  "local_id": 1,
  "paciente_id": 678,
  "profissional_id": 45,
  "especialidade_id": 12,
  "procedimento_id": 89,
  "data": "15-11-2025",
  "horario": "14:30",
  "valor": "150.00",
  "plano": "Particular",
  "convenio_id": null,
  "convenio_plano_id": null,
  "tabela_id": 1
}
```

**Response (200 OK):**
```json
{
  "content": {
    "agendamento_id": 12345,
    "message": "Agendamento criado com sucesso"
  }
}
```

---

#### PATCH /api/appointments/update_status
Atualiza o status de um agendamento (ex: paciente chegou).

**Request Body:**
```json
{
  "agendamento_id": 12345,
  "status_id": 2
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Status atualizado com sucesso"
}
```

---

#### DELETE /api/appointments/:id
Cancela um agendamento.

**URL Parameters:**
- `id` (integer): ID do agendamento

**Query Parameters:**
- `reason_id` (integer, obrigatório): ID do motivo de cancelamento

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Agendamento cancelado com sucesso"
}
```

---

#### GET /api/appointments/available
Lista horários disponíveis para agendamento.

**Query Parameters:**
- `profissional_id` (integer): ID do profissional
- `especialidade_id` (integer): ID da especialidade
- `procedimento_id` (integer): ID do procedimento
- `data` (string): Data no formato DD-MM-YYYY (default: hoje)

**Response (200 OK):**
```json
{
  "content": [
    {
      "data": "15-11-2025",
      "horarios": ["08:00", "08:30", "09:00", "09:30", "10:00"]
    }
  ]
}
```

---

#### GET /api/appointments/invoices
Lista notas fiscais de um agendamento.

**Query Parameters:**
- `agendamento_id` (integer, obrigatório): ID do agendamento

**Response (200 OK):**
```json
{
  "content": [
    {
      "invoice_id": "NF-12345",
      "valor": "150.00",
      "status": "Pago",
      "data_emissao": "15-11-2025"
    }
  ]
}
```

---

#### POST /api/appointments/create_password
Gera senha de atendimento para o paciente.

**Request Body:**
```json
{
  "unidade_id": 3,
  "unidade_nome": "Clínica Centro",
  "paciente_nome": "João Silva",
  "situacao": "Chegada"
}
```

**Response (200 OK):**
```json
{
  "url": "https://storage.example.com/senha-A123.pdf"
}
```

---

#### GET /api/appointments/coupon
Gera cupom de pagamento em PDF.

**Query Parameters:**
- `ids` (string): IDs dos agendamentos separados por vírgula
- `payment_form` (string): Forma de pagamento
- `last_digits` (string, opcional): Últimos dígitos do cartão
- `nsu` (string, opcional): NSU da transação

**Response (200 OK):**
```json
{
  "url": "https://storage.example.com/cupom-12345.pdf"
}
```

---

### 👥 Pacientes (Patients)

#### GET /api/patients
Lista pacientes com paginação.

**Query Parameters:**
- `offset` (integer): Início da paginação (default: 0)
- `limit` (integer): Limite de registros (default: 10)

**Response (200 OK):**
```json
{
  "content": [
    {
      "paciente_id": 678,
      "nome_completo": "João Silva",
      "cpf": "123.456.789-00",
      "data_nascimento": "15-05-1980",
      "telefone": "(11) 98765-4321",
      "email": "joao@exemplo.com"
    }
  ],
  "total": 150
}
```

---

#### GET /api/patients/search
Busca paciente por CPF.

**Query Parameters:**
- `cpf` (string, obrigatório): CPF do paciente (apenas números)

**Response (200 OK):**
```json
{
  "content": {
    "paciente_id": 678,
    "nome_completo": "João Silva",
    "cpf": "123.456.789-00",
    "data_nascimento": "15-05-1980",
    "genero": "M",
    "telefone": "(11) 98765-4321",
    "email": "joao@exemplo.com"
  }
}
```

**Response (404 Not Found):**
```json
{
  "message": "Paciente não encontrado"
}
```

---

#### GET /api/patients/:id
Busca paciente por ID.

**URL Parameters:**
- `id` (integer): ID do paciente

**Response (200 OK):**
```json
{
  "content": {
    "paciente_id": 678,
    "nome_completo": "João Silva",
    "cpf": "123.456.789-00",
    "data_nascimento": "15-05-1980",
    "genero": "M",
    "telefone": "(11) 98765-4321",
    "email": "joao@exemplo.com",
    "endereco": {
      "rua": "Rua Exemplo",
      "numero": "123",
      "cidade": "São Paulo",
      "estado": "SP",
      "cep": "01234-567"
    }
  }
}
```

---

#### POST /api/patients
Cria ou atualiza um paciente.

**Request Body:**
```json
{
  "patient": {
    "nome_completo": "João Silva",
    "cpf": "12345678900",
    "data_nascimento": "15-05-1980",
    "genero": "M",
    "email": "joao@exemplo.com",
    "telefone": "11987654321",
    "tabela_id": 1,
    "convenio_id": null,
    "plano_id": null,
    "matricula": null,
    "validade": null
  }
}
```

**Response (200 OK):**
```json
{
  "content": {
    "paciente_id": 678,
    "nome_completo": "João Silva",
    "message": "Paciente criado/atualizado com sucesso"
  }
}
```

---

### 👨‍⚕️ Profissionais (Professionals)

#### GET /api/professionals
Lista profissionais disponíveis.

**Query Parameters:**
- `especialidade_id` (integer, opcional): Filtrar por especialidade
- `unidade_id` (integer, opcional): Filtrar por unidade

**Response (200 OK):**
```json
{
  "content": [
    {
      "profissional_id": 45,
      "nome": "Dr. Maria Santos",
      "crm": "123456",
      "especialidade": "Cardiologia",
      "especialidade_id": 12
    }
  ]
}
```

---

#### GET /api/professionals/:id
Busca detalhes de um profissional.

**URL Parameters:**
- `id` (integer): ID do profissional

**Response (200 OK):**
```json
{
  "content": {
    "profissional_id": 45,
    "nome": "Dr. Maria Santos",
    "crm": "123456",
    "especialidades": [
      {
        "especialidade_id": 12,
        "nome": "Cardiologia"
      }
    ],
    "unidades": [
      {
        "unidade_id": 3,
        "nome": "Clínica Centro"
      }
    ]
  }
}
```

---

#### GET /api/professionals/availability
Verifica disponibilidade de um profissional.

**Query Parameters:**
- `profissional_id` (integer, obrigatório): ID do profissional
- `data` (string): Data no formato DD-MM-YYYY

**Response (200 OK):**
```json
{
  "content": {
    "disponivel": true,
    "horarios": ["08:00", "08:30", "09:00", "09:30", "10:00"]
  }
}
```

---

### 🏥 Especialidades (Specialties)

#### GET /api/specialties
Lista especialidades disponíveis.

**Response (200 OK):**
```json
{
  "content": [
    {
      "especialidade_id": 12,
      "nome": "Cardiologia",
      "ativo": true
    },
    {
      "especialidade_id": 15,
      "nome": "Ortopedia",
      "ativo": true
    }
  ]
}
```

---

#### GET /api/specialties/available
Lista especialidades com horários disponíveis.

**Query Parameters:**
- `unidade_id` (integer, opcional): Filtrar por unidade
- `data` (string, opcional): Data no formato DD-MM-YYYY

**Response (200 OK):**
```json
{
  "content": [
    {
      "especialidade_id": 12,
      "nome": "Cardiologia",
      "tem_horarios": true,
      "profissionais_disponiveis": 3
    }
  ]
}
```

---

### 🏢 Unidades (Units)

#### GET /api/units
Lista unidades de atendimento.

**Response (200 OK):**
```json
{
  "content": [
    {
      "unidade_id": 3,
      "nome_fantasia": "Clínica Centro",
      "endereco": "Rua Exemplo, 123",
      "telefone": "(11) 1234-5678",
      "horario_funcionamento": "08:00 - 18:00",
      "ativo": true
    }
  ]
}
```

---

### 📍 Locais (Locals)

#### GET /api/locals
Lista locais de atendimento (consultórios).

**Response (200 OK):**
```json
{
  "content": [
    {
      "local_id": 1,
      "nome": "Consultório 1",
      "unidade_id": 3,
      "ativo": true
    }
  ]
}
```

---

### 🔬 Procedimentos (Procedures)

#### GET /api/procedures
Lista procedimentos disponíveis.

**Query Parameters:**
- `especialidade_id` (integer, opcional): Filtrar por especialidade

**Response (200 OK):**
```json
{
  "content": [
    {
      "procedimento_id": 89,
      "nome": "Consulta em Cardiologia",
      "especialidade_id": 12,
      "valor": "150.00"
    }
  ]
}
```

---

#### GET /api/procedures/available
Lista procedimentos com disponibilidade.

**Query Parameters:**
- `especialidade_id` (integer, obrigatório): ID da especialidade
- `profissional_id` (integer, opcional): ID do profissional

**Response (200 OK):**
```json
{
  "content": [
    {
      "procedimento_id": 89,
      "nome": "Consulta em Cardiologia",
      "disponivel": true,
      "valor": "150.00"
    }
  ]
}
```

---

### 🏥 Convênios (Insurances)

#### GET /api/insurances
Lista convênios aceitos.

**Response (200 OK):**
```json
{
  "content": [
    {
      "convenio_id": 5,
      "nome": "Unimed",
      "ativo": true,
      "planos": [
        {
          "plano_id": 12,
          "nome": "Unimed Basic"
        }
      ]
    }
  ]
}
```

---

### 📊 Tabelas de Preço (Tables)

#### GET /api/tables
Lista tabelas de preço.

**Response (200 OK):**
```json
{
  "content": [
    {
      "tabela_id": 1,
      "nome": "Particular",
      "ativo": true
    },
    {
      "tabela_id": 2,
      "nome": "Convênio",
      "ativo": true
    }
  ]
}
```

---

#### GET /api/tables/available
Lista tabelas de preço disponíveis para determinado contexto.

**Query Parameters:**
- `convenio_id` (integer, opcional): ID do convênio

**Response (200 OK):**
```json
{
  "content": [
    {
      "tabela_id": 1,
      "nome": "Particular",
      "disponivel": true
    }
  ]
}
```

---

### 💰 Financeiro (Financials)

#### POST /api/financials
Cria uma conta financeira para o agendamento.

**Request Body:**
```json
{
  "account": {
    "agendamento_id": 12345
  },
  "paciente_id": 678
}
```

**Response (200 OK):**
```json
{
  "content": {
    "account_id": "ACC-789",
    "valor": "150.00",
    "status": "Pendente"
  }
}
```

---

#### POST /api/financials/create_invoice
Gera nota fiscal para o agendamento.

**Request Body:**
```json
{
  "agendamento_id": 12345
}
```

**Response (200 OK):**
```json
{
  "content": {
    "invoice_id": "NF-12345",
    "valor": "150.00",
    "status": "Gerada",
    "data_emissao": "15-11-2025"
  }
}
```

---

#### POST /api/financials/pay
Processa pagamento de uma nota fiscal.

**Request Body:**
```json
{
  "payment": {
    "invoiceId": "NF-12345",
    "paymentName": "Cartão de Crédito",
    "paymentMethod": "credit_card",
    "creditCardTransaction": {
      "transactionNumber": "123456789",
      "authorizationNumber": "ABC123",
      "flagCardId": 1,
      "installments": 1
    }
  }
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Pagamento realizado com sucesso",
  "transaction_id": "TRX-789"
}
```

---

#### POST /api/financials/create_pix
Gera QR Code PIX para pagamento.

**Request Body:**
```json
{
  "value": "150.00"
}
```

**Response (201 Created):**
```json
{
  "txid": "PIX-123456789",
  "qrcode": "00020126580014br.gov.bcb.pix...",
  "qrcode_image": "data:image/png;base64,iVBORw0KGg...",
  "valor": "150.00",
  "expiracao": "2025-11-15T15:30:00Z"
}
```

---

#### GET /api/financials/pix
Consulta status de pagamento PIX.

**Query Parameters:**
- `txid` (string, obrigatório): ID da transação PIX

**Response (200 OK):**
```json
{
  "txid": "PIX-123456789",
  "status": "CONCLUIDA",
  "valor": "150.00",
  "horario": "2025-11-15T14:35:00Z"
}
```

**Possíveis status:**
- `ATIVA`: Aguardando pagamento
- `CONCLUIDA`: Pago
- `REMOVIDA_PELO_USUARIO_RECEBEDOR`: Cancelada
- `REMOVIDA_PELO_PSP`: Expirada

---

#### GET /api/financials/payment_log
Busca log de pagamento.

**Query Parameters:**
- `invoice_id` (string, obrigatório): ID da nota fiscal

**Response (200 OK):**
```json
{
  "id": 123,
  "invoice_id": "NF-12345",
  "agendamento_id": 12345,
  "kind": "success",
  "payment_method": "credit_card",
  "valor": "150.00",
  "created_at": "2025-11-15T14:30:00Z"
}
```

---

### 💳 ClubFlex

#### GET /api/clubflex/check
Verifica elegibilidade do paciente no ClubFlex.

**Query Parameters:**
- `cpf` (string, obrigatório): CPF do paciente (apenas números)

**Response (200 OK):**
```json
{
  "status": "ok",
  "cpf": "12345678900",
  "type_sub": "PF",
  "name": "João Silva"
}
```

**Possíveis status:**
- `ok`: Elegível
- `inadimplente`: Bloqueado por inadimplência
- `cancelado`: Cancelado
- `nao_encontrado`: Não é assinante

---

#### GET /api/clubflex/status
Busca status completo do assinante.

**Query Parameters:**
- `cpf` (string, obrigatório): CPF do paciente

**Response (200 OK):**
```json
{
  "holder": {
    "name": "João Silva",
    "cpf": "12345678900",
    "status": "ATIVO",
    "plano": "ClubFlex Basic"
  }
}
```

---

### 📱 Sessões de Usuário (User Sessions)

#### POST /api/user_session_screens
Registra navegação do usuário nas telas do totem.

**Request Body:**
```json
{
  "user_session_id": 456,
  "screen_name": "agendamento",
  "action": "view"
}
```

**Response (201 Created):**
```json
{
  "id": 789,
  "user_session_id": 456,
  "screen_name": "agendamento",
  "action": "view",
  "created_at": "2025-11-15T14:30:00Z"
}
```

---

#### POST /api/user_session_actions
Registra ações do usuário no totem.

**Request Body:**
```json
{
  "user_session_id": 456,
  "action": "confirm_appointment",
  "result": "success"
}
```

**Response (201 Created):**
```json
{
  "id": 790,
  "user_session_id": 456,
  "action": "confirm_appointment",
  "result": "success",
  "created_at": "2025-11-15T14:30:00Z"
}
```

---

### 🔧 Utilidades (Statics)

#### GET /static/healthcheck
Verifica saúde da API.

**Response (200 OK):**
```json
{
  "status": "ok",
  "timestamp": "2025-11-15T14:30:00Z",
  "version": "1.0.0"
}
```

---

#### GET /static/terms_of_services
Retorna termos de serviço.

**Response (200 OK):**
```json
{
  "content": "Termos de serviço...",
  "version": "1.0",
  "updated_at": "2025-01-01"
}
```

---

#### GET /static/privacy_policies
Retorna políticas de privacidade.

**Response (200 OK):**
```json
{
  "content": "Políticas de privacidade...",
  "version": "1.0",
  "updated_at": "2025-01-01"
}
```

---

## Códigos de Status

| Código | Descrição |
|--------|-----------|
| 200 | OK - Requisição bem-sucedida |
| 201 | Created - Recurso criado com sucesso |
| 400 | Bad Request - Parâmetros inválidos |
| 401 | Unauthorized - Token inválido ou ausente |
| 404 | Not Found - Recurso não encontrado |
| 422 | Unprocessable Entity - Validação falhou |
| 500 | Internal Server Error - Erro no servidor |

---

## Variáveis de Ambiente

### Feegow (Sistema de Gestão)
```
FEEGOW_API_ENDPOINT=https://api.feegow.com.br
FEEGOW_API_TOKEN=seu_token_aqui
```

### ClubFlex (Plano de Saúde)
```
CLUBFLEX_API_ENDPOINT=https://api.clubflex.com.br
CLUBFLEX_API_LOGIN=seu_login
CLUBFLEX_API_PASSWORD=sua_senha
```

### Itaú (Pagamento PIX)
```
MS_PIX_URL=https://pix.itau.com.br
MS_PIX_TOKEN=seu_token_pix
```

### Banco de Dados
```
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
```

### Redis (Cache/Sidekiq)
```
REDIS_URL=redis://localhost:6379/0
```

---

## Logs e Auditoria

A API registra todas as chamadas para serviços externos na tabela `service_logs`:

```ruby
ServiceLog.create(
  origin: "user_session_id",      # Origem da requisição
  provider: :feegow,               # Provedor (feegow, clubflex, itau)
  params: { ... },                 # Parâmetros enviados
  response: { ... },               # Resposta recebida
  status: 200,                     # Status HTTP
  method: "post",                  # Método HTTP
  url_path: "full_url",           # URL completa
  requested_at: DateTime.now      # Timestamp
)
```

---

## Rate Limiting

A API implementa rate limiting baseado em IP:
- **Limite**: 100 requisições por minuto por IP
- **Header de resposta**: `X-RateLimit-Remaining`

---

## Versionamento

A API não utiliza versionamento na URL. Mudanças breaking changes serão comunicadas com antecedência.

---

## Suporte

Para questões técnicas, contate: dev@segmedic.com.br
