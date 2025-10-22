# Arquitetura Geral do Sistema

## 📋 Visão Geral

Sistema de automação de contatos para o time de cobranças e relacionamento com clientes, utilizando AWS Lambda com execução agendada via EventBridge (Cron).

## 🏗️ Arquitetura em Camadas

```mermaid
flowchart TB
    subgraph AWS["☁️ AWS Cloud"]
        EB[EventBridge<br/>Cron Schedule]
        Lambda[Lambda Functions]
    end
    
    subgraph App["📦 Aplicação"]
        Controllers[Controllers<br/>Handlers]
        UseCases[Use Cases<br/>Regras de Negócio]
        Repos[Repositories<br/>Integrações]
    end
    
    subgraph External["🌐 APIs Externas"]
        CRM[RD Station CRM]
        Escallo[Escallo<br/>Dialer + WhatsApp]
    end
    
    EB -->|Trigger| Lambda
    Lambda --> Controllers
    Controllers --> UseCases
    UseCases --> Repos
    Repos --> CRM
    Repos --> Escallo
    
    style AWS fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style App fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style External fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 🔄 Fluxo Geral de Execução

```mermaid
flowchart LR
    A[⏰ Cron Trigger] --> B[🔧 Lambda Handler]
    B --> C[📊 Busca Leads<br/>no CRM]
    C --> D[🔄 Processa e<br/>Normaliza]
    D --> E{Tipo}
    E -->|Telefone| F[📞 Escallo<br/>Dialer]
    E -->|WhatsApp| G[💬 Escallo<br/>WhatsApp]
    
    style A fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style B fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style C fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style D fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style E fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style F fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
    style G fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 📂 Estrutura de Código

```mermaid
graph TB
    subgraph Controllers["Controllers"]
        C1[campaign-dialer-call]
        C2[campaign-send-message]
        C3[webhook-send-message-whatsapp]
    end
    
    subgraph UseCases["Use Cases"]
        U1[dialer-leads]
        U2[get-all-deals-by-stage]
        U3[send-message-whatsapp]
    end
    
    subgraph Repos["Repositories"]
        R1[rd-crm-charge-team]
        R2[rd-crm-rc-team]
        R3[escallo-dialer]
        R4[escallo-whatsapp]
    end
    
    C1 --> U1
    C1 --> U2
    C2 --> U2
    C2 --> U3
    C3 --> U3
    
    U1 --> R3
    U2 --> R1
    U2 --> R2
    U3 --> R4
    
    style Controllers fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style UseCases fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style Repos fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 🎯 Tipos de Automação

```mermaid
flowchart TB
    Start[Sistema de Automações]
    
    Start --> Dialer[📞 Discagem Telefônica]
    Start --> WhatsApp[💬 Mensagens WhatsApp]
    
    Dialer --> D1[pipe-1-60]
    Dialer --> D2[pipe-61-180]
    Dialer --> D3[pipe-181-365]
    Dialer --> D4[pipe-366]
    
    WhatsApp --> W1[laboratory-proposal]
    WhatsApp --> W2[general-proposal]
    
    style Start fill:#e1e1e1,stroke:#666,stroke-width:3px,color:#000
    style Dialer fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style WhatsApp fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style D1 fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style D2 fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style D3 fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style D4 fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style W1 fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style W2 fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
```

## 🔌 Integrações Externas

```mermaid
flowchart LR
    App[Aplicação Lambda]
    
    App -->|GET /deals| CRM1[RD Station CRM<br/>Charges Team]
    App -->|GET /deals| CRM2[RD Station CRM<br/>RC Team]
    App -->|POST /campanha/telefonia| ESC1[Escallo<br/>Dialer API]
    App -->|POST /campanha/texto| ESC2[Escallo<br/>WhatsApp API]
    
    style App fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style CRM1 fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style CRM2 fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style ESC1 fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
    style ESC2 fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 🔐 Inversão de Dependências

```mermaid
flowchart TB
    subgraph UseCase["Use Case"]
        UC[DialerLeads<br/>ou<br/>SendMessageWhatsapp]
    end
    
    subgraph Interface["Interface"]
        I1[DialerRepository]
        I2[WhatsappRepository]
    end
    
    subgraph Implementation["Implementação"]
        Impl1[EscalloDialerRepository]
        Impl2[EscalloWhatsappRepository]
    end
    
    UC -.depende de.-> I1
    UC -.depende de.-> I2
    I1 -.implementa.-> Impl1
    I2 -.implementa.-> Impl2
    
    style UseCase fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style Interface fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style Implementation fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 📊 Fluxo de Dados: Discador

```mermaid
sequenceDiagram
    participant E as EventBridge
    participant L as Lambda
    participant C as RD CRM
    participant D as Escallo Dialer
    
    E->>L: Trigger (Cron)
    L->>L: Valida Pipeline ID
    L->>C: Busca Deals por Estágio
    C-->>L: Retorna Lista de Deals
    L->>L: Normaliza Telefones
    L->>L: Remove Duplicatas
    L->>D: Envia Lista de Contatos
    D-->>L: Confirma Recebimento
    L-->>E: Execução Concluída
```

## 📊 Fluxo de Dados: WhatsApp

```mermaid
sequenceDiagram
    participant E as EventBridge
    participant L as Lambda
    participant C as RD CRM
    participant W as Escallo WhatsApp
    
    E->>L: Trigger (Cron Diário)
    L->>L: Valida Pipeline ID
    L->>C: Busca Deals de Hoje
    C-->>L: Retorna Lista de Deals
    L->>L: Normaliza Telefones
    L->>L: Remove Duplicatas
    L->>W: Envia Lista de Mensagens
    W-->>L: Confirma Recebimento
    L-->>E: Execução Concluída
```

## ⚙️ Componentes Principais

| Componente | Responsabilidade | Tecnologia |
|------------|------------------|------------|
| **EventBridge** | Agendamento de execuções | AWS EventBridge (Cron) |
| **Lambda Functions** | Execução das automações | Node.js 22 + TypeScript |
| **Controllers** | Orquestração do fluxo | TypeScript |
| **Use Cases** | Lógica de negócio | TypeScript |
| **Repositories** | Integração com APIs | Axios + TypeScript |
| **CRM** | Fonte de dados de leads | RD Station CRM API |
| **Dialer/WhatsApp** | Envio de contatos | Escallo API |

## 🔄 Pipeline CI/CD

```mermaid
flowchart LR
    A[Git Push] --> B{Branch?}
    B -->|Feature| C[Run Tests<br/>CI]
    B -->|Main| D[Run Tests<br/>CI]
    D --> E[Deploy AWS<br/>CD]
    E --> F[Lambda<br/>Atualizada]
    
    style A fill:#e1e1e1,stroke:#666,stroke-width:2px,color:#000
    style B fill:#f5f5f5,stroke:#888,stroke-width:2px,color:#000
    style C fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style D fill:#e8e8e8,stroke:#777,stroke-width:2px,color:#000
    style E fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
    style F fill:#d4d4d4,stroke:#666,stroke-width:2px,color:#000
```

## 📝 Princípios Arquiteturais

### Clean Architecture

- **Independência de Frameworks**: Use cases não dependem de bibliotecas externas
- **Testabilidade**: Lógica de negócio testável sem dependências externas
- **Independência de UI**: Não há interface, apenas APIs
- **Independência de Banco**: Usa APIs externas como fonte de dados
- **Inversão de Dependências**: Interfaces definem contratos

### SOLID

- **S** - Single Responsibility: Cada classe tem uma única responsabilidade
- **O** - Open/Closed: Aberto para extensão, fechado para modificação
- **L** - Liskov Substitution: Implementações intercambiáveis via interfaces
- **I** - Interface Segregation: Interfaces específicas e enxutas
- **D** - Dependency Inversion: Dependências através de abstrações

## 🎯 Padrões Utilizados

- **Repository Pattern**: Abstração de acesso a dados externos
- **Factory Pattern**: Criação de instâncias de use cases
- **Dependency Injection**: Injeção de dependências nos construtores
- **Strategy Pattern**: Diferentes estratégias de busca (charges vs rc team)

## 📚 Documentações Detalhadas

- **[Campaign Dialer Call](./campaign-dialer-call/)** - Discagem automática
- **[Laboratory Proposal Message](./laboratory-proposal-message/)** - WhatsApp Lab
- **[General Proposal Message](./general-proposal-message/)** - WhatsApp Geral

---

📖 **Este documento fornece uma visão geral da arquitetura. Para detalhes de cada automação, consulte as pastas específicas.**
