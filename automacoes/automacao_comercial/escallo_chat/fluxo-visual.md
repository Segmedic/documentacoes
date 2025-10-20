# Fluxo Visual - Escallo Chat

## 🔄 Diagrama Principal

```mermaid
flowchart TD
    Start([Início da Automação]) --> Init[Inicializar Cliente Escallo]
    
    Init --> GetData[Buscar Report 087 - Chats do Dia]
    
    GetData --> Process[Processar Registros]
    
    Process --> Loop{Para cada chat}
    
    Loop --> CheckAgent{É Agente Comercial?}
    
    CheckAgent -->|Não| Skip1[Pular para próximo]
    CheckAgent -->|Sim| CheckDuplicate{Contato já processado?}
    
    CheckDuplicate -->|Sim| Skip2[Pular para próximo]
    CheckDuplicate -->|Não| CheckAgentID{Tem ID de agente?}
    
    CheckAgentID -->|Não| Skip3[Pular para próximo]
    CheckAgentID -->|Sim| ValidateLead{isLead válido?}
    
    ValidateLead -->|Não| Skip4[Pular para próximo]
    ValidateLead -->|Sim| CreateLead[Criar Lead Event]
    
    CreateLead --> SetOrigin[origin = escallo_chat]
    
    SetOrigin --> SendQueue[Enviar para SQS]
    
    SendQueue --> MarkProcessed[Marcar contato como processado]
    
    MarkProcessed --> HasMore{Mais chats?}
    
    Skip1 --> HasMore
    Skip2 --> HasMore
    Skip3 --> HasMore
    Skip4 --> HasMore
    
    HasMore -->|Sim| Loop
    HasMore -->|Não| End([Fim])
    
    style Start fill:#e1e4e8
    style End fill:#e1e4e8
    style SendQueue fill:#d1d5da
    style CreateLead fill:#f6f8fa
    style ValidateLead fill:#f6f8fa
    style CheckAgent fill:#f6f8fa
```

## 📊 Validação de Lead (isLead)

```mermaid
flowchart LR
    subgraph Registro["Registro87 - Chat"]
        R[Dados do Chat]
    end
    
    subgraph Validacoes["Validações"]
        V1{Tem nome do agente?}
        V2{Tem nome do cliente?}
        V3{Tem valor de contato?}
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
    V3 -->|Vazio| NOK
    V3 -->|Preenchido| OK
    
    style R fill:#f6f8fa
    style V1 fill:#e1e4e8
    style V2 fill:#e1e4e8
    style V3 fill:#e1e4e8
    style OK fill:#d1d5da
    style NOK fill:#e1e4e8
```

## 🎯 Fluxo de Decisão Simplificado

```mermaid
flowchart TD
    subgraph Entrada["Entrada"]
        A[Chat do Report 087]
    end
    
    subgraph Validacoes["Validações"]
        B{Agente Comercial?}
        C{Contato duplicado?}
        D{Tem ID agente?}
        E{isLead válido?}
    end
    
    subgraph Saida["Saída"]
        F[Processar Lead]
        G[Ignorar]
    end
    
    A --> B
    B -->|Não| G
    B -->|Sim| C
    C -->|Sim| G
    C -->|Não| D
    D -->|Não| G
    D -->|Sim| E
    E -->|Não| G
    E -->|Sim| F
    
    style A fill:#f6f8fa
    style B fill:#e1e4e8
    style C fill:#e1e4e8
    style D fill:#e1e4e8
    style E fill:#e1e4e8
    style F fill:#d1d5da
    style G fill:#e1e4e8
```

## 🔄 Integração com Escallo

```mermaid
graph LR
    subgraph Automacao["Automação"]
        Auto[Escallo Chat]
    end
    
    subgraph Escallo["Sistema Escallo"]
        Rep087[(Report 087 - Chats)]
    end
    
    subgraph Queue["Fila"]
        SQS[(AWS SQS)]
    end
    
    Auto -->|1. Query: Chats de hoje| Rep087
    Rep087 -->|2. Lista de Registros| Auto
    Auto -->|3. POST Lead Events| SQS
    
    style Auto fill:#d1d5da
    style Rep087 fill:#e1e4e8
    style SQS fill:#e1e4e8
```

## 📦 Estrutura de Dados Completa

```mermaid
classDiagram
    class Registro87 {
        +number id
        +string dataHoraInicial
        +string dataHoraFinal
        +ClienteContato clienteContato
        +MidiaSocial midiaSocial
        +string protocolo
        +string direcao
        +string situacao
        +string status
        +Agente agente
        +MotivoInicial motivoInicial
        +Motivo motivo
        +Classificacao classificacao
        +string duracaoContato
        +number qtdeMensagens
        +Atendimentos atendimentos
    }
    
    class ClienteContato {
        +string id
        +string nome
        +string tipo
        +string valor
    }
    
    class Agente {
        +string id
        +string nome
        +string codigo
    }
    
    class LeadEvent {
        +string origin
        +Registro87 payload
    }
    
    Registro87 --> ClienteContato : contém
    Registro87 --> Agente : contém
    LeadEvent --> Registro87 : payload
    
    note for Registro87 "Dados completos do\natendimento por chat"
    note for ClienteContato "Pode ser telefone,\nemail ou WhatsApp"
```

## 💬 Canais de Atendimento

```mermaid
mindmap
  root((Escallo Chat))
    Canais Digitais
      Chat Web
      WhatsApp Business
      Facebook Messenger
      Instagram Direct
      Email
    Dados Capturados
      Nome Cliente
      Contato
      Protocolo
      Duração
      Qtd Mensagens
    Validações
      Agente Comercial
      Dados Completos
      Sem Duplicatas
```

## 🔢 Deduplicação por Contato

```mermaid
sequenceDiagram
    participant Loop as Loop de Chats
    participant Array as Array de Contatos
    participant Queue as Fila SQS
    
    Loop->>Array: Verificar se clienteContato.valor existe
    
    alt Contato Novo
        Array-->>Loop: Não existe
        Loop->>Loop: Processar lead
        Loop->>Queue: Enviar para fila
        Loop->>Array: Adicionar contato
    else Contato Duplicado
        Array-->>Loop: Já existe
        Loop->>Loop: Pular para próximo
    end
```

## 👥 Verificação de Agente Comercial

```mermaid
flowchart LR
    subgraph Input["Entrada"]
        A[agente.id do Chat]
    end
    
    subgraph Check["Verificação"]
        B{ID está em AGENTES_ESCALLO_RD?}
    end
    
    subgraph Output["Resultado"]
        C[Agente Comercial - Processar]
        D[Agente Comum - Ignorar]
    end
    
    A --> B
    B -->|Sim - 20 agentes| C
    B -->|Não| D
    
    style A fill:#f6f8fa
    style B fill:#e1e4e8
    style C fill:#d1d5da
    style D fill:#e1e4e8
```

## 🎯 Origem Única

```mermaid
graph TB
    subgraph ChatsValidos["Todos os Chats Válidos"]
        C1[Chat Comercial Normal]
        C2[Chat Ação Record]
        C3[Chat Transferência]
        C4[WhatsApp]
        C5[Facebook Messenger]
        C6[Instagram Direct]
    end
    
    subgraph Origem["Origem Única"]
        O[escallo_chat]
    end
    
    C1 --> O
    C2 --> O
    C3 --> O
    C4 --> O
    C5 --> O
    C6 --> O
    
    style C1 fill:#e1e4e8
    style C2 fill:#e1e4e8
    style C3 fill:#e1e4e8
    style C4 fill:#e1e4e8
    style C5 fill:#e1e4e8
    style C6 fill:#e1e4e8
    style O fill:#d1d5da
```

## 📊 Diferença: Chat vs Ligação

```mermaid
graph TB
    subgraph EscalloChat["Escallo Chat"]
        EC1[Report 087]
        EC2[Apenas Agentes Comerciais]
        EC3[Sem validação de direção]
        EC4[1 origem: escallo_chat]
        EC5[Sem áudio]
        EC6[Multicanal]
    end
    
    subgraph EscalloLigacao["Escallo Ligação"]
        EL1[Report 086 + 002]
        EL2[Agentes OU Filas Record]
        EL3[Apenas Entrada]
        EL4[2 origens: normal/record]
        EL5[Com áudio]
        EL6[Apenas telefone]
    end
    
    style EC1 fill:#d1d5da
    style EC2 fill:#d1d5da
    style EC3 fill:#d1d5da
    style EC4 fill:#d1d5da
    style EC5 fill:#d1d5da
    style EC6 fill:#d1d5da
    style EL1 fill:#e1e4e8
    style EL2 fill:#e1e4e8
    style EL3 fill:#e1e4e8
    style EL4 fill:#e1e4e8
    style EL5 fill:#e1e4e8
    style EL6 fill:#e1e4e8
```

## 🔍 Tipo de Contato

```mermaid
flowchart TD
    subgraph ClienteContato["clienteContato.valor"]
        A[Valor do Contato]
    end
    
    subgraph Tipos["Tipos Possíveis"]
        T1[Telefone: 11999999999]
        T2[Email: cliente@email.com]
        T3[WhatsApp: 5511999999999]
        T4[ID Social: @username]
    end
    
    subgraph Dedup["Deduplicação"]
        D[Array de Contatos Processados]
    end
    
    A --> T1
    A --> T2
    A --> T3
    A --> T4
    
    T1 --> D
    T2 --> D
    T3 --> D
    T4 --> D
    
    style A fill:#f6f8fa
    style T1 fill:#e1e4e8
    style T2 fill:#e1e4e8
    style T3 fill:#e1e4e8
    style T4 fill:#e1e4e8
    style D fill:#d1d5da
```

## 📊 Métricas do Chat

```mermaid
graph LR
    subgraph Registro["Registro87"]
        R[Dados do Chat]
    end
    
    subgraph Metricas["Métricas Disponíveis"]
        M1[duracaoContato]
        M2[qtdeMensagens]
        M3[dataHoraInicial]
        M4[dataHoraFinal]
    end
    
    subgraph Analise["Possíveis Análises"]
        A1[Tempo de Atendimento]
        A2[Engajamento]
        A3[Horário de Pico]
        A4[Qualidade do Lead]
    end
    
    R --> M1
    R --> M2
    R --> M3
    R --> M4
    
    M1 --> A1
    M2 --> A2
    M3 --> A3
    M4 --> A4
    
    style R fill:#f6f8fa
    style M1 fill:#e1e4e8
    style M2 fill:#e1e4e8
    style M3 fill:#e1e4e8
    style M4 fill:#e1e4e8
    style A1 fill:#d1d5da
    style A2 fill:#d1d5da
    style A3 fill:#d1d5da
    style A4 fill:#d1d5da
```

## 🎯 Fluxo Completo Simplificado

```mermaid
flowchart TD
    A[Buscar Chats do Dia] --> B{Para cada chat}
    B --> C{É Agente Comercial?}
    C -->|Não| D[Pular]
    C -->|Sim| E{Contato já processado?}
    E -->|Sim| D
    E -->|Não| F{Tem ID agente?}
    F -->|Não| D
    F -->|Sim| G{isLead válido?}
    G -->|Não| D
    G -->|Sim| H[Enviar para SQS]
    H --> I[Marcar contato]
    I --> B
    D --> B
    B -->|Fim| J[Concluir]
    
    style A fill:#e1e4e8
    style H fill:#d1d5da
    style J fill:#e1e4e8
```

## 📋 Checklist de Validação

```mermaid
flowchart TD
    subgraph Validacoes["Checklist de Validação"]
        V1[1. Agente é comercial?]
        V2[2. Contato não duplicado?]
        V3[3. Tem ID de agente?]
        V4[4. Tem nome do agente?]
        V5[5. Tem nome do cliente?]
        V6[6. Tem valor de contato?]
    end
    
    subgraph Resultado["Resultado"]
        OK[Todas OK - Enviar Lead]
        NOK[Alguma Falha - Ignorar]
    end
    
    V1 --> V2
    V2 --> V3
    V3 --> V4
    V4 --> V5
    V5 --> V6
    
    V6 -->|Todas OK| OK
    V1 -->|Falha| NOK
    V2 -->|Falha| NOK
    V3 -->|Falha| NOK
    V4 -->|Falha| NOK
    V5 -->|Falha| NOK
    V6 -->|Falha| NOK
    
    style V1 fill:#f6f8fa
    style V2 fill:#f6f8fa
    style V3 fill:#f6f8fa
    style V4 fill:#f6f8fa
    style V5 fill:#f6f8fa
    style V6 fill:#f6f8fa
    style OK fill:#d1d5da
    style NOK fill:#e1e4e8
```

## 🎯 Comparação: Todas as Automações

```mermaid
graph TB
    subgraph EscalloChat["Escallo Chat"]
        EC[Report 087 - Multicanal]
    end
    
    subgraph EscalloLigacao["Escallo Ligação"]
        EL[Report 086/002 - Telefone]
    end
    
    subgraph LeadsAmanha["Leads Amanhã"]
        LA[Feegow - Agendamentos D+1]
    end
    
    subgraph LeadsConvenios["Leads Convênios"]
        LC[Medula - Atendimentos D-1]
    end
    
    subgraph RecuperacaoAgd["Recuperação Agendamento"]
        RA[Feegow - Carrinhos D-1 a D+15]
    end
    
    subgraph RetencaoB2B["Retenção B2B"]
        RB[ClubFlex + Medula - Último Mês]
    end
    
    style EC fill:#d1d5da
    style EL fill:#e1e4e8
    style LA fill:#f6f8fa
    style LC fill:#e1e4e8
    style RA fill:#f6f8fa
    style RB fill:#e1e4e8
```
