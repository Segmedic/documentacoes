# API Reference - Sistema de Agendamento Online

## 📋 Informações Gerais

### Base URL

``` cURL
Production: https://tuk21bf0ec.execute-api.sa-east-1.amazonaws.com
Development: http://localhost:4000
```

### Autenticação

A API utiliza integração com a Feegow API através de tokens de acesso configurados internamente. Não é necessário autenticação externa para os endpoints públicos.

### Formato de Resposta Padrão

```json
{
  "success": boolean,
  "content": any
}
```

---

## 🔍 Endpoints

## 📝 Leads

### Criar Lead

```http
POST /lead
```

Cria um novo registro de lead com ID único para rastreamento do processo de agendamento.

**Request Body:** Vazio

**Response:**

```json
{
  "success": true,
  "content": "a1b2c3d4"
}
```

**Status Codes:**

- `200` - Lead criado com sucesso

---

### Listar Leads

```http
GET /lead
```

Retorna todos os leads cadastrados no sistema.

**Response:**

```json
{
  "success": true,
  "content": [
    {
      "CLIENT_ID": "a1b2c3d4",
      "BEGIN_DATE": "2025-11-10T10:30:00.000Z",
      "CLIENT_NAME": "João Silva",
      "CLIENT_EMAIL": "joao@email.com",
      "CLIENT_PHONE": "21999999999"
    }
  ]
}
```

---

### Atualizar Lead

```http
PATCH /lead
```

Atualiza informações de um lead existente.

**Headers:**

``` cURL
id: string (required) - ID do lead
```

**Request Body:**

```json
{
  "CLIENT_NAME": "João Silva",
  "CLIENT_EMAIL": "joao@email.com",
  "CLIENT_PHONE": "21999999999",
  "FEEGOW_CONVENIO": "5",
  "FEEGOW_ESPECIALIDADE": "254",
  "FEEGOW_PROCEDIMENTO": "254",
  "carteirinhaId": "123456789"
}
```

**Response:**

```json
{
  "success": true,
  "content": "Lead a1b2c3d4 updated"
}
```

---

### Deletar Lead

```http
DELETE /lead
```

Remove um lead do sistema.

**Headers:**

``` cURL
id: string (required) - ID do lead
```

**Response:**

```json
{
  "success": true,
  "content": "Lead a1b2c3d4 deleted"
}
```

---

## 👤 Pacientes

### Buscar Paciente

```http
POST /patient
```

Busca informações de um paciente existente pelo CPF.

**Request Body:**

```json
{
  "client_cpf": "12345678900"
}
```

**Response - Sucesso:**

```json
{
  "success": true,
  "content": {
    "paciente_id": 12345,
    "nome_completo": "João Silva",
    "cpf": "12345678900",
    "email": "joao@email.com",
    "telefone": "21999999999",
    "data_nascimento": "1990-01-15"
  }
}
```

**Response - Paciente não encontrado:**

```json
{
  "success": false,
  "content": "Paciente não encontrado"
}
```

---

### Criar Paciente

```http
POST /patient-store
```

Cadastra um novo paciente no sistema Feegow.

**Request Body:**

```json
{
  "clientName": "João Silva",
  "clientCpf": "12345678900",
  "clientEmail": "joao@email.com",
  "clientPhone": "21999999999",
  "clientBirthDate": "1990-01-15T00:00:00.000Z",
  "clientGenre": "M"
}
```

**Validações:**

- `clientName`: obrigatório, string
- `clientCpf`: obrigatório, string com 11 dígitos
- `clientEmail`: obrigatório, formato de email válido
- `clientPhone`: obrigatório, string
- `clientBirthDate`: obrigatório, formato ISO8601
- `clientGenre`: obrigatório, "M" ou "F"

**Response:**

```json
{
  "success": true,
  "content": 12345
}
```

*Retorna o ID do paciente criado*

---

## 🏥 Convênios

### Listar Convênios

```http
GET /insurance
```

Retorna lista de convênios disponíveis para agendamento online.

**Response:**

```json
{
  "success": true,
  "content": [
    {
      "id": 5,
      "nome": "BRADESCO",
      "exibir_agendamento_online": 1
    },
    {
      "id": 10,
      "nome": "UNIMED",
      "exibir_agendamento_online": 1
    }
  ]
}
```

---

## 🩺 Especialidades

### Listar Especialidades

```http
GET /specialities
```

Retorna todas as especialidades médicas disponíveis.

**Response:**

```json
{
  "success": true,
  "content": [
    {
      "especialidade_id": 254,
      "nome": "Cardiologia"
    },
    {
      "especialidade_id": 258,
      "nome": "Clínica Geral"
    }
  ]
}
```

---

## 💊 Procedimentos

### Listar Procedimentos (Estático)

```http
GET /procedures
```

Retorna lista de procedimentos do arquivo JSON estático.

**Response:**

```json
{
  "success": true,
  "content": [
    {
      "procedimento_id": 246,
      "procedimento_name": "Alergologia"
    },
    {
      "procedimento_id": 254,
      "procedimento_name": "Cardiologia",
      "telemedicina_id": 100779
    }
  ]
}
```

---

### Buscar Procedimentos por Convênio

```http
POST /procedures
```

Busca procedimentos disponíveis baseado em convênio e outros filtros.

**Request Body:**

```json
{
  "insurance": "5",
  "clubflex": false,
  "procedimentoId": 254,
  "especialidadeId": 254
}
```

**Response:**

```json
{
  "success": true,
  "content": [
    {
      "procedimento_id": 254,
      "nome": "Consulta Cardiologia",
      "ativo": 1
    }
  ]
}
```

---

### Consultar Preço de Procedimento

```http
POST /preco-procedimento
```

Consulta o preço de um procedimento específico.

**Request Body:**

```json
{
  "procedimentoId": 254,
  "convenioId": 5,
  "unidadeId": 1
}
```

**Response:**

```json
{
  "success": true,
  "content": {
    "valor": 150.00,
    "moeda": "BRL"
  }
}
```

---

## 👨‍⚕️ Profissionais

### Listar Profissionais

```http
GET /professionals
```

Retorna lista de profissionais disponíveis.

**Query Parameters:**

- `especialidadeId`: number (opcional)
- `unidadeId`: number (opcional)
- `convenioId`: number (opcional)

**Exemplo:**

``` cURL
GET /professionals?especialidadeId=254&unidadeId=1
```

**Response:**

```json
{
  "success": true,
  "content": [
    {
      "profissional_id": 100,
      "nome": "Dr. Carlos Oliveira",
      "especialidade": "Cardiologia",
      "crm": "12345-RJ"
    }
  ]
}
```

---

### Listar Profissionais com Limite

```http
GET /professionals/limit
```

Retorna lista limitada de profissionais (útil para exibição inicial).

**Response:**

```json
{
  "success": true,
  "content": [
    {
      "profissional_id": 100,
      "nome": "Dr. Carlos Oliveira"
    }
  ]
}
```

---

## 🏢 Unidades

### Listar Unidades por Convênio

```http
GET /units
```

Retorna unidades disponíveis para um convênio e plano específicos.

**Query Parameters:**

- `convenio`: string (required)
- `plano`: string (required)

**Exemplo:**

``` cURL
GET /units?convenio=5&plano=10
```

**Response:**

```json
{
  "units": [
    {
      "unidade_id": 1,
      "nome": "Unidade Copacabana",
      "endereco": "Rua Example, 123"
    }
  ]
}
```

---

## 📅 Agendamentos

### Buscar Horários Disponíveis

```http
POST /schedule
```

Busca horários disponíveis para agendamento baseado nos filtros fornecidos.

**Request Body:**

```json
{
  "startDate": "2025-11-15",
  "endDate": "2025-11-30",
  "procedimentoId": 254,
  "especialidadeId": 254,
  "profissionalId": 100,
  "unidadeId": 1,
  "convenioId": 5,
  "planoId": 10
}
```

**Validações:**

- `startDate`: obrigatório, formato YYYY-MM-DD
- `endDate`: obrigatório, formato YYYY-MM-DD

**Response:**

```json
{
  "success": true,
  "content": [
    {
      "data": "2025-11-15",
      "horario": "09:00",
      "profissional_id": 100,
      "profissional_nome": "Dr. Carlos Oliveira",
      "unidade_id": 1,
      "unidade_nome": "Unidade Copacabana"
    }
  ],
  "total": 15
}
```

---

### Obter Detalhes de Agendamento

```http
GET /schedule
```

Busca informações de agendamentos existentes.

**Query Parameters:**

- `pacienteId`: number (opcional)
- `agendamentoId`: number (opcional)

**Response:**

```json
{
  "success": true,
  "content": {
    "agendamento_id": 12345,
    "data": "2025-11-15",
    "horario": "09:00",
    "status": "Confirmado"
  }
}
```

---

### Criar Agendamento

```http
POST /schedule-store
```

Cria um novo agendamento no sistema.

**Request Body:**

```json
{
  "pacienteId": 12345,
  "profissionalId": 100,
  "unidadeId": 1,
  "procedimentoId": 254,
  "data": "2025-11-15",
  "horario": "09:00",
  "convenioId": 5,
  "planoId": 10,
  "observacoes": "Primeira consulta"
}
```

**Validações:**

- `pacienteId`: obrigatório, number
- `profissionalId`: obrigatório, number
- `unidadeId`: obrigatório, number
- `data`: obrigatório, formato YYYY-MM-DD
- `horario`: obrigatório, formato HH:mm

**Response:**

```json
{
  "success": true,
  "content": {
    "agendamento_id": 12345,
    "status": "Confirmado"
  }
}
```

---

### Atualizar Agendamento

```http
PUT /schedule/:id
```

Atualiza informações de um agendamento existente.

**URL Parameters:**

- `id`: number - ID do agendamento

**Request Body:**

```json
{
  "data": "2025-11-16",
  "horario": "10:00",
  "observacoes": "Remarcado a pedido do paciente"
}
```

**Response:**

```json
{
  "success": true,
  "content": "Agendamento atualizado com sucesso"
}
```

---

### Cancelar Agendamento

```http
DELETE /schedule/:id
```

Cancela um agendamento existente.

**URL Parameters:**

- `id`: number - ID do agendamento

**Request Body:**

```json
{
  "motivoCancelamento": 1,
  "observacoes": "Paciente cancelou"
}
```

**Motivos de Cancelamento:**

- `1` - Cancelado pelo paciente
- `2` - Cancelado pelo profissional
- `3` - Cancelado pela clínica

**Response:**

```json
{
  "success": true,
  "content": "Agendamento cancelado com sucesso"
}
```

---

## 📧 E-mails

### Enviar Email de Consultas

```http
POST /sendMailConsultas
```

Envia email relacionado a consultas agendadas.

**Request Body:**

```json
{
  "to": "paciente@email.com",
  "agendamentoId": 12345,
  "tipo": "confirmacao"
}
```

**Response:**

```json
{
  "success": true,
  "content": "Email enviado com sucesso"
}
```

---

### Enviar Email de Exames

```http
POST /sendMailExames
```

Envia email relacionado a exames.

**Request Body:**

```json
{
  "to": "paciente@email.com",
  "exameId": 12345,
  "tipo": "resultado"
}
```

**Response:**

```json
{
  "success": true,
  "content": "Email enviado com sucesso"
}
```

---

### Enviar Pesquisa NPS

```http
POST /pesquisa
```

Envia pesquisa de satisfação NPS para o paciente.

**Request Body:**

```json
{
  "email": "paciente@email.com",
  "nome": "João Silva",
  "agendamentoId": 12345
}
```

**Response:**

```json
{
  "success": true,
  "content": "Pesquisa enviada com sucesso"
}
```

---

## 🏃 Clubflex

### Verificar Paciente Clubflex

```http
GET /clubflex/:cpf
```

Verifica se um CPF está cadastrado no programa Clubflex.

**URL Parameters:**

- `cpf`: string - CPF do paciente (apenas números)

**Exemplo:**

``` cURL
GET /clubflex/12345678900
```

**Response - Cadastrado:**

```json
{
  "success": true,
  "content": {
    "ativo": true,
    "nome": "João Silva",
    "plano": "Premium"
  }
}
```

**Response - Não cadastrado:**

```json
{
  "success": false,
  "content": "CPF não encontrado no Clubflex"
}
```

---

## 📎 Upload de Arquivos

### Upload de Pedido Médico

```http
POST /pedido-medico
```

Faz upload de arquivo de pedido médico.

**Content-Type:** `multipart/form-data`

**Form Data:**

- `file`: File (required) - Arquivo do pedido médico
- `pacienteId`: number (required)
- `agendamentoId`: number (optional)

**Response:**

```json
{
  "success": true,
  "content": {
    "fileUrl": "https://s3.amazonaws.com/bucket/pedido-12345.pdf",
    "fileId": "abc123"
  }
}
```

---

## 🏥 Nuria - Elegibilidade (Ainda não funciona)

### Verificar Elegibilidade

```http
POST /elegible
```

Verifica elegibilidade de um paciente através da integração com Nuria.

**Request Body:**

```json
{
  "cpf": "12345678900",
  "convenioId": 5,
  "carteirinha": "123456789",
  "dataNascimento": "1990-01-15"
}
```

**Response - Elegível:**

```json
{
  "success": true,
  "content": {
    "elegivel": true,
    "plano": "Enfermaria",
    "titular": "João Silva"
  }
}
```

**Response - Não elegível:**

```json
{
  "success": false,
  "content": {
    "elegivel": false,
    "motivo": "Carteirinha inválida"
  }
}
```

---

## 📖 Documentação Interativa

### Swagger UI

```http
GET /docs
```

Acessa a documentação interativa da API através do Swagger UI.

### Swagger JSON

```http
GET /swagger.json
```

Retorna a especificação OpenAPI em formato JSON.

---

## ❌ Códigos de Erro

### Erros Comuns

#### 400 - Bad Request

```json
{
  "success": false,
  "content": "Dados inválidos",
  "errors": [
    {
      "field": "clientEmail",
      "message": "Email inválido"
    }
  ]
}
```

**404 - Not Found**

```json
{
  "success": false,
  "content": "Recurso não encontrado"
}
```

**500 - Internal Server Error**

```json
{
  "success": false,
  "content": "Erro interno do servidor",
  "details": "Detalhes do erro"
}
```

---

## 📝 Notas Importantes

1. **Validação de Dados**: Todos os endpoints que recebem dados validam através de schemas Zod
2. **Integração Feegow**: A maioria dos endpoints se comunica com a API Feegow
3. **Formato de Datas**: Utilizar formato ISO8601 para datas
4. **IDs**: Todos os IDs são numéricos, exceto o CLIENT_ID de leads que é hexadecimal
5. **CPF**: Deve ser enviado apenas com números, sem formatação

---

## 🔄 Changelog

### v1.0.0 (2025-11-10)

- Documentação inicial da API
- Todos os endpoints principais documentados
