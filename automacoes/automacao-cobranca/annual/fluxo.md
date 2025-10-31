# Fluxo Visual - Annual Renewal

## Diagrama Mermaid

``` mermaid
---
title: Fluxo da Automação - Annual Renewal
---
flowchart TD
    %% Estilos neutros para GitHub
    classDef scheduleClass fill:#e8e8e8,stroke:#666,stroke-width:2px,color:#000
    classDef databaseClass fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
    classDef processClass fill:#b8b8b8,stroke:#666,stroke-width:2px,color:#000
    classDef queueClass fill:#f0f0f0,stroke:#666,stroke-width:2px,color:#000
    classDef workerClass fill:#c8c8c8,stroke:#666,stroke-width:2px,color:#000
    classDef decisionClass fill:#d8d8d8,stroke:#666,stroke-width:2px,color:#000
    classDef externalClass fill:#e0e0e0,stroke:#666,stroke-width:2px,color:#000
    classDef endClass fill:#a8a8a8,stroke:#666,stroke-width:2px,color:#fff
    classDef utilClass fill:#c0c0c0,stroke:#666,stroke-width:2px,color:#000

    Start([Início - Cron Job]):::scheduleClass
    Schedule[Lambda Schedule<br/>annualRennewal.handler]:::scheduleClass
    
    Start -->|Todo dia 1º do mês<br/>às 3h UTC| Schedule
    
    DB[(Banco DNA<br/>MySQL)]:::databaseClass
    Schedule --> DB
    
    DateUtils[Utilitários de Data:<br/>lastYear = ano anterior<br/>actualyMonth = mês atual]:::utilClass
    Schedule --> DateUtils
    
    Query[Query SQL<br/>listAnnualPlan lastYear]:::processClass
    DateUtils --> Query
    DB --> Query
    
    FilterMonth{Último pagamento<br/>no mesmo mês<br/>do ano anterior?}:::decisionClass
    Query --> FilterMonth
    
    FilterStatus{Status !=<br/>CANCELED?}:::decisionClass
    FilterMonth -->|Sim| FilterStatus
    
    Skip1[Pular plano]:::processClass
    FilterMonth -->|Não| Skip1
    FilterStatus -->|Não| Skip1
    
    Skip1 --> CheckMore1{Há mais<br/>planos?}:::decisionClass
    CheckMore1 -->|Sim| FilterMonth
    CheckMore1 -->|Não| End1([Fim do Schedule]):::endClass
    
    Queue[AWS SQS Queue<br/>annual]:::queueClass
    FilterStatus -->|Sim| Queue
    
    Queue --> CheckMore2{Há mais<br/>planos?}:::decisionClass
    CheckMore2 -->|Sim| FilterMonth
    CheckMore2 -->|Não| End2([Fim do Schedule]):::endClass
    
    Worker[Lambda Worker<br/>annualRennewal.handler]:::workerClass
    Queue -->|Consome mensagem<br/>batchSize: 1| Worker
    
    Parse[Use Case:<br/>AnnuaRennewallUseCase]:::processClass
    Worker --> Parse
    
    GetData[Buscar dados:<br/>- Titular<br/>- Plano<br/>- Faturas abertas<br/>- Cartão de Crédito]:::processClass
    Parse --> GetData
    GetData --> DB
    
    CheckData{Dados<br/>completos?}:::decisionClass
    GetData --> CheckData
    
    LogError1[Log: Falta dados<br/>para assinatura]:::processClass
    CheckData -->|Não| LogError1
    LogError1 --> NextRecord1
    
    BuildDeal[Montar DealDTO:<br/>- Calcular dias inadimplência<br/>- Definir stage renovação<br/>- Custom fields<br/>- Anotações]:::processClass
    CheckData -->|Sim| BuildDeal
    
    RDStation[RD Station CRM<br/>API]:::externalClass
    BuildDeal --> RDStation
    
    ProcessLead[RdService:<br/>processLeadAnnual]:::processClass
    BuildDeal --> ProcessLead
    
    FindOrg[Buscar/Criar<br/>Organização]:::processClass
    ProcessLead --> FindOrg
    FindOrg --> RDStation
    
    FindDeal[Buscar/Criar<br/>Deal]:::processClass
    FindOrg --> FindDeal
    FindDeal --> RDStation
    
    CheckNew{Deal<br/>é novo?}:::decisionClass
    FindDeal --> CheckNew
    
    UpdateDeal[Atualizar Deal:<br/>- Custom fields<br/>- User ID se stage inválido]:::processClass
    CheckNew -->|Não| UpdateDeal
    UpdateDeal --> RDStation
    UpdateDeal --> ReturnID1[Retornar ID do Deal]:::processClass
    
    AddNotes[Adicionar Anotações:<br/>- Link Backoffice<br/>- IDs das Faturas abertas]:::processClass
    CheckNew -->|Sim| AddNotes
    AddNotes --> RDStation
    AddNotes --> ReturnID2[Retornar ID do Deal]:::processClass
    
    CheckSuccess{Processamento<br/>OK?}:::decisionClass
    ReturnID1 --> CheckSuccess
    ReturnID2 --> CheckSuccess
    
    LogError2[Log: Falha ao<br/>processar lead]:::processClass
    CheckSuccess -->|Não| LogError2
    LogError2 --> NextRecord2
    
    LogSuccess[Log: Deal<br/>processado com sucesso]:::processClass
    CheckSuccess -->|Sim| LogSuccess
    LogSuccess --> NextRecord3
    
    NextRecord1{Próximo<br/>registro?}:::decisionClass
    NextRecord2{Próximo<br/>registro?}:::decisionClass
    NextRecord3{Próximo<br/>registro?}:::decisionClass
    
    NextRecord1 -->|Sim| Parse
    NextRecord2 -->|Sim| Parse
    NextRecord3 -->|Sim| Parse
    
    Wait[Aguardar 5 segundos]:::processClass
    NextRecord1 -->|Não| Wait
    NextRecord2 -->|Não| Wait
    NextRecord3 -->|Não| Wait
    
    CloseConn[Fechar conexão DB]:::processClass
    Wait --> CloseConn
    
    End3([Fim do Worker]):::endClass
    CloseConn --> End3

    %% Notas explicativas
    Note1[📝 Critérios SQL:<br/>- Plano LIKE '%ANUAL%'<br/>- Fatura status != 'CANCELED'<br/>- YEAR payment_date <= lastYear]:::queueClass
    Note1 -.-> Query
    
    Note2[📝 Exemplo:<br/>Hoje: 1º out/2025<br/>Busca: payment <= 2024<br/>Filtra: pagamento em out/2024<br/>Resultado: Renovações de out/2025]:::queueClass
    Note2 -.-> FilterMonth
    
    Note3[📝 Deal ID:<br/>Formato: idsubscription-anual<br/>Stage: FIRST_STAGE_ANNUAL_RENNEWAL]:::queueClass
    Note3 -.-> BuildDeal
```
