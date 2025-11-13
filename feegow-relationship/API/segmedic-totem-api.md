# Documentação de Integração - API Feegow

## Visão Geral
Esta API integra-se com o sistema Feegow para gerenciar agendamentos, pacientes, profissionais e processos financeiros de uma clínica médica.

---

## Configuração

### Variáveis de Ambiente Necessárias
- `FEEGOW_API_ENDPOINT`: URL base da API Feegow
- `FEEGOW_API_TOKEN`: Token de autenticação (x-access-token)

### Autenticação
Todas as requisições para a API Feegow incluem:
```ruby
headers: {
  'Content-Type': 'application/json',
  'x-access-token': ENV['FEEGOW_API_TOKEN']
}
```

---

## Endpoints Utilizados

### 1. Agendamentos (Appointments)

#### `POST appoints/new-appoint`
**Objetivo**: Criar novo agendamento  
**Usado em**: `FeegowService#create_appointment`  
**Parâmetros**:
- `local_id`: ID do local de atendimento
- `paciente_id`: ID do paciente
- `profissional_id`: ID do profissional
- `especialidade_id`: ID da especialidade
- `procedimento_id`: ID do procedimento
- `data`: Data do agendamento
- `horario`: Horário do agendamento
- `valor`: Valor do procedimento
- `plano`: Nome do plano (se houver)
- `convenio_id`: ID do convênio (opcional)
- `notas`: Sempre inclui "#agTOTEM" para identificação

**Retorna**: ID do agendamento criado

---

#### `GET appoints/search`
**Objetivo**: Buscar agendamentos por filtros  
**Usado em**: 
- `FeegowService#show_appointment` - Um agendamento específico
- `FeegowService#list_appointments` - Lista de agendamentos

**Parâmetros**:
- `agendamento_id`: ID específico (opcional)
- `paciente_id`: Buscar por paciente (opcional)
- `data_start`: Data inicial (opcional)
- `data_end`: Data final (opcional)
- `list_procedures`: Incluir procedimentos (opcional)

**Retorna**: Agendamento(s) com dados completos

---

#### `GET v2/appoints/available-schedule`
**Objetivo**: Listar horários disponíveis para agendamento  
**Usado em**: `FeegowService#list_available_appointments`  
**Parâmetros**:
- `data_start`: Data inicial
- `data_end`: Data final
- `unidade_id`: ID da unidade
- `convenio_id`: ID do convênio (opcional)
- `tipo`: 'P' para procedimento ou 'E' para especialidade
- `procedimento_id` ou `especialidade_id`: Conforme o tipo
- `profissional_id`: Filtrar por profissional (opcional)

**Lógica adicional**: Filtra horários disponíveis nas próximas 3 horas a partir do momento atual

---

#### `POST appoints/cancel-appoint`
**Objetivo**: Cancelar agendamento  
**Usado em**: `FeegowService.cancel_appointment` (método de classe)  
**Parâmetros**:
- `agendamento_id`: ID do agendamento
- `motivo_id`: ID do motivo do cancelamento

---

#### `POST appoints/reschedule`
**Objetivo**: Reagendar consulta  
**Usado em**: `FeegowService.reschedule_appointment` (método de classe)  
**Parâmetros**:
- `agendamento_id`: ID do agendamento
- `motivo_id`: ID do motivo
- `horario`: Novo horário
- `data`: Nova data

---

#### `POST appoints/statusUpdate`
**Objetivo**: Atualizar status do agendamento  
**Usado em**: `FeegowService#update_status_appointment`  
**Parâmetros**:
- `AgendamentoID`: ID do agendamento
- `StatusID`: Novo status
- `HoraChegada`: Hora da chegada (automático)

---

#### `GET appoints/queue-position`
**Objetivo**: Gerar senha de atendimento  
**Usado em**: `FeegowService#create_password`  
**Parâmetros**:
- `unidade_id`: ID da unidade
- `tipo_senha`: Tipo de senha baseado na situação

**Tipos de senha**:
- 102: Padrão
- 19: Clubflex
- 103: Exames
- 104: Outro
- 107: Orçamentos
- 108: Pagamento Particular
- 109: Convênio
- 112: Atendimento de Revisão

---

### 2. Pacientes (Patients)

#### `GET patient/search`
**Objetivo**: Buscar dados de um paciente  
**Usado em**: 
- `FeegowService#show_patient`
- `Api::SessionsController#create` (durante login)

**Parâmetros**:
- `paciente_id`: ID do paciente

**Enriquecimento de dados**: Adiciona informações da tabela de preços do paciente

---

#### `GET patient/list`
**Objetivo**: Listar pacientes com filtros  
**Usado em**: `FeegowService#list_patients`  
**Parâmetros**: Vários filtros possíveis (nome, CPF, etc.)

---

#### `POST patient/create`
**Objetivo**: Criar novo paciente  
**Usado em**: `FeegowService#create_patient`  
**Parâmetros obrigatórios**:
- `nome_completo`: Nome completo
- `cpf`: CPF do paciente

**Parâmetros opcionais**:
- `data_nascimento`: Data de nascimento
- `genero`: Gênero
- `email`: Email
- `telefone`: Telefone
- `convenio_id`: ID do convênio
- `plano_id`: ID do plano
- `matricula`: Matrícula do convênio
- `validade`: Validade da carteirinha
- `tabela_id`: ID da tabela de preços

**Lógica especial Clubflex**: 
- Se não houver `tabela_id` nem `convenio_id`, verifica elegibilidade Clubflex
- Se aprovado, atribui `tabela_id = 250`

---

#### `POST patient/edit`
**Objetivo**: Atualizar dados do paciente  
**Usado em**: `FeegowService#update_patient`  
**Parâmetros**: Similar ao `patient/create` com `paciente_id` obrigatório  
**Lógica especial Clubflex**: Mesma do `create_patient`

---

#### `GET patient/list-privates`
**Objetivo**: Listar tabelas de preços particulares  
**Usado em**: `FeegowService#list_tables`

---

### 3. Profissionais (Professionals)

#### `GET professional/list`
**Objetivo**: Listar profissionais por especialidade  
**Usado em**: `FeegowService.list_professionals` (método de classe)  
**Parâmetros**:
- `especialidade_id`: ID da especialidade
- `ativo`: Status (1 para ativos)

---

#### `GET professional/search`
**Objetivo**: Buscar dados de um profissional específico  
**Usado em**: `FeegowService#show_professional`  
**Parâmetros**:
- `profissional_id`: ID do profissional

---

#### `GET professional/insurance`
**Objetivo**: Listar convênios aceitos por profissional  
**Usado em**: `FeegowService.list_accepted_insurance` (método de classe)  
**Parâmetros**:
- `profissional_id`: ID do profissional

---

### 4. Empresa (Company)

#### `GET company/list-unity`
**Objetivo**: Listar todas as unidades da empresa  
**Usado em**: `FeegowService#list_units`  
**Retorna**: Matriz + unidades filiais

---

#### `GET company/list-local`
**Objetivo**: Listar locais de atendimento  
**Usado em**: `FeegowService#list_locals`  
**Cache**: 1 hora para otimização

---

### 5. Financeiro (Financial)

#### `POST financial/create-account`
**Objetivo**: Criar conta financeira para agendamento  
**Usado em**: `FeegowService#create_account`  
**Parâmetros**:
- `agendamento_id`: ID do agendamento
- `date`: Data atual (automático)

**Logging**: Salva registro em `PaymentLog`

---

#### `POST core/financial/invoice/create`
**Objetivo**: Criar fatura detalhada com itens e parcelamentos  
**Usado em**: `FeegowService#create_invoice`  
**Parâmetros complexos**:
- `type`: Tipo da transação (C = Crédito)
- `items`: Array de procedimentos com detalhes de execução
- `installments`: Array de parcelas
- `category`: Categoria de recebimento (PIX, Cartão, Dinheiro)
- `costCenter`: Centro de custo (Administração G3)

---

#### `GET financial/list-invoice`
**Objetivo**: Listar faturas/cobranças  
**Usado em**: `FeegowService#list_invoices`  
**Parâmetros**:
- `tipo_transacao`: Tipo (C = Crédito)
- `data_start`: Data inicial (1 mês atrás)
- `data_end`: Data final (1 mês à frente)
- `invoice_id` ou `agendamento_id`: Para busca específica

---

#### `POST financial/pay-movement`
**Objetivo**: Registrar pagamento de uma movimentação  
**Usado em**: `FeegowService#pay_account`  
**Parâmetros**:
- `invoiceId`: ID da fatura
- `movementId`: ID da movimentação
- `accountId`: ID da conta
- `amount`: Valor
- `paymentName`: Nome do pagamento
- `paymentMethod`: Método de pagamento
- `paymentDate`: Data do pagamento
- `associationId`: Tipo de associação (3 = paciente)
- `creditCardTransaction.*`: Dados do cartão (se aplicável)

---

#### `GET financial/credit-card-flags`
**Objetivo**: Listar bandeiras de cartão aceitas  
**Usado em**: `FeegowService.list_credit_card_flags` (método de classe)

---

### 6. Convênios (Insurance)

#### `GET insurance/list`
**Objetivo**: Listar todos os convênios  
**Usado em**: `FeegowService#list_insurances`

---

### 7. Procedimentos (Procedures)

#### `GET procedures/list`
**Objetivo**: Listar procedimentos disponíveis  
**Usado em**: `FeegowService#list_procedures`  
**Parâmetros**:
- `procedimento_id`: Filtrar por ID (opcional)

---

### 8. Especialidades (Specialties)

#### `GET specialties/list`
**Objetivo**: Listar especialidades disponíveis  
**Usado em**: `FeegowService.list_specialties` (método de classe)

---

### 9. Relatórios (Reports)

#### `POST reports/generate` (report=price-table)
**Objetivo**: Gerar relatório de tabelas de preços  
**Usado em**: `FeegowService#list_price_tables`  
**Parâmetros**:
- `report`: "price-table"
- `GRUPO_PROCEDIMENTO_IDS[]`: IDs dos grupos (112, 98)
- `ativo[]`: Status (1)
- `DATA_INICIO` e `DATA_FIM`: Período

**Filtros adicionais aplicados**:
- Apenas tabelas disponíveis (`AVAILABLE_PRICE_TABLES`)
- Filtro por `procedimento_ids` (opcional)
- Filtro por `table_id` (opcional)

---

#### `POST reports/generate` (report=bills-to-receive)
**Objetivo**: Gerar relatório de contas a receber do dia  
**Usado em**: `FeegowService#list_reports`  
**Parâmetros**:
- `report`: "bills-to-receive"
- `TIPO_DATA[]`: "COMPETENCIA"
- `DATA_INICIO` e `DATA_FIM`: Hoje

**Cache**: 1 hora para otimização

---

## Fluxos Principais

### Fluxo de Login
1. Usuário faz login com CPF
2. Sistema verifica `external_id` do usuário (ID no Feegow)
3. Se existir, busca dados atualizados via `patient/search`
4. Retorna token JWT + dados do paciente

### Fluxo de Criação de Agendamento
1. Busca horários disponíveis (`available-schedule`)
2. Verifica dados do paciente (`patient/search`)
3. Cria agendamento (`appoints/new-appoint`)
4. Gera senha de atendimento (`queue-position`)
5. Cria conta financeira (`financial/create-account`)
6. Envia cupom/senha por email

### Fluxo de Pagamento
1. Cria/busca fatura (`create-account` ou `create-invoice`)
2. Processa pagamento via gateway (Itaú PIX)
3. Registra pagamento no Feegow (`pay-movement`)
4. Atualiza status do agendamento

### Fluxo de Cadastro de Paciente com Clubflex
1. Verifica se paciente não tem convênio nem tabela definida
2. Consulta elegibilidade Clubflex (`ClubflexService`)
3. Se aprovado, atribui `tabela_id = 250`
4. Cria/atualiza paciente no Feegow

---

## Logs e Monitoramento

### ServiceLog
Todas as chamadas principais registram:
- `origin`: Sessão/origem da requisição
- `provider`: "feegow"
- `params`: Parâmetros enviados
- `response`: Resposta recebida
- `status`: Código HTTP
- `method`: GET/POST
- `url_path`: URL completa
- `requested_at`: Timestamp

### PaymentLog
Todos os processos financeiros registram:
- `agendamento_id`: ID do agendamento
- `invoice_id`: ID da fatura
- `movement_id`: ID da movimentação
- `amount`: Valor
- `feegow_response`: Resposta completa da API
- `kind`: "success" ou "fail"

---

## Cache e Otimização

### Endpoints com Cache (1 hora)
- `company/list-local`: Locais de atendimento
- `reports/generate` (bills-to-receive): Contas a receber do dia

### Estratégias de Performance
- Dados estáticos em arquivos JSON (`plano_tabelas.json`, `worklist_especialidades.json`)
- Batch loading de dados relacionados para evitar N+1 queries
- Filtros no frontend para reduzir chamadas desnecessárias

---

## Controladores que Usam FeegowService

- `Api::SessionsController`: Login e busca de paciente
- `Api::AppointmentsController`: CRUD de agendamentos
- `Api::PatientsController`: CRUD de pacientes
- `Api::AvailableAppointmentsController`: Horários disponíveis
- `Api::ProfessionalsController`: Dados de profissionais
- `Api::SpecialtiesController`: Lista de especialidades
- `Api::PaymentsController`: Processamento financeiro
- `Api::PriceTablesController`: Tabelas de preços

---

## Observações Importantes

1. **Identificação de Agendamentos do Totem**: Todos incluem `notas: "#agTOTEM"`
2. **Tabela Clubflex**: ID fixo = 250
3. **Tabela Particular Padrão**: ID = 2
4. **Centro de Custo Padrão**: Administração G3
5. **Categorias de Recebimento**: Definidas em constantes no helper
6. **Timeout**: Não especificado explicitamente (usar padrão Faraday)
7. **Retry Logic**: Não implementada (considerar adicionar para chamadas críticas)

---

## Dependências

- **Gem Faraday**: Cliente HTTP para comunicação com API
- **FeegowHelper**: Métodos auxiliares e constantes
- **ClubflexService**: Integração paralela para validação de elegibilidade
- **ApplicationHelper**: Formatação de strings e leitura de arquivos de dados
