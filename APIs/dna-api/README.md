# 📚 Documentação da API ClubFlex

Bem-vindo à documentação completa da API ClubFlex! Este diretório contém toda a documentação necessária para entender, integrar e trabalhar com a API.

---

## 📂 Estrutura da Documentação

> 💡 **Dica:** Para uma navegação completa com índice detalhado, consulte o [**ÍNDICE**](./INDICE.md)

### 1. [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md)

**Público-alvo:** Desenvolvedores e equipe técnica

Documentação completa e detalhada de todos os endpoints da API, incluindo:

- Especificação de rotas e métodos HTTP
- Payloads de requisição e resposta
- Parâmetros obrigatórios e opcionais
- Códigos de status HTTP
- Exemplos práticos de uso
- Informações sobre autenticação e autorização

**📖 Use quando precisar:**

- Implementar integrações com a API
- Consultar estrutura de dados
- Entender permissões e perfis de acesso
- Debugar problemas de integração

---

### 2. [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md)

**Público-alvo:** Stakeholders, gerentes de produto, equipe de negócios

Guia de fácil compreensão sobre o funcionamento da API, incluindo:

- Descrição das principais funcionalidades
- Perfis de usuário e suas permissões
- Fluxos de negócio explicados de forma simples
- Processos de cobrança e pagamento
- Segurança e privacidade de dados
- Perguntas frequentes

**📖 Use quando precisar:**

- Entender o que a API faz de forma geral
- Explicar funcionalidades para não-técnicos
- Compreender processos de negócio
- Avaliar capacidades da plataforma

---

### 3. [Fluxos Visuais](./03-FLUXOS-VISUAIS.md)

**Público-alvo:** Todos (diagramas facilitam o entendimento)

Diagramas visuais em Mermaid representando:

- Arquitetura geral da API
- Fluxo de autenticação
- Processo de criação de assinatura
- Pagamentos recorrentes e gestão de falhas
- Gestão de dependentes
- Consultas e relatórios
- Integração com webhooks
- Tratamento de erros

**📖 Use quando precisar:**

- Visualizar a arquitetura do sistema
- Entender fluxos complexos de forma visual
- Apresentar o sistema para novos membros da equipe
- Documentar processos e procedimentos

---

### 4. [Serviços Externos](./04-SERVICOS-EXTERNOS.md)

**Público-alvo:** Desenvolvedores e equipe de infraestrutura

Documentação completa das integrações com **10 APIs externas**:

**💳 Pagamentos e Transações (4 serviços)**

- **Vindi:** Gestão de pagamentos recorrentes
- **eRede:** Gateway de pagamento com cartão
- **BTG Pactual:** PIX e PIX automático
- **Cielo:** Consulta de BIN de cartões

**📧 Comunicação (2 serviços)**

- **Mailjet:** Envio de e-mails transacionais (SMTP)
- **Zenvia:** Envio de SMS (MFA e notificações)

**📄 Documentação Fiscal (1 serviço)**

- **eNotas:** Emissão de notas fiscais eletrônicas

**🔍 Dados e Validação (3 serviços)**

- **ViaCEP/BrasilAPI:** Consulta de CEP (com fallback automático)
- **Google reCAPTCHA:** Proteção anti-bot
- **Microsoft Teams:** Alertas para equipe técnica

Para cada serviço, inclui:

- Propósito da integração
- Endpoints utilizados
- Exemplos de requisições e respostas
- Fluxos de integração
- Tratamento de erros e contingência
- Planos de contingência específicos
- Variáveis de configuração
- Links para documentação oficial

**📖 Use quando precisar:**

- Entender dependências externas
- Implementar ou modificar integrações
- Debugar problemas com serviços externos
- Planejar estratégias de contingência
- Configurar variáveis de ambiente

---

## 🚀 Começando

### Para Desenvolvedores

1. Comece pela [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md) para entender os endpoints
2. Consulte os [Fluxos Visuais](./03-FLUXOS-VISUAIS.md) para entender a arquitetura
3. Veja [Serviços Externos](./04-SERVICOS-EXTERNOS.md) para entender as integrações

### Para Stakeholders

1. Leia a [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md) para entender as capacidades
2. Consulte os [Fluxos Visuais](./03-FLUXOS-VISUAIS.md) para visualizar os processos

### Para Equipe de Produto

1. Comece pela [Documentação Simplificada](./02-DOCUMENTACAO-SIMPLIFICADA.md)
2. Analise os [Fluxos Visuais](./03-FLUXOS-VISUAIS.md) para entender jornadas do usuário
3. Consulte a [Documentação Técnica](./01-DOCUMENTACAO-TECNICA.md) quando precisar de detalhes

---

## 🔑 Principais Conceitos

### Entidades Principais

- **Holder (Titular):** Cliente que possui uma assinatura
- **Subscription (Assinatura):** Contrato de plano recorrente
- **Plan (Plano):** Tipo de plano de benefícios oferecido
- **Invoice (Fatura):** Cobrança mensal gerada
- **Company (Empresa):** Empresa parceira/filial
- **Broker (Corretor):** Vendedor de planos
- **Dependent (Dependente):** Pessoa vinculada ao titular

### Fluxo Básico

1. Cliente escolhe um plano no site
2. Preenche dados pessoais (cria pré-assinatura)
3. Completa cadastro com dados de pagamento
4. Sistema processa primeira cobrança
5. Assinatura é ativada e cartão de benefícios gerado
6. Cobranças mensais acontecem automaticamente

---

## 🛠️ Tecnologias

- **Framework:** Spring Boot 2.6.7
- **Linguagem:** Java 17
- **Banco de Dados:** MySQL
- **Autenticação:** JWT (JSON Web Tokens)
- **Documentação:** Swagger/OpenAPI
- **Integrações Principais:**
  - Vindi (pagamentos recorrentes)
  - eRede (gateway de cartão)
  - BTG Pactual (PIX)
  - eNotas (notas fiscais)
  - Mailjet (e-mails)
  - Zenvia (SMS)
  - ViaCEP/BrasilAPI (CEP)
  - Google reCAPTCHA (anti-bot)
  - Cielo (consulta BIN)
  - Microsoft Teams (alertas)

---

## 📊 Diagramas Rápidos

### Arquitetura Simplificada

```
┌─────────────┐
│   Clientes  │
│ (Web/Mobile)│
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ API ClubFlex│ ◄─── JWT Auth
└──────┬──────┘
       │
       ├───────► [MySQL Database]
       │
       ├───────► [Vindi - Pagamentos Recorrentes]
       │
       ├───────► [eRede - Gateway de Cartão]
       │
       ├───────► [BTG Pactual - PIX]
       │
       ├───────► [eNotas - Notas Fiscais]
       │
       ├───────► [Mailjet - E-mails]
       │
       ├───────► [Zenvia - SMS]
       │
       ├───────► [ViaCEP/BrasilAPI - CEP]
       │
       ├───────► [Google reCAPTCHA - Anti-bot]
       │
       ├───────► [Cielo - Consulta BIN]
       │
       └───────► [Microsoft Teams - Alertas]
```

### Perfis de Acesso

```
ADMIN
  └─► MANAGER
       └─► SUPERVISOR
            └─► ATTENDANT
                 └─► HOLDER

BROKER (paralelo aos internos)
```

---

## 🔒 Segurança

### Autenticação

Todos os endpoints protegidos requerem um token JWT no header:

```
Authorization: Bearer {seu-token-aqui}
```

### Níveis de Acesso

Cada endpoint define quais perfis podem acessá-lo através da anotação `@RequireAuthentication`.

### Dados Sensíveis

- Números de cartão são tokenizados (nunca armazenados completos)
- CVV nunca é armazenado
- Senhas são hasheadas com bcrypt
- Conformidade com LGPD

---

## 📈 Métricas e Monitoramento

A API possui integração com New Relic para monitoramento de:

- Performance de endpoints
- Taxa de erro
- Tempo de resposta
- Uso de recursos

---

## 🐛 Reportando Problemas

Ao reportar um problema, inclua:

1. Endpoint afetado
2. Payload enviado (sem dados sensíveis)
3. Resposta recebida
4. Logs relevantes
5. Passos para reproduzir

---

## 📞 Contato

- **Equipe Técnica:** Para questões de desenvolvimento
- **Equipe de Produto:** Para questões de funcionalidades
- **Suporte:** Para questões operacionais

---

## 🔄 Versionamento

**Versão Atual:** 0.0.1-SNAPSHOT  
**Última atualização:** Novembro 2024

---

## 📖 Documentos Adicionais

- **[📑 ÍNDICE COMPLETO](./INDICE.md)** - Navegação detalhada por tópicos e perfis
- **[📖 GLOSSÁRIO](./GLOSSARIO.md)** - Definição de todos os termos técnicos e de negócio
- **[💡 EXEMPLOS PRÁTICOS](./EXEMPLOS-PRATICOS.md)** - Código de exemplo e casos de uso reais

---

## 📝 Notas de Versão

### v0.0.1-SNAPSHOT

- Estrutura inicial da API
- Gestão completa de assinaturas
- Integração com Vindi, eRede e eNotas
- Sistema de autenticação JWT
- Gestão de planos, empresas e corretores

---

## 🎯 Próximos Passos

1. ✅ Ler a documentação relevante ao seu perfil
2. ✅ Explorar os diagramas de fluxo
3. ✅ Testar endpoints em ambiente de desenvolvimento
4. ✅ Implementar sua integração

---

**Dica:** Todos os diagramas Mermaid podem ser visualizados diretamente no GitHub. Basta abrir os arquivos .md na interface web do GitHub para ver os diagramas renderizados.

---

💡 **Esta documentação está em constante evolução.** Se você encontrar algo que pode ser melhorado, sinta-se à vontade para contribuir!
