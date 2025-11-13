# Documentação - Integração com API Feegow

## 📋 Visão Geral

Este documento descreve como o sistema de agendamento se integra com a API da Feegow, uma plataforma de gestão médica que fornece funcionalidades de agendamento, cadastro de pacientes, profissionais e muito mais.

## 🔧 Configuração

### Variáveis de Ambiente
- `FEEGOW_API_ENDPOINT`: URL base da API Feegow
- `FEEGOW_API_TOKEN`: Token de autenticação

### Headers HTTP
- `Content-Type: application/json`
- `x-access-token: [FEEGOW_API_TOKEN]`

## 📁 Arquitetura

### Service Principal
**Arquivo**: `app/services/feegow_service.rb`
- Classe responsável por todas as comunicações com a API Feegow
- Utiliza a gem Faraday para requisições HTTP
- Implementa cache para otimizar requisições frequentes

### Helper
**Arquivo**: `app/helpers/feegow_helper.rb`
- Constantes de especialidades e procedimentos
- Métodos utilitários para validação de status de agendamento
- Checagem de regiões e unidades

## 🔌 Endpoints Utilizados

### 1. **Agendamentos (Appointments)**

#### `appoints/available-schedule`
- **Método**: GET
- **Uso**: Buscar horários disponíveis para agendamento
- **Implementado em**:
  - `FeegowService#list_appoints_availability`
  - `FeegowService#list_available_appointments`
  - `FeegowService#list_available_schedule`
- **Controllers**: `V1::AppointmentsController`
- **Parâmetros principais**:
  - `data_start`, `data_end`: Período de busca
  - `tipo`: 'P' (Procedimento) ou 'E' (Especialidade)
  - `procedimento_id` ou `especialidade_id`
  - `profissional_id`: Opcional
  - `unidade_id`: Opcional
  - `convenio_id`: Opcional

#### `v2/appoints/available-schedule`
- **Método**: GET
- **Uso**: Versão 2 da busca de horários (mais otimizada)
- **Features adicionais**:
  - Filtro automático de horários (mínimo 1 hora de antecedência)
  - Validação de atualização de convênio (espera de 5 dias para exames)

#### `appoints/new-appoint`
- **Método**: POST
- **Uso**: Criar novo agendamento
- **Implementado em**: `FeegowService#create_appointment`
- **Parâmetros**:
  - `local_id`, `paciente_id`, `profissional_id`
  - `especialidade_id`, `procedimento_id`
  - `plano`, `data`, `horario`, `valor`
  - `convenio_id`: Opcional
  - `notas`: Observações do agendamento

### 2. **Pacientes (Patients)**

#### `patient/search`
- **Método**: GET
- **Uso**: Buscar paciente por CPF ou ID
- **Implementado em**: `FeegowService#show_patient`
- **Usado em**:
  - `V1::AuthController` (login)
  - `V1::UserController` (dados do usuário)
  - `User#patient` (model)
  - `Session#set_schedule_params`
- **Parâmetros**: `paciente_cpf` ou `paciente_id`

#### `patient/list`
- **Método**: GET
- **Uso**: Listar pacientes
- **Implementado em**: `FeegowService#list_patients`

#### `patient/create`
- **Método**: POST
- **Uso**: Cadastrar novo paciente
- **Implementado em**: `FeegowService#create_patient`
- **Parâmetros**:
  - Obrigatórios: `nome_completo`, `cpf`
  - Opcionais: `data_nascimento`, `genero`, `email`, `telefone`
  - Convênio: `convenio_id`, `plano_id`, `matricula`, `validade`
  - `tabela_id`: Tabela de preços

#### `patient/edit`
- **Método**: POST
- **Uso**: Atualizar dados do paciente
- **Implementado em**: `FeegowService#update_patient`
- **Usado em**: `User#set_external_id`

#### `patient/list-privates`
- **Método**: GET
- **Uso**: Listar tabelas de preços privadas
- **Implementado em**: `FeegowService#list_tables`

### 3. **Profissionais (Professionals)**

#### `professional/list`
- **Método**: GET
- **Uso**: Listar profissionais
- **Implementado em**: `FeegowService#list_professionals`
- **Parâmetros**: `ativo: 1` (apenas profissionais ativos)

#### `professional/search`
- **Método**: GET
- **Uso**: Buscar profissional por ID
- **Implementado em**: `FeegowService#show_professional`

#### `professional/insurance`
- **Método**: GET
- **Uso**: Buscar convênios aceitos por profissional
- **Implementado em**: `FeegowService#get_professional_insurances`
- **Usado em**: `V1::AppointmentsController` (validação de convênios)

### 4. **Unidades e Locais**

#### `company/list-unity`
- **Método**: GET
- **Uso**: Listar unidades da empresa
- **Implementado em**: `FeegowService#list_units`
- **Controller**: `V1::UnitsController`
- **Retorno**: Matriz + Unidades

#### `company/list-local`
- **Método**: GET
- **Uso**: Listar locais de atendimento
- **Implementado em**: `FeegowService#list_locals`
- **Controller**: `V1::LocalsController`
- **Cache**: 1 hora

### 5. **Procedimentos (Procedures)**

#### `procedures/list`
- **Método**: GET
- **Uso**: Listar todos os procedimentos
- **Implementado em**: `FeegowService#list_procedures`
- **Usado em**: `Procedure.generate` (sincronização)

### 6. **Especialidades (Specialties)**

#### `specialties/list`
- **Método**: GET
- **Uso**: Listar especialidades médicas
- **Implementado em**: `FeegowService#list_specialties`
- **Controller**: `V1::SpecialtiesController`

### 7. **Convênios (Insurance)**

#### `insurance/list`
- **Método**: GET
- **Uso**: Listar convênios/planos de saúde
- **Implementado em**: `FeegowService#list_insurances`
- **Usa**: `InsuranceAdapter` para transformar dados

### 8. **Relatórios (Reports)**

#### `reports/generate?report=price-table`
- **Método**: POST
- **Uso**: Gerar relatório de tabelas de preços
- **Implementado em**: `FeegowService#list_price_tables`
- **Usado em**:
  - `SessionAppointment#set_price` (definir preços)
  - `V1::FeegowsController`
  - `V1::TotemController`
- **Parâmetros**:
  - `DATA_INICIO`, `DATA_FIM`: Datas do relatório
  - `GRUPO_PROCEDIMENTO_ID` ou `GRUPO_PROCEDIMENTO_IDS[]`
  - `ativo[]=1`: Apenas ativos

#### `reports/generate?report=bills-to-receive`
- **Método**: POST
- **Uso**: Relatório de contas a receber
- **Implementado em**: `FeegowService#list_reports`
- **Cache**: 1 hora

### 9. **Financeiro (Financial)**

#### `financial/list-invoice`
- **Método**: GET
- **Uso**: Listar faturas/notas fiscais
- **Implementado em**: `FeegowService#list_invoices`
- **Parâmetros**:
  - `tipo_transacao=C`: Tipo de transação
  - `data_start`, `data_end`: Período (1 mês antes e depois)
  - `invoice_id` ou `agendamento_id`: Filtros

## 🎯 Fluxos Principais

### Fluxo de Autenticação
1. Usuário informa CPF e telefone
2. Sistema busca paciente na Feegow (`patient/search`)
3. Se encontrado, cria/autentica usuário local
4. Sincroniza dados com `external_id` do Feegow

### Fluxo de Agendamento
1. Buscar horários disponíveis (`v2/appoints/available-schedule`)
2. Filtrar por convênios do profissional (`professional/insurance`)
3. Validar idade e restrições do procedimento
4. Adicionar ao carrinho (Session)
5. Confirmar agendamento (`appoints/new-appoint`)

### Fluxo de Preços
1. Buscar tabelas de preços (`reports/generate?report=price-table`)
2. Identificar preço Clubflex e Particular
3. Aplicar preço conforme elegibilidade do paciente
4. Armazenar em `SessionAppointment`

### Fluxo de Sincronização de Procedimentos
1. Buscar procedimentos da Feegow (`procedures/list`)
2. Comparar com backup local
3. Criar/atualizar registros no banco de dados
4. Sincronizar especialidades e grupos

## 📊 Models que Interagem com Feegow

### User
- `#patient`: Busca dados do paciente na Feegow
- `#set_external_id`: Cria/atualiza paciente e sincroniza ID
- `#set_insurance`: Atualiza convênio do paciente

### Session
- `#set_schedule_params`: Busca dados do paciente e elegibilidade Clubflex
- `#schedule_appointments`: Cria agendamentos na Feegow

### SessionAppointment
- `#set_price`: Busca tabelas de preços para o procedimento
- `#feegow_appointment_params`: Prepara parâmetros para criar agendamento

### Procedure
- `.generate`: Sincroniza procedimentos da Feegow com banco local

## 🔄 Cache e Otimizações

### Endpoints com Cache
- `company/list-local`: 1 hora
- `reports/generate?report=bills-to-receive`: 1 hora

### Otimizações
- Uso de threads para buscar convênios de múltiplos profissionais
- Cache de preços históricos (7 dias) para evitar chamadas desnecessárias
- Reutilização de dados de sessões anteriores

## 📝 Logging

Todos os métodos POST para Feegow registram logs em `ServiceLog`:
- Origem da requisição
- Provider: `:feegow`
- Parâmetros enviados
- Resposta recebida
- Status HTTP
- Método HTTP

**Models que geram logs**:
- `SessionAppointment`
- `Session`
- `User`

## 🚫 Status de Agendamento Inválidos

Constante `APPOINTMENT_INVALID_STATUS` no `FeegowHelper`:
- ID 2: Em atendimento
- ID 3: Atendido
- ID 4: Aguardando | Atendimento
- ID 5: Chamando | Atendimento
- ID 6: Não compareceu
- ID 11: Desmarcado pelo paciente

## 🔑 Pontos de Integração nos Controllers

- `ApplicationController`: Inicializa `@feegow` em todas as requisições
- `V1::AuthController`: Login e busca de pacientes
- `V1::AppointmentsController`: Agendamentos disponíveis
- `V1::FeegowsController`: Tabelas de preços
- `V1::LocalsController`: Locais de atendimento
- `V1::UnitsController`: Unidades
- `V1::SpecialtiesController`: Especialidades
- `V1::UserController`: Dados do paciente
- `V1::TotemController`: Tabelas de preços para totem

## 🛡️ Tratamento de Erros

O sistema trata erros da Feegow verificando:
- Códigos HTTP (200 = sucesso, 422 = erro, etc)
- Presença de `content` na resposta
- Validações específicas por endpoint

**Exemplo**:
```ruby
if patient_response[:code] == 200
  # Sucesso
elsif patient_response[:code] == 422
  # Erro - usuário não encontrado
end
```

## 📌 Observações Importantes

1. **IDs Externos**: Pacientes locais mantém `external_id` sincronizado com ID da Feegow
2. **Tabelas de Preços**: Sistema diferencia entre Clubflex e Particular
3. **Convênios**: Validação de convênios aceitos por profissional antes de exibir horários
4. **Datas**: Formato usado pela Feegow: `dd-mm-yyyy`
5. **Horários**: Formato: `HH:MM:SS`
6. **Tipos de Busca**: 'P' para Procedimento, 'E' para Especialidade

---

**Última atualização**: Novembro 2025
