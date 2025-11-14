# Fluxo da API - Segmedic Totem

## Visão Geral do Sistema

```mermaid
graph TB
    subgraph "Cliente"
        A[Totem de Autoatendimento]
    end
    
    subgraph "API Segmedic"
        B[API Rails]
        C[Banco de Dados PostgreSQL]
        D[Cache Redis]
    end
    
    subgraph "Serviços Externos"
        E[Feegow API]
        F[ClubFlex API]
        G[Itaú PIX API]
    end
    
    A -->|HTTP/JSON| B
    B -->|Consultas| C
    B -->|Cache| D
    B -->|Gestão Médica| E
    B -->|Convênio| F
    B -->|Pagamento PIX| G
    
    style A fill:#e1e1e1,stroke:#666,stroke-width:2px
    style B fill:#d1d1d1,stroke:#666,stroke-width:2px
    style C fill:#c1c1c1,stroke:#666,stroke-width:2px
    style D fill:#c1c1c1,stroke:#666,stroke-width:2px
    style E fill:#b1b1b1,stroke:#666,stroke-width:2px
    style F fill:#b1b1b1,stroke:#666,stroke-width:2px
    style G fill:#b1b1b1,stroke:#666,stroke-width:2px
```

---

## Fluxo 1: Agendamento com Convênio

```mermaid
sequenceDiagram
    participant T as Totem
    participant A as API
    participant F as Feegow
    participant C as ClubFlex
    
    T->>A: POST /api/patients/search?cpf=123
    A->>F: Buscar Paciente
    F-->>A: Dados do Paciente
    A-->>T: Retorna Paciente
    
    T->>A: GET /api/clubflex/check?cpf=123
    A->>C: Verificar Elegibilidade
    C-->>A: Status: OK
    A-->>T: Paciente Elegível
    
    T->>A: GET /api/specialties
    A->>F: Listar Especialidades
    F-->>A: Lista de Especialidades
    A-->>T: Retorna Especialidades
    
    T->>A: GET /api/professionals?especialidade_id=12
    A->>F: Buscar Profissionais
    F-->>A: Lista de Médicos
    A-->>T: Retorna Médicos
    
    T->>A: GET /api/appointments/available
    A->>F: Buscar Horários Disponíveis
    F-->>A: Horários Livres
    A-->>T: Retorna Horários
    
    T->>A: POST /api/appointments
    A->>F: Criar Agendamento
    F-->>A: Agendamento Criado
    A-->>T: Confirmação
    
    T->>A: POST /api/appointments/create_password
    A->>F: Gerar Senha Atendimento
    F-->>A: Senha Gerada
    A-->>T: PDF da Senha
```

---

## Fluxo 2: Agendamento Particular com Pagamento PIX

```mermaid
sequenceDiagram
    participant T as Totem
    participant A as API
    participant F as Feegow
    participant I as Itaú PIX
    
    T->>A: POST /api/patients/search?cpf=123
    A->>F: Buscar Paciente
    F-->>A: Paciente Não Encontrado
    A-->>T: Retorna 404
    
    T->>A: POST /api/patients
    A->>F: Criar Paciente
    F-->>A: Paciente Criado
    A-->>T: Cadastro OK
    
    T->>A: GET /api/specialties
    A->>F: Listar Especialidades
    F-->>A: Especialidades + Valores
    A-->>T: Retorna Lista
    
    T->>A: GET /api/appointments/available
    A->>F: Horários Disponíveis
    F-->>A: Horários
    A-->>T: Retorna Horários
    
    T->>A: POST /api/appointments
    A->>F: Criar Agendamento
    F-->>A: Agendamento ID
    A-->>T: Confirmação
    
    T->>A: POST /api/financials/create_invoice
    A->>F: Gerar Nota Fiscal
    F-->>A: Invoice ID
    A-->>T: Retorna Invoice
    
    T->>A: POST /api/financials/create_pix
    A->>I: Gerar QR Code PIX
    I-->>A: QR Code + TXID
    A-->>T: Exibe QR Code
    
    loop Verificar Pagamento
        T->>A: GET /api/financials/pix?txid=123
        A->>I: Consultar Status
        I-->>A: Status Pagamento
        A-->>T: Status Atualizado
    end
    
    T->>A: POST /api/financials/pay
    A->>F: Registrar Pagamento
    F-->>A: Pagamento Confirmado
    A-->>T: Sucesso
    
    T->>A: POST /api/appointments/create_password
    A->>F: Gerar Senha
    F-->>A: Senha
    A-->>T: PDF Senha
```

---

## Fluxo 3: Check-in do Paciente

```mermaid
sequenceDiagram
    participant T as Totem
    participant A as API
    participant F as Feegow
    
    T->>A: POST /api/patients/search?cpf=123
    A->>F: Buscar Paciente
    F-->>A: Dados Paciente
    A-->>T: Retorna Paciente
    
    T->>A: GET /api/appointments?paciente_id=678
    A->>F: Buscar Agendamentos do Dia
    F-->>A: Lista de Consultas
    A-->>T: Exibe Consultas
    
    T->>A: PATCH /api/appointments/update_status
    Note over T,A: {agendamento_id: 123, status_id: 2}
    A->>F: Atualizar Status para "Chegou"
    F-->>A: Status Atualizado
    A-->>T: Confirmação
    
    T->>A: POST /api/appointments/create_password
    A->>F: Gerar Senha Atendimento
    F-->>A: Senha Gerada
    A-->>T: PDF da Senha
    
    T->>A: POST /api/user_session_actions
    Note over T,A: Registrar ação de check-in
    A-->>T: Log Criado
```

---

## Fluxo 4: Cancelamento de Agendamento

```mermaid
sequenceDiagram
    participant T as Totem
    participant A as API
    participant F as Feegow
    
    T->>A: GET /api/patients/search?cpf=123
    A->>F: Buscar Paciente
    F-->>A: Dados Paciente
    A-->>T: Retorna Paciente
    
    T->>A: GET /api/appointments?paciente_id=678
    A->>F: Buscar Agendamentos Futuros
    F-->>A: Lista de Consultas
    A-->>T: Exibe Consultas
    
    T->>A: DELETE /api/appointments/123?reason_id=5
    A->>F: Cancelar Agendamento
    F-->>A: Agendamento Cancelado
    A-->>T: Confirmação
    
    T->>A: POST /api/user_session_actions
    Note over T,A: Registrar cancelamento
    A-->>T: Log Criado
```

---

## Fluxo 5: Pagamento com Cartão de Crédito

```mermaid
sequenceDiagram
    participant T as Totem
    participant A as API
    participant F as Feegow
    
    Note over T: Agendamento já criado
    
    T->>A: POST /api/financials/create_invoice
    A->>F: Gerar Nota Fiscal
    F-->>A: Invoice ID + Valor
    A-->>T: Dados Cobrança
    
    Note over T: Paciente insere cartão
    Note over T: Terminal processa pagamento
    
    T->>A: POST /api/financials/pay
    Note over T,A: Dados da transação do cartão
    A->>F: Registrar Pagamento
    F-->>A: Pagamento Confirmado
    A-->>T: Sucesso
    
    T->>A: GET /api/appointments/coupon
    Note over T,A: Gerar comprovante
    A-->>T: PDF do Cupom
```

---

## Fluxo de Dados Internos

```mermaid
graph LR
    subgraph "Controllers"
        A1[AppointmentsController]
        A2[PatientsController]
        A3[FinancialsController]
        A4[ClubflexController]
    end
    
    subgraph "Services"
        B1[FeegowService]
        B2[ClubflexService]
        B3[ItauService]
        B4[EmissaoCupomService]
        B5[EmissaoSenhaService]
    end
    
    subgraph "Models"
        C1[User]
        C2[UserSession]
        C3[PaymentLog]
        C4[ServiceLog]
    end
    
    A1 --> B1
    A1 --> B4
    A1 --> B5
    A2 --> B1
    A3 --> B1
    A3 --> B3
    A4 --> B2
    
    B1 --> C3
    B1 --> C4
    B2 --> C4
    B3 --> C3
    
    A1 --> C2
    A2 --> C1
    
    style A1 fill:#e1e1e1,stroke:#666,stroke-width:2px
    style A2 fill:#e1e1e1,stroke:#666,stroke-width:2px
    style A3 fill:#e1e1e1,stroke:#666,stroke-width:2px
    style A4 fill:#e1e1e1,stroke:#666,stroke-width:2px
    style B1 fill:#d1d1d1,stroke:#666,stroke-width:2px
    style B2 fill:#d1d1d1,stroke:#666,stroke-width:2px
    style B3 fill:#d1d1d1,stroke:#666,stroke-width:2px
    style B4 fill:#d1d1d1,stroke:#666,stroke-width:2px
    style B5 fill:#d1d1d1,stroke:#666,stroke-width:2px
    style C1 fill:#c1c1c1,stroke:#666,stroke-width:2px
    style C2 fill:#c1c1c1,stroke:#666,stroke-width:2px
    style C3 fill:#c1c1c1,stroke:#666,stroke-width:2px
    style C4 fill:#c1c1c1,stroke:#666,stroke-width:2px
```

---

## Arquitetura de Integração

```mermaid
graph TB
    subgraph "Frontend - Totem"
        UI[Interface do Usuário]
    end
    
    subgraph "Backend - API Rails"
        AUTH[Autenticação JWT]
        CTRL[Controllers]
        SVC[Services Layer]
        MDL[Models/Database]
        CACHE[Redis Cache]
        QUEUE[Sidekiq Jobs]
    end
    
    subgraph "APIs Externas"
        FEEGOW[Feegow<br/>Sistema de Gestão]
        CLUB[ClubFlex<br/>Convênio]
        PIX[Itaú<br/>Pagamento PIX]
    end
    
    UI -->|JSON/HTTP| AUTH
    AUTH -->|Token Válido| CTRL
    CTRL --> SVC
    SVC --> MDL
    SVC --> CACHE
    SVC --> QUEUE
    
    SVC -->|REST API| FEEGOW
    SVC -->|REST API| CLUB
    SVC -->|REST API| PIX
    
    FEEGOW -.->|Logs| MDL
    CLUB -.->|Logs| MDL
    PIX -.->|Logs| MDL
    
    style UI fill:#e1e1e1,stroke:#666,stroke-width:2px
    style AUTH fill:#d1d1d1,stroke:#666,stroke-width:2px
    style CTRL fill:#d1d1d1,stroke:#666,stroke-width:2px
    style SVC fill:#c1c1c1,stroke:#666,stroke-width:2px
    style MDL fill:#b1b1b1,stroke:#666,stroke-width:2px
    style CACHE fill:#b1b1b1,stroke:#666,stroke-width:2px
    style QUEUE fill:#b1b1b1,stroke:#666,stroke-width:2px
    style FEEGOW fill:#a1a1a1,stroke:#666,stroke-width:2px
    style CLUB fill:#a1a1a1,stroke:#666,stroke-width:2px
    style PIX fill:#a1a1a1,stroke:#666,stroke-width:2px
```

---

## Ciclo de Vida de um Agendamento

```mermaid
stateDiagram-v2
    [*] --> Iniciado: Paciente acessa totem
    Iniciado --> BuscandoPaciente: Informa CPF
    BuscandoPaciente --> PacienteEncontrado: Encontrado
    BuscandoPaciente --> CadastrandoPaciente: Não encontrado
    CadastrandoPaciente --> PacienteEncontrado: Cadastro completo
    
    PacienteEncontrado --> VerificandoConvenio: Possui convênio?
    VerificandoConvenio --> EscolhendoEspecialidade: Sim, elegível
    VerificandoConvenio --> EscolhendoEspecialidade: Não, particular
    
    EscolhendoEspecialidade --> EscolhendoMedico: Especialidade selecionada
    EscolhendoMedico --> EscolhendoHorario: Médico selecionado
    EscolhendoHorario --> AgendamentoCriado: Horário confirmado
    
    AgendamentoCriado --> GerandoPagamento: Particular
    AgendamentoCriado --> GerandoSenha: Convênio
    
    GerandoPagamento --> ProcessandoPagamento: Invoice criada
    ProcessandoPagamento --> PagamentoAprovado: Pagamento OK
    ProcessandoPagamento --> PagamentoRecusado: Falha
    PagamentoRecusado --> GerandoPagamento: Tentar novamente
    
    PagamentoAprovado --> GerandoSenha: Pagamento confirmado
    GerandoSenha --> SenhaEmitida: Senha gerada
    SenhaEmitida --> [*]: Processo completo
```

---

## Modelo de Dados Simplificado

```mermaid
erDiagram
    USER ||--o{ USER_SESSION : creates
    USER_SESSION ||--o{ USER_SESSION_SCREEN : has
    USER_SESSION ||--o{ USER_SESSION_ACTION : has
    USER_SESSION ||--o{ PAYMENT_LOG : generates
    USER_SESSION ||--o{ SERVICE_LOG : generates
    
    USER {
        int id
        string email
        datetime created_at
    }
    
    USER_SESSION {
        int id
        int user_id
        string schedule_kind
        int convenio_id
        datetime created_at
    }
    
    USER_SESSION_SCREEN {
        int id
        int user_session_id
        string screen_name
        string action
        datetime created_at
    }
    
    USER_SESSION_ACTION {
        int id
        int user_session_id
        string action
        string result
        datetime created_at
    }
    
    PAYMENT_LOG {
        int id
        int user_id
        string invoice_id
        int agendamento_id
        string kind
        decimal valor
        string payment_method
        datetime created_at
    }
    
    SERVICE_LOG {
        int id
        string origin
        string provider
        json params
        json response
        int status
        string method
        string url_path
        datetime requested_at
    }
```

---

## Fluxo de Autenticação e Sessão

```mermaid
sequenceDiagram
    participant T as Totem
    participant A as API
    participant DB as Database
    
    T->>A: POST /api/sign_in
    Note over T,A: {email, password}
    A->>DB: Validar Credenciais
    DB-->>A: Usuário Válido
    A->>A: Gerar JWT Token
    A-->>T: Retorna Token
    
    Note over T: Armazena token localmente
    
    T->>A: GET /api/specialties
    Note over T,A: Authorization: Bearer {token}
    A->>A: Validar Token
    A->>DB: Buscar Dados
    DB-->>A: Dados
    A-->>T: Retorna Response
    
    Note over T: Token expira após X horas
    
    T->>A: GET /api/appointments
    Note over T,A: Token expirado
    A-->>T: 401 Unauthorized
    T->>A: POST /api/sign_in
    Note over T: Renova autenticação
```

---

## Performance e Cache

```mermaid
graph LR
    subgraph "Request Flow"
        A[Request] --> B{Cache?}
        B -->|Hit| C[Retorna do Redis]
        B -->|Miss| D[Consulta Feegow]
        D --> E[Salva no Redis]
        E --> F[Retorna Response]
    end
    
    subgraph "Cache Strategy"
        G[Especialidades<br/>TTL: 24h]
        H[Unidades<br/>TTL: 24h]
        I[Profissionais<br/>TTL: 6h]
        J[Horários<br/>TTL: 5min]
    end
    
    style A fill:#e1e1e1,stroke:#666,stroke-width:2px
    style B fill:#d1d1d1,stroke:#666,stroke-width:2px
    style C fill:#c1c1c1,stroke:#666,stroke-width:2px
    style D fill:#b1b1b1,stroke:#666,stroke-width:2px
    style E fill:#b1b1b1,stroke:#666,stroke-width:2px
    style F fill:#c1c1c1,stroke:#666,stroke-width:2px
    style G fill:#a1a1a1,stroke:#666,stroke-width:2px
    style H fill:#a1a1a1,stroke:#666,stroke-width:2px
    style I fill:#a1a1a1,stroke:#666,stroke-width:2px
    style J fill:#a1a1a1,stroke:#666,stroke-width:2px
```

---

## Tratamento de Erros

```mermaid
graph TD
    A[Request] --> B{Validação}
    B -->|Inválido| C[400 Bad Request]
    B -->|Válido| D{Autenticação}
    D -->|Falha| E[401 Unauthorized]
    D -->|OK| F{Chamada Externa}
    
    F -->|Feegow Error| G{Retry?}
    F -->|ClubFlex Error| G
    F -->|PIX Error| G
    F -->|Sucesso| H[200 OK]
    
    G -->|Sim| I[Tentar Novamente]
    G -->|Não| J[500 Server Error]
    I --> F
    
    C --> K[Log Error]
    E --> K
    J --> K
    K --> L[ServiceLog Table]
    
    style A fill:#e1e1e1,stroke:#666,stroke-width:2px
    style B fill:#d1d1d1,stroke:#666,stroke-width:2px
    style D fill:#d1d1d1,stroke:#666,stroke-width:2px
    style F fill:#c1c1c1,stroke:#666,stroke-width:2px
    style G fill:#b1b1b1,stroke:#666,stroke-width:2px
    style H fill:#a1a1a1,stroke:#666,stroke-width:2px
    style C fill:#a1a1a1,stroke:#666,stroke-width:2px
    style E fill:#a1a1a1,stroke:#666,stroke-width:2px
    style J fill:#a1a1a1,stroke:#666,stroke-width:2px
```
