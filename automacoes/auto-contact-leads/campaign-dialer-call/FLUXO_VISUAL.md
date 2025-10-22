# Fluxo Visual - Campaign Dialer Call

## 📊 Diagrama do Fluxo de Automação

``` mermaid
flowchart TD
    Start([Início: Cron Schedule]) --> CheckDay{É Domingo?}
    
    CheckDay -->|Sim| EndSunday([Encerra Execução])
    CheckDay -->|Não| ValidatePipeline{Pipeline ID<br/>válido?}
    
    ValidatePipeline -->|Não| LogError[Log: Pipeline Inválido]
    LogError --> EndError([Encerra com Erro])
    
    ValidatePipeline -->|Sim| GetKey[Obtém Chave da Campanha]
    
    GetKey --> InitRepo[Inicializa<br/>RdCrmChargeTeamRepository]
    
    InitRepo --> LoopStages[Itera por Estágios<br/>do Pipeline]
    
    LoopStages --> LoopPages{Tem mais<br/>páginas?}
    
    LoopPages -->|Sim| FetchDeals[Busca Deals<br/>200 por página]
    
    FetchDeals --> ProcessContacts[Processa Contatos:<br/>- Normaliza Telefone<br/>- Extrai Nome<br/>- Remove Duplicatas]
    
    ProcessContacts --> Wait[Aguarda 2s<br/>Rate Limiting]
    
    Wait --> LoopPages
    
    LoopPages -->|Não| CheckResults{Encontrou<br/>Leads?}
    
    CheckResults -->|Não| LogNoLeads[Log: Nenhum Lead]
    LogNoLeads --> EndNoLeads([Encerra])
    
    CheckResults -->|Sim| FormatPayload[Formata Payload Escallo:<br/>- expiraLista: 720min<br/>- cancelarPendentes: true<br/>- chaveExterna: key]
    
    FormatPayload --> SendToDialer[Envia POST para<br/>Escallo API]
    
    SendToDialer --> CheckDialer{Envio<br/>bem-sucedido?}
    
    CheckDialer -->|Não| ThrowError[Lança DialingError]
    ThrowError --> EndDialerError([Encerra com Erro])
    
    CheckDialer -->|Sim| LogSuccess[Log: Sucesso]
    LogSuccess --> EndSuccess([Execução Concluída])
    
    style Start fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style EndSunday fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style EndError fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style EndNoLeads fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style EndDialerError fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style EndSuccess fill:#d4d4d4,stroke:#666,stroke-width:3px,color:#000
    
    style CheckDay fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style ValidatePipeline fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style LoopPages fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style CheckResults fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style CheckDialer fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    
    style GetKey fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style InitRepo fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style LoopStages fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style FetchDeals fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style ProcessContacts fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Wait fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style FormatPayload fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style SendToDialer fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
```

## 🔄 Fluxo Simplificado de Dados

``` mermaid
graph LR
    A[AWS Lambda<br/>Trigger] --> B[RD Station CRM]
    B --> C[Use Case:<br/>GetAllDeals]
    C --> D[Use Case:<br/>DialerLeads]
    D --> E[Escallo<br/>Dialer API]
    
    style A fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style B fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style C fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style D fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style E fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 🏗️ Arquitetura de Camadas

```  mermaid
flowchart TB
    subgraph Lambda["🔸 AWS Lambda Function"]
        Handler[campaign-dialer-call<br/>handler]
    end
    
    subgraph UseCases["🔸 Use Cases Layer"]
        GetDeals[GetAllDealsUseCase]
        DialerLeads[DialerLeads]
    end
    
    subgraph Repos["🔸 Repositories Layer"]
        RdCrm[RdCrmChargeTeamRepository]
        Escallo[EscalloDialerRepository]
    end
    
    subgraph External["🔸 APIs Externas"]
        RdApi[RD Station CRM API]
        EscalloApi[Escallo Dialer API]
    end
    
    Handler --> GetDeals
    Handler --> DialerLeads
    GetDeals --> RdCrm
    DialerLeads --> Escallo
    RdCrm --> RdApi
    Escallo --> EscalloApi
    
    style Lambda fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style UseCases fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Repos fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style External fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## ⏱️ Cronograma de Execução

```  mermaid
gantt
    title Calendário de Execuções por Pipeline
    dateFormat YYYY-MM-DD
    axisFormat %d
    
    section Pipeline 1-60 Dias
    Dias Ímpares às 12h UTC    :2025-10-01, 2025-10-31
    
    section Pipeline 61-180 Dias
    Dias Ímpares às 12h UTC    :2025-10-01, 2025-10-31
    
    section Pipeline 181-365 Dias
    Dias Ímpares às 12h UTC    :2025-10-01, 2025-10-31
    
    section Pipeline +366 Dias
    Dias 3, 7, 14, 21 às 12h UTC :2025-10-01, 2025-10-31
```

## 📋 Mapeamento de Pipelines

| Pipeline | ID | Chave | Frequência |
|----------|-----|-------|-----------|
| 1-60 Dias | `66db5321b075c70026b57949` | `1_60DIAS` | Dias ímpares |
| 61-180 Dias | `66db64909885e10023aee3d5` | `61_180DIAS` | Dias ímpares |
| 181-365 Dias | `66db64b0f64a1f001a00d2e5` | `181_365DIAS` | Dias ímpares |
| +366 Dias | `66db64bebdef0d0020f400a5` | `+365DIAS` | Dias 3,7,14,21 |

## 🎯 Estágios dos Pipelines

Todos os pipelines possuem 6 estágios processados em sequência:

1. **Novos**
2. **[Sistema] Contato 1**
3. **[Manual] Sem Conexão**
4. **[Sistema] Contato 2**
5. **[Manual] Sem Resposta**
6. **[Sistema] Resgate**

---

## 📝 Notas

- **Cores neutras**: Escolhidas especificamente para boa visualização no tema claro e escuro do GitHub
- **Fluxo detalhado**: Mostra todas as etapas e decisões do processo
- **Arquitetura**: Demonstra a separação de responsabilidades entre camadas
- **Cronograma**: Visualização temporal das execuções agendadas
