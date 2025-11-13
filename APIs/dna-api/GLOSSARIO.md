# 📖 Glossário - API ClubFlex

Este glossário define os principais termos técnicos e de negócio utilizados na API ClubFlex e em sua documentação.

---

## 🔤 Termos em Ordem Alfabética

### A

**API (Application Programming Interface)**
Interface que permite a comunicação entre diferentes sistemas de software. A API ClubFlex permite que aplicações front-end, mobile e sistemas externos interajam com o backend.

**Assinatura (Subscription)**
Contrato de plano recorrente entre o cliente (titular) e a ClubFlex. Define o plano contratado, forma de pagamento, data de início e status.

**Atendente (Attendant)**
Perfil de usuário da equipe de suporte ao cliente, com permissões para consultar e auxiliar clientes.

**Autenticação (Authentication)**
Processo de verificar a identidade de um usuário através de credenciais (email e senha). Gera um token JWT para acesso à API.

**Autorização (Authorization)**
Processo de verificar se um usuário autenticado tem permissão para realizar determinada ação ou acessar determinado recurso.

---

### B

**Backoffice**
Sistema administrativo interno utilizado por atendentes, supervisores e gerentes para gerenciar assinaturas, clientes e operações.

**Benefício (Benefit)**
Vantagem oferecida aos assinantes dos planos, como descontos em farmácias, academias, consultas médicas, etc.

**BIN (Bank Identification Number)**
Primeiros 6 dígitos de um cartão que identificam o banco emissor e a bandeira.

**Boleto Bancário**
Forma de pagamento em que o cliente recebe um código de barras para pagar em bancos, lotéricas ou internet banking.

**BrasilAPI**
API pública brasileira utilizada como fallback para consulta de CEP quando ViaCEP está indisponível.

**Broker → Ver Corretor**

**BTG Pactual**
Banco utilizado para geração de cobranças PIX e PIX automático (débito recorrente).

---

### C

**Callback → Ver Webhook**

**Cartão de Benefícios (Benefit Card)**
Cartão físico ou virtual gerado para o titular e dependentes, usado para acessar os benefícios do plano.

**CEP (Código de Endereçamento Postal)**
Código de 8 dígitos que identifica localidades e logradouros no Brasil.

**Cielo**
Adquirente de cartões utilizado para consulta de BIN (identificação de bandeira e banco emissor).

**Circuit Breaker**
Padrão de design que previne falhas em cascata ao detectar serviços externos indisponíveis e interromper temporariamente chamadas a eles.

**CNPJ (Cadastro Nacional da Pessoa Jurídica)**
Identificador único de empresas no Brasil (14 dígitos).

**Cobrança Recorrente**
Sistema automático de cobranças mensais em assinaturas, gerenciado pela Vindi.

**Company → Ver Empresa**

**Corretor (Broker)**
Pessoa ou empresa parceira que vende planos da ClubFlex. Recebe comissões pelas vendas realizadas.

**CPF (Cadastro de Pessoas Físicas)**
Identificador único de pessoas físicas no Brasil (11 dígitos).

**CRUD (Create, Read, Update, Delete)**
Operações básicas em bancos de dados: Criar, Ler, Atualizar e Deletar.

**CVV (Card Verification Value)**
Código de segurança de 3 ou 4 dígitos no verso do cartão de crédito/débito.

---

### D

**Dependente (Dependent)**
Pessoa vinculada a um titular de assinatura (cônjuge, filho, etc.) que também tem acesso aos benefícios do plano.

**Downgrade**
Alteração de um plano para outro de menor valor ou benefícios.

---

### E

**eNotas**
Serviço externo utilizado para emissão automática de notas fiscais eletrônicas (NF-e e NFS-e).

**Endpoint**
URL específica da API que executa uma função, como `/holder/{id}` para buscar dados de um titular.

**eRede**
Gateway de pagamento utilizado para processar transações com cartão de crédito e débito.

**Empresa (Company)**
Empresa parceira ou filial da ClubFlex que pode estar vinculada a planos e corretores específicos.

**Estorno (Refund)**
Devolução de pagamento já realizado, em casos de cancelamento ou erro.

---

### F

**Fallback**
Sistema de contingência que automaticamente utiliza um serviço alternativo quando o principal falha. Exemplo: ViaCEP ↔ BrasilAPI.

**Fatura (Invoice/Bill)**
Cobrança mensal gerada para uma assinatura, contendo o valor a ser pago e data de vencimento.

**Fluxo (Flow)**
Sequência de etapas que um processo percorre do início ao fim.

---

### G

**Gateway de Pagamento**
Serviço intermediário que processa pagamentos com cartão, conectando a API aos bancos e operadoras.

**Google reCAPTCHA**
Serviço do Google utilizado para proteção anti-bot em formulários, validando se o usuário é humano.

---

### H

**Hash**
Função criptográfica unidirecional usada para armazenar senhas de forma segura.

**Holder → Ver Titular**

**HTTP (HyperText Transfer Protocol)**
Protocolo de comunicação usado na web para transferência de dados entre cliente e servidor.

---

### I

**Invoice → Ver Fatura**

---

### J

**JSON (JavaScript Object Notation)**
Formato leve de intercâmbio de dados, fácil para humanos lerem e máquinas processarem.

**JWT (JSON Web Token)**
Padrão de token de autenticação que contém informações do usuário codificadas e assinadas digitalmente.

---

### L

**LGPD (Lei Geral de Proteção de Dados)**
Lei brasileira que regula o tratamento de dados pessoais, garantindo privacidade e segurança.

---

### M

**Mailjet**
Serviço SMTP utilizado para envio de e-mails transacionais (contratos, notificações, confirmações).

**Mensalidade**
Valor pago mensalmente pelo titular para manter a assinatura ativa.

**MFA (Multi-Factor Authentication)**
Autenticação de múltiplos fatores, geralmente usando senha + código SMS.

**Microsoft Teams**
Plataforma de colaboração da Microsoft, utilizada para envio de alertas e notificações para a equipe técnica via webhooks.

---

### N

**NF-e (Nota Fiscal Eletrônica)**
Documento fiscal digital emitido para pessoa jurídica.

**NFS-e (Nota Fiscal de Serviço Eletrônica)**
Documento fiscal digital emitido para prestação de serviços.

---

### P

### P

**Payload**
Dados enviados em uma requisição HTTP (corpo da requisição).

**PCI-DSS (Payment Card Industry Data Security Standard)**
Padrão de segurança para processamento de dados de cartão de crédito.

**PIX**
Sistema de pagamento instantâneo brasileiro, operado pelo Banco Central. Permite transferências 24/7 em poucos segundos.

**PIX Automático**
Modalidade de débito recorrente via PIX, onde o cliente autoriza cobranças automáticas mensais.

---

**Pessoa Física (PF)**
Indivíduo/pessoa natural, identificado por CPF.

**Pessoa Jurídica (PJ)**
Empresa, identificada por CNPJ.

**PIX**
Sistema de pagamento instantâneo brasileiro, disponível 24/7.

**Plan → Ver Plano**

**Plano (Plan)**
Tipo de assinatura oferecida pela ClubFlex, com benefícios e valores específicos (ex: Plano Básico, Premium).

**Pré-assinatura**
Primeira etapa de criação de assinatura, onde dados básicos do cliente são coletados.

**Profile → Ver Perfil**

---

### R

**Rate Limiting**
Limitação da quantidade de requisições que um cliente pode fazer à API em determinado período.

**Refund → Ver Estorno**

**REST (Representational State Transfer)**
Estilo arquitetural para APIs web que usa métodos HTTP (GET, POST, PUT, DELETE).

**Retry**
Tentativa automática de reexecutar uma operação que falhou.

---

### S

**SEFAZ (Secretaria da Fazenda)**
Órgão governamental responsável por autorizar e validar notas fiscais eletrônicas.

**SMS (Short Message Service)**
Mensagens de texto curtas enviadas para celulares, utilizadas para MFA e notificações.

**SMTP (Simple Mail Transfer Protocol)**
Protocolo padrão para envio de e-mails pela internet.

**Status Code → Ver Código de Status HTTP**

**Subscription → Ver Assinatura**

**Supervisor**
Perfil de usuário com permissões elevadas para gestão de equipe e relatórios avançados.

---

### T

**TID (Transaction ID)**
Identificador único de uma transação de pagamento no gateway eRede.

**Titular (Holder)**
Cliente que contrata e é responsável por uma assinatura. Pode adicionar dependentes.

**Token**
Código criptografado gerado após autenticação, usado para identificar o usuário em requisições subsequentes.

**Tokenização**
Processo de substituir dados sensíveis (como número de cartão) por um identificador único não sensível.

---

### U

**Upgrade**
Alteração de um plano para outro de maior valor ou benefícios.

---

### V

### V

**ViaCEP**
API pública brasileira para consulta de endereços através do CEP. Serviço principal, com BrasilAPI como fallback.

**Vindi**
Plataforma de gestão de pagamentos recorrentes, responsável por gerenciar assinaturas, cobranças automáticas e webhooks.

---

**Virtual (Empresa)**
Empresa fictícia criada no sistema para fins de organização, sem existência real.

---

### W

**Webhook**
Notificação automática enviada por um serviço externo (como Vindi, BTG Pactual) para informar sobre eventos (pagamento confirmado, PIX recebido, assinatura cancelada, etc.).

---

### Z

**Zenvia**
Plataforma de comunicação utilizada para envio de SMS (códigos de verificação MFA, notificações urgentes).

---

## 📊 Termos por Categoria

### 💳 Pagamentos

- Assinatura Recorrente
- BIN (Bank Identification Number)
- Boleto Bancário
- BTG Pactual
- Cartão de Crédito
- Cartão de Débito
- CVV
- Estorno (Refund)
- Fatura (Invoice)
- Gateway de Pagamento
- Mensalidade
- PIX
- PIX Automático
- TID
- Tokenização
- Vindi

---

### 👥 Usuários e Perfis

- Atendente (Attendant)
- Corretor (Broker)
- Dependente (Dependent)
- Gerente (Manager)
- Supervisor
- Titular (Holder)

### 🏢 Entidades de Negócio

- Assinatura (Subscription)
- Benefício (Benefit)
- Cartão de Benefícios
- Empresa (Company)
- Fatura (Invoice)
- Plano (Plan)

### 🔐 Segurança

- Autenticação
- Autorização
- Google reCAPTCHA
- Hash
- JWT
- LGPD
- MFA (Multi-Factor Authentication)
- PCI-DSS
- Token
- Tokenização

### 🔌 Integrações

- API
- BrasilAPI
- BTG Pactual
- Callback
- CEP
- Cielo
- eNotas
- eRede
- Mailjet
- Microsoft Teams
- REST
- SMS
- SMTP
- ViaCEP
- Vindi
- Webhook
- Zenvia

### 💻 Técnico

- Backoffice
- BIN
- Circuit Breaker
- CRUD
- Endpoint
- Fallback
- Fluxo
- HTTP
- JSON
- Payload
- PIX
- PIX Automático
- Rate Limiting
- Retry
- Status Code
- TID

---

## 🔄 Sinônimos e Traduções

| Português | Inglês | Observação |
|-----------|--------|------------|
| Titular | Holder | Pessoa que possui a assinatura |
| Assinatura | Subscription | Contrato de plano recorrente |
| Corretor | Broker | Vendedor de planos |
| Fatura | Invoice / Bill | Cobrança mensal |
| Plano | Plan | Tipo de assinatura |
| Dependente | Dependent | Pessoa vinculada ao titular |
| Empresa | Company | Empresa parceira |
| Estorno | Refund | Devolução de pagamento |
| Atendente | Attendant | Equipe de suporte |

---

## ❓ Dúvidas Comuns

### Qual a diferença entre Autenticação e Autorização?

- **Autenticação:** Verifica QUEM você é (login com email e senha)
- **Autorização:** Verifica O QUE você pode fazer (permissões baseadas em perfil)

### Qual a diferença entre Titular e Dependente?

- **Titular:** Pessoa que contrata e paga pela assinatura
- **Dependente:** Pessoa vinculada ao titular que também usufrui dos benefícios

### Qual a diferença entre Plano e Assinatura?

- **Plano:** Produto/serviço oferecido (ex: Plano Premium)
- **Assinatura:** Contrato específico de um cliente com um plano

### Qual a diferença entre Vindi e eRede?

- **Vindi:** Gerencia assinaturas recorrentes e cobranças automáticas
- **eRede:** Processa transações pontuais com cartão em tempo real

### O que é um Webhook?

É uma notificação automática enviada por um sistema externo para informar sobre eventos. Exemplo: Vindi envia webhook quando um pagamento é confirmado.

---

## 📚 Siglas e Acrônimos

| Sigla | Significado | Descrição |
|-------|-------------|-----------|
| API | Application Programming Interface | Interface de programação de aplicações |
| CNPJ | Cadastro Nacional da Pessoa Jurídica | Identificador de empresas |
| CPF | Cadastro de Pessoas Físicas | Identificador de pessoas |
| CRUD | Create, Read, Update, Delete | Operações básicas de banco de dados |
| CVV | Card Verification Value | Código de segurança do cartão |
| HTTP | HyperText Transfer Protocol | Protocolo de comunicação web |
| JSON | JavaScript Object Notation | Formato de dados |
| JWT | JSON Web Token | Token de autenticação |
| LGPD | Lei Geral de Proteção de Dados | Lei de privacidade brasileira |
| NF-e | Nota Fiscal Eletrônica | Documento fiscal digital |
| NFS-e | Nota Fiscal de Serviço Eletrônica | Documento fiscal de serviços |
| PCI-DSS | Payment Card Industry Data Security Standard | Padrão de segurança de cartões |
| PF | Pessoa Física | Indivíduo |
| PIX | Sistema de Pagamentos Instantâneos | Pagamento instantâneo brasileiro |
| PJ | Pessoa Jurídica | Empresa |
| REST | Representational State Transfer | Estilo arquitetural de API |
| SEFAZ | Secretaria da Fazenda | Órgão fiscal governamental |
| TID | Transaction ID | Identificador de transação |

---

## 🔗 Links Relacionados

- [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md)
- [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md)
- [Fluxos Visuais](./03-FLUXOS-VISUAIS.md)
- [Serviços Externos](./04-SERVICOS-EXTERNOS.md)
- [README Principal](./README.md)
- [Índice](./INDICE.md)

---

**Última atualização:** Novembro 2024
