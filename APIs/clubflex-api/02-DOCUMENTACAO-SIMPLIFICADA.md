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

- Endereço completo
- Dados de dependentes (se aplicável)
- Forma de pagamento preferida

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

- Todos os dados pessoais são criptografados
- Conformidade com a LGPD (Lei Geral de Proteção de Dados)
- Acesso restrito por nível de permissão
- Logs de auditoria de todas as operações

### Autenticação

- Sistema de login seguro com senha
- Tokens de autenticação com validade limitada
- Opção de autenticação de dois fatores (2FA)
- Recuperação de senha por e-mail

### Dados Sensíveis

- Números de cartão são armazenados de forma tokenizada
- CVV nunca é armazenado
- Dados bancários são criptografados
- Informações médicas (se aplicável) têm proteção adicional

---

## 🔔 Notificações e Comunicação

O sistema envia notificações automáticas para:

- Confirmação de cadastro
- Aprovação de assinatura
- Lembrete de vencimento de fatura
- Confirmação de pagamento
- Falha no pagamento
- Atualização de dados cadastrais
- Novos benefícios disponíveis

**Canais de comunicação:**

- E-mail
- SMS (opcional)
- Notificações no app (quando aplicável)

---

## 📱 Integração com Serviços Externos

A API se integra com diversos serviços para oferecer uma experiência completa:

### Vindi (Pagamentos)

- Processamento de cobranças recorrentes
- Gestão de cartões de crédito
- Emissão de boletos
- Geração de PIX

### eRede (Gateway de Pagamento)

- Processamento de transações com cartão
- Validação de cartões
- Prevenção de fraudes

### eNotas (Notas Fiscais)

- Emissão automática de notas fiscais
- Envio por e-mail ao cliente
- Conformidade fiscal

---

## ❓ Perguntas Frequentes

### Como um cliente cancela sua assinatura?

O cliente pode solicitar o cancelamento através da área logada ou entrando em contato com o suporte. O sistema processa o cancelamento e interrompe as cobranças futuras.

### O que acontece se o pagamento falhar?

O sistema tenta cobrar novamente após alguns dias. Se persistir a falha, a assinatura pode ser suspensa temporariamente até a regularização do pagamento.

### É possível mudar de plano?

Sim, o cliente pode fazer upgrade ou downgrade do plano. A diferença de valor é calculada proporcionalmente.

### Como adicionar dependentes?

O titular pode adicionar dependentes através da área logada, informando os dados necessários. O valor adicional é cobrado na próxima fatura.

### Os dados estão seguros?

Sim, utilizamos criptografia de ponta, seguimos as melhores práticas de segurança e estamos em conformidade com a LGPD.

---

## 📞 Suporte

Para questões sobre o funcionamento da API ou dúvidas de negócio, entre em contato com:

- **Equipe Técnica**: Para questões de integração e desenvolvimento
- **Equipe de Produto**: Para questões de funcionalidades e processos de negócio
- **Equipe de Suporte**: Para questões operacionais e atendimento ao cliente

---

**Versão do documento:** 1.0  
**Última atualização:** Novembro 2024
