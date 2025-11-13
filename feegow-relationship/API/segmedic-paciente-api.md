# Documentação - Integração API Feegow

## Visão Geral

Esta aplicação integra-se com a API Feegow para gerenciar dados de pacientes, agendamentos, procedimentos, profissionais e outros recursos médicos. A integração é realizada através do serviço `FeegowService` e helpers relacionados.

## Configuração

### Variáveis de Ambiente
- `FEEGOW_API_ENDPOINT`: URL base da API Feegow
- `FEEGOW_API_TOKEN`: Token de autenticação (enviado no header `x-access-token`)

### Autenticação
Todas as requisições incluem o header:
```
x-access-token: [FEEGOW_API_TOKEN]
Content-Type: application/json
```

## Arquivos Principais

### 1. `app/services/feegow_service.rb`
Serviço principal que centraliza todas as chamadas à API Feegow usando Faraday.

**Características:**
- Suporta modo síncrono e assíncrono (`async: true`)
- Modo assíncrono usa `Async::HTTP::Faraday` para requisições paralelas
- Retorna hash padronizado: `{ body: ..., code: ... }`

### 2. `app/helpers/feegow_helper.rb`
Helper com constantes e métodos auxiliares para a API Feegow.

**Contém:**
- Status de agendamentos inválidos (`APPOINTMENT_INVALID_STATUS`)
- Tabelas de preços (`TABLES`)
- Especialidades (`SPECIALTIES`)
- Procedimentos (`PROCEDURES`)
- Métodos auxiliares para requisições HTTP

## Endpoints da API Feegow Utilizados

### Pacientes (Patients)

#### `GET /patient/search`
**Uso:** Buscar paciente por CPF ou ID
- `find_patient(cpf)` - Busca por CPF
- `find_patient_by_id(id)` - Busca por ID

#### `POST /patient/create`
**Uso:** Criar novo paciente
- `create_patient(params)`
- Parâmetros: nome_completo, cpf, data_nascimento, email

#### `POST /patient/edit`
**Uso:** Atualizar dados do paciente
- `update_patient(paciente_id, data)`
- Registra log de ação em `LogAction`
- Campos atualizáveis: nome_completo, cpf, data_nascimento, genero, email, telefone, convenio_id, plano_id, matricula, validade

#### `POST /patient/upload-base64`
**Uso:** Upload de documentos do paciente em base64
- `upload_patient_document(patient_document, b64)`

#### `find_or_create_patient(name, cpf, email, birthday)`
**Lógica:** Busca paciente por CPF, atualiza ou cria novo se necessário

### Agendamentos (Appointments)

#### `GET /appoints/search`
**Uso:** Listar e buscar agendamentos
- `list_appointments(patient_id, start_date, end_date, status)` - Lista todos os agendamentos (modo assíncrono)
- `get_appointment(appointment_id)` - Busca agendamento específico
- **Importante:** `list_appointments` divide requisições em períodos de 179 dias desde 01/01/2022

#### `GET /appoints/status`
**Uso:** Listar status de agendamentos
- `list_appointments_status()`

#### `POST /appoints/statusUpdate`
**Uso:** Confirmar agendamento
- `confirm_appointment(appointment_id)`
- Define StatusID: 7

#### `POST /appoints/cancel-appoint`
**Uso:** Cancelar agendamento
- `cancel_appointment(appointment_id)`
- Define motivo_id: 1

### Procedimentos (Procedures)

#### `GET /procedures/list`
**Uso:** Listar procedimentos
- `list_procedures(procedure_id=nil)`
- Pode filtrar por procedimento específico ou listar todos

### Profissionais (Professionals)

#### `GET /professional/list`
**Uso:** Listar profissionais
- `list_professionals(status=1)`
- Status: 1 = ativo

#### `GET /professional/search`
**Uso:** Buscar profissional específico
- `show_professional(professional_id)`

### Especialidades (Specialties)

#### `GET /specialties/list`
**Uso:** Listar especialidades
- `list_specialties()`

### Propostas (Proposals)

#### `GET /proposal/list`
**Uso:** Listar propostas do paciente
- `list_proposals(patient_id, start_date, end_date)`

#### `POST /proposal/change-status`
**Uso:** Alterar status da proposta
- `confirm_proposal(proposal_id)` - Define status_id: 2
- `cancel_proposal(proposal_id)` - Define status_id: 3

### Laudos Médicos (Medical Reports)

#### `GET /medical-reports/search`
**Uso:** Buscar laudo por agendamento
- `get_medical_report_by_appointment(appointment_id)`

#### `GET /medical-reports/get-laudos-list`
**Uso:** Listar laudos médicos
- `list_medical_reports(patient_id, start_date, end_date)` - Modo assíncrono
- **Importante:** Divide requisições em períodos de 364 dias desde 01/01/2021

### Relatórios (Reports)

#### `POST /reports/generate?report=bills-to-receive`
**Uso:** Gerar relatório de contas a receber
- `get_bills_report(start_date, end_date)`
- Formato de data: dd/mm/yyyy

#### `POST /reports/generate?report=generated-invoice`
**Uso:** Gerar relatório de notas fiscais
- `get_invoices_report(start_date, end_date)`
- Formato de data: dd/mm/yyyy

### Unidades (Units)

#### `GET /company/list-unity`
**Uso:** Listar unidades da empresa
- `get_units()`

## Controllers que Utilizam Feegow

### `Api::V1::AppointmentsController`
**Endpoints:**
- `GET /appointments` - Lista agendamentos do paciente (assíncrono)
- `POST /appointments/:id/confirm` - Confirma agendamento
- `POST /appointments/:id/cancel` - Cancela agendamento
- `GET /appointments/:id/medical_report` - Obtém laudo médico em base64
- `GET /appointments/:id/nfe_link` - Obtém NF-e em base64
- `GET /appointments/:id/preparation_link` - Obtém link de preparo
- `GET /appointments/status_enum` - Lista status disponíveis

**Funcionalidades especiais:**
- Enriquece dados de agendamentos com informações de especialidade, procedimento e profissional
- Busca preparos em arquivo JSON local
- Busca unidades em backup local via `BackupService`

### `Api::V1::ProposalsController`
**Endpoints:**
- `GET /proposals` - Lista propostas do paciente
- `POST /proposals/:id/confirm` - Confirma proposta
- `POST /proposals/:id/cancel` - Cancela proposta

### `Api::V1::UsersController`
**Endpoints:**
- `POST /users/user_infos` - Verifica existência de usuário no Feegow
- `POST /users` - Cria usuário no Feegow e Keycloak
- `GET /users/me` - Obtém dados do usuário atual
- `POST /users/validate_questions` - Valida respostas e retorna dados Feegow
- `PUT /users/update_information` - Atualiza informações do usuário

**Integração:**
- Sincroniza criação de usuários entre Feegow e Keycloak
- Usa ID do Feegow como identificador principal (`feegow-cliente-id`)

### `Api::V1::ReportsController`
**Endpoints:**
- `GET /reports` - Lista relatórios locais
- `GET /reports/:id/medical_report` - Obtém laudo em base64

## Jobs Assíncronos (Sidekiq)

### `ProcedureImportJob`
Importa todos os procedimentos da API Feegow para o banco local.
- Modelo: `Procedure`
- Campos: name, external_id, payload (JSON completo)

### `ProfessionalImportJob`
Importa todos os profissionais da API Feegow para o banco local.
- Modelo: `Professional`
- Campos: name, external_id, payload (JSON completo)

### `SpecialtyImportJob`
Importa todas as especialidades da API Feegow para o banco local.
- Modelo: `Specialty`
- Campos: name, external_id, payload (JSON completo)

## Estratégias de Performance

### 1. Requisições Assíncronas
Para operações que precisam fazer múltiplas requisições (ex: buscar agendamentos ou laudos de vários períodos), o serviço usa o adapter `Async::HTTP::Faraday`:

```ruby
fs = FeegowService.new(async: true)
```

Isso permite executar requisições em paralelo, reduzindo significativamente o tempo de resposta.

### 2. Cache Local
Dados de procedimentos, profissionais e especialidades são importados e mantidos no banco local para:
- Reduzir chamadas à API Feegow
- Melhorar performance
- Enriquecer dados de agendamentos

### 3. Backups Locais
Arquivos JSON locais armazenam:
- `app/backups/preparos.json` - Informações de preparo de procedimentos
- `app/backups/unidades.json` - Dados de unidades (via `BackupService`)

## Auditoria e Logs

Todas as ações críticas são registradas em `LogAction`:
- Criação/atualização de pacientes
- Confirmação/cancelamento de agendamentos
- Confirmação/cancelamento de propostas
- Acesso a laudos e notas fiscais

**Campos registrados:**
- `patient_id`: ID do paciente no Feegow
- `action`: Nome da ação
- `outcome`: success/failure
- `status`: HTTP status code
- `response`: Resposta da API
- `params`: Parâmetros enviados

## Fluxos Principais

### 1. Criação de Usuário
1. Busca/cria paciente no Feegow (`find_or_create_patient`)
2. Cria usuário no Keycloak com ID do Feegow
3. Retorna sucesso ou erro identificando a etapa falha

### 2. Listagem de Agendamentos
1. Requisição assíncrona dividida em períodos de 179 dias
2. Coleta status de agendamentos
3. Enriquece dados com procedimentos, profissionais e especialidades locais
4. Adiciona informações de unidade e preparo
5. Filtra por período se especificado
6. Ordena resultados conforme solicitado

### 3. Obtenção de NF-e
1. Busca invoice no banco local (se existir)
2. Caso não exista:
   - Busca agendamento no Feegow
   - Gera relatório de contas a receber (±3 dias da data)
   - Localiza conta correspondente ao agendamento
   - Gera relatório de notas fiscais
   - Localiza nota fiscal correspondente
3. Retorna PDF em base64

## Considerações Técnicas

### Divisão de Períodos
- **Agendamentos:** Períodos de 179 dias desde 01/01/2022
- **Laudos:** Períodos de 364 dias desde 01/01/2021

Esta divisão é necessária devido a limitações da API Feegow.

### Formato de Datas
- **Buscas:** dd-mm-yyyy ou yyyy-mm-dd (dependendo do endpoint)
- **Relatórios:** dd/mm/yyyy
- **Criação:** yyyy-mm-dd

### Status de Agendamentos Inválidos
IDs: 2, 3, 4, 5, 6, 11
- Em atendimento, Atendido, Aguardando, Chamando, Não compareceu, Desmarcado pelo paciente

### Tabelas de Preço Disponíveis
- PARTICULAR (ID: 2)
- INTERCLINICA (ID: 83)
- CLUBFLEX (ID: 250)
- INTERCLINICA 500 (ID: 310)
- CLUBFLEX EMPRESARIAL

## Melhorias Futuras (TODOs no código)

1. **RDicom Service:** Buscar arquivo de laudo no Feegow e salvar no parâmetro "link"
2. **Tratamento de Status:** Implementar tratamento adequado de códigos de status em requisições assíncronas
3. **Error Handling:** Adicionar tratamento de erros mais robusto para requisições paralelas

## Dependências

- **Faraday:** Cliente HTTP
- **Async::HTTP::Faraday:** Adapter para requisições assíncronas
- **Net::HTTP:** Usado no helper para requisições alternativas
- **Sidekiq:** Para processamento de jobs assíncronos de importação
