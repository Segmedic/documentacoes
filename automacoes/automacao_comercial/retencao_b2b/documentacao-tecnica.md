# Documentação Técnica - Retenção de Ex-Colaboradores B2B

## 📋 Visão Geral

A automação de **Retenção de Ex-Colaboradores B2B** tem como objetivo identificar dependentes que foram removidos de planos empresariais (PJ) do ClubFlex no último mês e criar leads qualificados para ações de retenção, oferecendo planos individuais ou outros benefícios.

## 🎯 Objetivo

Capturar ex-dependentes do ClubFlex que:

- Foram **removidos** de planos PJ (empresariais) no último mês
- **Não** estão bloqueados ou ainda ativos (OK)
- Possuem CPF válido
- Têm telefone para contato
- Podem ser contatados para ofertas de retenção

## 🔧 Tecnologias e Dependências

### Serviços Externos

- **ClubFlex (MySQL)**: Base de dados de assinaturas e dependentes
- **Medula (PostgreSQL)**: Data warehouse com histórico de atendimentos
- **AWS SQS**: Fila para envio de leads

### Repositórios

- `ClubflexRepository`: Consultas ao banco ClubFlex
- `MedulaRepository`: Consultas ao data warehouse Medula

### Utilitários

- `diferencaFormatada`: Calcula diferença entre datas de forma legível

## 📊 Fluxo de Execução

### 1. Inicialização

```typescript
handler()
```

- Cria conexão com ClubFlex (MySQL)
- Cria conexão com Medula (PostgreSQL)
- Inicializa os serviços

### 2. Busca de Dependentes Removidos

Executa query no ClubFlex que busca:

- Status = 'REMOVED'
- CPF válido (regex valida formato)
- Tipo de assinatura = 'PJ' (empresarial)
- Data de remoção entre 1 mês atrás e ontem
- Ordenado por data de remoção (DESC)

### 3. Deduplicação e Filtragem

Processa a lista usando `reduce` com lógica complexa:

```typescript
map: Map<cpf, lead>    // Leads únicos válidos
skip: Set<cpf>         // CPFs a ignorar
```

**Lógica de Filtragem:**

- Se CPF já está em `skip` → **Ignora**
- Se status = 'BLOCKED' ou 'OK' → **Adiciona ao skip e remove do map**
- Se status = 'REMOVED' e CPF novo → **Adiciona ao map**

**Resultado:** Apenas CPFs com status REMOVED e únicos

### 4. Enriquecimento com Dados do Medula

Para cada CPF válido:

- Busca histórico de atendimentos no Medula
- Obtém unidades mais utilizadas
- Obtém procedimentos mais realizados

### 5. Construção do Lead Enriquecido

Para cada lead, calcula e adiciona:

- **Período de assinatura**: Tempo que foi assinante
- **Tempo sem assinatura**: Tempo desde a remoção
- **Dados pessoais**: Nome, email, telefone, sexo, idade
- **Histórico de uso**: Unidades e procedimentos mais usados
- **Link Feegow**: URL direta para o paciente no sistema

**Prioridade de Dados:**

- Dados do Medula (se existir) > Dados do ClubFlex
- Se não há dados no Medula, usa dados do ClubFlex

### 6. Validação Final

Filtra leads que:

- ✅ Possuem telefone válido (não vazio)
- ❌ Remove leads sem telefone

### 7. Envio em Lote

- Usa `Promise.all` para enviar todos os leads em paralelo
- Cada lead é enviado para a fila SQS

### 8. Finalização

No bloco `finally`:

- Destrói conexão ClubFlex
- Destrói conexão Medula

## 📦 Estrutura de Dados

### GetDependentsRemovedResponse (ClubFlex)

```typescript
{
  cpf: string,
  status: 'REMOVED' | 'OK' | 'BLOCKED',
  name: string,
  email: string,
  phone: string,
  sex: string,
  date_remove: Date,
  date_of_insert: Date,
  type: 'titular' | 'dependente'
}
```

### Lead Enriquecido (Payload Final)

```typescript
{
  name: string,                          // Nome do paciente
  email: string,                         // Email de contato
  phone: string,                         // Telefone (obrigatório)
  period_of_subscription: string,        // Ex: "2 meses, 15 dias"
  time_without_subscription: string,     // Ex: "10 dias"
  sexo: 'Masculino' | 'Feminino',       // Sexo
  age: number | "Nao informado",         // Idade
  procedure_max_served: string,          // Procedimento mais usado
  unit_max_served: string,               // Unidade mais usada
  patient_feegow: string                 // URL do Feegow
}
```

### LeadEvent

```typescript
{
  origin: "retencao_excolaboradores_b2b",
  payload: LeadEnriquecido
}
```

## 🔍 Critérios de Filtragem

### Query ClubFlex - Filtros SQL

```sql
WHERE 
  d.status = 'REMOVED'                                    -- Removidos
  AND d.cpf REGEXP '^(?!([0-9])\\1{10})[0-9]{11}$'      -- CPF válido
  AND s.type_sub = 'PJ'                                  -- Plano empresarial
  AND d.date_of_removal >= CURRENT_DATE - INTERVAL 1 MONTH  -- Último mês
  AND d.date_of_removal < CURRENT_DATE                  -- Até ontem
```

### Validação de CPF (Regex)

- **Formato**: 11 dígitos numéricos
- **Rejeita**: CPFs com todos os dígitos iguais (111.111.111-11, etc)

### Deduplicação em Runtime

```typescript
// Se já processado, pula
if (skip.has(cpf)) return;

// Se BLOCKED ou OK, marca para pular
if (status === 'BLOCKED' || status === 'OK') {
  skip.add(cpf);
  map.delete(cpf);
}

// Se REMOVED e novo, adiciona
if (!map.has(cpf)) {
  map.set(cpf, item);
}
```

### Validação Final

- **Obrigatório**: Telefone não vazio
- **Opcional**: Outros campos podem ser "Não informado"

## ⚙️ Configurações

### Período de Busca

- **Início**: 1 mês atrás
- **Fim**: Ontem (CURRENT_DATE - 1)

### Tipo de Assinatura

- **Aceitos**: Apenas 'PJ' (planos empresariais)
- **Rejeitados**: Planos individuais

### Status Aceitos

- **Processados**: REMOVED
- **Ignorados**: OK, BLOCKED

## 🚨 Tratamento de Erros

- Try/catch no handler principal
- Log de erros no console
- Garantia de destruição de conexões no bloco `finally`

## 📝 Observações Importantes

1. **Enriquecimento de Dados**: Combina dados de 2 sistemas (ClubFlex + Medula)
2. **Priorização**: Dados do Medula são preferidos quando disponíveis
3. **Cálculo de Tempo**: Usa função `diferencaFormatada` para tempos legíveis
4. **Link Feegow**: Gera URL direta para o paciente no sistema médico
5. **Validação de Telefone**: Único campo obrigatório para envio
6. **Deduplicação Complexa**: Lógica de Map/Set para garantir unicidade
7. **Status Prioritário**: BLOCKED e OK excluem REMOVED do mesmo CPF
8. **Envio Paralelo**: Todos os leads são enviados simultaneamente

## 🔄 Dependências de Outros Módulos

- `src/utils/constantes.ts`: Função `diferencaFormatada`
- `src/aws/sqs.ts`: Envio para fila
- `src/clubflex/clubflexRepository.ts`: Acesso ao banco ClubFlex
- `src/medula/medulaRepository.ts`: Acesso ao data warehouse Medula

## 🎯 Diferenças vs Outras Automações

| Característica | Retenção B2B | Leads Amanhã | Leads Convênios |
|----------------|--------------|--------------|-----------------|
| Período | Último mês | D+1 | D-1 |
| Foco | Ex-dependentes ClubFlex | Agendamentos sem convênio | Atendimentos com convênio |
| Enriquecimento | Sim (2 sistemas) | Não | Não |
| Cálculo de Tempo | Sim | Não | Não |
| Deduplicação | Map/Set complexo | Set simples | DISTINCT ON |
| Fonte Primária | ClubFlex | Feegow | Medula |
| Validação Telefone | Obrigatória | Obrigatória | Não |

## 📊 Fontes de Dados

### ClubFlex (MySQL)

- **dependent**: Tabela de dependentes
- **subscription**: Tabela de assinaturas
- **holder**: Tabela de titulares

### Medula (PostgreSQL)

- **feegow_agendamentos_dw**: Histórico de agendamentos
- **feegow_atendimentos_dw**: Histórico de atendimentos

## 🎯 Casos de Uso Comercial

Esta automação é útil para:

1. **Retenção**: Oferecer plano individual para ex-dependentes
2. **Winback**: Reconquistar clientes que perderam o plano empresarial
3. **Upsell**: Oferecer ClubFlex individual
4. **Personalização**: Usar histórico de uso para ofertas direcionadas
5. **Timing**: Contato no momento ideal (até 1 mês após remoção)

## 🎯 Próximos Passos (Sugestões)

1. Adicionar métricas de conversão (quantos aceitaram a oferta)
2. Segmentar por tempo sem assinatura (mais recentes = maior prioridade)
3. Adicionar score baseado em frequência de uso
4. Implementar retry logic para falhas no envio
5. Adicionar validação de email além de telefone
6. Considerar envio escalonado ao invés de paralelo total
7. Adicionar logs estruturados com detalhes dos leads
8. Criar dashboard com taxa de remoção por empresa
