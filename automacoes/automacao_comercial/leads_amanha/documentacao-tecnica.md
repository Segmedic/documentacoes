# Documentação Técnica - Leads de Amanhã

## 📋 Visão Geral

A automação de **Leads de Amanhã** tem como objetivo identificar pacientes com agendamentos para o dia seguinte que não possuem convênio médico e que ainda não estão no funil de vendas do CRM, enviando esses leads qualificados para conversão.

## 🎯 Objetivo

Capturar leads de agendamentos do dia seguinte (D+1) que:

- Não possuem convênio médico (são particulares)
- Não são membros ClubFlex
- Não possuem oportunidades ativas no CRM
- Atendem critérios específicos de qualificação

## 🔧 Tecnologias e Dependências

### Bibliotecas

- **date-fns**: Manipulação de datas (adicionar 1 dia)

### Serviços Externos

- **Feegow**: Sistema de gestão médica (agendamentos do dia seguinte)
- **Medula**: Verificação de oportunidades no CRM
- **ClubFlex**: Verificação de membros do clube de benefícios
- **AWS SQS**: Fila para envio de leads

### Repositórios

- `MedulaRepository`: Consultas ao CRM
- `ClubflexRepository`: Consultas ao ClubFlex

## 📊 Fluxo de Execução

### 1. Definição do Período

```typescript
start = new Date()        // Hoje
end = add(start, { days: 1 })  // Amanhã
```

Busca agendamentos de **hoje até amanhã** (janela de 24h).

### 2. Inicialização

- Cria conexões com Medula e ClubFlex
- Inicializa o cliente Feegow

### 3. Busca de Agendamentos

Busca todos os agendamentos do período no Feegow e extrai:

- **Telefones**: Normalizados (sem formatação)
- **CPFs**: Para verificação ClubFlex
- **Dados completos**: Para processamento posterior

### 4. Verificações de Negócio (Paralelas)

```typescript
Promise.all([
  medulaService.verifyLeadsHaveOportunityInCrm(phones),
  clubflexService.verifyIsClubFlex(cpfs)
])
```

- Verifica se os telefones têm oportunidades no CRM
- Verifica se os CPFs são membros ClubFlex

### 5. Filtragem de Leads Qualificados

Para cada agendamento, aplica os seguintes filtros:

#### 5.1. Validações de Qualidade

✅ **Status válido**: Não pode ser "Desmarcado pelo paciente"  
✅ **Procedimento válido**: Não pode ser "Visita Representante"  
✅ **Tipo de consulta**: Não pode ser retorno (Retorno !== "Sim")  
✅ **Convênio**: Não pode ter convênio (NomeConvenio === "")  
✅ **Telefone**: Deve ter telefone válido  
✅ **Tabela válida**: Deve estar em uma das tabelas permitidas

#### 5.2. Validações de Negócio

✅ **NÃO** é ClubFlex  
✅ **NÃO** possui oportunidade no CRM  
✅ **NÃO** foi processado anteriormente (deduplicação)

### 6. Envio para Fila

Para cada lead qualificado:

- Cria um `LeadEvent` com origem "leads_amanha"
- Envia para a fila SQS
- Adiciona o PacienteID ao Set de processados (evita duplicatas)

### 7. Finalização

- Destrói conexões com ClubFlex e Medula
- Retorna mensagem de conclusão

## 📦 Estrutura de Dados

### LeadEvent

```typescript
{
  origin: "leads_amanha",
  payload: {
    PacienteID: number,
    Cel1: string,
    CPF: string,
    NomeConvenio: string,
    StaConsulta: string,
    NomeProcedimento: string,
    Retorno: string,
    NomeTabela: string,
    normalizedPhone: string,
    // ... outros campos do agendamento Feegow
  }
}
```

## 🔍 Critérios de Filtragem

### Tabelas Válidas (Particulares)

As seguintes tabelas são aceitas:

- Particular
- Interclinica
- Interclinica 500
- Interclinicas Coleta Domiciliar
- Interclinicas Faturado
- Sindicato Dos Rodoviários 2024.3
- Sindicato dos Rodoviários
- Sindicato dos Rodoviários - Faturado

### Exclusões Automáticas

1. Agendamentos desmarcados pelo paciente
2. Visitas de representante
3. Consultas de retorno
4. Pacientes com convênio
5. Pacientes sem telefone
6. Pacientes ClubFlex
7. Pacientes com oportunidades no CRM
8. Pacientes já processados (duplicatas)

## ⚙️ Configurações

### Período de Busca

- **Início**: Hoje (data/hora atual)
- **Fim**: Amanhã (+ 24 horas)

### Normalização

- Telefones são normalizados antes das comparações
- Remove formatação e mantém apenas números

## 🚨 Tratamento de Erros

- Não possui try/catch explícito no handler
- Erros serão propagados para o runtime
- Garantia de destruição de conexões ao final (sem finally)

## 📝 Observações Importantes

1. **Deduplicação em Tempo Real**: Usa `Set<number>` para evitar processar o mesmo paciente duas vezes
2. **Normalização de Telefone**: Essencial para matching com CRM
3. **Execução Paralela**: Usa `Promise.all` para otimizar consultas externas
4. **Janela de Tempo**: Busca agendamentos de hoje até amanhã (não apenas amanhã)
5. **Tabelas Específicas**: Apenas tabelas particulares pré-definidas são aceitas

## 🔄 Dependências de Outros Módulos

- `date-fns`: Cálculo de datas
- `src/utils/formatAnyThing.ts`: Normalização de telefone
- `src/utils/constantes.ts`: Lista de tabelas válidas
- `src/aws/sqs.ts`: Envio para fila
- `src/feegow/client.ts`: Cliente da API Feegow

## 🎯 Diferenças vs Outras Automações

| Característica | Leads Amanhã | Recuperação Agendamento |
|----------------|--------------|-------------------------|
| Período | Hoje até amanhã | Ontem até +15 dias |
| Foco | Agendamentos futuros | Carrinhos abandonados |
| Validação | Múltiplos filtros de qualidade | Foco em especialidade |
| Deduplicação | Em tempo real (Set) | Baseada em CPF único |

## 🎯 Próximos Passos (Sugestões)

1. Adicionar try/catch para melhor tratamento de erros
2. Adicionar logs estruturados para rastreabilidade
3. Implementar métricas (quantos leads enviados vs filtrados)
4. Considerar cache para tabelas válidas
5. Adicionar retry logic para falhas no envio à fila
