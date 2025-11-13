# 🗂️ Índice da Documentação API ClubFlex

## 📑 Visão Geral

Esta documentação está organizada em 4 documentos principais, cada um focado em um aspecto específico da API ClubFlex.

---

## 📚 Documentos Disponíveis

### 🔧 [01 - Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md)

<details>
<summary><b>📋 Conteúdo Completo</b></summary>

#### Informações Gerais

- Versão e tecnologias
- URL base
- Autenticação JWT
- Perfis de usuário

#### Endpoints Documentados

**🔐 Autenticação e Usuários**

- `POST /user/remember/passwd` - Recuperação de senha

**🏢 Empresas (Company)**

- `GET /company` - Listar empresas
- `POST /company` - Criar empresa
- `PUT /company` - Atualizar empresa
- `GET /company/plan/{planId}` - Empresas por plano
- `GET /company/broker/{brokerId}` - Empresas por corretor

**📋 Planos (Plan)**

- `GET /plan/list/avaliable/site` - Planos disponíveis no site
- `GET /plan/list/active` - Todos planos ativos
- `GET /plan/list/active/{type}` - Planos ativos por tipo
- `GET /plan/list/all` - Todos os planos
- `GET /plan/list/all/{type}` - Todos planos por tipo
- `GET /plan/{planId}` - Detalhes do plano

**👥 Titulares (Holder)**

- `POST /holder/filter` - Filtrar titulares ativos
- `POST /holder/filter/inactive` - Filtrar titulares inativos
- `POST /pj/filter` - Filtrar titulares PJ ativos
- `GET /holder/pj/company` - Empresas para filtro PJ
- `POST /pj/filter/inactive` - Filtrar titulares PJ inativos
- `POST /holder/parceria/farma` - Titulares parceria farmácia
- `POST /dependent/filter` - Filtrar dependentes
- `GET /holder/{id}` - Detalhes do titular
- `POST /holder` - Atualizar titular

**📝 Assinaturas (Subscription)**

- `POST /subscription` - Criar pré-assinatura
- `PUT /subscription` - Completar assinatura

**🏦 Corretores (Broker)**

- `GET /broker` - Listar corretores
- `POST /broker` - Criar corretor
- `PUT /broker` - Atualizar corretor

**💳 Faturas (Invoice)**

- `GET /invoice` - Buscar faturas
- `PUT /invoice/{invoiceId}/payment-type` - Alterar tipo de pagamento

**👨‍👩‍👧‍👦 Dependentes (Dependent)**

- `GET /dependent` - Listar dependentes

**🎁 Benefícios (Benefit)**

- `GET /benefit` - Listar benefícios

**📞 Callbacks**

- `POST /callbacks/vindi` - Webhook Vindi

#### Recursos Adicionais

- Códigos de status HTTP
- Estrutura de resposta padrão
- Sistema de paginação
- Enumerações importantes
- Rate limiting
- Versionamento

</details>

---

### 📖 [02 - Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md)

<details>
<summary><b>📋 Conteúdo Completo</b></summary>

#### Seções Principais

**📖 O que é a API ClubFlex**

- Visão geral do sistema

**🎯 Principais Funcionalidades**

1. Gestão de Assinaturas (PF e PJ)
2. Gerenciamento de Planos
3. Controle de Pagamentos
4. Gestão de Titulares e Dependentes
5. Controle de Empresas e Corretores
6. Sistema de Benefícios

**👥 Perfis de Usuário**

- Titular
- Atendente
- Corretor
- Supervisor
- Gerente
- Admin

**🔄 Fluxo Principal de Contratação**

- Passo a passo completo
- Do acesso ao site até ativação

**💰 Como Funciona a Cobrança**

- Cobrança recorrente
- Tipos de cobrança (Cartão, Boleto, PIX)
- Tratamento de falhas

**📊 Relatórios e Consultas**

- Para gestores
- Para corretores
- Para atendentes

**🔒 Segurança e Privacidade**

- Proteção de dados
- Autenticação
- Dados sensíveis
- Conformidade LGPD

**🔔 Notificações e Comunicação**

- Tipos de notificação
- Canais de comunicação

**📱 Integração com Serviços Externos**

- Vindi (Pagamentos)
- eRede (Gateway)
- eNotas (Notas Fiscais)

**❓ Perguntas Frequentes**

- Cancelamento
- Falha de pagamento
- Mudança de plano
- Adição de dependentes
- Segurança de dados

</details>

---

### 🎨 [03 - Fluxos Visuais](./03-FLUXOS-VISUAIS.md)

<details>
<summary><b>📋 Conteúdo Completo</b></summary>

#### Diagramas Disponíveis

**📊 Visão Geral da Arquitetura**

- Diagrama completo do sistema
- Clientes, API, banco de dados e integrações

**🔐 Fluxo de Autenticação**

- Login e geração de token
- Validação em requisições subsequentes
- Tratamento de erros

**📝 Fluxo de Criação de Assinatura**

- Etapa 1: Pré-assinatura
- Etapa 2: Completar assinatura
- Processamento de pagamento
- Ativação

**💳 Fluxo de Processamento de Pagamento Recorrente**

- Execução diária
- Processamento por forma de pagamento
- Webhooks de confirmação

**👥 Fluxo de Gestão de Dependentes**

- Adicionar dependente
- Remover dependente
- Visualizar dependentes

**🔄 Fluxo de Tratamento de Falha de Pagamento**

- Diagrama de estados
- Tentativas de cobrança
- Suspensão e cancelamento
- Regularização

**📊 Fluxo de Consulta e Relatórios**

- Diferentes perfis de usuário
- Endpoints de consulta
- Processamento e formatação
- Exportação (Excel, PDF, gráficos)

**🔄 Fluxo de Integração com Vindi (Webhook)**

- Eventos recebidos
- Processamento assíncrono
- Atualização de status

**🛠️ Fluxo de Tratamento de Erros**

- Validação de token
- Validação de permissões
- Validação de dados
- Regras de negócio
- Erros de banco
- Erros de APIs externas

**Legenda de Cores:**

- Diagramas em tons neutros de cinza para melhor visualização no GitHub

</details>

---

### 🔌 [04 - Serviços Externos](./04-SERVICOS-EXTERNOS.md)

<details>
<summary><b>📋 Conteúdo Completo</b></summary>

#### 📋 Visão Geral

- Resumo executivo das 10 integrações
- Mapa de integrações (diagrama Mermaid)
- Tabela comparativa de criticidade

#### 💳 Pagamentos e Transações (4 serviços)

**💳 Vindi - Plataforma de Pagamentos Recorrentes**

**Endpoints Documentados:**

1. **Gestão de Clientes (Customers)**
   - `POST /customers` - Criar cliente
   - `GET /customers/{id}` - Buscar cliente
   - `PUT /customers/{id}` - Atualizar cliente

2. **Meios de Pagamento (Payment Profiles)**
   - `POST /payment_profiles` - Registrar cartão
   - `DELETE /payment_profiles/{id}` - Remover cartão

3. **Assinaturas (Subscriptions)**
   - `POST /subscriptions` - Criar assinatura
   - `GET /subscriptions/{id}` - Consultar assinatura
   - `PUT /subscriptions/{id}` - Atualizar assinatura
   - `DELETE /subscriptions/{id}` - Cancelar assinatura

4. **Faturas (Bills)**
   - `POST /bills` - Criar fatura
   - `GET /bills/{id}` - Consultar fatura
   - `POST /bills/{id}/charges` - Nova tentativa de cobrança
   - `POST /bills/{id}/refund` - Estornar fatura

5. **Webhooks**
   - Configuração e eventos recebidos

**💰 eRede - Gateway de Pagamento**

**Endpoints Documentados:**

1. **Criar Transação**
   - `POST /v1/transactions` - Processar transação

2. **Consultar Transação**
   - `GET /v1/transactions/{tid}` - Consultar status

3. **Cancelar Transação**
   - `POST /v1/transactions/{tid}/refunds` - Estornar

**🏦 BTG Pactual - PIX Automático**

**Endpoints Documentados:**

1. **Criar Cobrança PIX**
   - `POST /pix` - Gerar QR Code PIX

2. **Consultar Cobrança PIX**
   - `GET /pix/{chargeId}` - Verificar status

3. **Criar Autorização PIX Automático**
   - `POST /pix/automatic/authorize` - Débito recorrente

4. **Agendar PIX Automático**
   - `POST /pix/automatic/schedule` - Agendar cobrança

5. **Cancelar Autorização**
   - `DELETE /pix/automatic/{authorizationId}` - Cancelar autorização

**💳 Cielo - Consulta BIN de Cartão**

**Endpoint Documentado:**

- `GET /cardBin/{firstSixDigits}` - Identificar cartão

#### 📧 Comunicação (2 serviços)

**📧 Mailjet - Envio de E-mails**

- Configuração SMTP
- Templates de e-mail (contratos, notificações)
- Envio assíncrono via fila

**📱 Zenvia - Envio de SMS**

- Autenticação via username/password
- Códigos MFA
- Notificações urgentes

#### 📄 Documentação Fiscal (1 serviço)

**📄 eNotas - Emissão de Notas Fiscais**

**Endpoints Documentados:**

1. **Gestão de Clientes**
   - `POST /empresas/{id}/clientes` - Cadastrar cliente

2. **Emissão de Notas Fiscais**
   - `POST /empresas/{id}/nfes` - Emitir NF

3. **Consulta de Notas Fiscais**
   - `GET /empresas/{id}/nfes/{nfeId}` - Consultar NF

4. **Cancelamento de Notas Fiscais**
   - `POST /empresas/{id}/nfes/{nfeId}/cancelamento` - Cancelar NF

#### 🔍 Dados e Validação (3 serviços)

**🌐 ViaCEP / BrasilAPI - Consulta de CEP**

- Sistema de fallback automático
- ViaCEP (principal) ↔ BrasilAPI (backup)
- Fallback de 60 minutos
- APIs públicas sem autenticação

**🔐 Google reCAPTCHA - Proteção Anti-Bot**

- Validação de formulários
- Modo de escape para testes
- Integração com Google API

**🔔 Microsoft Teams - Alertas e Notificações**

- Webhooks para canal do Teams
- Alertas de erros críticos
- Notificações de monitoramento

#### 📊 Recursos Adicionais

- Resumo comparativo das 10 integrações (tabela de criticidade)
- Boas práticas de integração (retry, security, performance, monitoring)
- Estratégias de contingência (para cada serviço individualmente)
- Variáveis de configuração (todas as properties necessárias)
- Métricas e SLA (disponibilidade e tempo de resposta)
- Troubleshooting (guia de resolução de problemas)
- Links para documentação oficial e suporte

</details>

---

## 🎯 Guia Rápido por Perfil

### 👨‍💻 Desenvolvedor Backend

1. ✅ [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md) - Todos os endpoints
2. ✅ [Serviços Externos](./04-SERVICOS-EXTERNOS.md) - Integrações
3. ✅ [Fluxos Visuais](./03-FLUXOS-VISUAIS.md) - Arquitetura

### 👨‍💼 Product Manager

1. ✅ [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md) - Funcionalidades
2. ✅ [Fluxos Visuais](./03-FLUXOS-VISUAIS.md) - Processos

### 🎨 UX/UI Designer

1. ✅ [Fluxos Visuais](./03-FLUXOS-VISUAIS.md) - Jornadas do usuário
2. ✅ [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md) - Funcionalidades

### 📊 Business Analyst

1. ✅ [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md) - Regras de negócio
2. ✅ [Fluxos Visuais](./03-FLUXOS-VISUAIS.md) - Processos visuais

### 🔧 DevOps / SRE

1. ✅ [Serviços Externos](./04-SERVICOS-EXTERNOS.md) - Dependências externas
2. ✅ [Fluxos Visuais](./03-FLUXOS-VISUAIS.md) - Arquitetura
3. ✅ [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md) - Endpoints e autenticação

### 📱 Desenvolvedor Frontend/Mobile

1. ✅ [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md) - Endpoints da API
2. ✅ [Fluxos Visuais](./03-FLUXOS-VISUAIS.md) - Fluxos de integração

### 👔 Stakeholder/Executivo

1. ✅ [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md) - Visão geral
2. ✅ [Fluxos Visuais](./03-FLUXOS-VISUAIS.md) - Diagramas de alto nível

---

## 🔍 Busca Rápida

### Por Funcionalidade

**Autenticação e Segurança**

- [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md#autenticação) - Como autenticar
- [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md#segurança-e-privacidade) - Segurança explicada
- [Fluxos Visuais](./03-FLUXOS-VISUAIS.md#fluxo-de-autenticação) - Diagrama de autenticação

**Criar Assinatura**

- [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md#assinaturas-subscription) - Endpoints
- [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md#fluxo-principal-de-contratação) - Processo explicado
- [Fluxos Visuais](./03-FLUXOS-VISUAIS.md#fluxo-de-criação-de-assinatura) - Diagrama do fluxo

**Pagamentos**

- [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md#faturas-invoice) - Endpoints de fatura
- [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md#como-funciona-a-cobrança) - Como funciona
- [Fluxos Visuais](./03-FLUXOS-VISUAIS.md#fluxo-de-processamento-de-pagamento-recorrente) - Diagrama de cobrança
- [Serviços Externos](./04-SERVICOS-EXTERNOS.md#vindi---plataforma-de-pagamentos-recorrentes) - Integração Vindi

**Gestão de Dependentes**

- [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md#dependentes-dependent) - Endpoints
- [Fluxos Visuais](./03-FLUXOS-VISUAIS.md#fluxo-de-gestão-de-dependentes) - Diagrama do processo

**Integrações Externas**

- [Serviços Externos](./04-SERVICOS-EXTERNOS.md) - Todas as 10 integrações
- [Fluxos Visuais - Arquitetura](./03-FLUXOS-VISUAIS.md#visão-geral-da-arquitetura) - Diagrama de integrações

**Serviços Específicos**

- [Vindi - Pagamentos Recorrentes](./04-SERVICOS-EXTERNOS.md#vindi---plataforma-de-pagamentos-recorrentes)
- [eRede - Gateway de Cartão](./04-SERVICOS-EXTERNOS.md#erede---gateway-de-pagamento)
- [BTG Pactual - PIX](./04-SERVICOS-EXTERNOS.md#btg-pactual---pix-automático)
- [eNotas - Notas Fiscais](./04-SERVICOS-EXTERNOS.md#enotas---emissão-de-notas-fiscais)
- [Mailjet - E-mails](./04-SERVICOS-EXTERNOS.md#mailjet---envio-de-e-mails)
- [Zenvia - SMS](./04-SERVICOS-EXTERNOS.md#zenvia---envio-de-sms)
- [ViaCEP/BrasilAPI - CEP](./04-SERVICOS-EXTERNOS.md#viacep--brasilapi---consulta-de-cep)
- [Google reCAPTCHA - Anti-bot](./04-SERVICOS-EXTERNOS.md#google-recaptcha---proteção-anti-bot)
- [Cielo - Consulta BIN](./04-SERVICOS-EXTERNOS.md#cielo---consulta-bin-de-cartão)
- [Microsoft Teams - Alertas](./04-SERVICOS-EXTERNOS.md#microsoft-teams---alertas-e-notificações)

---

## 📌 Atalhos Úteis

| Preciso de... | Veja aqui |
|---------------|-----------|
| 🔍 Listar todos os endpoints | [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md) |
| 📊 Ver diagrama de arquitetura | [Fluxos Visuais - Arquitetura](./03-FLUXOS-VISUAIS.md#visão-geral-da-arquitetura) |
| 💡 Entender como funciona a cobrança | [Documentação Simplificada - Cobrança](./02-DOCUMENTACAO-SIMPLIFICADA.md#como-funciona-a-cobrança) |
| 🔌 Ver integrações com Vindi | [Serviços Externos - Vindi](./04-SERVICOS-EXTERNOS.md#vindi---plataforma-de-pagamentos-recorrentes) |
| 👥 Entender perfis de usuário | [Documentação Simplificada - Perfis](./02-DOCUMENTACAO-SIMPLIFICADA.md#perfis-de-usuário) |
| 🔄 Ver fluxo de falha de pagamento | [Fluxos Visuais - Falha Pagamento](./03-FLUXOS-VISUAIS.md#fluxo-de-tratamento-de-falha-de-pagamento) |
| 🛠️ Ver tratamento de erros | [Fluxos Visuais - Erros](./03-FLUXOS-VISUAIS.md#fluxo-de-tratamento-de-erros) |
| 📄 Integração com eNotas | [Serviços Externos - eNotas](./04-SERVICOS-EXTERNOS.md#enotas---emissão-de-notas-fiscais) |

---

## 📝 Convenções da Documentação

### Ícones Utilizados

- 🔐 Autenticação/Segurança
- 🏢 Empresas
- 📋 Planos
- 👥 Titulares/Usuários
- 📝 Assinaturas
- 🏦 Corretores
- 💳 Faturas/Pagamentos
- 👨‍👩‍👧‍👦 Dependentes
- 🎁 Benefícios
- 📞 Webhooks/Callbacks
- 🔌 Integrações
- 📊 Diagramas/Fluxos
- ⚠️ Atenção
- ✅ Sucesso
- ❌ Erro
- 💡 Dica

### Cores nos Diagramas

- Diagramas em tons neutros de cinza para melhor visualização no GitHub
- Sem cores para manter aspecto profissional e neutro

---

## 🔄 Atualizações

**Última atualização:** Novembro 2024  
**Versão:** 1.0

---

## 💬 Feedback

Esta documentação foi útil? Encontrou algum erro ou tem sugestões de melhoria?  
Entre em contato com a equipe de desenvolvimento!

---

**[◀️ Voltar para README Principal](./README.md)**
