# Guia Simplificado da API ClubFlex

## 📖 O que é a API ClubFlex?

A API ClubFlex é o sistema central que gerencia toda a plataforma de assinaturas de benefícios da ClubFlex. Ela permite que clientes contratem planos, gerenciem suas informações e acessem benefícios de forma segura e eficiente.

---

## 🎯 Principais Funcionalidades

### 1. Gestão de Assinaturas

Permite que novos clientes se cadastrem e contratem planos de benefícios, escolhendo entre diferentes modalidades:

- **Pessoa Física (PF)**: Para indivíduos
- **Pessoa Jurídica (PJ)**: Para empresas

**O que é possível fazer:**

- Criar nova assinatura
- Atualizar dados da assinatura
- Cancelar assinatura
- Consultar status da assinatura
- Adicionar ou remover dependentes

---

### 2. Gerenciamento de Planos

Controla todos os planos de benefícios disponíveis para contratação.

**O que é possível fazer:**

- Visualizar planos disponíveis
- Consultar detalhes de cada plano
- Filtrar planos por tipo (PF/PJ)
- Ver valores e benefícios inclusos

---

### 3. Controle de Pagamentos

Gerencia todo o processo de cobrança e pagamento das assinaturas.

**Formas de pagamento aceitas:**

- Cartão de crédito
- Cartão de débito
- Boleto bancário
- PIX

**O que é possível fazer:**

- Processar pagamentos
- Consultar faturas
- Alterar forma de pagamento
- Visualizar histórico de pagamentos
- Gerar segunda via de boleto

---

### 4. Gestão de Titulares e Dependentes

Administra os dados de todos os usuários da plataforma.

**O que é possível fazer:**

- Cadastrar novos titulares
- Atualizar informações pessoais
- Adicionar dependentes ao plano
- Consultar dados de titulares
- Filtrar titulares por diversos critérios

---

### 5. Controle de Empresas e Corretores

Gerencia a rede de empresas parceiras e corretores que vendem os planos.

**O que é possível fazer:**

- Cadastrar novas empresas
- Registrar corretores
- Vincular empresas a planos específicos
- Consultar vendas por corretor
- Gerar relatórios de performance

---

### 6. Sistema de Benefícios

Controla todos os benefícios disponíveis nos planos contratados.

**O que é possível fazer:**

- Listar benefícios disponíveis
- Consultar detalhes de cada benefício
- Verificar benefícios por plano
- Acessar parcerias (ex: descontos em farmácias)

---

## 👥 Perfis de Usuário

A API trabalha com diferentes níveis de acesso:

| Perfil | Descrição | Permissões |
|--------|-----------|------------|
| **Titular** | Cliente que possui uma assinatura | Visualizar e editar seus próprios dados, gerenciar dependentes, consultar faturas |
| **Atendente** | Equipe de suporte ao cliente | Consultar dados de clientes, auxiliar em problemas, processar solicitações |
| **Corretor** | Vendedor de planos | Criar novas assinaturas, consultar seus clientes, gerar relatórios de vendas |
| **Supervisor** | Supervisor de equipe | Todas as permissões de atendente + gestão de equipe e relatórios avançados |
| **Gerente** | Gestor da operação | Acesso completo a relatórios, configurações e gestão de usuários |
| **Admin** | Administrador do sistema | Acesso total ao sistema, incluindo configurações críticas |

---

## 🔄 Fluxo Principal de Contratação

### Passo 1: Cliente Acessa o Site

O cliente navega pelos planos disponíveis e escolhe o que melhor se adequa às suas necessidades.

### Passo 2: Início do Cadastro

O cliente informa seus dados básicos:

- Nome completo
- CPF/CNPJ
- E-mail
- Telefone
- Data de nascimento

### Passo 3: Escolha do Plano

O cliente seleciona:

- Tipo de plano (Individual, Familiar, Empresarial)
- Empresa vinculada (se aplicável)
- Corretor responsável (se houver)

### Passo 4: Dados Complementares

O cliente fornece:

- **Endereço completo**:
  - Digite apenas o CEP e o sistema preenche automaticamente rua, bairro, cidade e estado
  - Sistema usa ViaCEP (com BrasilAPI como backup para garantir disponibilidade)
- **Dados de dependentes** (se aplicável): nome, CPF, data de nascimento
- **Forma de pagamento preferida**: cartão, boleto ou PIX

---

### Passo 5: Processamento do Pagamento

O sistema:

- Valida os dados do cartão/boleto
- Processa o primeiro pagamento
- Gera a primeira fatura

### Passo 6: Ativação da Assinatura

Após confirmação do pagamento:

- Assinatura é ativada
- Cartão de benefícios é gerado
- E-mail de boas-vindas é enviado
- Cliente recebe acesso à área logada

---

## 💰 Como Funciona a Cobrança

### Cobrança Recorrente

As assinaturas são cobradas mensalmente de forma automática:

1. **Dia da cobrança**: Mesmo dia do mês em que a assinatura foi contratada
2. **Tentativas de cobrança**: O sistema tenta cobrar automaticamente no cartão cadastrado
3. **Falha no pagamento**: Se a cobrança falhar, o sistema:
   - Tenta novamente após alguns dias
   - Envia notificações ao cliente
   - Permite troca de forma de pagamento

### Tipos de Cobrança

**Cartão de Crédito/Débito:**

- Cobrança automática
- Sem necessidade de ação do cliente
- Confirmação imediata

**Boleto Bancário:**

- Boleto enviado por e-mail
- Cliente precisa pagar manualmente
- Compensação em 1-3 dias úteis

**PIX:**

- QR Code gerado automaticamente
- Pagamento instantâneo
- Confirmação em tempo real

---

## 📊 Relatórios e Consultas

### Para Gestores

- Total de assinaturas ativas
- Receita mensal
- Taxa de cancelamento
- Novos clientes por período
- Performance por corretor
- Performance por empresa

### Para Corretores

- Minhas vendas
- Comissões a receber
- Clientes ativos
- Taxa de conversão

### Para Atendentes

- Assinaturas com problemas de pagamento
- Solicitações pendentes
- Tickets de suporte abertos

---

## 🔒 Segurança e Privacidade

### Proteção de Dados

- ✅ Todos os dados pessoais são criptografados
- ✅ Conformidade com a LGPD (Lei Geral de Proteção de Dados)
- ✅ Acesso restrito por nível de permissão
- ✅ Logs de auditoria de todas as operações
- ✅ Proteção anti-bot (Google reCAPTCHA) em formulários
- ✅ Monitoramento 24/7 de atividades suspeitas

### Autenticação

- 🔐 Sistema de login seguro com senha criptografada
- 🎫 Tokens de autenticação com validade limitada
- 📱 Opção de autenticação de dois fatores (MFA via SMS)
- 📧 Recuperação de senha segura por e-mail
- 🤖 Validação anti-bot em cadastros (previne contas falsas)

### Dados Sensíveis

- 💳 Números de cartão são armazenados de forma tokenizada (nunca salvamos o número completo)
- 🔒 CVV nunca é armazenado em nenhuma circunstância
- 🏦 Dados bancários são criptografados de ponta a ponta
- 🔐 Informações médicas (se aplicável) têm proteção adicional
- 📊 Conformidade com PCI-DSS (padrão de segurança da indústria de cartões)

### Sistema de Validação

- ✅ **Google reCAPTCHA**: Verifica se você é humano em formulários importantes
- ✅ **Validação de cartão**: Cielo valida dados do cartão em tempo real
- ✅ **Validação de endereço**: CEP é validado automaticamente
- ✅ **Validação de CPF/CNPJ**: Sistema verifica se documentos são válidos

---

## 🔔 Notificações e Comunicação

O sistema utiliza múltiplos canais para manter os clientes informados:

### 📧 E-mails (via Mailjet)

**Quando você recebe:**

- ✅ Confirmação de cadastro e contrato
- ✅ Aprovação de assinatura
- ✅ Confirmação de pagamento mensal
- ✅ Lembrete de vencimento (3 dias antes)
- ✅ Nota fiscal após pagamento
- ✅ Alerta de falha no pagamento
- ✅ Atualização de dados cadastrais
- ✅ Novos benefícios disponíveis

**Todos os e-mails são enviados automaticamente para o e-mail cadastrado.**

---

### 📱 SMS (via Zenvia)

**Quando você recebe:**

- 🔐 Códigos de verificação para login (se ativado)
- ⚠️ Alertas urgentes de pagamento
- 🔔 Notificações críticas sobre a conta
- ✅ Confirmação de alterações importantes

**SMS é usado apenas para comunicações urgentes e códigos de segurança.**

---

### 🔒 Autenticação em Duas Etapas (MFA)

Para maior segurança, você pode ativar a autenticação em dois fatores:

1. Digite seu e-mail e senha normalmente
2. Receba um código por SMS no celular cadastrado
3. Digite o código para confirmar o acesso
4. Sua conta fica protegida contra acessos não autorizados

**Recomendamos ativar para maior segurança!**

---

## 📱 Integração com Serviços Externos

A API ClubFlex se integra com **10 serviços externos** diferentes para oferecer uma experiência completa e segura. Cada serviço tem uma função específica:

---

### 💳 Serviços de Pagamento (4 serviços)

#### **Vindi** - Pagamentos Recorrentes

**Para que serve:** Gerencia as cobranças mensais automáticas das assinaturas.

**O que faz:**

- Armazena dados de cartões de forma segura (tokenizada)
- Processa cobranças automáticas todo mês
- Envia notificações de pagamentos confirmados ou recusados
- Gerencia todo o ciclo de vida da assinatura

**Benefício para o cliente:** Pagamento automático sem necessidade de lembrar todo mês.

---

#### **eRede** - Processamento de Cartões

**Para que serve:** Processa transações com cartão de crédito e débito em tempo real.

**O que faz:**

- Valida dados do cartão
- Comunica com bancos e operadoras
- Aprova ou recusa transações instantaneamente
- Processa a primeira cobrança da assinatura

**Benefício para o cliente:** Confirmação imediata se o pagamento foi aprovado.

---

#### **BTG Pactual** - PIX

**Para que serve:** Gera cobranças via PIX (QR Code) para pagamento instantâneo.

**O que faz:**

- Cria QR Codes PIX para pagamento
- Confirma pagamentos em tempo real
- Permite configurar PIX automático (débito recorrente)
- Processa pagamentos 24 horas por dia, 7 dias por semana

**Benefício para o cliente:** Pagamento rápido, sem taxas, disponível a qualquer hora.

---

#### **Cielo** - Identificação de Cartões

**Para que serve:** Identifica automaticamente a bandeira e tipo do cartão.

**O que faz:**

- Reconhece se é Visa, Mastercard, Elo, etc.
- Identifica se é crédito ou débito
- Valida os primeiros dígitos do cartão
- Melhora a experiência de cadastro

**Benefício para o cliente:** Sistema preenche automaticamente informações do cartão.

---

### 📧 Serviços de Comunicação (2 serviços)

#### **Mailjet** - Envio de E-mails

**Para que serve:** Envia todos os e-mails transacionais do sistema.

**O que faz:**

- Envia contrato de assinatura por e-mail
- Notifica sobre pagamentos confirmados
- Alerta sobre falhas de pagamento
- Envia lembretes de vencimento
- Entrega notas fiscais por e-mail

**Benefício para o cliente:** Recebe todas as informações importantes na caixa de entrada.

---

#### **Zenvia** - Envio de SMS

**Para que serve:** Envia mensagens de texto para o celular do cliente.

**O que faz:**

- Envia códigos de verificação (autenticação dupla)
- Notifica sobre pagamentos urgentes
- Alerta sobre vencimentos próximos
- Confirma alterações de dados importantes

**Benefício para o cliente:** Recebe alertas importantes mesmo sem internet.

---

### 📄 Serviço de Documentação Fiscal (1 serviço)

#### **eNotas** - Notas Fiscais

**Para que serve:** Emite notas fiscais eletrônicas automaticamente.

**O que faz:**

- Gera nota fiscal após cada pagamento confirmado
- Envia NF por e-mail automaticamente
- Mantém conformidade com legislação fiscal
- Armazena histórico de notas emitidas

**Benefício para o cliente:** Recebe nota fiscal automaticamente, sem precisar solicitar.

---

### 🔍 Serviços de Dados e Validação (3 serviços)

#### **ViaCEP / BrasilAPI** - Consulta de Endereço

**Para que serve:** Preenche automaticamente o endereço ao digitar o CEP.

**O que faz:**

- Busca endereço completo pelo CEP
- Preenche rua, bairro, cidade e estado automaticamente
- Sistema de backup: se ViaCEP falhar, usa BrasilAPI
- Torna o cadastro mais rápido e preciso

**Benefício para o cliente:** Cadastro mais rápido, sem precisar digitar endereço completo.

---

#### **Google reCAPTCHA** - Proteção Anti-Bot

**Para que serve:** Protege formulários contra robôs e ataques automatizados.

**O que faz:**

- Valida se quem está cadastrando é uma pessoa real
- Previne criação de contas falsas
- Protege contra tentativas de fraude
- Bloqueia ataques automatizados

**Benefício para o cliente:** Plataforma mais segura e protegida.

---

#### **Microsoft Teams** - Alertas Internos

**Para que serve:** Notifica a equipe técnica sobre problemas no sistema.

**O que faz:**

- Envia alertas de erros críticos
- Notifica sobre falhas em integrações
- Monitora saúde do sistema
- Permite resposta rápida a incidentes

**Benefício para o cliente:** Problemas são identificados e resolvidos rapidamente.

---

### 📊 Resumo das Integrações

| Serviço | Função | Quando é usado |
|---------|--------|----------------|
| **Vindi** | Pagamentos recorrentes | Cobranças mensais automáticas |
| **eRede** | Gateway de cartão | Primeira compra e transações pontuais |
| **BTG Pactual** | PIX | Quando cliente escolhe pagar via PIX |
| **Cielo** | Validação de cartão | Cadastro de novo cartão |
| **Mailjet** | E-mails | Contratos, confirmações, notificações |
| **Zenvia** | SMS | Códigos de verificação, alertas urgentes |
| **eNotas** | Notas fiscais | Após confirmação de pagamento |
| **ViaCEP/BrasilAPI** | Consulta CEP | Preenchimento de endereço |
| **Google reCAPTCHA** | Anti-bot | Cadastros e formulários |
| **Microsoft Teams** | Alertas técnicos | Monitoramento interno |

---

### 🔒 Segurança nas Integrações

**Todas as integrações são protegidas por:**

- ✅ Conexões criptografadas (HTTPS/TLS)
- ✅ Autenticação via tokens e chaves secretas
- ✅ Validação de dados em todas as requisições
- ✅ Monitoramento 24/7 de disponibilidade
- ✅ Planos de contingência em caso de falha

**Sistema de Redundância:**

- Se um serviço falhar, o sistema tenta alternativas
- Exemplo: ViaCEP indisponível → usa BrasilAPI automaticamente
- Operações críticas têm retry automático
- Nada bloqueia a experiência do cliente

---

## ❓ Perguntas Frequentes

### Como um cliente cancela sua assinatura?

O cliente pode solicitar o cancelamento através da área logada ou entrando em contato com o suporte. O sistema processa o cancelamento e interrompe as cobranças futuras.

### O que acontece se o pagamento falhar?

O sistema tenta cobrar novamente após alguns dias. Durante esse período, você recebe notificações por e-mail e SMS. Se persistir a falha, a assinatura pode ser suspensa temporariamente até a regularização do pagamento.

### É possível mudar de plano?

Sim, o cliente pode fazer upgrade ou downgrade do plano. A diferença de valor é calculada proporcionalmente e ajustada na próxima fatura.

### Como adicionar dependentes?

O titular pode adicionar dependentes através da área logada, informando os dados necessários (nome, CPF, data de nascimento). O valor adicional é cobrado na próxima fatura.

### Os dados estão seguros?

Sim! Utilizamos:

- ✅ Criptografia de ponta a ponta
- ✅ Proteção anti-bot (Google reCAPTCHA)
- ✅ Autenticação de dois fatores (MFA)
- ✅ Conformidade com LGPD e PCI-DSS
- ✅ Dados de cartão tokenizados (nunca armazenamos números completos)

### Como funciona o pagamento por PIX?

Ao escolher PIX, o sistema gera um QR Code automaticamente (via BTG Pactual). Você escaneia com o app do seu banco e confirma o pagamento. A confirmação é instantânea, 24/7.

### Posso configurar PIX automático?

Sim! PIX automático permite débito recorrente mensal direto da sua conta. Basta autorizar uma vez e os pagamentos mensais são processados automaticamente.

### Recebo nota fiscal?

Sim! Após cada pagamento confirmado, o sistema emite automaticamente a nota fiscal (via eNotas) e envia por e-mail. Você não precisa solicitar.

### Por que preciso verificar o reCAPTCHA?

O reCAPTCHA (desafio "Não sou um robô") protege o sistema contra robôs e tentativas de fraude. É uma camada extra de segurança para todos os usuários.

### O que é autenticação de dois fatores (MFA)?

É uma camada extra de segurança: além da senha, você recebe um código por SMS para confirmar que é realmente você tentando acessar. Recomendamos ativar!

### Por que o endereço é preenchido automaticamente?

Quando você digita o CEP, o sistema consulta automaticamente o endereço completo (via ViaCEP ou BrasilAPI). Isso torna o cadastro mais rápido e preciso.

### E se o sistema de CEP estiver fora do ar?

Não se preocupe! Temos sistema de backup automático: se ViaCEP falhar, usamos BrasilAPI. Se ambos estiverem indisponíveis (muito raro), você pode preencher manualmente.

---

## 📞 Suporte

Para questões sobre o funcionamento da API ou dúvidas de negócio, entre em contato com:

- **Equipe Técnica**: Para questões de integração e desenvolvimento
- **Equipe de Produto**: Para questões de funcionalidades e processos de negócio
- **Equipe de Suporte**: Para questões operacionais e atendimento ao cliente

---

**Versão do documento:** 1.0  
**Última atualização:** Novembro 2024
