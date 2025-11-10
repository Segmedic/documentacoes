# Documentação - Sistema de Agendamento Online

Bem-vindo à documentação completa do Sistema de Agendamento Online da Segmedic.

## 📚 Índice de Documentos

### 1. [API Reference](./API-REFERENCE.md) - Documentação Técnica

**Para:** Desenvolvedores e Engenheiros de Software

Documentação técnica completa com todos os endpoints da API, incluindo:

- Especificação de rotas (GET, POST, PATCH, DELETE, PUT)
- Payloads de requisição com exemplos
- Respostas esperadas (sucesso e erro)
- Códigos de status HTTP
- Validações e regras de negócio
- Exemplos práticos de uso

**Quando usar:**

- Implementar integração com a API
- Debugar problemas técnicos
- Desenvolver novas features
- Criar testes automatizados

---

### 2. [Documentação Simplificada](./DOCUMENTACAO-SIMPLIFICADA.md) - Guia para Stakeholders

**Para:** Product Owners, Gestores, Analistas de Negócio

Guia não-técnico explicando o funcionamento do sistema, incluindo:

- Visão geral das funcionalidades
- Fluxos de negócio explicados
- Casos de uso práticos
- Métricas e indicadores
- Glossário de termos
- Checklist de sucesso

**Quando usar:**

- Entender o funcionamento do sistema
- Planejar novos recursos
- Analisar processos de negócio
- Treinar novos membros da equipe
- Apresentar para stakeholders

---

### 3. [Fluxo Visual da API](./FLUXO-API.md) - Diagramas Mermaid

**Para:** Todos os públicos

Representação visual dos fluxos do sistema através de diagramas, incluindo:

- Fluxo completo de agendamento
- Gestão de agendamentos (update/cancel)
- Comunicação por email
- Arquitetura de integração
- Legenda de cores

**Quando usar:**

- Visualizar o journey do usuário
- Entender dependências entre sistemas
- Identificar pontos de melhoria
- Onboarding de novos membros
- Apresentações e reuniões

---

## 🚀 Como Navegar

### Para Desenvolvedores

```
1. Comece com FLUXO-API.md (para entender o big picture)
2. Aprofunde em API-REFERENCE.md (para implementar)
3. Consulte DOCUMENTACAO-SIMPLIFICADA.md (para contexto de negócio)
```

### Para Product Owners / Gestores

```
1. Comece com DOCUMENTACAO-SIMPLIFICADA.md (entender funcionalidades)
2. Visualize FLUXO-API.md (ver fluxos visuais)
3. Consulte API-REFERENCE.md quando necessário (detalhes técnicos)
```

### Para Designers / UX

```
1. Comece com FLUXO-API.md (entender fluxos)
2. Leia DOCUMENTACAO-SIMPLIFICADA.md (casos de uso)
3. Use API-REFERENCE.md para validar possibilidades técnicas
```

---

## 🔍 Busca Rápida

### Preciso saber sobre

#### Agendamento

- **Como funciona:** [Documentação Simplificada - Seção 6](./DOCUMENTACAO-SIMPLIFICADA.md#6-criação-de-agendamentos-)
- **Endpoints:** [API Reference - Agendamentos](./API-REFERENCE.md#-agendamentos)
- **Fluxo visual:** [Fluxo API - Diagrama Geral](./FLUXO-API.md#diagrama-geral-do-sistema)

#### Pacientes

- **Como funciona:** [Documentação Simplificada - Seção 2](./DOCUMENTACAO-SIMPLIFICADA.md#2-cadastro-de-pacientes-)
- **Endpoints:** [API Reference - Pacientes](./API-REFERENCE.md#-pacientes)

#### Convênios

- **Como funciona:** [Documentação Simplificada - Seção 3](./DOCUMENTACAO-SIMPLIFICADA.md#3-consulta-de-convênios-)
- **Endpoints:** [API Reference - Convênios](./API-REFERENCE.md#-convênios)

#### Emails

- **Como funciona:** [Documentação Simplificada - Seção 8](./DOCUMENTACAO-SIMPLIFICADA.md#8-comunicação-por-email-)
- **Endpoints:** [API Reference - E-mails](./API-REFERENCE.md#-e-mails)
- **Fluxo visual:** [Fluxo API - Comunicação Email](./FLUXO-API.md#fluxo-de-comunicação-por-email)

#### Integrações

- **Arquitetura:** [Fluxo API - Integração](./FLUXO-API.md#arquitetura-de-integração)
- **Feegow:** [API Reference - todos endpoints](./API-REFERENCE.md)
- **Nuria:** [API Reference - Elegibilidade](./API-REFERENCE.md#-nuria---elegibilidade)
- **Clubflex:** [API Reference - Clubflex](./API-REFERENCE.md#-clubflex)

---

## 📊 Estrutura dos Documentos

```
docs/
├── README.md                          # Este arquivo (índice)
├── API-REFERENCE.md                   # Documentação técnica completa
├── DOCUMENTACAO-SIMPLIFICADA.md       # Guia para stakeholders
└── FLUXO-API.md                       # Diagramas visuais
```

---

## 🎯 Recursos Adicionais

### Documentação Interativa

Para explorar a API de forma interativa, acesse:

```
http://localhost:4000/docs
```

*(em ambiente de desenvolvimento)*

### Swagger JSON

Para obter a especificação OpenAPI em JSON:

```
http://localhost:4000/swagger.json
```

---

## 🔄 Atualizações

Esta documentação é atualizada regularmente. Última atualização: **10 de novembro de 2025**

### Próximas Atualizações Planejadas

- [ ] Exemplos de código em diferentes linguagens
- [ ] Guia de troubleshooting
- [ ] Documentação de webhooks (quando disponível)
- [ ] Postman Collection
- [ ] Tutoriais passo-a-passo

---

## 📝 Convenções

### Cores nos Diagramas

- 🟢 Verde: Sucesso / Início
- 🔴 Vermelho: Erro / Fim com falha
- 🔵 Azul: Integrações externas
- 🟣 Roxo: Processamento interno
- 🟡 Amarelo: Decisões
- 🟠 Laranja: Atualizações de Lead

### Nomenclatura de Endpoints

- **GET**: Buscar/Listar recursos
- **POST**: Criar novos recursos
- **PATCH**: Atualizar parcialmente
- **PUT**: Atualizar completamente
- **DELETE**: Remover recursos

---

## 🆘 Suporte

### Dúvidas Técnicas

- Verifique a [API Reference](./API-REFERENCE.md)
- Consulte o [Swagger UI](http://localhost:4000/docs)
- Entre em contato com o time de desenvolvimento

### Dúvidas de Negócio

- Consulte a [Documentação Simplificada](./DOCUMENTACAO-SIMPLIFICADA.md)
- Analise os [Fluxos Visuais](./FLUXO-API.md)
- Entre em contato com o Product Owner

---

## 🤝 Contribuindo

Se você identificar algum erro ou tiver sugestões de melhoria para esta documentação:

1. Crie uma issue descrevendo o problema ou sugestão
2. Ou envie um pull request com as correções
3. Mantenha o padrão e estrutura existentes

---

## 📋 Checklist de Uso

### Antes de Integrar com a API

- [ ] Li a documentação simplificada
- [ ] Entendi os fluxos visuais
- [ ] Revisei os endpoints necessários
- [ ] Tenho as credenciais de acesso
- [ ] Conheço o ambiente de testes

### Ao Implementar uma Feature

- [ ] Consultei a API Reference
- [ ] Entendi o fluxo completo
- [ ] Implementei tratamento de erros
- [ ] Testei casos de sucesso e falha
- [ ] Documentei as alterações

---

**Versão da Documentação:** 1.0.0  
**Última Atualização:** 10 de novembro de 2025  
**Mantido por:** Equipe de Desenvolvimento Segmedic
