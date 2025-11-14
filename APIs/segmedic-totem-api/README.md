# 📚 Documentação da API Segmedic Totem

Bem-vindo à documentação completa da API Segmedic Totem. Esta pasta contém toda a documentação necessária para entender, usar e manter a API.

---

## 📑 Arquivos Disponíveis

| Arquivo | Descrição | Audiência |
|---------|-----------|-----------|
| **[README.md](./README.md)** | Este arquivo - índice geral | Todos |
| **[API_TECHNICAL_DOCUMENTATION.md](./API_TECHNICAL_DOCUMENTATION.md)** | Documentação técnica completa | Desenvolvedores |
| **[API_SIMPLE_GUIDE.md](./API_SIMPLE_GUIDE.md)** | Guia em linguagem simples | Stakeholders |
| **[API_FLOW_DIAGRAMS.md](./API_FLOW_DIAGRAMS.md)** | Diagramas visuais Mermaid | Dev/Arquitetos |
| **[EXTERNAL_SERVICES.md](./EXTERNAL_SERVICES.md)** | Integrações externas | Dev/DevOps |
| **[ROUTES_REFERENCE.md](./ROUTES_REFERENCE.md)** | Referência rápida de rotas | Desenvolvedores |

---

## 📚 Índice de Documentos

### 1. [Documentação Técnica da API](./API_TECHNICAL_DOCUMENTATION.md)
**Audiência:** Desenvolvedores, Engenheiros de Software

Documentação técnica completa com:
- Todos os endpoints da API
- Parâmetros de request e response
- Exemplos de payloads
- Códigos de status HTTP
- Autenticação e headers
- Variáveis de ambiente

**Quando usar:**
- Integrar com a API
- Desenvolver novos recursos
- Debugar problemas
- Entender estrutura de dados

---

### 2. [Guia Simplificado da API](./API_SIMPLE_GUIDE.md)
**Audiência:** Stakeholders, Product Owners, Gerentes de Projeto

Documentação em linguagem simples com:
- O que é e para que serve o sistema
- Principais funcionalidades
- Fluxos de uso do totem
- Benefícios para clínica e pacientes
- Perguntas frequentes
- Glossário de termos

**Quando usar:**
- Apresentar o sistema para não-técnicos
- Treinamento de equipe
- Entender regras de negócio
- Comunicação com stakeholders

---

### 3. [Diagramas de Fluxo](./API_FLOW_DIAGRAMS.md)
**Audiência:** Desenvolvedores, Arquitetos, Analistas de Sistema

Fluxos visuais em Mermaid:
- Visão geral do sistema
- Fluxo de agendamento com convênio
- Fluxo de agendamento particular com PIX
- Fluxo de check-in
- Fluxo de cancelamento
- Fluxo de pagamento com cartão
- Arquitetura de integração
- Ciclo de vida de agendamento
- Modelo de dados
- Performance e cache
- Tratamento de erros

**Quando usar:**
- Entender fluxo completo
- Planejar mudanças
- Onboarding de novos desenvolvedores
- Apresentações técnicas

---

### 4. [Serviços Externos](./EXTERNAL_SERVICES.md)
**Audiência:** Desenvolvedores, DevOps, Arquitetos

Documentação detalhada das integrações:
- **Feegow API**: Sistema de gestão médica
- **ClubFlex API**: Sistema de convênio
- **Itaú PIX API**: Pagamentos PIX

Para cada serviço:
- Propósito e uso
- Configuração e autenticação
- Endpoints utilizados
- Exemplos de request/response
- Estratégia de cache
- Logging e auditoria
- Pontos de atenção
- Tratamento de erros

**Quando usar:**
- Configurar ambiente
- Debugar integrações
- Entender dependências externas
- Monitorar performance

---

## 🚀 Guia Rápido

### Para Desenvolvedores

1. **Primeira vez no projeto?**
   - Leia: [Guia Simplificado](./API_SIMPLE_GUIDE.md)
   - Veja: [Diagramas de Fluxo](./API_FLOW_DIAGRAMS.md)
   - Consulte: [Documentação Técnica](./API_TECHNICAL_DOCUMENTATION.md)

2. **Vai integrar com a API?**
   - Leia: [Documentação Técnica](./API_TECHNICAL_DOCUMENTATION.md)
   - Configure: [Serviços Externos](./EXTERNAL_SERVICES.md)

3. **Precisa entender um fluxo específico?**
   - Veja: [Diagramas de Fluxo](./API_FLOW_DIAGRAMS.md)

---

### Para Stakeholders

1. **Quer entender o que é o sistema?**
   - Leia: [Guia Simplificado](./API_SIMPLE_GUIDE.md)

2. **Precisa visualizar como funciona?**
   - Veja: [Diagramas de Fluxo](./API_FLOW_DIAGRAMS.md) (seção Visão Geral)

3. **Quer saber sobre integrações?**
   - Leia: [Serviços Externos](./EXTERNAL_SERVICES.md) (seção Visão Geral)

---

## 🏗️ Arquitetura Resumida

```
┌──────────────┐
│    Totem     │ (Cliente)
└──────┬───────┘
       │ HTTP/JSON
       ▼
┌──────────────┐
│  API Rails   │ (Backend)
└──────┬───────┘
       │
       ├─► PostgreSQL (Banco de Dados)
       ├─► Redis (Cache)
       │
       └─► APIs Externas:
           ├─► Feegow (Gestão Médica)
           ├─► ClubFlex (Convênio)
           └─► Itaú PIX (Pagamentos)
```

---

## 🔑 Principais Recursos

### Agendamentos
- ✅ Criar agendamento
- ✅ Listar agendamentos
- ✅ Cancelar agendamento
- ✅ Fazer check-in
- ✅ Gerar senha de atendimento
- ✅ Consultar horários disponíveis

### Pacientes
- ✅ Cadastrar paciente
- ✅ Buscar por CPF
- ✅ Atualizar dados

### Pagamentos
- ✅ Pagamento com PIX
- ✅ Pagamento com cartão de crédito
- ✅ Gerar comprovantes
- ✅ Consultar status

### Convênios
- ✅ Verificar elegibilidade
- ✅ Validar ClubFlex
- ✅ Listar convênios aceitos

---

## 🛠️ Tecnologias

- **Backend**: Ruby on Rails 7
- **Banco de Dados**: PostgreSQL
- **Cache**: Redis
- **Jobs**: Sidekiq
- **Autenticação**: JWT (Devise)
- **APIs Externas**: Faraday (HTTP client)
- **Documentação**: Markdown + Mermaid

---

## 📊 Monitoramento

A API registra todas as ações em tabelas de log:

- **service_logs**: Chamadas para APIs externas
- **payment_logs**: Transações financeiras
- **user_session_screens**: Navegação no totem
- **user_session_actions**: Ações do usuário

---

## 🔒 Segurança

- ✅ Autenticação JWT
- ✅ HTTPS obrigatório
- ✅ Tokens em variáveis de ambiente
- ✅ Logs completos de auditoria
- ✅ Conformidade com LGPD
- ✅ Dados sensíveis mascarados

---

## 📝 Padrões de Código

### Nomenclatura
- **Controllers**: `PascalCase` + `Controller` (ex: `AppointmentsController`)
- **Services**: `PascalCase` + `Service` (ex: `FeegowService`)
- **Models**: `PascalCase` singular (ex: `User`, `PaymentLog`)
- **Métodos**: `snake_case` (ex: `create_appointment`)

### Estrutura de Response
```json
{
  "body": {
    "content": { ... },
    "message": "..."
  },
  "code": 200
}
```

### Tratamento de Erros
```ruby
begin
  # código
rescue => e
  { body: { message: e.message }, code: 400 }
end
```

---

## 🚦 Ambientes

### Desenvolvimento
```
Base URL: http://localhost:3000
Database: segmedic_totem_api_development
Redis: localhost:6379/0
```

### Produção
```
Base URL: https://api.segmedic.com.br
Database: PostgreSQL (RDS)
Redis: Redis (ElastiCache)
```

---

## 📞 Suporte

### Técnico
- **E-mail**: dev@segmedic.com.br
- **Slack**: #segmedic-totem-dev

### Negócio
- **E-mail**: suporte@segmedic.com.br
- **Telefone**: (11) 1234-5678

---

## 🔄 Atualizações

Esta documentação é mantida pela equipe de desenvolvimento e deve ser atualizada sempre que:
- Novos endpoints forem criados
- Fluxos forem alterados
- Integrações forem adicionadas/modificadas
- Regras de negócio mudarem

**Última atualização**: 14 de novembro de 2025

---

## 📖 Como Contribuir com a Documentação

1. **Documentação Técnica**: Atualizar ao adicionar/modificar endpoints
2. **Guia Simplificado**: Atualizar ao mudar regras de negócio
3. **Diagramas**: Atualizar ao modificar fluxos ou arquitetura
4. **Serviços Externos**: Atualizar ao adicionar/modificar integrações

### Formato
- Use Markdown para documentação
- Use Mermaid para diagramas
- Inclua exemplos reais (sem dados sensíveis)
- Mantenha linguagem clara e objetiva

---

## 📌 Links Úteis

- [Documentação do Rails](https://guides.rubyonrails.org/)
- [Mermaid Live Editor](https://mermaid.live/) (para editar diagramas)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [Redis Docs](https://redis.io/docs/)

---

**Desenvolvido com ❤️ pela equipe Segmedic**
