# Documentação Técnica - Leads de Convênios

## 📋 Visão Geral

A automação de **Leads de Convênios** tem como objetivo identificar pacientes que realizaram atendimentos com convênio médico no dia anterior e enviá-los como leads para o CRM, possibilitando ações comerciais de upsell, cross-sell ou migração para planos particulares.

## 🎯 Objetivo

Capturar pacientes que:

- Realizaram atendimentos **com convênio** no dia anterior (D-1)
- Completaram o atendimento (presente na tabela de atendimentos)
- Não são de convênios específicos excluídos (Bradesco, Mediservice)
- Não realizaram visitas (apenas procedimentos médicos válidos)

## 🔧 Tecnologias e Dependências

### Bibliotecas

- **date-fns**: Manipulação de datas (subtrair 1 dia, formatar data)

### Serviços Externos

- **Medula (PostgreSQL)**: Data warehouse com dados de agendamentos e atendimentos
- **AWS SQS**: Fila para envio de leads

### Repositórios

- `MedulaRepository`: Consultas ao data warehouse Medula

## 📊 Fluxo de Execução

### 1. Inicialização

```typescript
handler()
```

- Cria conexão com o repositório Medula (PostgreSQL)
- Inicializa o serviço Medula

### 2. Cálculo da Data

```typescript
const today = new Date()
const yesterday = subDays(today, 1)
const yesterdayInBrazilFormat = format(yesterday, 'yyyy-MM-dd')
```

Calcula a data de ontem no formato brasileiro (YYYY-MM-DD).

### 3. Busca de Atendimentos com Convênio

Executa query no data warehouse que:

- Busca agendamentos do dia anterior
- Faz JOIN com atendimentos realizados (confirmação de presença)
- Filtra apenas registros com convênio preenchido
- Remove convênios específicos (Bradesco, Mediservice)
- Remove visitas de representantes
- Usa `DISTINCT ON (CPF)` para evitar duplicatas

### 4. Validação

Se não houver leads:

- Lança erro "Não há leads de convenio"

### 5. Envio para Fila

Para cada paciente encontrado:

- Cria um `LeadEvent` com origem "leads_convenios"
- Envia para a fila SQS
- **Não há deduplicação adicional** (já tratado pelo DISTINCT ON)

### 6. Finalização

O fluxo termina sem destruir conexões explicitamente (diferente de outras automações).

## 📦 Estrutura de Dados

### PatientConvenio (Payload)

```typescript
{
  NomePaciente: string,
  CPF: string,
  email1: string,
  nomesexo: string,
  data: string,           // Data do agendamento
  nomeprocedimento: string,
  nomeprofissional: string,
  nomeunidade: string,
  nomeconvenio: string,   // Nome do convênio utilizado
  nascimento: string,
  Cel1: string
}
```

### LeadEvent

```typescript
{
  origin: "leads_convenios",
  payload: PatientConvenio
}
```

## 🔍 Critérios de Filtragem

### Query SQL - Principais Filtros

```sql
WHERE  
    fad."data" = $1                                    -- Data de ontem
    AND COALESCE(fad.nomeconvenio, '') <> ''          -- Tem convênio
    AND fad.nomeconvenio NOT LIKE '%Bradesco%'        -- Não é Bradesco
    AND fad.nomeconvenio NOT LIKE '%Mediservice%'     -- Não é Mediservice
    AND fad.nomeprocedimento NOT LIKE '%Visita%'      -- Não é visita
```

### JOIN com Atendimentos

```sql
INNER JOIN feegow_atendimentos_dw ON agendamentoid = agendamentoid
```

Garante que o paciente **compareceu** ao atendimento (não apenas agendou).

### Deduplicação

```sql
DISTINCT ON (fad."CPF")
ORDER BY fad."CPF", fad."data"
```

Remove duplicatas pelo CPF, mantendo apenas o primeiro registro ordenado por data.

## ⚙️ Configurações

### Período de Busca

- **Data**: Ontem (D-1)
- **Formato**: YYYY-MM-DD

### Convênios Excluídos

- Bradesco (todos os planos)
- Mediservice (todos os planos)

### Procedimentos Excluídos

- Visitas de representantes

## 🚨 Tratamento de Erros

- Lança erro se não houver leads encontrados
- Não possui try/catch explícito
- Erros serão propagados para o runtime
- **Importante**: Não há destruição de conexões no finally

## 📝 Observações Importantes

1. **Simplicidade**: É a automação mais simples do sistema
2. **Sem Verificações Adicionais**: Não verifica CRM ou ClubFlex
3. **Sem Deduplicação em Runtime**: Confia no DISTINCT ON do SQL
4. **JOIN Importante**: O INNER JOIN garante que apenas atendimentos realizados sejam considerados
5. **Data Warehouse**: Usa dados consolidados do Medula, não API Feegow diretamente
6. **Convênios Específicos**: Bradesco e Mediservice são explicitamente excluídos

## 🔄 Dependências de Outros Módulos

- `date-fns`: Cálculo e formatação de datas
- `src/aws/sqs.ts`: Envio para fila
- `src/medula/medulaRepository.ts`: Acesso ao data warehouse
- `src/medula/medulaService.ts`: Lógica de negócio Medula

## 🎯 Diferenças vs Outras Automações

| Característica | Leads Convênios | Leads Amanhã | Recuperação Agendamento |
|----------------|-----------------|--------------|-------------------------|
| Período | Ontem (D-1) | Hoje até amanhã (D+1) | Ontem até +15 dias |
| Foco | Atendimentos com convênio | Agendamentos sem convênio | Carrinhos abandonados |
| Verificações | Nenhuma adicional | CRM + ClubFlex | CRM + ClubFlex + Especialidade |
| Deduplicação | DISTINCT ON (SQL) | Set em runtime | Map por CPF |
| Fonte de Dados | Data Warehouse Medula | API Feegow | API Feegow + BD Interno |
| Complexidade | Baixa | Média | Alta |

## 📊 Fonte de Dados

### Tabelas do Medula (PostgreSQL)

- **feegow_agendamentos_dw**: Data warehouse de agendamentos
- **feegow_atendimentos_dw**: Data warehouse de atendimentos realizados

### Relacionamento

```sql
feegow_agendamentos_dw (agendado)
    ↓ INNER JOIN
feegow_atendimentos_dw (compareceu)
    ↓
Leads de Convênio
```

## 🎯 Casos de Uso Comercial

Esta automação é útil para:

1. **Upsell**: Oferecer planos particulares ou ClubFlex
2. **Cross-sell**: Oferecer outros serviços não cobertos pelo convênio
3. **Retenção**: Identificar pacientes que podem migrar para particular
4. **Análise**: Entender quais convênios os pacientes estão usando

## 🔐 Segurança

- Connection string hardcoded (considerar usar variáveis de ambiente)
- SSL configurado com `rejectUnauthorized: false`

## 🎯 Próximos Passos (Sugestões)

1. Adicionar destruição de conexões no finally
2. Implementar logging estruturado
3. Adicionar métricas (quantos leads por convênio)
4. Considerar verificações CRM para evitar duplicatas
5. Mover connection string para variáveis de ambiente
6. Adicionar try/catch para melhor tratamento de erros
7. Considerar não lançar erro quando não há leads (pode ser normal)
8. Adicionar validação de campos obrigatórios antes do envio
