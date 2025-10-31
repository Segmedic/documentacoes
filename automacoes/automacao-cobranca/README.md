# Documentação das Automações

Esta pasta contém a documentação técnica e visual de todas as automações do sistema de cobrança.

## 📚 Índice de Automações

### 1. [Tickets Renewal](./ticketsRennewal/)

Automação responsável por identificar e processar assinaturas com pagamento via carnê (tickets) que possuem faturas em aberto.

- **Frequência**: Toda segunda-feira às 3h UTC
- **Handler Schedule**: `src/schedules/ticketsRennewal.ts`
- **Handler Worker**: `src/workers/ticketsRennewal.ts`
- **Documentação**: [README](./ticketsRennewal/README.md) | [Guia Rápido](./ticketsRennewal/GUIA-RAPIDO.md)
- **Fluxo Visual**: [Diagrama Mermaid](./ticketsRennewal/fluxo.md)

### 2. [Annual Renewal](./annual/)

Automação responsável por identificar e processar planos anuais que precisam ser renovados, criando deals no CRM para ação comercial proativa.

- **Frequência**: Todo dia 1º de cada mês às 3h UTC
- **Handler Schedule**: `src/schedules/annualRennewal.ts`
- **Handler Worker**: `src/workers/annualRennewal.ts`
- **Documentação**: [README](./annual/README.md) | [Guia Rápido](./annual/GUIA-RAPIDO.md)
- **Fluxo Visual**: [Diagrama Mermaid](./annual/fluxo.md)

### 3. [Sync](./sync/) 🔥

**Automação principal** do sistema. Sincroniza diariamente todas as assinaturas inadimplentes com o CRM, organizando-as automaticamente em stages por tempo de atraso.

- **Frequência**: Diariamente às 3h UTC
- **Handler Schedule**: `src/schedules/sync.ts`
- **Handler Worker**: `src/workers/sync.ts`
- **Documentação**: [README](./sync/README.md) | [Guia Rápido](./sync/GUIA-RAPIDO.md)
- **Fluxo Visual**: [Diagrama Mermaid](./sync/fluxo.md)

### 4. [Update](./update/) 🔄

**Automação de limpeza** que sincroniza o CRM com o banco de dados. Remove deals de clientes regularizados e realiza fechamento mensal obrigatório.

- **Frequência**: Diariamente às 22h UTC
- **Handler Schedule**: `src/schedules/update.ts`
- **Handler Worker**: `src/workers/update.ts`
- **Documentação**: [README](./update/README.md) | [Guia Rápido](./update/GUIA-RAPIDO.md)
- **Fluxo Visual**: [Diagrama Mermaid](./update/fluxo.md)

---

## � Documentação Consolidada

Para uma visão integrada e comparativa de todas as automações, consulte:

- 🔍 **[Comparativo das Automações](./COMPARATIVO.md)** - Análise detalhada das 4 automações, incluindo:
  - Timeline diária de execução
  - Tabela comparativa completa
  - Matriz de critérios
  - Relacionamento e complementaridade
  - Análise de volume e performance
  - Tratamento de conflitos
  - Troubleshooting integrado

---

## �📋 Estrutura de Documentação

Cada automação possui sua própria pasta com três arquivos principais:

### README.md

Documentação técnica completa contendo:

- 📋 Visão geral e objetivo
- ⚙️ Configuração (serverless, agendamento)
- 🔄 Fluxo de execução detalhado
- 📊 Critérios de seleção
- 🏗️ Estrutura de dados
- 🔍 Campos customizados
- 🚨 Tratamento de erros
- 📈 Métricas e monitoramento
- 🔗 Dependências
- 🛠️ Manutenção

### GUIA-RAPIDO.md

Guia de referência rápida contendo:

- 📋 Resumo executivo
- 🎯 Critérios de processamento (tabelas)
- 🔄 Fluxo simplificado
- 📊 Exemplos práticos
- ⚙️ Configuração essencial
- 🔍 Monitoramento e métricas
- 🆘 Troubleshooting rápido

### fluxo.md

Diagrama visual em Mermaid mostrando:

- Fluxo completo da automação
- Decisões e condições
- Integrações externas
- Processamento de dados
- Tratamento de erros

---

## 🏗️ Arquitetura Geral

Todas as automações seguem o padrão:

``` bash
┌─────────────────┐
│  CloudWatch     │
│  Cron Event     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Lambda         │
│  Schedule       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  AWS SQS        │
│  Queue          │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Lambda         │
│  Worker         │
└─────────────────┘
```

## 🔧 Tecnologias Utilizadas

- **Serverless Framework**: Orquestração e deploy
- **AWS Lambda**: Processamento serverless
- **AWS SQS**: Filas de mensagens
- **AWS CloudWatch**: Agendamento e logs
- **TypeScript**: Linguagem de programação
- **MySQL**: Banco de dados (DNA)
- **RD Station CRM**: Sistema de CRM

## 📦 Automações Disponíveis

| Automação | Frequência | Objetivo | Criticidade |
|-----------|------------|----------|-------------|
| **sync** 🔥 | Diariamente às 3h | Sincronização geral de inadimplências | 🔴 Crítica |
| **ticketsRennewal** | Segundas às 3h | Renovação de carnês com faturas abertas | 🟠 Alta |
| **annual** | 1º dia do mês às 3h | Renovação proativa de planos anuais | 🟡 Média |
| **update** 🔄 | Diariamente às 22h | Limpeza de deals regularizados no CRM | 🟢 Suporte |

---

## 🚀 Como Usar Esta Documentação

1. **Para desenvolvedores novos**: Comece lendo o README de cada automação
2. **Para entender o fluxo**: Visualize o diagrama Mermaid no GitHub
3. **Para manutenção**: Consulte as seções de tratamento de erros e monitoramento
4. **Para deploy**: Veja as configurações do serverless.yml

## 📝 Convenções

- Todos os diagramas usam cores neutras compatíveis com GitHub
- Horários estão em UTC
- Logs seguem padrão estruturado
- Nomes de arquivos seguem kebab-case

## 🤝 Contribuindo

Ao adicionar ou modificar automações:

1. Crie uma pasta com o nome da automação
2. Adicione README.md com documentação técnica
3. Adicione GUIA-RAPIDO.md com resumo executivo
4. Crie fluxo.md com diagrama Mermaid
5. Atualize este índice

---

**Última atualização**: 31 de outubro de 2025
