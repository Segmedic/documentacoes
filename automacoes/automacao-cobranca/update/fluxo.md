# Fluxo Visual - Update

## Diagrama Mermaid

``` mermaid
---
title: Fluxo da Automação - Update
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
    classDef apiClass fill:#c0c0c0,stroke:#666,stroke-width:2px,color:#000

    Start([Início - Cron Job]):::scheduleClass
    Schedule[Lambda Schedule<br/>update.handler]:::scheduleClass
    
    Start -->|Todos os dias<br/>às 22h UTC| Schedule
    
    RDApi[RD Station CRM<br/>API]:::externalClass
    Schedule --> RDApi
    
    GetPipe1[Buscar Pipeline 1-60 dias<br/>Paginação: até 49 páginas<br/>Delay: 2s entre páginas]:::apiClass
    GetPipe2[Buscar Pipeline 61-180 dias<br/>Paginação: até 49 páginas<br/>Delay: 2s entre páginas]:::apiClass
    GetPipe3[Buscar Pipeline 181-365 dias<br/>Paginação: até 49 páginas<br/>Delay: 2s entre páginas]:::apiClass
    GetPipe4[Buscar Pipeline 366+ dias<br/>Paginação: até 49 páginas<br/>Delay: 2s entre páginas]:::apiClass
    
    Schedule --> GetPipe1
    GetPipe1 --> RDApi
    
    Wait1[Aguardar 3 segundos]:::processClass
    GetPipe1 --> Wait1
    Wait1 --> GetPipe2
    GetPipe2 --> RDApi
    
    Wait2[Aguardar 3 segundos]:::processClass
    GetPipe2 --> Wait2
    Wait2 --> GetPipe3
    GetPipe3 --> RDApi
    
    Wait3[Aguardar 3 segundos]:::processClass
    GetPipe3 --> Wait3
    Wait3 --> GetPipe4
    GetPipe4 --> RDApi
    
    Concat[Concatenar todos<br/>os deals coletados]:::processClass
    GetPipe4 --> Concat
    
    LoopDeals{Para cada<br/>deal}:::decisionClass
    Concat --> LoopDeals
    
    ExtractSub[Extrair subscription ID<br/>do nome da organização<br/>Regex: apenas números]:::processClass
    LoopDeals -->|Processar| ExtractSub
    
    ValidateSub{Subscription ID<br/>é válido?}:::decisionClass
    ExtractSub --> ValidateSub
    
    Skip[Pular deal]:::processClass
    ValidateSub -->|Não| Skip
    Skip --> LoopDeals
    
    CreatePayload[Criar UpdateDTO:<br/>- subscription<br/>- dealId]:::processClass
    ValidateSub -->|Sim| CreatePayload
    
    Queue[AWS SQS Queue<br/>update]:::queueClass
    CreatePayload --> Queue
    Queue --> LoopDeals
    
    LoopDeals -->|Fim| End1([Fim do Schedule]):::endClass
    
    Worker[Lambda Worker<br/>update.handler]:::workerClass
    Queue -->|Consome mensagem<br/>batchSize: 1| Worker
    
    ValidatePayload{Payload<br/>válido?}:::decisionClass
    Worker --> ValidatePayload
    
    LogInvalid[Log erro:<br/>Payload inválido]:::processClass
    ValidatePayload -->|Não| LogInvalid
    LogInvalid --> End2([Exceção]):::endClass
    
    DB[(Banco DNA<br/>MySQL)]:::databaseClass
    ValidatePayload -->|Sim| DB
    
    UseCase[Use Case:<br/>UpdateUseCase]:::processClass
    ValidatePayload --> UseCase
    
    GetInvoices[Buscar faturas abertas<br/>da assinatura]:::processClass
    UseCase --> GetInvoices
    GetInvoices --> DB
    
    GetSub[Buscar dados<br/>da assinatura]:::processClass
    GetInvoices --> GetSub
    GetSub --> DB
    
    CheckDate[Verificar:<br/>É último dia do mês?]:::processClass
    GetSub --> CheckDate
    
    IsLastDay{É último<br/>dia do mês?}:::decisionClass
    CheckDate --> IsLastDay
    
    MarkLost1[Marcar deal como LOST<br/>no RD Station]:::processClass
    IsLastDay -->|Sim| MarkLost1
    MarkLost1 --> RDApi
    MarkLost1 --> Return1[Retornar]:::processClass
    
    CheckInvoices{Tem faturas<br/>abertas?}:::decisionClass
    IsLastDay -->|Não| CheckInvoices
    
    Continue[Continuar<br/>Deal permanece ativo]:::processClass
    CheckInvoices -->|Sim| Continue
    Continue --> Return2[Retornar]:::processClass
    
    MarkLost2[Marcar deal como LOST<br/>no RD Station<br/>Regularizado]:::processClass
    CheckInvoices -->|Não| MarkLost2
    MarkLost2 --> RDApi
    MarkLost2 --> Return3[Retornar]:::processClass
    
    CloseConn[Fechar conexão DB]:::processClass
    Return1 --> CloseConn
    Return2 --> CloseConn
    Return3 --> CloseConn
    
    End3([Fim do Worker]):::endClass
    CloseConn --> End3

    %% Notas explicativas
    Note1[📝 Coleta Massiva:<br/>- Todos os 4 pipelines<br/>- Até 9.800 deals<br/>- Rate limiting: 2s/3s<br/>- Limite: 49 páginas/stage]:::queueClass
    Note1 -.-> Schedule
    
    Note2[📝 Extração ID:<br/>Organization name<br/>'Assinatura 12345' → 12345<br/>Apenas números]:::queueClass
    Note2 -.-> ExtractSub
    
    Note3[📝 Limpeza Mensal:<br/>Todo último dia do mês<br/>marca deals como lost<br/>independente do status]:::queueClass
    Note3 -.-> IsLastDay
    
    Note4[📝 Regularização:<br/>Sem faturas abertas =<br/>Cliente regularizou<br/>Remove do funil]:::queueClass
    Note4 -.-> CheckInvoices
    
    Note5[📝 Lost Lead Process:<br/>- Mantém custom fields<br/>- Atualiza status<br/>- win: false<br/>- Preserva user_id]:::queueClass
    Note5 -.-> MarkLost1
```
