# Fluxo Visual - General Proposal Message

## 📊 Diagrama do Fluxo de Automação

```mermaid
flowchart TD
    Start([Início: Cron Schedule<br/>Diário às 12h UTC]) --> ValidatePipeline{Pipeline ID<br/>válido?}
    
    ValidatePipeline -->|Não| LogError[Log: Pipeline Inválido<br/>ou Chave não Encontrada]
    LogError --> EndError([Encerra com Erro])
    
    ValidatePipeline -->|Sim| GetKey[Obtém Chave da Campanha:<br/>KEY_CAMPAIGN_PROPOSTAS_GERAIS]
    
    GetKey --> InitRepo[Inicializa<br/>RdCrmRcTeamRepository<br/>Token: CRM_TOKEN_RC_TEAM]
    
    InitRepo --> SetFilters[Define Filtros de Busca:<br/>- created_at_period: true<br/>- start_date: Hoje<br/>- end_date: Hoje<br/>- Pipeline: Propostas Gerais]
    
    SetFilters --> LoopPages{Tem mais<br/>páginas?}
    
    LoopPages -->|Sim| FetchDeals[Busca Deals do Dia Atual<br/>200 por página<br/>Max 50 páginas]
    
    FetchDeals --> ProcessContacts[Processa Contatos:<br/>- Normaliza Telefone 0XX<br/>- Extrai Nome<br/>- Remove Duplicatas]
    
    ProcessContacts --> Wait[Aguarda 2s<br/>Rate Limiting]
    
    Wait --> LoopPages
    
    LoopPages -->|Não| CheckResults{Encontrou<br/>Leads?}
    
    CheckResults -->|Não| LogNoLeads[Log: Nenhum Lead Encontrado]
    LogNoLeads --> EndNoLeads([Encerra])
    
    CheckResults -->|Sim| ValidateContacts{Lista de<br/>contatos válida?}
    
    ValidateContacts -->|Não| ThrowValidation[Lança SendMessageError:<br/>Lista vazia]
    ThrowValidation --> EndValidation([Encerra com Erro])
    
    ValidateContacts -->|Sim| FormatPayload[Formata Payload Escallo:<br/>- expiraLista: 60min<br/>- cancelarPendentes: false<br/>- chaveExterna: key]
    
    FormatPayload --> SendToWhatsapp[Envia POST para<br/>Escallo WhatsApp API<br/>/campanha/texto/lista]
    
    SendToWhatsapp --> CheckWhatsapp{Envio<br/>bem-sucedido?}
    
    CheckWhatsapp -->|Não| ThrowError[Lança SendMessageError:<br/>Falha no envio]
    ThrowError --> EndWhatsappError([Encerra com Erro])
    
    CheckWhatsapp -->|Sim| LogSuccess[Log: Mensagens<br/>Enviadas com Sucesso]
    LogSuccess --> EndSuccess([Execução Concluída])
    
    style Start fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style EndError fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style EndNoLeads fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style EndValidation fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style EndWhatsappError fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style EndSuccess fill:#d4d4d4,stroke:#666,stroke-width:3px,color:#000
    
    style ValidatePipeline fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style LoopPages fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style CheckResults fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style ValidateContacts fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style CheckWhatsapp fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    
    style GetKey fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style InitRepo fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style SetFilters fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style FetchDeals fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style ProcessContacts fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Wait fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style FormatPayload fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style SendToWhatsapp fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
```

## 🔄 Fluxo Simplificado de Dados

```mermaid
graph LR
    A[AWS Lambda<br/>Cron Diário] --> B[RD Station CRM<br/>RC Team]
    B --> C[Use Case:<br/>GetAllDeals<br/>Filtro: Hoje]
    C --> D[Use Case:<br/>SendMessageWhatsapp]
    D --> E[Escallo<br/>WhatsApp API]
    E --> F[Cliente<br/>WhatsApp]
    
    style A fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style B fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style C fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style D fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style E fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
    style F fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 🏗️ Arquitetura de Camadas

```mermaid
flowchart TB
    subgraph Lambda["🔸 AWS Lambda Function"]
        Handler[campaign-send-message<br/>handler<br/>compartilhado]
    end
    
    subgraph UseCases["🔸 Use Cases Layer"]
        GetDeals[GetAllDealsUseCase]
        SendMessage[SendMessageWhatsapp]
    end
    
    subgraph Repos["🔸 Repositories Layer"]
        RdCrm[RdCrmRcTeamRepository]
        Escallo[EscalloWhatsappRepository]
    end
    
    subgraph External["🔸 APIs Externas"]
        RdApi[RD Station CRM API<br/>RC Team Token]
        EscalloApi[Escallo WhatsApp API]
    end
    
    Handler --> GetDeals
    Handler --> SendMessage
    GetDeals --> RdCrm
    SendMessage --> Escallo
    RdCrm --> RdApi
    Escallo --> EscalloApi
    
    style Lambda fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style UseCases fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Repos fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style External fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## ⏱️ Timeline de Execução Diária

```mermaid
gantt
    title Execução Diária - General Proposal Message
    dateFormat HH:mm
    axisFormat %H:%M
    
    section Trigger
    Cron Schedule (12h UTC)        :milestone, 12:00, 0m
    
    section Busca CRM
    Validação Pipeline             :active, 12:00, 1m
    Busca Deals do Dia             :active, 12:01, 5m
    Normalização Contatos          :active, 12:06, 2m
    
    section Envio WhatsApp
    Preparação Payload             :active, 12:08, 1m
    Envio para Escallo             :active, 12:09, 3m
    
    section Finalização
    Logs e Conclusão               :milestone, 12:12, 0m
```

## 📋 Comparação: Três Automações WhatsApp

```mermaid
graph TB
    subgraph General["General Proposal Message"]
        G1[Pipeline: Propostas Gerais]
        G2[ID: 647e4cdab66552000db15fd4]
        G3[Chave: 3b685d99-e284...]
        G4[Público: Diverso]
    end
    
    subgraph Laboratory["Laboratory Proposal Message"]
        L1[Pipeline: Propostas Lab]
        L2[ID: 65391eec1e66020013a4a869]
        L3[Chave: b6f7016d-dfe1...]
        L4[Público: Laboratórios]
    end
    
    subgraph Shared["Configuração Compartilhada"]
        S1[Handler: campaign-send-message]
        S2[Frequência: Diária 12h UTC]
        S3[Filtro: Deals de Hoje]
        S4[Expiração: 60min]
        S5[Token: RC_TEAM]
        S6[cancelarPendentes: false]
    end
    
    General --> Shared
    Laboratory --> Shared
    
    style General fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Laboratory fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Shared fill:#d4d4d4,stroke:#666,stroke-width:3px,color:#000
```

## 🎯 Comparação Detalhada de Pipelines

```mermaid
flowchart LR
    subgraph Input["Entrada de Dados"]
        I1[Pipeline ID<br/>diferente]
        I2[Chave Campanha<br/>diferente]
    end
    
    subgraph Process["Processamento Idêntico"]
        P1[Mesmo Código]
        P2[Mesmas Validações]
        P3[Mesmo Repositório]
        P4[Mesma API]
    end
    
    subgraph Output["Saída"]
        O1[Mensagens<br/>WhatsApp]
        O2[Públicos<br/>diferentes]
    end
    
    Input --> Process
    Process --> Output
    
    style Input fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style Process fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Output fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 📊 Fluxo de Filtros e Processamento

```mermaid
flowchart LR
    A[Todos os Deals<br/>do Pipeline<br/>Propostas Gerais] --> B{Filtro:<br/>created_at_period}
    B --> C[Apenas Deals<br/>de Hoje]
    C --> D{Filtro:<br/>win = null}
    D --> E[Apenas Não-Ganhos]
    E --> F[Paginação:<br/>200/página]
    F --> G[Deduplicação:<br/>por Telefone]
    G --> H[Lista Final<br/>para WhatsApp]
    
    style A fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style B fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style C fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style D fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style E fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style F fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style G fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style H fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 🔑 Informações do Pipeline

```mermaid
graph TD
    Pipeline[Pipeline: Propostas Gerais<br/>ID: 647e4cdab66552000db15fd4]
    
    Pipeline --> Key[Chave Campanha:<br/>3b685d99-e284-45fd-b739-1d3207fb5c8e]
    Pipeline --> Schedule[Schedule:<br/>Diário às 12h UTC]
    Pipeline --> Token[Token:<br/>CRM_TOKEN_RC_TEAM]
    Pipeline --> Expire[Expiração:<br/>60 minutos]
    Pipeline --> Cancel[Cancelar Pendentes:<br/>false]
    Pipeline --> Type[Tipo:<br/>Propostas Gerais/Diversas]
    
    style Pipeline fill:#e1e1e1,stroke:#666,stroke-width:3px,color:#000
    style Key fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Schedule fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Token fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Expire fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Cancel fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Type fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
```

## 🎯 Visão Geral do Ecossistema

```mermaid
flowchart TB
    subgraph Dialer["Automações de Discagem"]
        D1[pipe-1-60]
        D2[pipe-61-180]
        D3[pipe-181-365]
        D4[pipe-366]
    end
    
    subgraph WhatsApp["Automações WhatsApp"]
        W1[laboratory-proposal-message]
        W2[general-proposal-message]
    end
    
    subgraph Systems["Sistemas Externos"]
        CRM[RD Station CRM]
        ESC[Escallo]
    end
    
    Dialer -->|Telefone| ESC
    WhatsApp -->|WhatsApp| ESC
    Dialer -->|Busca| CRM
    WhatsApp -->|Busca| CRM
    
    style Dialer fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style WhatsApp fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Systems fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 📊 Tabela Comparativa - General vs Laboratory

| Característica | General Proposal | Laboratory Proposal |
|----------------|------------------|---------------------|
| **Pipeline ID** | `647e4cdab66552000db15fd4` | `65391eec1e66020013a4a869` |
| **Chave Campanha** | `3b685d99-e284-45fd-b739-1d3207fb5c8e` | `b6f7016d-dfe1-4acf-a4ba-853bf900c264` |
| **Tipo de Proposta** | Gerais/Diversas | Laboratoriais |
| **Frequência** | Diária às 12h UTC | Diária às 12h UTC |
| **Canal** | WhatsApp | WhatsApp |
| **Filtro Temporal** | Deals de hoje | Deals de hoje |
| **Expiração Lista** | 60 minutos | 60 minutos |
| **Cancelar Pendentes** | false | false |
| **Token CRM** | RC_TEAM | RC_TEAM |
| **Handler** | campaign-send-message | campaign-send-message |
| **Max Páginas** | 50 | 50 |
| **Deals/Página** | 200 | 200 |
| **Rate Limit** | 2s entre requests | 2s entre requests |

## 🎯 Pontos-Chave da Automação

| Característica | Valor |
|----------------|-------|
| **Frequência** | Diária (todos os dias) |
| **Horário** | 12h UTC (09h BRT) |
| **Canal** | WhatsApp |
| **Filtro Temporal** | Apenas deals criados hoje |
| **Expiração Lista** | 60 minutos |
| **Cancelar Pendentes** | Não |
| **Token CRM** | RC_TEAM |
| **Max Páginas** | 50 |
| **Deals/Página** | 200 |
| **Rate Limit** | 2s entre requests |
| **Código Compartilhado** | Sim (com laboratory-proposal) |
| **Diferenciação** | Pipeline ID + Chave |

## 📝 Notas

- **Cores neutras**: Paleta em tons de cinza para visualização no GitHub
- **Fluxo detalhado**: Mostra todas as etapas, validações e decisões
- **Comparação visual**: Diferenças e semelhanças entre pipelines
- **Arquitetura clara**: Separação de responsabilidades entre camadas
- **Ecossistema completo**: Visão geral de todas as automações
- **Código reutilizável**: Mesma implementação, configurações diferentes
