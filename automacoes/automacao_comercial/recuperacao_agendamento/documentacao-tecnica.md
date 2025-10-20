# Documentação Técnica - Recuperação de Agendamento

## 📋 Visão Geral

A automação de **Recuperação de Agendamento** tem como objetivo identificar pacientes que abandonaram o processo de agendamento (carrinho abandonado) e que ainda não possuem agendamentos futuros na mesma especialidade, enviando esses leads para conversão no CRM.

## 🎯 Objetivo

Recuperar pacientes que iniciaram mas não completaram um agendamento, filtrando aqueles que:

- Não possuem agendamento confirmado na mesma especialidade
- Não possuem oportunidades ativas no CRM
- Não são clientes ClubFlex
- Não possuem convênio médico

## 🔧 Tecnologias e Dependências

### Serviços Externos

- **Feegow**: Sistema de gestão médica (agendamentos, pacientes, especialidades)
- **Medula**: Verificação de oportunidades no CRM
- **ClubFlex**: Verificação de membros do clube de benefícios
- **AWS SQS**: Fila para envio de leads

### Repositórios

- `ConsultationRepository`: Dados de agendamentos v1
- `ScheduleV2Repository`: Dados de agendamentos v2
- `MedulaRepository`: Consultas ao CRM
- `ClubflexRepository`: Consultas ao ClubFlex

## 📊 Fluxo de Execução

### 1. Inicialização

```typescript
handler()
```

- Cria conexões com os repositórios (Medula, ClubFlex, Consultation, ScheduleV2)
- Inicializa os serviços necessários

### 2. Coleta de Dados

Executa em paralelo:

- **Especialidades** do Feegow
- **Agendamentos** dos últimos 15 dias (de ontem até 15 dias à frente)
- **Leads de carrinho abandonado** de hoje

### 3. Primeira Filtragem - Verificação de Especialidade

```typescript
leadsToSend = cart_recovery_of_schedule_today.filter(...)
```

Para cada lead do carrinho:

- Se não tem especialidade definida → **INCLUI** (permite envio)
- Se tem especialidade → verifica se existe agendamento confirmado
  - Busca agendamentos com status 6, 11 ou 16 (agendados/confirmados)
  - Verifica se é do mesmo paciente
  - Verifica se é da mesma especialidade
  - Se encontrar → **EXCLUI** (já tem agendamento)
  - Se não encontrar → **INCLUI** (pode enviar)

### 4. Extração de Telefones e CPFs

- Cria conjuntos únicos de telefones e CPFs dos agendamentos
- Normaliza telefones (remove formatação)
- Normaliza CPFs (remove pontos e traços)

### 5. Verificações de Negócio

Executa em paralelo:

- **Verificação CRM**: Consulta se os telefones têm oportunidades ativas
- **Verificação ClubFlex**: Consulta se os CPFs são membros ClubFlex

### 6. Segunda Filtragem - Regras de Negócio

```typescript
leadsVerifyInCrm = leadsToSend.filter(...)
```

Filtra leads que:

- ✅ Possuem CPF e telefone válidos
- ✅ **NÃO** são ClubFlex
- ✅ **NÃO** possuem oportunidade no CRM

### 7. Envio para Fila

Para cada lead aprovado:

- Cria um objeto `LeadEvent` com origem "recuperacao_agendamento"
- Envia para a fila SQS da AWS

### 8. Finalização

- Destrói todas as conexões (schedulesService, medulaService, clubflexService)

## 📦 Estrutura de Dados

### LeadRecAgd

```typescript
{
  patient_id: string | number;
  patient_email?: string;
  cpf: string;
  patient_phone?: string;
  patient_name?: string;
  schedule_professional?: string | null;
  schedule_speciality?: string | null;
  schedule_day?: string | null;
  schedule_unit?: string | null;
}
```

### LeadEvent

```typescript
{
  origin: "recuperacao_agendamento",
  payload: LeadRecAgd
}
```

## 🔍 Critérios de Filtragem

### Status de Agendamento Considerados

- `StaID = 6`: Agendado
- `StaID = 11`: Confirmado
- `StaID = 16`: Em atendimento

### Exclusões

1. Pacientes com agendamento confirmado na mesma especialidade
2. Pacientes sem CPF ou telefone
3. Pacientes ClubFlex
4. Pacientes com oportunidades ativas no CRM

## ⚙️ Configurações

### Período de Busca

- **Início**: Ontem (`yesterday`)
- **Fim**: 15 dias à frente (`fifteenDaysAhead`)

## 🚨 Tratamento de Erros

- Try/catch no handler principal
- Log de erros no console
- Garantia de destruição de conexões no bloco `finally`

## 📝 Observações Importantes

1. **Performance**: Usa `Promise.all` para execução paralela sempre que possível
2. **Deduplicação**: Usa `Set` para garantir unicidade de telefones e CPFs
3. **Normalização**: Telefones e CPFs são normalizados antes das comparações
4. **Múltiplas Fontes**: Busca leads tanto no sistema v1 quanto v2
5. **Verificação de Convênio**: O `findLeadsOfCartOfRecoveryOfToday` já filtra pacientes sem convênio

## 🔄 Dependências de Outros Módulos

- `src/utils/date.ts`: Funções de data (`fifteenDaysAhead`, `yesterday`)
- `src/utils/formatAnyThing.ts`: Normalização de telefone
- `src/aws/sqs.ts`: Envio para fila
- `src/feegow/client.ts`: Cliente da API Feegow

## 🎯 Próximos Passos (Sugestões)

1. Adicionar métricas de quantos leads são enviados
2. Implementar retry em caso de falha no envio
3. Adicionar logs estruturados para melhor rastreabilidade
4. Considerar cache para consultas repetitivas ao Feegow
