# Fluxo Visual - Laboratory Proposal Message

## 📊 Diagrama do Fluxo de Automação

```mermaid
flowchart TD
    Start([Início: Cron Schedule<br/>Diário às 12h UTC]) --> ValidatePipeline{Pipeline ID<br/>válido?}
    
    ValidatePipeline -->|Não| LogError[Log: Pipeline Inválido<br/>ou Chave não Encontrada]
    LogError --> EndError([Encerra com Erro])
    
    ValidatePipeline -->|Sim| GetKey[Obtém Chave da Campanha:<br/>KEY_CAMPAIGN_PROPOSTAS_LABORATORIAS]
    
    GetKey --> InitRepo[Inicializa<br/>RdCrmRcTeamRepository<br/>Token: CRM_TOKEN_RC_TEAM]
    
    InitRepo --> SetFilters[Define Filtros de Busca:<br/>- created_at_period: true<br/>- start_date: Hoje<br/>- end_date: Hoje]
    
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
        Handler[campaign-send-message<br/>handler]
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
    title Execução Diária - Laboratory Proposal Message
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

## 📋 Comparação: Dialer Call vs WhatsApp Message

```mermaid
graph TB
    subgraph Dialer["Campaign Dialer Call"]
        D1[Busca por Estágios]
        D2[Lista expira: 720min]
        D3[cancelarPendentes: true]
        D4[Canal: Telefone]
        D5[Token: CHARGES_TEAM]
        D6[Execução: Dias Específicos]
    end
    
    subgraph WhatsApp["Laboratory Proposal Message"]
        W1[Busca por Data: Hoje]
        W2[Lista expira: 60min]
        W3[cancelarPendentes: false]
        W4[Canal: WhatsApp]
        W5[Token: RC_TEAM]
        W6[Execução: Diária]
    end
    
    style Dialer fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style WhatsApp fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
```

## 📊 Fluxo de Filtros e Processamento

```mermaid
flowchart LR
    A[Todos os Deals<br/>do Pipeline] --> B{Filtro:<br/>created_at_period}
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
    Pipeline[Pipeline: Propostas Laboratoriais<br/>ID: 65391eec1e66020013a4a869]
    
    Pipeline --> Key[Chave Campanha:<br/>b6f7016d-dfe1-4acf-a4ba-853bf900c264]
    Pipeline --> Schedule[Schedule:<br/>Diário às 12h UTC]
    Pipeline --> Token[Token:<br/>CRM_TOKEN_RC_TEAM]
    Pipeline --> Expire[Expiração:<br/>60 minutos]
    Pipeline --> Cancel[Cancelar Pendentes:<br/>false]
    
    style Pipeline fill:#e1e1e1,stroke:#666,stroke-width:3px,color:#000
    style Key fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Schedule fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Token fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Expire fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Cancel fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
```

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

## 📝 Notas

- **Cores neutras**: Paleta em tons de cinza para visualização no GitHub
- **Fluxo detalhado**: Mostra todas as etapas, validações e decisões
- **Comparação visual**: Diferenças entre automações de dialer e WhatsApp
- **Timeline realista**: Estimativa de tempo de execução por etapa
- **Arquitetura clara**: Separação de responsabilidades entre camadas
