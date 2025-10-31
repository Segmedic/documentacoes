# Fluxo Visual - Tickets Renewal

## Diagrama Mermaid

``` mermaid
---
title: Fluxo da Automação - Tickets Renewal
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

    Start([Início - Cron Job]):::scheduleClass
    Schedule[Lambda Schedule<br/>ticketsRennewal.handler]:::scheduleClass
    
    Start -->|Toda segunda-feira<br/>às 3h UTC| Schedule
    
    DB[(Banco DNA<br/>MySQL)]:::databaseClass
    Schedule --> DB
    
    Query[Query SQL<br/>listTicketToRennewal]:::processClass
    DB --> Query
    
    Filter{Filtrar:<br/>ult_pg <= 4 meses?}:::decisionClass
    Query --> Filter
    
    Skip1[Pular assinatura]:::processClass
    Filter -->|Não| Skip1
    Skip1 --> CheckMore1{Há mais<br/>assinaturas?}:::decisionClass
    CheckMore1 -->|Sim| Filter
    CheckMore1 -->|Não| End1([Fim do Schedule]):::endClass
    
    Queue[AWS SQS Queue<br/>ticketsRennewal]:::queueClass
    Filter -->|Sim| Queue
    
    Queue --> CheckMore2{Há mais<br/>assinaturas?}:::decisionClass
    CheckMore2 -->|Sim| Filter
    CheckMore2 -->|Não| End2([Fim do Schedule]):::endClass
    
    Worker[Lambda Worker<br/>ticketsRennewal.handler]:::workerClass
    Queue -->|Consome mensagem<br/>batchSize: 1| Worker
    
    Parse[Use Case:<br/>TicketsRennewalUseCase]:::processClass
    Worker --> Parse
    
    GetData[Buscar dados:<br/>- Titular<br/>- Plano<br/>- Faturas<br/>- Cartão de Crédito]:::processClass
    Parse --> GetData
    GetData --> DB
    
    CheckData{Dados<br/>completos?}:::decisionClass
    GetData --> CheckData
    
    LogError1[Log: Falta dados<br/>para assinatura]:::processClass
    CheckData -->|Não| LogError1
    LogError1 --> NextRecord1
    
    BuildDeal[Montar DealDTO:<br/>- Calcular dias inadimplência<br/>- Definir stage<br/>- Custom fields<br/>- Anotações]:::processClass
    CheckData -->|Sim| BuildDeal
    
    RDStation[RD Station CRM<br/>API]:::externalClass
    BuildDeal --> RDStation
    
    ProcessLead[RdService:<br/>processLeadTicket]:::processClass
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
    
    AddNotes[Adicionar Anotações:<br/>- Link Backoffice<br/>- IDs das Faturas]:::processClass
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
    Note1[📝 Critérios SQL:<br/>- payment_type IN 'TICKET', 'TICKETS'<br/>- status != 'CANCELED'<br/>- plano LIKE '%CARNE%'<br/>- qtd_aberto <= 1]:::queueClass
    Note1 -.-> Query
    
    Note2[📝 Custom Fields:<br/>- Tempo inadimplente<br/>- CPF, Plano, Status<br/>- Forma pagamento<br/>- Score, Código retorno]:::queueClass
    Note2 -.-> BuildDeal
```
