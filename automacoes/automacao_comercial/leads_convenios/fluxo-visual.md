# Fluxo Visual - Leads de Convênios

## 🔄 Diagrama Principal

```mermaid
flowchart TD
    Start([Início da Automação]) --> Init[Inicializar Conexão Medula]
    
    Init --> CalcDate[Calcular Data de Ontem]
    
    CalcDate --> Query[Executar Query no Data Warehouse]
    
    Query --> CheckResult{Encontrou leads?}
    
    CheckResult -->|Não| Error[Lançar Erro: Não há leads de convenio]
    CheckResult -->|Sim| ProcessLeads[Processar Lista de Pacientes]
    
    ProcessLeads --> Loop{Para cada paciente}
    
    Loop --> CreateEvent[Criar Lead Event]
    
    CreateEvent --> SendQueue[Enviar para Fila SQS]
    
    SendQueue --> HasMore{Mais pacientes?}
    
    HasMore -->|Sim| Loop
    HasMore -->|Não| End([Fim])
    
    Error --> EndError([Fim com Erro])
    
    style Start fill:#e1e4e8
    style End fill:#e1e4e8
    style EndError fill:#d1d5da
    style SendQueue fill:#d1d5da
    style Query fill:#f6f8fa
    style CheckResult fill:#f6f8fa
```

## 📊 Query e Filtros no Data Warehouse

```mermaid
flowchart TD
    subgraph DataWarehouse["Data Warehouse Medula - PostgreSQL"]
        DW1[(feegow_agendamentos_dw)]
        DW2[(feegow_atendimentos_dw)]
    end
    
    subgraph Filtros["Filtros SQL"]
        F1[Data = Ontem]
        F2[Tem Convênio Preenchido]
        F3[NÃO é Bradesco]
        F4[NÃO é Mediservice]
        F5[NÃO é Visita]
        F6[DISTINCT ON CPF]
    end
    
    subgraph Resultado["Resultado"]
        R[Lista de Pacientes Únicos com Convênio]
    end
    
    DW1 -->|INNER JOIN| DW2
    DW2 --> F1
    F1 --> F2
    F2 --> F3
    F3 --> F4
    F4 --> F5
    F5 --> F6
    F6 --> R
    
    style DW1 fill:#e1e4e8
    style DW2 fill:#e1e4e8
    style F1 fill:#f6f8fa
    style F2 fill:#f6f8fa
    style F3 fill:#f6f8fa
    style F4 fill:#f6f8fa
    style F5 fill:#f6f8fa
    style F6 fill:#d1d5da
    style R fill:#d1d5da
```

## 🎯 Filtros de Convênio

```mermaid
flowchart LR
    subgraph Entrada["Entrada"]
        A[Todos os Atendimentos de Ontem]
    end
    
    subgraph Validacoes["Validações"]
        B{Tem Convênio?}
        C{É Bradesco?}
        D{É Mediservice?}
    end
    
    subgraph Saida["Saída"]
        E[Aceitar Lead]
        F[Rejeitar Lead]
    end
    
    A --> B
    B -->|Não| F
    B -->|Sim| C
    C -->|Sim| F
    C -->|Não| D
    D -->|Sim| F
    D -->|Não| E
    
    style A fill:#f6f8fa
    style B fill:#fafbfc
    style C fill:#fafbfc
    style D fill:#fafbfc
    style E fill:#d1d5da
    style F fill:#e1e4e8
```

## 📅 Janela de Tempo

```mermaid
gantt
    title Período de Busca dos Atendimentos
    dateFormat YYYY-MM-DD
    axisFormat %d/%m
    
    section Período
    Ontem (D-1)                 :milestone, m1, 2024-01-10, 0d
    Dia da Busca                :active, 2024-01-10, 1d
    Hoje (Execução)             :crit, milestone, m2, 2024-01-11, 0d
```

## 🔄 INNER JOIN - Garantia de Atendimento

```mermaid
flowchart LR
    subgraph Agendamentos["Agendamentos"]
        A1[Paciente agendou]
        A2[Status: Agendado]
    end
    
    subgraph Atendimentos["Atendimentos"]
        B1[Paciente compareceu]
        B2[Atendimento realizado]
    end
    
    subgraph Join["INNER JOIN"]
        C[Somente quem AGENDOU E COMPARECEU]
    end
    
    subgraph Resultado["Resultado"]
        D[Leads Qualificados]
    end
    
    A1 --> Join
    A2 --> Join
    B1 --> Join
    B2 --> Join
    Join --> D
    
    style A1 fill:#f6f8fa
    style A2 fill:#f6f8fa
    style B1 fill:#f6f8fa
    style B2 fill:#f6f8fa
    style Join fill:#d1d5da
    style D fill:#d1d5da
```

## 🔄 Integração com Sistemas

```mermaid
graph LR
    subgraph Automacao["Automação"]
        Auto[Leads de Convênios]
    end
    
    subgraph DataWarehouse["Data Warehouse"]
        Medula[(Medula PostgreSQL)]
    end
    
    subgraph Queue["Fila"]
        SQS[(AWS SQS)]
    end
    
    Auto -->|Query SQL: Atendimentos D-1| Medula
    Medula -->|Lista de Pacientes| Auto
    Auto -->|POST Lead Event| SQS
    
    style Auto fill:#d1d5da
    style Medula fill:#e1e4e8
    style SQS fill:#e1e4e8
```

## 📦 Estrutura de Dados

```mermaid
classDiagram
    class PatientConvenio {
        +string NomePaciente
        +string CPF
        +string email1
        +string nomesexo
        +string data
        +string nomeprocedimento
        +string nomeprofissional
        +string nomeunidade
        +string nomeconvenio
        +string nascimento
        +string Cel1
    }
    
    class LeadEvent {
        +string origin
        +PatientConvenio payload
    }
    
    class SQS {
        +send(LeadEvent)
    }
    
    LeadEvent --> PatientConvenio : contém
    SQS ..> LeadEvent : processa
    
    note for PatientConvenio "Dados extraídos do\nData Warehouse Medula"
```

## 🚦 Deduplicação via SQL

```mermaid
flowchart TD
    subgraph Entrada["Múltiplos Registros"]
        A[CPF 123 - Consulta 1]
        B[CPF 123 - Consulta 2]
        C[CPF 456 - Exame 1]
    end
    
    subgraph SQL["DISTINCT ON CPF"]
        D[Agrupa por CPF]
        E[ORDER BY data]
        F[Mantém primeiro registro]
    end
    
    subgraph Saida["Registros Únicos"]
        G[CPF 123 - Consulta 1]
        H[CPF 456 - Exame 1]
    end
    
    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    F --> G
    F --> H
    
    style A fill:#f6f8fa
    style B fill:#f6f8fa
    style C fill:#f6f8fa
    style D fill:#e1e4e8
    style E fill:#e1e4e8
    style F fill:#e1e4e8
    style G fill:#d1d5da
    style H fill:#d1d5da
```

## 🎯 Convênios Aceitos vs Rejeitados

```mermaid
mindmap
  root((Convênios))
    Aceitos
      Unimed
      Amil
      SulAmerica
      Porto Seguro
      Outros não excluídos
    Rejeitados
      Bradesco
        Bradesco Saúde
        Bradesco Seguros
      Mediservice
        Mediservice Premium
        Mediservice Basic
```

## 📊 Fluxo de Processamento Simplificado

```mermaid
sequenceDiagram
    participant Auto as Automação
    participant Medula as Data Warehouse
    participant SQS as Fila SQS
    
    Auto->>Auto: Calcular data de ontem
    Auto->>Medula: Query: atendimentos com convênio
    Medula-->>Auto: Lista de pacientes (DISTINCT)
    
    alt Lista vazia
        Auto->>Auto: Lançar erro
    else Lista com dados
        loop Para cada paciente
            Auto->>Auto: Criar LeadEvent
            Auto->>SQS: Enviar lead
        end
    end
    
    Auto->>Auto: Finalizar
```

## 🔍 Validações de Procedimento

```mermaid
flowchart LR
    subgraph Procedimentos["Tipos de Procedimento"]
        P1[Consulta Médica]
        P2[Exame Laboratorial]
        P3[Procedimento Cirúrgico]
        P4[Visita Representante]
    end
    
    subgraph Validacao["Validação"]
        V{NOT LIKE Visita?}
    end
    
    subgraph Resultado["Resultado"]
        R1[Aceitar]
        R2[Rejeitar]
    end
    
    P1 --> V
    P2 --> V
    P3 --> V
    P4 --> V
    
    V -->|Consulta| R1
    V -->|Exame| R1
    V -->|Cirurgia| R1
    V -->|Visita| R2
    
    style P1 fill:#f6f8fa
    style P2 fill:#f6f8fa
    style P3 fill:#f6f8fa
    style P4 fill:#f6f8fa
    style V fill:#e1e4e8
    style R1 fill:#d1d5da
    style R2 fill:#e1e4e8
```

## 🎯 Comparação com Outras Automações

```mermaid
graph TB
    subgraph LeadsConvenios["Leads Convênios"]
        LC1[Atendimentos D-1]
        LC2[COM convênio]
        LC3[Sem verificações adicionais]
    end
    
    subgraph LeadsAmanha["Leads Amanhã"]
        LA1[Agendamentos D+1]
        LA2[SEM convênio]
        LA3[Verifica CRM + ClubFlex]
    end
    
    subgraph RecuperacaoAgd["Recuperação Agendamento"]
        RA1[Carrinhos D-1 até D+15]
        RA2[SEM convênio]
        RA3[Verifica CRM + ClubFlex + Especialidade]
    end
    
    style LC1 fill:#d1d5da
    style LC2 fill:#d1d5da
    style LC3 fill:#d1d5da
    style LA1 fill:#e1e4e8
    style LA2 fill:#e1e4e8
    style LA3 fill:#e1e4e8
    style RA1 fill:#f6f8fa
    style RA2 fill:#f6f8fa
    style RA3 fill:#f6f8fa
```
