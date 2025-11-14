# Serviços Externos - Integrações da API

## Visão Geral

A API Segmedic Totem integra-se com três serviços externos principais para fornecer funcionalidades completas de agendamento, convênio e pagamento.

---

## 1. Feegow API

### 🎯 Propósito
O Feegow é o sistema principal de gestão médica que armazena e gerencia todas as informações de agendamentos, pacientes, profissionais e financeiro da clínica.

### 🔧 Configuração
```ruby
FEEGOW_API_ENDPOINT=https://api.feegow.com.br
FEEGOW_API_TOKEN=seu_token_aqui
```

### 📡 Autenticação
- **Tipo**: Token Bearer
- **Header**: `x-access-token: {FEEGOW_API_TOKEN}`
- **Formato**: JSON

---

### 📋 Endpoints Utilizados

#### **Agendamentos**

| Endpoint | Método | Uso na API Segmedic | Parâmetros |
|----------|--------|---------------------|------------|
| `/appoints/new-appoint` | POST | Criar novo agendamento | `local_id`, `paciente_id`, `profissional_id`, `especialidade_id`, `procedimento_id`, `data`, `horario`, `valor`, `plano`, `convenio_id` |
| `/appoints/search` | GET | Buscar agendamentos | `agendamento_id`, `paciente_id`, `data_start`, `data_end` |
| `/appoints/cancel-appoint` | POST | Cancelar agendamento | `agendamento_id`, `motivo_id` |
| `/appoints/reschedule` | POST | Reagendar consulta | `agendamento_id`, `motivo_id`, `horario`, `data` |
| `/appoints/statusUpdate` | POST | Atualizar status (check-in) | `AgendamentoID`, `StatusID`, `HoraChegada` |
| `/v2/appoints/available-schedule` | GET | Buscar horários disponíveis | `data_start`, `data_end`, `unidade_id`, `tipo`, `especialidade_id` ou `procedimento_id`, `profissional_id`, `convenio_id` |
| `/appoints/queue-position` | GET | Gerar senha de atendimento | `unidade_id`, `tipo_senha` |

**Exemplo de Request - Criar Agendamento:**
```json
POST /appoints/new-appoint
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
  "notas": "#agTOTEM"
}
```

**Exemplo de Response:**
```json
{
  "success": true,
  "content": {
    "agendamento_id": 12345
  }
}
```

---

#### **Pacientes**

| Endpoint | Método | Uso na API Segmedic | Parâmetros |
|----------|--------|---------------------|------------|
| `/patient/list` | GET | Listar pacientes | `offset`, `limit` |
| `/patient/search` | GET | Buscar paciente | `paciente_id` ou `paciente_cpf` |
| `/patient/create` | POST | Criar novo paciente | `nome_completo`, `cpf`, `data_nascimento`, `genero`, `email`, `telefone`, `tabela_id`, `convenio_id`, `plano_id`, `matricula`, `validade` |
| `/patient/edit` | POST | Atualizar paciente | `paciente_id` + campos a atualizar |
| `/patient/list-privates` | GET | Listar tabelas de preço | - |

**Exemplo de Request - Criar Paciente:**
```json
POST /patient/create
{
  "nome_completo": "João Silva",
  "cpf": "12345678900",
  "data_nascimento": "15-05-1980",
  "genero": "M",
  "email": "joao@exemplo.com",
  "telefone": "11987654321",
  "tabela_id": 1
}
```

**Exemplo de Response:**
```json
{
  "success": true,
  "content": {
    "paciente_id": 678,
    "nome": "João Silva",
    "cpf": "123.456.789-00"
  }
}
```

---

#### **Profissionais**

| Endpoint | Método | Uso na API Segmedic | Parâmetros |
|----------|--------|---------------------|------------|
| `/professional/list` | GET | Listar profissionais | `especialidade_id`, `ativo` |
| `/professional/search` | GET | Buscar profissional | `profissional_id` |
| `/professional/insurance` | GET | Listar convênios aceitos | `profissional_id` |

**Exemplo de Response - Buscar Profissional:**
```json
{
  "content": {
    "profissional_id": 45,
    "nome": "Dr. Maria Santos",
    "crm": "123456",
    "ativo": true,
    "especialidades": [
      {
        "especialidade_id": 12,
        "nome": "Cardiologia"
      }
    ]
  }
}
```

---

#### **Especialidades e Procedimentos**

| Endpoint | Método | Uso na API Segmedic | Parâmetros |
|----------|--------|---------------------|------------|
| `/specialties/list` | GET | Listar especialidades | - |
| `/procedures/list` | GET | Listar procedimentos | `procedimento_id`, `especialidade_id` |

---

#### **Unidades e Locais**

| Endpoint | Método | Uso na API Segmedic | Parâmetros |
|----------|--------|---------------------|------------|
| `/company/list-unity` | GET | Listar unidades | - |
| `/company/list-local` | GET | Listar consultórios | - |

---

#### **Convênios**

| Endpoint | Método | Uso na API Segmedic | Parâmetros |
|----------|--------|---------------------|------------|
| `/insurance/list` | GET | Listar convênios | - |

---

#### **Financeiro**

| Endpoint | Método | Uso na API Segmedic | Parâmetros |
|----------|--------|---------------------|------------|
| `/financial/create-account` | POST | Criar conta financeira | `agendamento_id`, `date` |
| `/core/financial/invoice/create` | POST | Criar invoice (nota) | Payload complexo com itens, parcelas, etc |
| `/financial/list-invoice` | GET | Listar notas fiscais | `tipo_transacao`, `data_start`, `data_end`, `invoice_id`, `agendamento_id` |
| `/financial/pay-movement` | POST | Registrar pagamento | `invoiceId`, `movementId`, `accountId`, `amount`, `paymentName`, `paymentMethod`, `paymentDate`, `creditCardTransaction` |
| `/financial/credit-card-flags` | GET | Listar bandeiras de cartão | - |

**Exemplo de Request - Pagar Conta:**
```bash
POST /financial/pay-movement?invoiceId=NF-123&movementId=456&accountId=678&amount=150.00&paymentName=Cartão%20de%20Crédito&paymentMethod=credit_card&paymentDate=2025-11-15&associationId=3&creditCardTransaction.transactionNumber=789&creditCardTransaction.authorizationNumber=ABC123&creditCardTransaction.flagCardId=1&creditCardTransaction.installments=1
```

---

#### **Relatórios**

| Endpoint | Método | Uso na API Segmedic | Parâmetros |
|----------|--------|---------------------|------------|
| `/reports/generate` | POST | Gerar relatórios | `report` (tipo), parâmetros específicos por tipo |

**Tipos de Relatório:**
- `price-table`: Tabela de preços
- `bills-to-receive`: Contas a receber

---

### 🔄 Cache Strategy

A API implementa cache para reduzir chamadas ao Feegow:

| Endpoint | TTL | Motivo |
|----------|-----|--------|
| `/company/list-local` | 1 hora | Consultórios mudam raramente |
| `/reports/generate` (contas) | 1 hora | Relatório pesado |

**Implementação:**
```ruby
cached_data = Rails.cache.read(url)
if cached_data.nil?
    response = @conn.get(url)
    data = { body: response.body["content"], code: response.status }
    Rails.cache.write(url, data, expires_in: 1.hour)
    return data
end
cached_data
```

---

### 📊 Logging

Todas as chamadas ao Feegow são registradas na tabela `service_logs`:

```ruby
ServiceLog.create(
  origin: user_session_id,      # De onde veio a requisição
  provider: :feegow,             # Provedor
  params: { ... },               # Parâmetros enviados
  response: { ... },             # Resposta recebida
  status: 200,                   # Status HTTP
  method: "post",                # GET, POST, etc
  url_path: "full_url",          # URL completa
  requested_at: DateTime.now    # Timestamp
)
```

---

### ⚠️ Pontos de Atenção

1. **Nota especial em agendamentos**: Todos os agendamentos criados pelo totem incluem `"notas": "#agTOTEM"` para identificação
2. **Formato de data**: Feegow usa `DD-MM-YYYY` enquanto a API Rails usa `YYYY-MM-DD`
3. **Horários disponíveis**: A API filtra horários para mostrar apenas 1-3 horas à frente do momento atual
4. **Valores monetários**: Podem vir como string "R$ 150,00" e precisam ser convertidos para float

---

## 2. ClubFlex API

### 🎯 Propósito
ClubFlex é um sistema de assinatura de plano de saúde. Pacientes assinantes têm desconto em consultas. A API verifica elegibilidade antes de permitir agendamento.

### 🔧 Configuração
```ruby
CLUBFLEX_API_ENDPOINT=https://api.clubflex.com.br
CLUBFLEX_API_LOGIN=seu_login
CLUBFLEX_API_PASSWORD=sua_senha
```

### 📡 Autenticação
- **Tipo**: Token dinâmico obtido via login
- **Header**: `token: {token_obtido}`
- **Renovação**: A cada requisição (token via `/user/backoffice/login`)

**Fluxo de Autenticação:**
```ruby
# 1. Login para obter token
POST /user/backoffice/login
{
  "login": "usuario",
  "password": "senha"
}

# Response:
{
  "object": "token_jwt_aqui"
}

# 2. Usar token nas requisições
GET /eligibility/customer/{cpf}
Headers: { "token": "token_jwt_aqui" }
```

---

### 📋 Endpoints Utilizados

| Endpoint | Método | Uso na API Segmedic | Parâmetros |
|----------|--------|---------------------|------------|
| `/user/backoffice/login` | POST | Autenticar e obter token | `login`, `password` |
| `/eligibility/customer/{cpf}` | GET | Verificar elegibilidade | `cpf` (na URL) |
| `/holder/filter` | POST | Buscar titular (PF) | `cpfCnpjHolder` |
| `/pj/filter` | POST | Buscar empresa (PJ) | `cpfCnpjHolder` |

---

### 📝 Verificação de Elegibilidade

**Request:**
```http
GET /eligibility/customer/12345678900
Headers: {
  "token": "token_aqui"
}
```

**Response - Paciente Elegível:**
```json
{
  "object": {
    "status": "OK",
    "typeSub": "PF",
    "name": "João Silva"
  }
}
```

**Response - Paciente Inadimplente:**
```json
{
  "object": {
    "status": "BLOCKED",
    "typeSub": "PF",
    "name": "João Silva"
  }
}
```

**Response - Paciente Cancelado:**
```json
{
  "object": {
    "status": "CANCELED",
    "typeSub": "PF",
    "name": "João Silva"
  }
}
```

**Response - Não é assinante:**
```json
{
  "object": null
}
```

---

### 🔄 Lógica de Elegibilidade

A API Segmedic processa os status da seguinte forma:

```ruby
case status
when "OK"
  # Paciente pode agendar com desconto ClubFlex
  # Tabela de preço: 250 (ClubFlex)
  result[:status] = "ok"
  
when "BLOCKED"
  # Paciente inadimplente, não pode usar benefícios
  result[:status] = "inadimplente"
  
when "CANCELED"
  # Assinatura cancelada
  result[:status] = "cancelado"
  
else
  # Não é assinante
  result[:status] = "nao_encontrado"
end
```

---

### 📊 Logging

Chamadas ao ClubFlex também são registradas:

```ruby
ServiceLog.create(
  origin: user_session_id,
  provider: :clubflex,
  params: { cpf: cpf },
  response: response.body,
  status: response.status,
  method: "get",
  requested_at: DateTime.now,
  url_path: "#{ENV['CLUBFLEX_API_ENDPOINT']}/eligibility/customer/#{cpf}"
)
```

---

### 🎫 Integração com Feegow

Quando paciente é elegível no ClubFlex:
1. API verifica status no ClubFlex
2. Se `status == "OK"`, define `tabela_id: 250` (tabela ClubFlex no Feegow)
3. Agendamento é criado no Feegow com essa tabela
4. Valor da consulta é automaticamente ajustado conforme tabela

---

### ⚠️ Pontos de Atenção

1. **Token expira**: Cada requisição faz novo login para garantir token válido
2. **CPF formato**: Aceita com ou sem máscara (123.456.789-00 ou 12345678900)
3. **Tipo de assinatura**: `PF` (Pessoa Física) ou `PJ` (Pessoa Jurídica via empresa)
4. **Prioridade**: Se paciente tem convênio E é ClubFlex, convênio tem prioridade

---

## 3. Itaú PIX API (Microserviço)

### 🎯 Propósito
Microserviço próprio que faz interface com a API PIX do Itaú para gerar QR Codes e verificar status de pagamentos.

### 🔧 Configuração
```ruby
MS_PIX_URL=https://pix-ms.segmedic.com.br
MS_PIX_TOKEN=seu_token_bearer
```

### 📡 Autenticação
- **Tipo**: Token Bearer
- **Header**: `Authorization: Bearer {MS_PIX_TOKEN}`
- **Formato**: JSON

---

### 📋 Endpoints Utilizados

| Endpoint | Método | Uso na API Segmedic | Parâmetros |
|----------|--------|---------------------|------------|
| `/generate_pix` | GET | Gerar QR Code PIX | `value` (valor em reais) |
| `/pix` | GET | Consultar status do PIX | `txid` (ID da transação) |

---

### 💰 Gerar PIX

**Request:**
```http
GET /generate_pix?value=150.00
Headers: {
  "Authorization": "Bearer token_aqui"
}
```

**Response:**
```json
{
  "txid": "PIX-123456789ABCDEF",
  "qrcode": "00020126580014br.gov.bcb.pix...",
  "qrcode_image": "data:image/png;base64,iVBORw0KGgoAAAANS...",
  "valor": 150.00,
  "expiracao": "2025-11-15T15:30:00Z",
  "location": "pix.example.com/qr/v2/123"
}
```

**Campos:**
- `txid`: Identificador único da transação
- `qrcode`: String Pix Copia e Cola
- `qrcode_image`: QR Code em Base64 (para exibir no totem)
- `valor`: Valor em reais
- `expiracao`: Timestamp de expiração (geralmente 30 minutos)

---

### 🔍 Consultar Status do PIX

**Request:**
```http
GET /pix?txid=PIX-123456789ABCDEF
Headers: {
  "Authorization": "Bearer token_aqui"
}
```

**Response - Aguardando Pagamento:**
```json
{
  "txid": "PIX-123456789ABCDEF",
  "status": "ATIVA",
  "valor": {
    "original": "150.00"
  },
  "horario": "2025-11-15T14:30:00Z"
}
```

**Response - Pago:**
```json
{
  "txid": "PIX-123456789ABCDEF",
  "status": "CONCLUIDA",
  "valor": {
    "original": "150.00"
  },
  "horario": "2025-11-15T14:35:22Z",
  "pagador": {
    "cpf": "***456789**",
    "nome": "JOAO SILVA"
  },
  "endToEndId": "E12345678202511151435AbCdEf123"
}
```

**Status Possíveis:**
- `ATIVA`: Aguardando pagamento
- `CONCLUIDA`: Pagamento confirmado
- `REMOVIDA_PELO_USUARIO_RECEBEDOR`: Cancelada pelo recebedor
- `REMOVIDA_PELO_PSP`: Expirada

---

### 🔄 Fluxo de Pagamento PIX no Totem

```
1. Paciente escolhe pagar com PIX
   ↓
2. API chama /generate_pix?value=150.00
   ↓
3. Exibe QR Code na tela do totem
   ↓
4. Loop: A cada 3 segundos chama /pix?txid=...
   ↓
5. Quando status == "CONCLUIDA"
   ↓
6. Registra pagamento no Feegow
   ↓
7. Emite senha de atendimento
```

---

### 📊 Logging

Pagamentos PIX são registrados em `payment_logs`:

```ruby
PaymentLog.create(
  user_id: current_user.id,
  invoice_id: invoice_id,
  agendamento_id: agendamento_id,
  kind: "success",              # ou "pending", "fail"
  payment_method: "pix",
  valor: 150.00,
  txid: "PIX-123456789",
  pix_response: response.body
)
```

---

### ⚠️ Pontos de Atenção

1. **Expiração**: QR Codes geralmente expiram em 30 minutos
2. **Polling**: Totem deve consultar status a cada 3-5 segundos
3. **Timeout**: Após 10 minutos sem pagamento, oferecer outras opções
4. **Valor**: Sempre em formato decimal com 2 casas (150.00, não 150)
5. **Confirmação**: Aguardar status CONCLUIDA antes de liberar senha

---

## Resumo Comparativo

| Serviço | Propósito | Autenticação | Principais Endpoints | Cache |
|---------|-----------|--------------|---------------------|-------|
| **Feegow** | Gestão médica completa | Token fixo (header) | 20+ endpoints (agenda, pacientes, financeiro) | Sim (1h) |
| **ClubFlex** | Verificação de convênio | Token dinâmico (renovado a cada chamada) | 4 endpoints (elegibilidade, holder) | Não |
| **Itaú PIX** | Pagamentos PIX | Bearer token fixo | 2 endpoints (gerar, consultar) | Não |

---

## Mapa de Dependências

```
┌─────────────────────────────────────────────────────┐
│                  API Segmedic Totem                  │
└─────────────────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
    ┌────────┐     ┌──────────┐    ┌─────────┐
    │ Feegow │     │ ClubFlex │    │Itaú PIX │
    └────────┘     └──────────┘    └─────────┘
         │
         └─► Agendamentos
         └─► Pacientes
         └─► Profissionais
         └─► Especialidades
         └─► Financeiro
         └─► Unidades
         └─► Procedimentos
         └─► Convênios (cadastro)
         
                    │
                    └─► Elegibilidade ClubFlex
                    └─► Status de assinatura
                    
                                   │
                                   └─► Gerar QR Code
                                   └─► Verificar pagamento
```

---

## Tratamento de Erros

### Feegow
- **Timeout**: 30 segundos
- **Retry**: Não há retry automático
- **Erro 401**: Token inválido (verificar variável de ambiente)
- **Erro 422**: Validação falhou (verificar parâmetros)

### ClubFlex
- **Timeout**: 15 segundos
- **Retry**: Token é renovado a cada chamada
- **Erro 401**: Credenciais inválidas
- **Response null**: Paciente não é assinante (não é erro)

### Itaú PIX
- **Timeout**: 10 segundos
- **Retry**: Implementar retry para generate_pix
- **Erro 401**: Token inválido
- **Status REMOVIDA**: PIX expirou ou foi cancelado

---

## Monitoramento

### Métricas Importantes

1. **Taxa de sucesso por serviço**
   ```sql
   SELECT provider, 
          COUNT(*) as total,
          SUM(CASE WHEN status < 300 THEN 1 ELSE 0 END) as success
   FROM service_logs
   WHERE created_at > NOW() - INTERVAL '24 hours'
   GROUP BY provider;
   ```

2. **Tempo médio de resposta**
   ```sql
   SELECT provider, 
          AVG(EXTRACT(EPOCH FROM (created_at - requested_at))) as avg_seconds
   FROM service_logs
   WHERE created_at > NOW() - INTERVAL '24 hours'
   GROUP BY provider;
   ```

3. **Erros mais comuns**
   ```sql
   SELECT provider, status, COUNT(*) as count
   FROM service_logs
   WHERE status >= 400
     AND created_at > NOW() - INTERVAL '24 hours'
   GROUP BY provider, status
   ORDER BY count DESC;
   ```

---

## Segurança

### Tokens e Credenciais
- ✅ Armazenar em variáveis de ambiente (nunca no código)
- ✅ Usar HTTPS para todas as chamadas
- ✅ Rotacionar tokens periodicamente
- ✅ Logar todas as requisições (sem dados sensíveis)

### Dados Sensíveis
- ❌ Nunca logar CPF completo
- ❌ Nunca logar dados de cartão
- ❌ Nunca logar tokens em texto plano
- ✅ Mascarar dados pessoais nos logs

---

## Suporte

### Feegow
- **Documentação**: https://docs.feegow.com.br
- **Suporte**: suporte@feegow.com

### ClubFlex
- **Contato**: Via acordo comercial
- **Ambiente**: Sandbox disponível

### Itaú PIX
- **Documentação Oficial**: https://developer.itau.com.br/pix
- **Microserviço**: Desenvolvido internamente
