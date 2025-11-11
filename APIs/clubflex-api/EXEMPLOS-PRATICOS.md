# 💡 Exemplos Práticos - API ClubFlex

Este documento contém exemplos práticos de uso da API ClubFlex para casos de uso comuns.

---

## 🚀 Começando

### Pré-requisitos

- URL base da API
- Credenciais de acesso (email e senha)
- Ferramenta para fazer requisições HTTP (Postman, curl, ou código)

---

## 🔐 1. Autenticação

### Exemplo 1.1: Login e Obtenção de Token

**Requisição:**

```bash
curl -X POST https://api.clubflex.com.br/user/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "usuario@email.com",
    "password": "senha123"
  }'
```

**Resposta:**

```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": 123,
      "name": "João Silva",
      "email": "usuario@email.com",
      "profile": "HOLDER"
    }
  },
  "message": null
}
```

**💡 Dica:** Salve o token para usar nas próximas requisições!

---

## 📋 2. Consultar Planos Disponíveis

### Exemplo 2.1: Listar Planos Ativos para Pessoa Física

**Requisição:**

```bash
curl -X GET "https://api.clubflex.com.br/plan/list/active/PF" \
  -H "Content-Type: application/json"
```

**Resposta:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Plano Básico",
      "description": "Plano individual com benefícios essenciais",
      "value": 79.90,
      "type": "PF",
      "active": true,
      "online": true,
      "benefits": [
        {
          "id": 1,
          "name": "Desconto em Farmácias",
          "description": "Até 70% de desconto em medicamentos"
        }
      ]
    },
    {
      "id": 2,
      "name": "Plano Premium",
      "description": "Plano completo com todos os benefícios",
      "value": 149.90,
      "type": "PF",
      "active": true,
      "online": true
    }
  ],
  "message": null
}
```

---

## 👤 3. Criar Nova Assinatura (Cliente Final)

### Exemplo 3.1: Passo 1 - Criar Pré-Assinatura

**Requisição:**

```bash
curl -X POST https://api.clubflex.com.br/subscription \
  -H "Content-Type: application/json" \
  -d '{
    "planId": 1,
    "holder": {
      "name": "Maria Santos",
      "cpfCnpj": "12345678900",
      "email": "maria@email.com",
      "phone": "11999999999",
      "birthDate": "1990-05-15"
    },
    "address": {
      "street": "Av. Paulista",
      "number": "1000",
      "complement": "Apto 101",
      "neighborhood": "Bela Vista",
      "city": "São Paulo",
      "state": "SP",
      "zipCode": "01310100"
    }
  }'
```

**Resposta:**

```json
{
  "success": true,
  "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJob2xkZXJJZCI6NDU2fQ...",
  "message": "Pré-assinatura criada. Use o token para completar."
}
```

---

### Exemplo 3.2: Passo 2 - Completar Assinatura com Cartão

**Requisição:**

```bash
curl -X PUT https://api.clubflex.com.br/subscription \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-da-pre-assinatura}" \
  -d '{
    "holder": {
      "id": 456,
      "name": "Maria Santos",
      "cpfCnpj": "12345678900",
      "email": "maria@email.com"
    },
    "planId": 1,
    "paymentType": "CREDIT_CARD",
    "creditCard": {
      "holderName": "MARIA SANTOS",
      "number": "4111111111111111",
      "cvv": "123",
      "expirationMonth": 12,
      "expirationYear": 2025,
      "choice": true
    },
    "dependents": []
  }'
```

**Resposta:**

```json
{
  "success": true,
  "data": {
    "subscriptionId": 789,
    "status": "ACTIVE",
    "vindiSubscriptionId": 12345,
    "cardRegistered": true,
    "cardNumber": "411111******1111",
    "firstInvoiceUrl": "https://vindi.com/faturas/98765"
  },
  "message": "Assinatura ativada com sucesso!"
}
```

---

### Exemplo 3.3: Completar Assinatura com Boleto

**Requisição:**

```bash
curl -X PUT https://api.clubflex.com.br/subscription \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-da-pre-assinatura}" \
  -d '{
    "holder": {
      "id": 456,
      "name": "Maria Santos",
      "cpfCnpj": "12345678900",
      "email": "maria@email.com"
    },
    "planId": 1,
    "paymentType": "BOLETO",
    "dependents": []
  }'
```

**Resposta:**

```json
{
  "success": true,
  "data": {
    "subscriptionId": 790,
    "status": "PENDING",
    "vindiSubscriptionId": 12346,
    "boletoUrl": "https://vindi.com/boletos/98766.pdf",
    "boletoBarcode": "23793.38128 60000.123456 78901.234567 8 12340000079900"
  },
  "message": "Assinatura criada. Aguardando pagamento do boleto."
}
```

---

## 🔍 4. Consultar Titulares (Backoffice)

### Exemplo 4.1: Filtrar Titulares Ativos

**Requisição:**

```bash
curl -X POST https://api.clubflex.com.br/holder/filter \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-atendente}" \
  -d '{
    "name": "Maria",
    "planId": 1,
    "page": 0,
    "size": 20
  }'
```

**Resposta:**

```json
{
  "success": true,
  "data": [
    {
      "id": 456,
      "name": "Maria Santos",
      "cpfCnpj": "12345678900",
      "email": "maria@email.com",
      "phone": "11999999999",
      "subscriptionStatus": "ACTIVE",
      "planName": "Plano Básico",
      "planValue": 79.90,
      "subscriptionStartDate": "2024-01-15",
      "paymentType": "CREDIT_CARD"
    }
  ],
  "message": null
}
```

---

### Exemplo 4.2: Obter Detalhes Completos de um Titular

**Requisição:**

```bash
curl -X GET https://api.clubflex.com.br/holder/456 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-atendente}"
```

**Resposta:**

```json
{
  "success": true,
  "data": {
    "id": 456,
    "name": "Maria Santos",
    "cpfCnpj": "12345678900",
    "email": "maria@email.com",
    "phone": "11999999999",
    "birthDate": "1990-05-15",
    "address": {
      "street": "Av. Paulista",
      "number": "1000",
      "complement": "Apto 101",
      "neighborhood": "Bela Vista",
      "city": "São Paulo",
      "state": "SP",
      "zipCode": "01310100"
    },
    "subscription": {
      "id": 789,
      "planId": 1,
      "planName": "Plano Básico",
      "value": 79.90,
      "status": "ACTIVE",
      "startDate": "2024-01-15",
      "paymentType": "CREDIT_CARD"
    },
    "cardNumber": "1234567890123456",
    "dependents": []
  },
  "message": null
}
```

---

## 👨‍👩‍👧 5. Gerenciar Dependentes

### Exemplo 5.1: Adicionar Dependente

**Requisição:**

```bash
curl -X POST https://api.clubflex.com.br/dependent \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-titular}" \
  -d '{
    "holderId": 456,
    "name": "João Santos",
    "cpf": "98765432100",
    "birthDate": "2015-03-20",
    "relationship": "FILHO",
    "active": true
  }'
```

**Resposta:**

```json
{
  "success": true,
  "data": {
    "id": 101,
    "name": "João Santos",
    "cpf": "98765432100",
    "birthDate": "2015-03-20",
    "relationship": "FILHO",
    "active": true,
    "cardNumber": "1234567890123457",
    "additionalValue": 39.90
  },
  "message": "Dependente adicionado. Novo valor da assinatura: R$ 119,80"
}
```

---

## 💳 6. Consultar Faturas

### Exemplo 6.1: Buscar Faturas do Mês

**Requisição:**

```bash
curl -X GET "https://api.clubflex.com.br/invoice?initialDueDate=2024-01-01&finalDueDate=2024-01-31&page=0&size=50" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-gerente}"
```

**Resposta:**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1001,
        "holderName": "Maria Santos",
        "amount": 79.90,
        "dueDate": "2024-01-15",
        "status": "PAID",
        "paymentType": "CREDIT_CARD",
        "paymentDate": "2024-01-14",
        "planName": "Plano Básico"
      },
      {
        "id": 1002,
        "holderName": "Carlos Oliveira",
        "amount": 149.90,
        "dueDate": "2024-01-20",
        "status": "PENDING",
        "paymentType": "BOLETO",
        "planName": "Plano Premium"
      }
    ],
    "page": 0,
    "size": 50,
    "totalElements": 2,
    "totalPages": 1
  },
  "message": null
}
```

---

## 🏢 7. Gestão de Empresas (Gerente)

### Exemplo 7.1: Criar Nova Empresa

**Requisição:**

```bash
curl -X POST https://api.clubflex.com.br/company \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-gerente}" \
  -d '{
    "name": "Tech Solutions LTDA",
    "cnpj": "12345678000190",
    "enabled": true,
    "virtual": false
  }'
```

**Resposta:**

```json
{
  "success": true,
  "data": {
    "id": 10,
    "name": "Tech Solutions LTDA",
    "cnpj": "12345678000190",
    "enabled": true,
    "virtual": false
  },
  "message": "Empresa criada com sucesso"
}
```

---

### Exemplo 7.2: Listar Empresas por Plano

**Requisição:**

```bash
curl -X GET "https://api.clubflex.com.br/company/plan/1" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-gerente}"
```

**Resposta:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Empresa A LTDA",
      "cnpj": "11111111000111",
      "enabled": true
    },
    {
      "id": 2,
      "name": "Empresa B LTDA",
      "cnpj": "22222222000122",
      "enabled": true
    }
  ],
  "message": null
}
```

---

## 🏦 8. Gestão de Corretores

### Exemplo 8.1: Criar Corretor

**Requisição:**

```bash
curl -X POST https://api.clubflex.com.br/broker \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-gerente}" \
  -d '{
    "name": "Corretora Excellence",
    "email": "contato@excellence.com",
    "phone": "11988887777",
    "enabled": true
  }'
```

**Resposta:**

```json
{
  "success": true,
  "data": {
    "id": 5,
    "name": "Corretora Excellence",
    "email": "contato@excellence.com",
    "phone": "11988887777",
    "enabled": true
  },
  "message": "Corretor criado com sucesso"
}
```

---

### Exemplo 8.2: Listar Corretores Ativos

**Requisição:**

```bash
curl -X GET "https://api.clubflex.com.br/broker?enabled=true" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-atendente}"
```

**Resposta:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Corretora ABC",
      "email": "contato@abc.com",
      "phone": "11999998888",
      "enabled": true
    },
    {
      "id": 5,
      "name": "Corretora Excellence",
      "email": "contato@excellence.com",
      "phone": "11988887777",
      "enabled": true
    }
  ],
  "message": null
}
```

---

## 🔄 9. Atualizar Dados

### Exemplo 9.1: Atualizar Dados do Titular

**Requisição:**

```bash
curl -X POST https://api.clubflex.com.br/holder \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-titular}" \
  -d '{
    "id": 456,
    "name": "Maria Santos Silva",
    "email": "maria.nova@email.com",
    "phone": "11988888888",
    "address": {
      "street": "Rua Nova",
      "number": "2000",
      "complement": "Casa",
      "neighborhood": "Jardim Paulista",
      "city": "São Paulo",
      "state": "SP",
      "zipCode": "01400000"
    }
  }'
```

**Resposta:**

```json
{
  "success": true,
  "data": true,
  "message": "Dados atualizados com sucesso"
}
```

---

### Exemplo 9.2: Alterar Forma de Pagamento de uma Fatura

**Requisição:**

```bash
curl -X PUT https://api.clubflex.com.br/invoice/1002/payment-type \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token-atendente}" \
  -d '{
    "newPaymentType": "PIX"
  }'
```

**Resposta:**

```json
{
  "success": true,
  "data": true,
  "message": "Forma de pagamento alterada para PIX"
}
```

---

## ❌ 10. Tratamento de Erros

### Exemplo 10.1: Token Inválido ou Expirado

**Requisição:**

```bash
curl -X GET https://api.clubflex.com.br/holder/456 \
  -H "Authorization: Bearer token-invalido"
```

**Resposta:**

```json
{
  "success": false,
  "data": null,
  "message": "Token inválido ou expirado"
}
```

**Status HTTP:** `401 Unauthorized`

---

### Exemplo 10.2: Sem Permissão

**Requisição:**

```bash
curl -X POST https://api.clubflex.com.br/company \
  -H "Authorization: Bearer {token-atendente}" \
  -d '{ "name": "Nova Empresa" }'
```

**Resposta:**

```json
{
  "success": false,
  "data": null,
  "message": "Usuário não possui permissão para esta operação"
}
```

**Status HTTP:** `403 Forbidden`

---

### Exemplo 10.3: Validação de Dados

**Requisição:**

```bash
curl -X POST https://api.clubflex.com.br/subscription \
  -H "Content-Type: application/json" \
  -d '{
    "planId": 1,
    "holder": {
      "name": "",
      "cpfCnpj": "123",
      "email": "email-invalido"
    }
  }'
```

**Resposta:**

```json
{
  "success": false,
  "data": null,
  "message": "Erros de validação encontrados",
  "errors": [
    {
      "field": "holder.name",
      "message": "Nome é obrigatório"
    },
    {
      "field": "holder.cpfCnpj",
      "message": "CPF inválido"
    },
    {
      "field": "holder.email",
      "message": "Email inválido"
    }
  ]
}
```

**Status HTTP:** `400 Bad Request`

---

## 🧪 11. Ambiente de Testes

### Dados de Teste

**Cartões de Teste (eRede):**

```
Aprovado:
Número: 4111 1111 1111 1111
CVV: 123
Validade: 12/2025

Negado:
Número: 4000 0000 0000 0002
CVV: 123
Validade: 12/2025
```

**CPFs de Teste:**

```
11111111111 (válido)
22222222222 (válido)
12345678900 (válido)
```

---

## 📚 Bibliotecas e SDKs

### JavaScript/Node.js

```javascript
const axios = require('axios');

const api = axios.create({
  baseURL: 'https://api.clubflex.com.br',
  headers: {
    'Content-Type': 'application/json'
  }
});

// Fazer login
async function login(email, password) {
  const response = await api.post('/user/login', { email, password });
  const token = response.data.data.token;
  
  // Configurar token para próximas requisições
  api.defaults.headers.common['Authorization'] = `Bearer ${token}`;
  
  return token;
}

// Listar planos
async function getPlans() {
  const response = await api.get('/plan/list/active');
  return response.data.data;
}

// Uso
(async () => {
  await login('usuario@email.com', 'senha123');
  const plans = await getPlans();
  console.log(plans);
})();
```

---

### Python

```python
import requests

class ClubFlexAPI:
    def __init__(self, base_url='https://api.clubflex.com.br'):
        self.base_url = base_url
        self.token = None
        self.session = requests.Session()
    
    def login(self, email, password):
        response = self.session.post(
            f'{self.base_url}/user/login',
            json={'email': email, 'password': password}
        )
        data = response.json()
        self.token = data['data']['token']
        self.session.headers.update({
            'Authorization': f'Bearer {self.token}'
        })
        return self.token
    
    def get_plans(self):
        response = self.session.get(f'{self.base_url}/plan/list/active')
        return response.json()['data']

# Uso
api = ClubFlexAPI()
api.login('usuario@email.com', 'senha123')
plans = api.get_plans()
print(plans)
```

---

## 🔗 Links Relacionados

- [Documentação Técnica Completa](./01-DOCUMENTACAO-TECNICA.md)
- [Fluxos Visuais](./03-FLUXOS-VISUAIS.md)
- [Glossário](./GLOSSARIO.md)

---

**Última atualização:** Novembro 2024
