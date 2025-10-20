# Fluxo Visual - Escallo Ligação

## 🔄 Diagrama Principal

```mermaid
flowchart TD
    Start([Início da Automação]) --> Init[Inicializar Cliente Escallo]
    
    Init --> GetData{Buscar Dados Paralelos}
    
    GetData -->|Report 086| GetCalls[Buscar Ligações do Dia]
    GetData -->|Report 002| GetAudios[Buscar Áudios do Dia]
    
    GetCalls --> Process[Processar Registros]
    GetAudios --> Process
    
    Process --> Loop{Para cada ligação}
    
    Loop --> CheckQueue{É Fila Ação Record?}
    
    CheckQueue -->|Sim| SetFlagQueue[filaAcaoRecord = true]
    CheckQueue -->|Não| SetFlagNoQueue[filaAcaoRecord = false]
    
    SetFlagQueue --> CheckAgent{É Agente Comercial?}
    SetFlagNoQueue --> CheckAgent
    
    CheckAgent -->|Sim| SetAgent[agenteComercial = true]
    CheckAgent -->|Não| SetNoAgent[agenteComercial = false]
    
    SetAgent --> CheckDuplicate
    SetNoAgent --> CheckDuplicate
    
    CheckDuplicate{Número já processado?}
    
    CheckDuplicate -->|Sim| Skip[Pular para próximo]
    CheckDuplicate -->|Não| CheckCondition{Atende condição?}
    
    CheckCondition -->|Não| Skip
    CheckCondition -->|Sim| FindAudio[Buscar Áudio por uniqueid]
    
    FindAudio --> ValidateLead{isLead válido?}
    
    ValidateLead -->|Não| Skip
    ValidateLead -->|Sim| DefineOrigin{Qual origem?}
    
    DefineOrigin -->|Fila Record| OriginRecord[escallo_ligacao_record]
    DefineOrigin -->|Fila Normal| OriginNormal[escallo_ligacao]
    
    OriginRecord --> CreateLead[Criar Lead Event]
    OriginNormal --> CreateLead
    
    CreateLead --> AddAudio[Adicionar Link de Áudio]
    AddAudio --> SendQueue[Enviar para SQS]
    SendQueue --> MarkProcessed[Marcar número como processado]
    
    MarkProcessed --> HasMore{Mais ligações?}
    Skip --> HasMore
    
    HasMore -->|Sim| Loop
    HasMore -->|Não| End([Fim])
    
    style Start fill:#e1e4e8
    style End fill:#e1e4e8
    style SendQueue fill:#d1d5da
    style CreateLead fill:#f6f8fa
    style ValidateLead fill:#f6f8fa
    style CheckCondition fill:#f6f8fa
```

## 📊 Lógica de Decisão - Agente e Fila

```mermaid
flowchart TD
    subgraph Input["Dados da Ligação"]
        A[Agente ID]
        B[Fila Atendimento]
    end
    
    subgraph Check1["Verificação de Agente"]
        C{Agente está em AGENTES_ESCALLO_RD?}
        D[agenteComercial = true]
        E[agenteComercial = false]
    end
    
    subgraph Check2["Verificação de Fila"]
        F{Fila é Ação Record ou Transferencia?}
        G[filaAcaoRecord = true]
        H[filaAcaoRecord = false]
    end
    
    subgraph Decision["Decisão Final"]
        I{agenteComercial OU filaAcaoRecord?}
        J[Processar Lead]
        K[Ignorar]
    end
    
    A --> C
    B --> F
    
    C -->|Sim| D
    C -->|Não| E
    
    F -->|Sim| G
    F -->|Não| H
    
    D --> I
    E --> I
    G --> I
    H --> I
    
    I -->|Verdadeiro| J
    I -->|Falso| K
    
    style A fill:#f6f8fa
    style B fill:#f6f8fa
    style D fill:#d1d5da
    style G fill:#d1d5da
    style J fill:#d1d5da
    style K fill:#e1e4e8
```

## 🎯 Tabela de Decisão Visual

```mermaid
graph TB
    subgraph Cenario1["Cenário 1: Agente Comercial + Fila Record"]
        C1A[Agente: SIM]
        C1B[Fila: SIM]
        C1R[Resultado: escallo_ligacao_record]
    end
    
    subgraph Cenario2["Cenário 2: Agente Comercial + Fila Normal"]
        C2A[Agente: SIM]
        C2B[Fila: NÃO]
        C2R[Resultado: escallo_ligacao]
    end
    
    subgraph Cenario3["Cenário 3: Agente Comum + Fila Record"]
        C3A[Agente: NÃO]
        C3B[Fila: SIM]
        C3R[Resultado: escallo_ligacao_record]
    end
    
    subgraph Cenario4["Cenário 4: Agente Comum + Fila Normal"]
        C4A[Agente: NÃO]
        C4B[Fila: NÃO]
        C4R[Resultado: IGNORAR]
    end
    
    C1A --> C1R
    C1B --> C1R
    C2A --> C2R
    C2B --> C2R
    C3A --> C3R
    C3B --> C3R
    C4A --> C4R
    C4B --> C4R
    
    style C1R fill:#d1d5da
    style C2R fill:#d1d5da
    style C3R fill:#d1d5da
    style C4R fill:#e1e4e8
```

## 📞 Validação de Lead (isLead)

```mermaid
flowchart LR
    subgraph Registro["Registro de Ligação"]
        R[Registro86]
    end
    
    subgraph Validacoes["Validações"]
        V1{Tem agente?}
        V2{Tem origem?}
        V3{É Entrada?}
    end
    
    subgraph Resultado["Resultado"]
        OK[Lead Válido]
        NOK[Lead Inválido]
    end
    
    R --> V1
    V1 -->|Vazio| NOK
    V1 -->|Preenchido| V2
    V2 -->|Vazio| NOK
    V2 -->|Preenchido| V3
    V3 -->|Saída| NOK
    V3 -->|Entrada| OK
    
    style R fill:#f6f8fa
    style V1 fill:#e1e4e8
    style V2 fill:#e1e4e8
    style V3 fill:#e1e4e8
    style OK fill:#d1d5da
    style NOK fill:#e1e4e8
```

## 🔄 Integração com Escallo

```mermaid
graph LR
    subgraph Automacao["Automação"]
        Auto[Escallo Ligação]
    end
    
    subgraph Escallo["Sistema Escallo"]
        Rep086[(Report 086 - Ligações)]
        Rep002[(Report 002 - Áudios)]
    end
    
    subgraph Queue["Fila"]
        SQS[(AWS SQS)]
    end
    
    Auto -->|1. Query: Ligações de hoje| Rep086
    Rep086 -->|2. Lista de Registros| Auto
    Auto -->|3. Query: Áudios de hoje| Rep002
    Rep002 -->|4. Lista de Áudios| Auto
    Auto -->|5. Match por uniqueid| Auto
    Auto -->|6. POST Lead Events| SQS
    
    style Auto fill:#d1d5da
    style Rep086 fill:#e1e4e8
    style Rep002 fill:#e1e4e8
    style SQS fill:#e1e4e8
```

## 📦 Estrutura de Dados

```mermaid
classDiagram
    class Registro86 {
        +string dataHoraInicial
        +string uniqueid
        +string primaryuuid
        +string origem
        +string destino
        +string agenteId
        +string agente
        +string filaAtendimentoId
        +string filaAtendimento
        +string direcao
        +string fcr
        +string classificacaoId
        +string classificacao
        +string observacao
        +string linkAudio
    }
    
    class LeadEvent {
        +string origin
        +Registro86 payload
    }
    
    class SQS {
        +send(LeadEvent)
    }
    
    LeadEvent --> Registro86 : contém
    SQS ..> LeadEvent : processa
    
    note for Registro86 "linkAudio é adicionado\napós busca no Report 002"
```

## 🎯 Filas de Atendimento

```mermaid
graph TB
    subgraph FilasRecord["Filas Ação Record"]
        F1[Ação Record]
        F2[Transferencia Ação Record]
    end
    
    subgraph FilasNormais["Outras Filas"]
        F3[Fila Comercial]
        F4[Fila Suporte]
        F5[Outras...]
    end
    
    subgraph Origem["Origem do Lead"]
        O1[escallo_ligacao_record]
        O2[escallo_ligacao]
    end
    
    F1 --> O1
    F2 --> O1
    F3 --> O2
    F4 --> O2
    F5 --> O2
    
    style F1 fill:#d1d5da
    style F2 fill:#d1d5da
    style F3 fill:#e1e4e8
    style F4 fill:#e1e4e8
    style F5 fill:#e1e4e8
    style O1 fill:#d1d5da
    style O2 fill:#e1e4e8
```

## 🔢 Deduplicação por Número

```mermaid
sequenceDiagram
    participant Loop as Loop de Ligações
    participant Array as Array de Números
    participant Queue as Fila SQS
    
    Loop->>Array: Verificar se número existe
    
    alt Número Novo
        Array-->>Loop: Não existe
        Loop->>Loop: Processar lead
        Loop->>Queue: Enviar para fila
        Loop->>Array: Adicionar número
    else Número Duplicado
        Array-->>Loop: Já existe
        Loop->>Loop: Pular para próximo
    end
```

## 📊 Matching de Áudio

```mermaid
flowchart LR
    subgraph Ligacoes["Report 086 - Ligações"]
        L1[Ligação 1 - uniqueid: ABC123]
        L2[Ligação 2 - uniqueid: DEF456]
        L3[Ligação 3 - uniqueid: GHI789]
    end
    
    subgraph Audios["Report 002 - Áudios"]
        A1[Audio - ligacao.uniqueid: ABC123]
        A2[Audio - ligacao.uniqueid: GHI789]
    end
    
    subgraph Result["Resultado"]
        R1[Ligação 1 + linkAudio]
        R2[Ligação 2 - sem áudio]
        R3[Ligação 3 + linkAudio]
    end
    
    L1 -->|Match| A1
    L1 --> R1
    A1 --> R1
    
    L2 --> R2
    
    L3 -->|Match| A2
    L3 --> R3
    A2 --> R3
    
    style L1 fill:#f6f8fa
    style L2 fill:#f6f8fa
    style L3 fill:#f6f8fa
    style A1 fill:#e1e4e8
    style A2 fill:#e1e4e8
    style R1 fill:#d1d5da
    style R2 fill:#e1e4e8
    style R3 fill:#d1d5da
```

## 🚦 Filtros de Direção

```mermaid
flowchart TD
    subgraph Tipos["Tipos de Ligação"]
        T1[Entrada - Inbound]
        T2[Saída - Outbound]
    end
    
    subgraph Validacao["Validação"]
        V{direcao == Entrada?}
    end
    
    subgraph Resultado["Resultado"]
        R1[Processar como Lead]
        R2[Ignorar]
    end
    
    T1 --> V
    T2 --> V
    
    V -->|Sim| R1
    V -->|Não| R2
    
    style T1 fill:#d1d5da
    style T2 fill:#e1e4e8
    style V fill:#f6f8fa
    style R1 fill:#d1d5da
    style R2 fill:#e1e4e8
```

## 👥 Agentes Comerciais

```mermaid
mindmap
  root((Agentes Comerciais - 20))
    Grupo 1
      334
      261
      412
      409
      321 Gleice
    Grupo 2
      339 Pedro
      352 Caroline
      456 Keismi
      459 Karolyn
    Grupo 3
      431 Daniel
      345 Patrick
      446 Carina
      519 Rafaela
    Grupo 4
      525 Livia
      555 Ana Beatriz
      552 Thuane
      546 Luiz Felipe
    Grupo 5
      314 André
      573 Maria Clara
      585 Samara
```

## 🎯 Fluxo Completo Simplificado

```mermaid
flowchart TD
    A[Buscar Ligações e Áudios do Dia] --> B{Para cada ligação}
    B --> C{Agente Comercial OU Fila Record?}
    C -->|Não| D[Pular]
    C -->|Sim| E{Número já processado?}
    E -->|Sim| D
    E -->|Não| F{isLead válido?}
    F -->|Não| D
    F -->|Sim| G[Adicionar áudio]
    G --> H[Enviar para SQS]
    H --> I[Marcar número]
    I --> B
    D --> B
    B -->|Fim| J[Concluir]
    
    style A fill:#e1e4e8
    style H fill:#d1d5da
    style J fill:#e1e4e8
```

## 📊 Comparação com Escallo Chat

```mermaid
graph TB
    subgraph EscalloLigacao["Escallo Ligação"]
        EL1[Report 086 + 002]
        EL2[Apenas Entrada]
        EL3[Com áudio]
        EL4[Filas Record diferenciadas]
        EL5[20 agentes]
    end
    
    subgraph EscalloChat["Escallo Chat"]
        EC1[Report 087]
        EC2[Entrada e Saída]
        EC3[Sem áudio]
        EC4[Filas Record diferenciadas]
        EC5[Mesmos agentes]
    end
    
    style EL1 fill:#d1d5da
    style EL2 fill:#d1d5da
    style EL3 fill:#d1d5da
    style EL4 fill:#d1d5da
    style EL5 fill:#d1d5da
    style EC1 fill:#e1e4e8
    style EC2 fill:#e1e4e8
    style EC3 fill:#e1e4e8
    style EC4 fill:#e1e4e8
    style EC5 fill:#e1e4e8
```
