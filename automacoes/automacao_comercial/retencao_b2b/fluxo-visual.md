# Fluxo Visual - Retenção de Ex-Colaboradores B2B

## 🔄 Diagrama Principal

```mermaid
flowchart TD
    Start([Início da Automação]) --> Init[Inicializar Conexões: ClubFlex e Medula]
    
    Init --> Query[Buscar Dependentes Removidos no ClubFlex]
    
    Query --> Deduplicate[Processar e Deduplica CPFs]
    
    Deduplicate --> Filter{Filtrar por Status}
    
    Filter -->|BLOCKED ou OK| Skip[Adicionar ao Skip Set]
    Filter -->|REMOVED| Keep[Manter no Map]
    
    Skip --> NextItem{Mais itens?}
    Keep --> NextItem
    
    NextItem -->|Sim| Filter
    NextItem -->|Não| ExtractCPFs[Extrair Lista de CPFs Únicos]
    
    ExtractCPFs --> EnrichData[Buscar Dados no Medula por CPF]
    
    EnrichData --> BuildLeads[Construir Leads Enriquecidos]
    
    BuildLeads --> Loop{Para cada lead}
    
    Loop --> CalcPeriod[Calcular Período de Assinatura]
    CalcPeriod --> CalcTime[Calcular Tempo Sem Assinatura]
    CalcTime --> MergeData[Mesclar Dados ClubFlex + Medula]
    MergeData --> BuildURL[Gerar URL Feegow]
    BuildURL --> ValidatePhone{Tem telefone?}
    
    ValidatePhone -->|Não| Exclude[Excluir Lead]
    ValidatePhone -->|Sim| AddToList[Adicionar à Lista]
    
    Exclude --> MoreLeads{Mais leads?}
    AddToList --> MoreLeads
    
    MoreLeads -->|Sim| Loop
    MoreLeads -->|Não| SendAll[Enviar Todos em Paralelo para SQS]
    
    SendAll --> Cleanup[Destruir Conexões]
    
    Cleanup --> End([Fim])
    
    style Start fill:#e1e4e8
    style End fill:#e1e4e8
    style SendAll fill:#d1d5da
    style EnrichData fill:#f6f8fa
    style Deduplicate fill:#f6f8fa
    style MergeData fill:#f6f8fa
```

## 📊 Deduplicação Complexa com Map e Set

```mermaid
flowchart TD
    subgraph Input["Entrada - Múltiplos Registros"]
        A[CPF 123 - REMOVED]
        B[CPF 123 - OK]
        C[CPF 456 - REMOVED]
        D[CPF 789 - BLOCKED]
    end
    
    subgraph Process["Processamento"]
        E{Status do CPF?}
        F[Map: CPFs REMOVED]
        G[Skip: CPFs BLOCKED/OK]
    end
    
    subgraph Output["Saída"]
        H[Apenas CPF 456 - REMOVED]
    end
    
    A --> E
    B --> E
    C --> E
    D --> E
    
    E -->|REMOVED e novo| F
    E -->|BLOCKED ou OK| G
    
    F --> H
    G -.Exclui de Map.-> F
    
    style A fill:#f6f8fa
    style B fill:#f6f8fa
    style C fill:#f6f8fa
    style D fill:#f6f8fa
    style E fill:#e1e4e8
    style F fill:#d1d5da
    style G fill:#e1e4e8
    style H fill:#d1d5da
```

## 🔄 Enriquecimento de Dados

```mermaid
flowchart LR
    subgraph ClubFlex["Dados ClubFlex"]
        CF1[Nome]
        CF2[Email]
        CF3[Telefone]
        CF4[CPF]
        CF5[Data Remoção]
        CF6[Data Inserção]
    end
    
    subgraph Medula["Dados Medula - Histórico"]
        M1[Nome Paciente]
        M2[Email]
        M3[Celular]
        M4[Sexo]
        M5[Idade]
        M6[Unidade Mais Usada]
        M7[Procedimento Mais Usado]
        M8[ID Feegow]
    end
    
    subgraph Calculo["Cálculos"]
        C1[Período de Assinatura]
        C2[Tempo Sem Assinatura]
        C3[URL Feegow]
    end
    
    subgraph LeadFinal["Lead Enriquecido"]
        L[Dados Combinados + Calculados]
    end
    
    CF1 --> L
    CF2 --> L
    CF3 --> L
    CF4 --> Medula
    CF5 --> C2
    CF6 --> C1
    
    M1 -.Prioridade.-> L
    M2 -.Prioridade.-> L
    M3 -.Prioridade.-> L
    M4 --> L
    M5 --> L
    M6 --> L
    M7 --> L
    M8 --> C3
    
    C1 --> L
    C2 --> L
    C3 --> L
    
    style CF1 fill:#f6f8fa
    style CF2 fill:#f6f8fa
    style CF3 fill:#f6f8fa
    style M1 fill:#e1e4e8
    style M2 fill:#e1e4e8
    style M3 fill:#e1e4e8
    style C1 fill:#d1d5da
    style C2 fill:#d1d5da
    style C3 fill:#d1d5da
    style L fill:#d1d5da
```

## 🎯 Filtros de Status

```mermaid
flowchart TD
    subgraph Entrada["CPFs do ClubFlex"]
        E1[Status: REMOVED]
        E2[Status: OK]
        E3[Status: BLOCKED]
    end
    
    subgraph Decisao["Decisão"]
        D{Qual Status?}
    end
    
    subgraph Acao["Ação"]
        A1[Processar como Lead]
        A2[Adicionar ao Skip Set]
        A3[Remover do Map se existir]
    end
    
    E1 --> D
    E2 --> D
    E3 --> D
    
    D -->|REMOVED| A1
    D -->|OK| A2
    D -->|BLOCKED| A2
    
    A2 --> A3
    
    style E1 fill:#d1d5da
    style E2 fill:#e1e4e8
    style E3 fill:#e1e4e8
    style D fill:#f6f8fa
    style A1 fill:#d1d5da
    style A2 fill:#e1e4e8
    style A3 fill:#e1e4e8
```

## 📅 Período de Busca

```mermaid
gantt
    title Janela de Tempo - Dependentes Removidos
    dateFormat YYYY-MM-DD
    axisFormat %d/%m
    
    section Período
    1 Mês Atrás                 :milestone, m1, 2024-01-10, 0d
    Janela de Busca (30 dias)   :active, 2024-01-10, 30d
    Ontem                       :milestone, m2, 2024-02-09, 0d
    Hoje (Execução)             :crit, milestone, m3, 2024-02-10, 0d
```

## 🔄 Integração com Sistemas

```mermaid
graph TB
    subgraph Automacao["Automação"]
        Auto[Retenção Ex-Colaboradores B2B]
    end
    
    subgraph Databases["Bases de Dados"]
        ClubFlex[(ClubFlex MySQL)]
        Medula[(Medula PostgreSQL)]
    end
    
    subgraph Queue["Fila"]
        SQS[(AWS SQS)]
    end
    
    Auto -->|1. Query: Dependentes REMOVED PJ| ClubFlex
    ClubFlex -->|2. Lista de CPFs| Auto
    Auto -->|3. Query: Histórico por CPF| Medula
    Medula -->|4. Dados Enriquecidos| Auto
    Auto -->|5. POST Lead Events| SQS
    
    style Auto fill:#d1d5da
    style ClubFlex fill:#e1e4e8
    style Medula fill:#e1e4e8
    style SQS fill:#e1e4e8
```

## 📦 Estrutura do Lead Enriquecido

```mermaid
classDiagram
    class ClubFlexData {
        +string cpf
        +string name
        +string email
        +string phone
        +string sex
        +Date date_remove
        +Date date_of_insert
        +string status
    }
    
    class MedulaData {
        +string NomePaciente
        +string email1
        +string Cel1
        +string nomesexo
        +number idade
        +number pacienteid
        +string unitsAndQuantityServed
        +string proceduresAndQuantityServed
    }
    
    class LeadEnriquecido {
        +string name
        +string email
        +string phone
        +string period_of_subscription
        +string time_without_subscription
        +string sexo
        +number age
        +string procedure_max_served
        +string unit_max_served
        +string patient_feegow
    }
    
    class LeadEvent {
        +string origin
        +LeadEnriquecido payload
    }
    
    ClubFlexData --> LeadEnriquecido : dados base
    MedulaData --> LeadEnriquecido : enriquecimento
    LeadEnriquecido --> LeadEvent : contém
    
    note for LeadEnriquecido "Dados Medula têm\nprioridade sobre ClubFlex"
```

## 🚦 Validação de CPF

```mermaid
flowchart LR
    subgraph Input["CPF"]
        A[String de Entrada]
    end
    
    subgraph Regex["Validação Regex"]
        B{11 dígitos?}
        C{Não são todos iguais?}
    end
    
    subgraph Output["Resultado"]
        D[CPF Válido]
        E[CPF Inválido]
    end
    
    A --> B
    B -->|Não| E
    B -->|Sim| C
    C -->|Sim - Todos iguais| E
    C -->|Não - Variam| D
    
    style A fill:#f6f8fa
    style B fill:#e1e4e8
    style C fill:#e1e4e8
    style D fill:#d1d5da
    style E fill:#e1e4e8
```

## 📊 Cálculo de Períodos

```mermaid
sequenceDiagram
    participant Lead as Lead
    participant Calc as Calculadora
    participant Utils as diferencaFormatada
    
    Lead->>Calc: date_of_insert, date_remove
    Calc->>Utils: Calcular diferença
    Utils-->>Calc: "2 meses, 15 dias"
    Calc-->>Lead: period_of_subscription
    
    Lead->>Calc: date_remove, hoje
    Calc->>Utils: Calcular diferença
    Utils-->>Calc: "10 dias"
    Calc-->>Lead: time_without_subscription
```

## 🎯 Query ClubFlex - Filtros SQL

```mermaid
flowchart TD
    subgraph Tables["Tabelas"]
        T1[(dependent)]
        T2[(subscription)]
    end
    
    subgraph Filtros["Filtros WHERE"]
        F1[status = REMOVED]
        F2[CPF válido regex]
        F3[type_sub = PJ]
        F4[date_of_removal >= 1 mês atrás]
        F5[date_of_removal < hoje]
    end
    
    subgraph Result["Resultado"]
        R[Dependentes Removidos de Planos PJ]
    end
    
    T1 -->|JOIN| T2
    T2 --> F1
    F1 --> F2
    F2 --> F3
    F3 --> F4
    F4 --> F5
    F5 --> R
    
    style T1 fill:#e1e4e8
    style T2 fill:#e1e4e8
    style F1 fill:#f6f8fa
    style F2 fill:#f6f8fa
    style F3 fill:#f6f8fa
    style F4 fill:#f6f8fa
    style F5 fill:#f6f8fa
    style R fill:#d1d5da
```

## 🔢 Prioridade de Dados

```mermaid
mindmap
  root((Lead Final))
    Sempre Medula se existe
      Nome
      Email
      Telefone
      Sexo
      Idade
    Exclusivo Medula
      Unidade Mais Usada
      Procedimento Mais Usado
      ID Feegow
      Link Feegow
    Calculados
      Período de Assinatura
      Tempo Sem Assinatura
    Fallback ClubFlex
      Se Medula não tem dados
      Usa dados do ClubFlex
```

## 🚀 Envio Paralelo

```mermaid
flowchart LR
    subgraph Leads["Leads Válidos"]
        L1[Lead 1]
        L2[Lead 2]
        L3[Lead 3]
        L4[Lead N]
    end
    
    subgraph PromiseAll["Promise.all"]
        P[Envio Simultâneo]
    end
    
    subgraph SQS["AWS SQS"]
        Q1[Fila]
        Q2[Fila]
        Q3[Fila]
        Q4[Fila]
    end
    
    L1 --> P
    L2 --> P
    L3 --> P
    L4 --> P
    
    P --> Q1
    P --> Q2
    P --> Q3
    P --> Q4
    
    style L1 fill:#f6f8fa
    style L2 fill:#f6f8fa
    style L3 fill:#f6f8fa
    style L4 fill:#f6f8fa
    style P fill:#d1d5da
    style Q1 fill:#e1e4e8
    style Q2 fill:#e1e4e8
    style Q3 fill:#e1e4e8
    style Q4 fill:#e1e4e8
```

## 🎯 Comparação com Outras Automações

```mermaid
graph TB
    subgraph RetencaoB2B["Retenção B2B"]
        R1[Ex-dependentes ClubFlex]
        R2[Último mês]
        R3[Enriquecimento 2 sistemas]
        R4[Cálculo de tempos]
    end
    
    subgraph LeadsAmanha["Leads Amanhã"]
        LA1[Agendamentos futuros]
        LA2[D+1]
        LA3[Sem enriquecimento]
        LA4[Sem cálculos]
    end
    
    subgraph LeadsConvenios["Leads Convênios"]
        LC1[Atendimentos realizados]
        LC2[D-1]
        LC3[Sem enriquecimento]
        LC4[Sem cálculos]
    end
    
    style R1 fill:#d1d5da
    style R2 fill:#d1d5da
    style R3 fill:#d1d5da
    style R4 fill:#d1d5da
    style LA1 fill:#e1e4e8
    style LA2 fill:#e1e4e8
    style LA3 fill:#e1e4e8
    style LA4 fill:#e1e4e8
    style LC1 fill:#f6f8fa
    style LC2 fill:#f6f8fa
    style LC3 fill:#f6f8fa
    style LC4 fill:#f6f8fa
```
