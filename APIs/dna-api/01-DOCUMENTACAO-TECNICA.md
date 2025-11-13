# Documentação Técnica da API - ClubFlex

## Informações Gerais

**Versão:** 0.0.1-SNAPSHOT  
**Framework:** Spring Boot 2.6.7  
**Java Version:** 17  
**Packaging:** WAR  
**Base URL:** `/api` (ou conforme configuração do servidor)

---

## Autenticação

A API utiliza autenticação baseada em **JWT (JSON Web Token)**. Todos os endpoints protegidos requerem um token válido no header da requisição.

### Header de Autenticação

```
Authorization: Bearer {token}
```

### Perfis de Usuário

- `HOLDER` - Titular de assinatura
- `ATTENDANT` - Atendente
- `BROKER` - Corretor
- `SUPERVISOR` - Supervisor
- `MANAGER` - Gerente
- `ADMIN` - Administrador

---

## Endpoints

### 🔐 Autenticação e Usuários

#### **POST** `/user/remember/passwd`

Solicitação de recuperação de senha.

**Request Body:**

```json
{
  "email": "usuario@email.com"
}
```

**Response:**

```json
{
  "success": true,
  "data": "Senha enviada para o usuario@email.com",
  "message": null
}
```

**Autenticação:** Não requerida

---

### 🏢 Empresas (Company)

#### **GET** `/company`

Lista todas as filiais da ClubFlex.

**Query Parameters:**

- `includeVirtual` (boolean, opcional): Incluir empresas virtuais. Default: `false`

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Empresa Exemplo LTDA",
      "cnpj": "12345678000190",
      "enabled": true,
      "virtual": false
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `MANAGER`, `SUPERVISOR`, `BROKER`

---

#### **POST** `/company`

Cria uma nova empresa.

**Request Body:**

```json
{
  "name": "Nova Empresa LTDA",
  "cnpj": "98765432000100",
  "enabled": true,
  "virtual": false
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "id": 2,
    "name": "Nova Empresa LTDA",
    "cnpj": "98765432000100",
    "enabled": true,
    "virtual": false
  },
  "message": null
}
```

**Perfis Autorizados:** `MANAGER`

---

#### **PUT** `/company`

Atualiza uma empresa existente.

**Request Body:**

```json
{
  "id": 2,
  "name": "Empresa Atualizada LTDA",
  "cnpj": "98765432000100",
  "enabled": true,
  "virtual": false
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "id": 2,
    "name": "Empresa Atualizada LTDA",
    "cnpj": "98765432000100",
    "enabled": true,
    "virtual": false
  },
  "message": null
}
```

**Perfis Autorizados:** `MANAGER`

---

#### **GET** `/company/plan/{planId}`

Lista empresas associadas a um plano específico.

**Path Parameters:**

- `planId` (Long): ID do plano

**Query Parameters:**

- `includeVirtual` (boolean, opcional): Incluir empresas virtuais. Default: `false`

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Empresa A",
      "cnpj": "12345678000190",
      "enabled": true,
      "virtual": false
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `MANAGER`, `SUPERVISOR`, `BROKER`

---

#### **GET** `/company/broker/{brokerId}`

Lista empresas associadas a um corretor específico.

**Path Parameters:**

- `brokerId` (Long): ID do corretor

**Query Parameters:**

- `includeVirtual` (boolean, opcional): Incluir empresas virtuais. Default: `false`

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Empresa B",
      "cnpj": "12345678000190",
      "enabled": true,
      "virtual": false
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `MANAGER`, `SUPERVISOR`, `BROKER`

---

### 📋 Planos (Plan)

#### **GET** `/plan/list/avaliable/site`

Lista planos ativos disponíveis para exibição no site (online e PF).

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Plano Básico",
      "description": "Plano com benefícios básicos",
      "value": 99.90,
      "active": true,
      "online": true
    }
  ],
  "message": null
}
```

**Autenticação:** Não requerida

---

#### **GET** `/plan/list/active`

Lista todos os planos ativos.

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Plano Premium",
      "description": "Plano completo",
      "value": 199.90,
      "active": true
    }
  ],
  "message": null
}
```

**Autenticação:** Não requerida

---

#### **GET** `/plan/list/active/{type}`

Lista planos ativos de um tipo específico de pessoa.

**Path Parameters:**

- `type` (TypeSub): Tipo de assinatura (`PF` ou `PJ`)

**Query Parameters:**

- `companyId` (Long, opcional): ID da empresa
- `brokerId` (Long, opcional): ID do corretor

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Plano Empresarial",
      "type": "PJ",
      "value": 299.90,
      "active": true
    }
  ],
  "message": null
}
```

**Autenticação:** Não requerida

---

#### **GET** `/plan/list/all`

Lista todos os planos (ativos e inativos).

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Plano A",
      "active": true
    },
    {
      "id": 2,
      "name": "Plano B",
      "active": false
    }
  ],
  "message": null
}
```

**Autenticação:** Não requerida

---

#### **GET** `/plan/list/all/{type}`

Lista todos os planos de um tipo específico.

**Path Parameters:**

- `type` (TypeSub): Tipo de assinatura (`PF` ou `PJ`)

**Query Parameters:**

- `companyId` (Long, opcional): ID da empresa
- `brokerId` (Long, opcional): ID do corretor

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Plano PF Básico",
      "type": "PF",
      "value": 79.90
    }
  ],
  "message": null
}
```

**Autenticação:** Não requerida

---

#### **GET** `/plan/{planId}`

Obtém detalhes de um plano específico pelo ID.

**Path Parameters:**

- `planId` (Long): ID do plano

**Response:**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Plano Premium",
    "description": "Plano com todos os benefícios",
    "value": 199.90,
    "active": true,
    "benefits": []
  },
  "message": null
}
```

**Perfis Autorizados:** `MANAGER`, `SUPERVISOR`

---

### 👥 Titulares (Holder)

#### **POST** `/holder/filter`

Filtra titulares ativos baseado em critérios.

**Request Body:**

```json
{
  "name": "João",
  "cpfCnpj": "12345678900",
  "email": "joao@email.com",
  "planId": 1,
  "companyId": 1,
  "brokerId": 1,
  "page": 0,
  "size": 20
}
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "João Silva",
      "cpfCnpj": "12345678900",
      "email": "joao@email.com",
      "subscriptionStatus": "ACTIVE",
      "planName": "Plano Premium"
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`

---

#### **POST** `/holder/filter/inactive`

Filtra titulares inativos baseado em critérios.

**Request Body:**

```json
{
  "name": "Maria",
  "page": 0,
  "size": 20
}
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 2,
      "name": "Maria Santos",
      "cpfCnpj": "98765432100",
      "subscriptionStatus": "INACTIVE",
      "cancelDate": "2024-01-15"
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`

---

#### **POST** `/pj/filter`

Filtra titulares pessoa jurídica ativos.

**Request Body:**

```json
{
  "cnpj": "12345678000190",
  "companyName": "Empresa Teste",
  "page": 0,
  "size": 20
}
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Empresa Teste LTDA",
      "cpfCnpj": "12345678000190",
      "subscriptionStatus": "ACTIVE"
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`

---

#### **GET** `/holder/pj/company`

Obtém lista de empresas para filtro de titulares PJ.

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "key": 1,
      "value": "Empresa A LTDA"
    },
    {
      "key": 2,
      "value": "Empresa B LTDA"
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`

---

#### **POST** `/pj/filter/inactive`

Filtra titulares pessoa jurídica inativos.

**Request Body:**

```json
{
  "cnpj": "12345678000190",
  "page": 0,
  "size": 20
}
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 3,
      "name": "Empresa Inativa LTDA",
      "cpfCnpj": "12345678000190",
      "subscriptionStatus": "INACTIVE"
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`

---

#### **POST** `/holder/parceria/farma`

Filtra titulares para parceria farmácia.

**Request Body:**

```json
{
  "name": "Carlos",
  "cpfCnpj": "12345678900"
}
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Carlos Souza",
      "cpfCnpj": "12345678900",
      "cardNumber": "1234567890123456"
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`

---

#### **POST** `/dependent/filter`

Filtra dependentes baseado em critérios.

**Request Body:**

```json
{
  "name": "Ana",
  "cpf": "98765432100",
  "holderId": 1
}
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Ana Silva",
      "cpf": "98765432100",
      "holderName": "João Silva",
      "relationship": "FILHO"
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`

---

#### **GET** `/holder/{id}`

Obtém detalhes completos de um titular.

**Path Parameters:**

- `id` (Long): ID do titular

**Response:**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "João Silva",
    "cpfCnpj": "12345678900",
    "email": "joao@email.com",
    "phone": "11999999999",
    "birthDate": "1990-05-15",
    "address": {
      "street": "Rua Exemplo",
      "number": "123",
      "city": "São Paulo",
      "state": "SP",
      "zipCode": "01234-567"
    },
    "subscription": {
      "id": 1,
      "planName": "Plano Premium",
      "status": "ACTIVE",
      "startDate": "2023-01-01"
    }
  },
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`

---

#### **POST** `/holder`

Atualiza dados de um titular.

**Request Body:**

```json
{
  "id": 1,
  "name": "João Silva Santos",
  "email": "joao.novo@email.com",
  "phone": "11988888888",
  "address": {
    "street": "Rua Nova",
    "number": "456",
    "city": "São Paulo",
    "state": "SP",
    "zipCode": "01234-567"
  }
}
```

**Response:**

```json
{
  "success": true,
  "data": true,
  "message": "Titular atualizado com sucesso"
}
```

**Perfis Autorizados:** `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`

---

### 📝 Assinaturas (Subscription)

#### **POST** `/subscription`

Cria uma nova assinatura simples (pré-assinatura).

**Request Body:**

```json
{
  "planId": 1,
  "holder": {
    "name": "Pedro Oliveira",
    "cpfCnpj": "11122233344",
    "email": "pedro@email.com",
    "phone": "11977777777",
    "birthDate": "1985-03-20"
  },
  "address": {
    "street": "Av. Paulista",
    "number": "1000",
    "complement": "Apto 101",
    "neighborhood": "Bela Vista",
    "city": "São Paulo",
    "state": "SP",
    "zipCode": "01310-100"
  }
}
```

**Response:**

```json
{
  "success": true,
  "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "message": "Pré-assinatura criada com sucesso. Token de autenticação gerado."
}
```

**Autenticação:** Opcional (se não autenticado, cria novo usuário)

---

#### **PUT** `/subscription`

Completa uma assinatura (finaliza o cadastro).

**Request Body:**

```json
{
  "holder": {
    "id": 1,
    "name": "Pedro Oliveira",
    "cpfCnpj": "11122233344",
    "email": "pedro@email.com"
  },
  "planId": 1,
  "companyId": 1,
  "brokerId": 2,
  "paymentType": "CREDIT_CARD",
  "creditCard": {
    "holderName": "PEDRO OLIVEIRA",
    "number": "4111111111111111",
    "cvv": "123",
    "expirationMonth": 12,
    "expirationYear": 2025,
    "choice": true
  },
  "dependents": [
    {
      "name": "Maria Oliveira",
      "cpf": "22233344455",
      "birthDate": "2010-05-10",
      "relationship": "FILHO"
    }
  ]
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "subscriptionId": 1,
    "status": "ACTIVE",
    "vindiSubscriptionId": 12345,
    "cardRegistered": true,
    "firstInvoiceUrl": "https://vindi.com/invoice/123456"
  },
  "message": "Assinatura criada com sucesso"
}
```

**Perfis Autorizados:** `HOLDER`, `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`

---

### 🏦 Corretores (Broker)

#### **GET** `/broker`

Lista todos os corretores.

**Query Parameters:**

- `name` (String, opcional): Filtro por nome
- `enabled` (Boolean, opcional): Filtro por status ativo/inativo

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Corretora ABC",
      "email": "contato@corretorabc.com",
      "phone": "11988887777",
      "enabled": true
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `MANAGER`, `SUPERVISOR`, `BROKER`

---

#### **POST** `/broker`

Cria um novo corretor.

**Request Body:**

```json
{
  "name": "Corretora XYZ",
  "email": "contato@corretoraxyz.com",
  "phone": "11977776666",
  "enabled": true
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "id": 2,
    "name": "Corretora XYZ",
    "email": "contato@corretoraxyz.com",
    "phone": "11977776666",
    "enabled": true
  },
  "message": "Corretor criado com sucesso"
}
```

**Perfis Autorizados:** `MANAGER`, `ADMIN`

---

#### **PUT** `/broker`

Atualiza um corretor existente.

**Request Body:**

```json
{
  "id": 2,
  "name": "Corretora XYZ Ltda",
  "email": "novo@corretoraxyz.com",
  "phone": "11966665555",
  "enabled": true
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "id": 2,
    "name": "Corretora XYZ Ltda",
    "email": "novo@corretoraxyz.com",
    "phone": "11966665555",
    "enabled": true
  },
  "message": "Corretor atualizado com sucesso"
}
```

**Perfis Autorizados:** `MANAGER`, `ADMIN`

---

### 💳 Faturas (Invoice)

#### **GET** `/invoice`

Busca faturas com filtros.

**Query Parameters:**

- `initialDueDate` (Date, obrigatório): Data inicial de vencimento (formato: yyyy-MM-dd)
- `finalDueDate` (Date, obrigatório): Data final de vencimento (formato: yyyy-MM-dd)
- `status` (InvoiceStatus, opcional): Status da fatura (`PENDING`, `PAID`, `CANCELED`, etc.)
- `paymentType` (PaymentType, opcional): Tipo de pagamento (`CREDIT_CARD`, `BOLETO`, `PIX`)
- `invoiceType` (InvoiceType, opcional): Tipo de fatura
- `page` (Integer, opcional): Número da página (default: 0)
- `size` (Integer, opcional): Tamanho da página (default: 20)

**Response:**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "holderName": "João Silva",
        "amount": 199.90,
        "dueDate": "2024-01-15",
        "status": "PAID",
        "paymentType": "CREDIT_CARD",
        "paymentDate": "2024-01-14"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1
  },
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `MANAGER`, `SUPERVISOR`

---

#### **PUT** `/invoice/{invoiceId}/payment-type`

Altera o tipo de pagamento de uma fatura.

**Path Parameters:**

- `invoiceId` (Long): ID da fatura

**Request Body:**

```json
{
  "newPaymentType": "BOLETO"
}
```

**Response:**

```json
{
  "success": true,
  "data": true,
  "message": "Tipo de pagamento alterado com sucesso"
}
```

**Perfis Autorizados:** `ATTENDANT`, `MANAGER`, `SUPERVISOR`

---

### 👨‍👩‍👧‍👦 Dependentes (Dependent)

#### **GET** `/dependent`

Lista dependentes de um titular.

**Query Parameters:**

- `holderId` (Long, opcional): ID do titular

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Ana Silva",
      "cpf": "98765432100",
      "birthDate": "2010-05-15",
      "relationship": "FILHO",
      "active": true
    }
  ],
  "message": null
}
```

**Perfis Autorizados:** `ATTENDANT`, `BROKER`, `MANAGER`, `SUPERVISOR`, `HOLDER`

---

### 🎁 Benefícios (Benefit)

#### **GET** `/benefit`

Lista todos os benefícios disponíveis.

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Desconto em Farmácias",
      "description": "Até 80% de desconto em medicamentos",
      "category": "PHARMACY",
      "active": true
    }
  ],
  "message": null
}
```

**Autenticação:** Não requerida

---

### 📞 Callbacks

#### **POST** `/callbacks/vindi`

Webhook para receber notificações da Vindi.

**Request Body:**

```json
{
  "event": {
    "type": "bill_paid",
    "created_at": "2024-01-15T10:30:00Z",
    "data": {
      "bill": {
        "id": 12345,
        "amount": 199.90,
        "status": "paid"
      }
    }
  }
}
```

**Response:**

```json
{
  "success": true,
  "message": "Callback processado com sucesso"
}
```

**Autenticação:** Não requerida (validação interna)

---

## Códigos de Status HTTP

| Código | Descrição |
|--------|-----------|
| `200` | Requisição bem-sucedida |
| `201` | Recurso criado com sucesso |
| `400` | Requisição inválida (erro de validação) |
| `401` | Não autenticado |
| `403` | Não autorizado (sem permissão) |
| `404` | Recurso não encontrado |
| `409` | Conflito (ex: recurso já existe) |
| `500` | Erro interno do servidor |

---

## Estrutura de Resposta Padrão

Todas as respostas seguem o padrão `StandardResponse`:

```json
{
  "success": boolean,
  "data": any,
  "message": string | null
}
```

### Resposta de Sucesso

```json
{
  "success": true,
  "data": { ... },
  "message": null
}
```

### Resposta de Erro

```json
{
  "success": false,
  "data": null,
  "message": "Mensagem de erro detalhada"
}
```

---

## Paginação

Endpoints que retornam listas paginadas utilizam a estrutura `PaginatedResponse`:

```json
{
  "success": true,
  "data": {
    "content": [ ... ],
    "page": 0,
    "size": 20,
    "totalElements": 100,
    "totalPages": 5,
    "first": true,
    "last": false
  },
  "message": null
}
```

---

## Enumerações Importantes

### PaymentType

- `CREDIT_CARD` - Cartão de crédito
- `DEBIT_CARD` - Cartão de débito
- `BOLETO` - Boleto bancário
- `PIX` - PIX

### InvoiceStatus

- `PENDING` - Pendente
- `PAID` - Pago
- `CANCELED` - Cancelado
- `OVERDUE` - Vencido

### TypeSub

- `PF` - Pessoa Física
- `PJ` - Pessoa Jurídica

### UserProfile

- `HOLDER` - Titular
- `ATTENDANT` - Atendente
- `BROKER` - Corretor
- `SUPERVISOR` - Supervisor
- `MANAGER` - Gerente
- `ADMIN` - Administrador

---

## Rate Limiting

A API não implementa rate limiting atualmente, mas é recomendado implementar controle de requisições no lado do cliente para evitar sobrecarga.

---

## Versionamento

A API atualmente não utiliza versionamento de endpoints. Futuras versões podem implementar versionamento através de:

- Path: `/api/v1/...`
- Header: `API-Version: 1.0`

---

## Suporte e Contato

Para questões técnicas ou suporte, entre em contato com a equipe de desenvolvimento.

**Última atualização:** Novembro 2024
