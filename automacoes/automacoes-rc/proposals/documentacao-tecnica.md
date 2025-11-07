# Documentação Técnica - Automação de Propostas

## Visão Geral

A automação de propostas é responsável por capturar propostas comerciais criadas na Feegow (orçamentos de procedimentos médicos) e convertê-las em oportunidades no RD Station CRM para acompanhamento comercial. O sistema agrupa procedimentos por paciente, distribui atendentes de forma round-robin e cria deals enriquecidos com dados completos do paciente.

## Arquitetura

### Componentes Principais

1. **Schedule Function** (`src/schedules/proposals.ts`)
   - Função Lambda agendada via EventBridge
   - Execução diária às 04:00 UTC (01:00 Brasília)
   - Timeout: 900 segundos (15 minutos)
   - Memória: 128 MB

2. **Worker Function** (`src/workers/proposals.ts`)
   - Função Lambda processadora de mensagens SQS
   - Timeout: 180 segundos (3 minutos)
   - Memória: 128 MB
   - Batch size: 1 mensagem por execução
   - Reserved concurrency: 2
   - Max concurrency: 2

3. **SQS Queue**
   - Delay de 15 segundos
   - Processamento com concorrência limitada (max 2)

## Fluxo de Execução

### Etapa 1: Coleta e Agrupamento de Propostas (Schedule)

**Handler:** `src/schedules/proposals.ts`

```typescript
async function handler(event, context)
```

**Processo:**

1. Instancia cliente Feegow
2. Busca propostas do dia anterior via API Feegow
3. Filtra apenas propostas com status "Em aberto" (PropostaStatusID = 1)
4. Agrupa propostas por paciente (múltiplos procedimentos → uma proposta)
5. Distribui atendentes de forma round-robin
6. Envia cada proposta agrupada para a fila SQS

**Configuração:**

- **Cron:** `cron(0 4 ? * * *)` - Executa todo dia às 04:00 UTC
- **Environment:** `QUEUE_URL` - URL da fila SQS
- **Data:** Dia anterior formatado como dd/MM/yyyy

### Etapa 2: Processamento de Propostas (Worker)

**Handler:** `src/workers/proposals.ts`

```typescript
async function handler(event: Event, context)
```

**Processo:**

1. Itera sobre cada mensagem SQS recebida
2. Busca dados completos do paciente na Feegow
3. Valida valor total dos procedimentos (> R$ 65,00)
4. Valida se paciente possui telefone
5. Converte proposta em Deal do RD Station
6. Busca agendamentos existentes do paciente
7. Enriquece anotações com status de agendamentos
8. Processa lead no RD Station

## Lógica de Negócio

### Agrupamento de Propostas por Paciente

**Função:** `groupProposals(data: any[])`

**Objetivo:** Consolidar múltiplos procedimentos do mesmo paciente em uma única proposta

**Processo:**

1. Cria objeto indexado por `PacienteID`
2. Para cada item da API Feegow:
   - Se paciente não existe no grupo, cria nova proposta
   - Adiciona procedimento ao array `items` da proposta
3. Retorna array de propostas agrupadas

**Estrutura da Proposta:**

```typescript
interface Proposal {
    id: number;              // ID do paciente
    pacienteId: number;
    unidadeId: number;
    nomePaciente: string;
    numeroPaciente: string;
    nomeUsuario: string;     // Usuário que criou a proposta
    nomeUnidade: string;
    profissional?: string;   // Profissional relacionado
    agente: string;          // Atendente atribuído (round-robin)
    items: ProposalItem[];   // Array de procedimentos
}

interface ProposalItem {
    procedimentoId: number;
    grupoId: number;
    value: number;           // Valor monetário
    name: string;            // Nome do procedimento
}
```

### Distribuição Round-Robin de Atendentes

**Constante:** `ATENDENTES`

```typescript
const ATENDENTES = [
  "Joseani Feliciano",
  "Nicole Botelho",
  "Karen Ribeiro",
  "Jossana Gusmão",
  "Hosana Santos"
]
```

**Lógica:**

```typescript
let indiceAtendente = 0;

// A cada nova proposta (novo paciente)
indiceAtendente = indiceAtendente == ATENDENTES.length 
    ? 0 
    : indiceAtendente += 1;

proposal.agente = ATENDENTES[indiceAtendente];
```

**Comportamento:**

- Distribui propostas igualmente entre os 5 atendentes
- Ciclo contínuo: após o último atendente, volta para o primeiro
- Garante balanceamento de carga entre a equipe

### Validações no Worker

**1. Validação de Valor Mínimo:**

```typescript
const totalValue = body.items.reduce((acc, item) => acc + item.value, 0);

if (totalValue >= 0.1 && totalValue <= 65) {
    console.log(`Proposta ignorada, valor total (${totalValue}) é menor ou igual a 65 reais.`);
    continue;
}
```

**Critério:** Propostas entre R$ 0,10 e R$ 65,00 são ignoradas

**Razão:** Valores muito baixos podem ser propostas de teste ou procedimentos sem valor comercial

**2. Validação de Telefone:**

```typescript
const deal = parseDeal(body, clientFeegow);
if(!deal) {
    console.log("Proposta ignorada, o paciente não possui número de telefone.");
    continue;
}
```

**Critério:** Paciente deve ter telefone cadastrado

**Razão:** Telefone é essencial para contato comercial

### Conversão de Proposta para Deal

**Função:** `parseDeal(proposal: Proposal, clientFeegow: Patient)`

**Dados do Paciente:**

- **ID:** `pacienteId`
- **Nome:** `nomePaciente`
- **Contato:** `numeroPaciente`
- **Email:** Primeiro email cadastrado na Feegow

**Custom Fields (12 campos):**

1. **FIELD_PRIORIDADE:** Prioridade baseada no grupo do primeiro procedimento
2. **CLIENT_ANOTACAO:** Observações do cadastro do paciente
3. **FIELD_UNIDADE:** Nome da unidade
4. **CLIENT_CPF:** CPF do paciente
5. **CLIENT_CEP:** CEP
6. **CLIENT_CIDADE:** Cidade
7. **CLIENT_BAIRRO:** Bairro
8. **CLIENT_COMPLEMENTO:** Complemento do endereço
9. **CLIENT_NUMERO:** Número do endereço
10. **CLIENT_LOGRADOURO:** Logradouro/Rua
11. **CLIENT_TELEFONE:** Número de telefone
12. **AGENTE_PROPOSTA:** Nome do atendente atribuído

**Prioridade por Grupo de Exames:**

Utiliza mapeamento `groupExames`:

```typescript
const groupExames = {
    "69": "Exames Laboratoriais",
    "94": "Audiometria",
    "101": "Ecocardiograma",
    "103": "Eletrocefalograma",
    "120": "Ultrasonografia",
    // ... outros grupos
}

const prioridadeNome = groupExames[procedimentos[0].grupo];
```

**Procedimentos:**

Cada item da proposta é convertido para:

```typescript
{
    grupo: item.grupoId.toString(),
    id: item.procedimentoId,
    name: item.name,
    valor: item.value
}
```

**Stage e Source:**

- **Stage:** `STAGE_PROPOSTA` (ID: 647e4cdab66552000db15fd5)
- **Source:** `SOURCE_PROPOSTAS` (ID: 646ba573893f28000fcff142)

### Enriquecimento com Status de Agendamentos

**Processo:**

1. Busca agendamentos existentes do paciente via API Feegow
2. Para cada procedimento da proposta:
   - Verifica se existe agendamento correspondente
   - Se existe, adiciona status do agendamento
3. Cria anotação formatada com lista de procedimentos e status

**Mapeamento de Status:**

```typescript
const StatusAppointment = {
    1: "Marcado - não confirmado",
    2: "Em atendimento",
    3: "Atendido",
    4: "Aguardando",
    5: "chamado",
    6: "Não compareceu",
    7: "Marcado - confirmado",
    11: "Desmarcado pelo paciente",
    15: "Remarcado",
    33: "Em espera",
    208: "Aguardando pagamento"
}
```

**Exemplo de Anotação:**

``` bash
Procedimentos:
246 - Alergologia - Marcado - confirmado
254 - Cardiologia - Atendido
Mamografia
```

**Adiciona Link Feegow:**

``` typescript
deal.anotacoes.push(FeegowClient.linkPaciente(pacienteId));
```

### Processamento no RD Station

**Serviço:** `src/rd/service.ts`

**Função:** `processLead(deal: Deal)`

**Etapas:**

1. **Busca ou Cria Organização:**
   - Busca organização pelo nome do paciente
   - Se não existir, cria nova organização

2. **Cria Deal:**
   - Cria negócio vinculado à organização
   - Preenche 12 custom fields
   - Adiciona produtos (procedimentos da proposta)
   - Define stage PROPOSTA e source PROPOSTAS

3. **Adiciona Anotações:**
   - Lista de procedimentos com status de agendamento
   - Link para o paciente na Feegow

## Dependências

### Integrações Externas

1. **Feegow API:**
   - `proposals(date)` - Busca propostas por data
   - `getPatient(patientId)` - Busca dados completos do paciente
   - `schedulesByPatientId(patientId)` - Busca agendamentos
   - `linkPaciente(patientId)` - Gera link de acesso

2. **RD Station CRM API:**
   - Criação de organizações
   - Criação de deals com produtos
   - Criação de anotações

### Bibliotecas

- `date-fns` - Manipulação de datas (format, sub)
- `aws-sdk` - Integração com SQS

## Diferenças em Relação às Outras Automações

| Aspecto | Proposals | Consultas | Convênios | Exames |
|---------|-----------|-----------|-----------|---------|
| **Fonte** | API Feegow | MySQL | API Feegow | MySQL |
| **Entidade** | Propostas comerciais | Leads site | Agendamentos | Leads exames |
| **Agrupamento** | Por paciente | Não | Não | Por nome (Set) |
| **Distribuição** | Round-robin atendentes | Não | Não | Não |
| **Validação $** | Valor > R$ 65 | Não | Não | Não |
| **Status ID** | PropostaStatusID = 1 | Não usado | StaID = 3 | Não usado |
| **Produtos RD** | Sim (procedimentos) | Não | Não | Não |
| **Custom Fields** | 12 campos | 10+ campos | 3 campos | 2 campos |
| **Prioridade** | Por grupo exame | Não | Não | Não |
| **Agendamentos** | Verifica e anota status | Valida datas | Não | Valida datas |
| **Stage** | PROPOSTA | RECUPERACAO | ATENDIMENTO_CONVENIOS | FUNEL_RECUPERACAO_EX |
| **Objetivo** | Conversão comercial | Recuperar leads | Monitorar atendidos | Recuperar exames |

## Particularidades da Automação de Propostas

### 1. Agrupamento por Paciente

**Única automação que agrupa dados:**

- Múltiplas linhas da API → 1 proposta consolidada
- Permite visão completa do orçamento do paciente
- Facilita negociação de pacotes

### 2. Distribuição de Atendentes

**Sistema Round-Robin exclusivo:**

- 5 atendentes rotativos
- Balanceamento automático de carga
- Cada nova proposta recebe próximo atendente da fila

### 3. Enriquecimento com Status

**Cruzamento de dados único:**

- Busca agendamentos existentes
- Relaciona procedimentos com agendamentos
- Anota status atual de cada procedimento
- Permite visão 360° do paciente

### 4. Validação de Valor Comercial

**Filtro financeiro:**

- Descarta propostas de baixo valor (≤ R$ 65)
- Foco em oportunidades comerciais relevantes
- Reduz ruído no CRM

### 5. Produtos no Deal

**Única que adiciona produtos:**

```typescript
deal_products: [
    {
        amount: 1,
        base_price: 250.00,
        description: "69",  // grupoId
        name: "Alergologia",
        price: 250.00,
        total: 250.00,
        recurrence: "spare"
    }
]
```

### 6. 12 Custom Fields

**Mais campos de endereço que outras automações:**

- CPF, CEP, Cidade, Bairro
- Complemento, Número, Logradouro
- Dados completos para envio de materiais/documentos

### 7. Prioridade Automática

**Baseada no primeiro procedimento:**

- Grupo do procedimento → Nome da prioridade
- Ex: Grupo "69" → "Exames Laboratoriais"
- Permite triagem e categorização automática

## Variáveis de Ambiente

- `QUEUE_URL` - URL da fila SQS de propostas
- Credenciais Feegow API
- Credenciais RD Station API

## Tratamento de Erros

1. **Proposta com valor baixo:**
   - Log: "Proposta ignorada, valor total (...) é menor ou igual a 65 reais"
   - Continua para próxima proposta

2. **Paciente sem telefone:**
   - Log: "Proposta ignorada, o paciente não possui número de telefone"
   - Continua para próxima proposta

3. **Erro ao buscar paciente ou agendamentos:**
   - Pode gerar exceção não tratada
   - Mensagem permanece na fila SQS para retry

## Configurações de Performance

### Schedule Function

- **Timeout:** 900s para processar e agrupar propostas
- **Memória:** 128 MB suficiente para agrupamento
- **Execução:** 1x por dia (04:00 UTC)

### Worker Function

- **Timeout:** 180s por proposta
- **Memória:** 128 MB
- **Batch size:** 1 (processamento individual)
- **Delay:** 15s entre mensagens
- **Concorrência:** Max 2 simultâneos
- **Reserved:** 2 (garantido)

### Otimizações

1. **Agrupamento no Schedule:** Reduz número de mensagens SQS
2. **Filtro de status:** Apenas propostas em aberto (status 1)
3. **Validação antecipada:** Valor e telefone antes de processar
4. **Busca única do paciente:** Dados completos em uma chamada

## Monitoramento

### Métricas Importantes

1. **Propostas por dia:** Total coletado da Feegow
2. **Taxa de filtragem:** Propostas descartadas vs processadas
3. **Distribuição de atendentes:** Balanceamento round-robin
4. **Valor médio das propostas:** Controle financeiro
5. **Taxa de propostas sem telefone:** Qualidade de cadastro
6. **Procedimentos por proposta:** Média de itens agrupados

### Logs Importantes

```typescript
console.log("DEAL", deal)  // Deal completo sendo criado
console.log(`Proposta ignorada, valor total (${totalValue})...`)
console.log("Proposta ignorada, o paciente não possui número de telefone.")
```

## Manutenção

### Pontos de Atenção

1. **Lista de Atendentes:**
   - Atualizar array `ATENDENTES` quando equipe mudar
   - Manter quantidade ímpar para melhor distribuição

2. **Valor Mínimo:**
   - Threshold de R$ 65 pode precisar ajuste
   - Considerar inflação e política comercial

3. **Status de Proposta:**
   - Atualmente filtra apenas PropostaStatusID = 1
   - Validar se outros status devem ser incluídos

4. **Grupos de Exames:**
   - Manter mapeamento `groupExames` atualizado
   - Novos grupos devem ter prioridade definida

5. **Round-Robin:**
   - Índice reseta a cada execução
   - Não mantém estado entre dias

### Troubleshooting

**Problema:** Atendente recebendo mais propostas que outros

**Solução:**

- Verificar lógica do round-robin
- Confirmar se array ATENDENTES está correto
- Índice pode estar desalinhado

**Problema:** Propostas válidas sendo ignoradas

**Solução:**

- Verificar threshold de valor (R$ 65)
- Validar cadastro de telefone na Feegow
- Conferir status da proposta (deve ser 1)

**Problema:** Produtos não aparecendo no RD

**Solução:**

- Verificar parsing de valores (replaceAll e replace)
- Validar estrutura deal_products
- Conferir se procedimentos têm valor válido

**Problema:** Prioridade vazia ou incorreta

**Solução:**

- Verificar mapeamento groupExames
- Validar grupoId do primeiro procedimento
- Adicionar novos grupos ao mapeamento
