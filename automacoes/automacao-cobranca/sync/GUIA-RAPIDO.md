# Automação: Sync - Guia Rápido

## 🎯 Resumo Executivo

Esta automação é a **mais importante do sistema**, sincronizando **diariamente todas as assinaturas inadimplentes** com o RD Station CRM, organizando-as automaticamente em stages específicos baseados no tempo de inadimplência (1-60, 61-180, 181-365, 366+ dias).

### Quando executa?

⏰ **Todos os dias às 3h UTC** (Meia-noite horário de Brasília)

### O que faz?

1. Busca **todas** as assinaturas com faturas vencidas
2. Valida regras especiais (tickets, faturas abertas)
3. Calcula dias de inadimplência
4. **Atribui stage automaticamente** por faixa de dias
5. Cria/atualiza deals no RD Station CRM

---

## 📊 Critérios de Seleção

### Incluídos ✅

- Faturas: Status `OPENED` e vencidas (< hoje)
- Assinatura: Status **diferente** de `CANCELED`
- Assinatura: **NÃO** aguardando primeiro pagamento
- Tipo: Pessoa Física (`PF`)
- Atraso: **Até 720 dias** (2 anos)
- Tem faturas abertas na validação adicional

### Excluídos ❌

- Faturas não vencidas
- Assinaturas canceladas
- Aguardando primeiro pagamento
- Pessoa Jurídica (PJ)
- Mais de 2 anos de atraso
- **Tickets/carnês DEFAULT com ≤ 2 dias de atraso**
- Assinaturas sem faturas abertas

---

## 🎯 Stages Automáticos

### Organização por Faixa de Inadimplência

| Dias de Atraso | Stage | Estratégia | Prioridade |
|----------------|-------|------------|------------|
| **1-60 dias** | `STAGES_1_60_DIAS` | Cobrança inicial/amigável | 🟢 Baixa |
| **61-180 dias** | `STAGES_61_180_DIAS` | Cobrança intensificada | 🟡 Média |
| **181-365 dias** | `STAGES_181_365_DIAS` | Cobrança crítica | 🟠 Alta |
| **366+ dias** | `STAGES_366_DIAS` | Judicial/Encerramento | 🔴 Crítica |

### Lógica de Atribuição

```typescript
// Padrão: 1-60 dias
let stage = STAGES_1_60_DIAS;

// Progressão automática
if (days > 60)  { stage = STAGES_61_180_DIAS }
if (days > 180) { stage = STAGES_181_365_DIAS }
if (days > 365) { stage = STAGES_366_DIAS }
```

---

## 🔄 Fluxo Simplificado

```text
Cron Job (Diário 3h)
    ↓
Busca todas assinaturas inadimplentes
    ↓
Para cada assinatura:
  ├─ Valida faturas abertas
  ├─ Valida regra tickets (> 2 dias)
  ├─ Calcula dias de atraso
  ├─ Define stage automaticamente
  └─ Cria/atualiza deal no CRM
```

---

## 💡 Regras Especiais

### 1. Carência de 2 Dias para Tickets

**Regra**: Tickets/carnês DEFAULT precisam ter **mais de 2 dias de atraso**

**Por quê?**: Evita cobrança prematura de carnês recém-emitidos

**Exemplo**:

- Vencimento: 01/10/2025
- Hoje: 02/10/2025 (1 dia) → ❌ **NÃO processa**
- Hoje: 03/10/2025 (2 dias) → ❌ **NÃO processa**
- Hoje: 04/10/2025 (3 dias) → ✅ **Processa**

### 2. Limite de 2 Anos

**Regra**: Assinaturas com mais de **720 dias** não são sincronizadas

**Motivo**: Foco em inadimplências recuperáveis

### 3. Apenas Pessoa Física

**Regra**: `type_sub = 'PF'`

**Motivo**: PJ (Pessoa Jurídica) tem fluxo diferente

### 4. Sem Faturas Abertas

**Regra**: Se `invoices.length == 0` → pula assinatura

**Motivo**: Nada a cobrar no momento

---

## 📋 Informações no CRM

### Dados do Deal

- **Assinatura**: `{id}` (apenas número, sem sufixo)
- **Titular**: Nome, email, telefones
- **Tipo**: Baseado no payment_type
- **Stage**: **Atribuído automaticamente** por dias
- **Classificação**: Baseada em score e tipo

### Campos Customizados

- ⏱️ **Tempo Inadimplente**: Dias de atraso
- 🆔 **CPF**: Do titular
- 💳 **Forma de Pagamento**: CREDIT_CARD, TICKET, etc.
- 📦 **Plano**: Nome do plano contratado
- ✅ **Status**: OK, BLOCKED, etc.
- 📊 **Score**: Do cartão (se disponível)
- 🚫 **Código/Erro**: Retorno ABECS (se aplicável)

### Anotações

- 🔗 Link direto para o backoffice
- 📄 Lista completa de faturas abertas

---

## ⚙️ Configurações Técnicas

### Schedule Lambda

- **Timeout**: 15 minutos (900s)
- **Memória**: 416 MB (maior que outras)
- **Handler**: `src/schedules/sync.handler`
- **Trigger**: Diário às 3h UTC

### Worker Lambda

- **Timeout**: 3 minutos (180s)
- **Memória**: 256 MB
- **Handler**: `src/workers/sync.handler`
- **Concorrência**: Máximo 2 execuções simultâneas
- **Batch Size**: 1 mensagem por vez

### Fila SQS

- **Delay**: 30 segundos
- **Concorrência Máxima**: 2
- **Processamento**: Sequencial com 5s entre registros

---

## 📈 Monitoramento

### Logs de Sucesso

```text
(Sucesso implícito - ID do deal retornado)
```

### Logs de Aviso

```text
Assinatura {id} não possui faturas abertas ou não é válida
Falha ao processar lead para a assinatura {id}
```

### Logs de Erro

```text
Erro ao processar assinatura {id}: {detalhes}
```

### Logs de Validação

```text
Cliente tem pagamento tipo ticket ou tickets por padrão mas ainda não deve há 2 dias
```

---

## 🔍 Troubleshooting

### Problema: Assinatura não apareceu no CRM

**Checklist de Validação**:

1. ✅ Tem faturas abertas?
2. ✅ Faturas estão vencidas (< hoje)?
3. ✅ Status ≠ CANCELED?
4. ✅ Não está aguardando primeiro pagamento?
5. ✅ É Pessoa Física (PF)?
6. ✅ Atraso < 720 dias?
7. ✅ Se ticket/carnê DEFAULT: atraso > 2 dias?

### Problema: Deal no stage errado

**Causa**: Dias de inadimplência mudaram desde última execução

**Solução**: Normal! A próxima execução (amanhã) atualizará o stage automaticamente.

**Exemplo**:

- Hoje (60 dias): Stage 1-60 dias
- Amanhã (61 dias): Sync move para 61-180 dias

### Problema: Volume muito alto

**Diagnóstico**:

- Verificar quantidade total de inadimplentes
- Analisar distribuição por faixas
- Identificar gargalos no processamento

**Ações**:

- Considerar aumentar `maxConcurrency`
- Aumentar `memorySize` se necessário
- Revisar limite de 720 dias
- Otimizar query SQL

### Problema: Timeout no schedule

**Causas**:

- Query SQL lenta (sem índices)
- Volume muito grande
- Latência de banco/rede

**Soluções**:

- Adicionar índices na tabela
- Aumentar timeout
- Verificar saúde do banco

---

## 📊 Capacidade de Processamento

### Estimativa com Configuração Atual

- **Concorrência**: 2 workers simultâneos
- **Delay**: 5s entre registros por worker
- **Throughput**: ~24 assinaturas/minuto
- **Capacidade/hora**: ~1.440 assinaturas
- **Capacidade máxima (15min)**: ~360 assinaturas

### Recomendações por Volume

| Assinaturas | Ação |
|-------------|------|
| < 300 | ✅ Configuração OK |
| 300-500 | ⚠️ Considerar aumentar concurrency |
| 500-1000 | 🔶 Aumentar para 3-4 workers |
| > 1000 | 🔴 Revisar arquitetura |

---

## 🎯 Diferenças vs Outras Automações

| Aspecto | Sync | Tickets Renewal | Annual |
|---------|------|-----------------|---------|
| **Escopo** | Todas inadimplências | Só carnês 4 meses | Só renovações |
| **Frequência** | Diária | Semanal | Mensal |
| **Stage** | Dinâmico | Fixo | Fixo |
| **ID Deal** | `{id}` | `{id}-ticket` | `{id}-anual` |
| **Volume** | Alto | Médio | Baixo |
| **Memória** | 416/256 MB | 168/168 MB | 168/168 MB |
| **Importância** | 🔥 Crítica | 📊 Alta | 📅 Média |

---

## 💼 Importância Estratégica

O **Sync** é a automação mais crítica porque:

1. 🔄 **Execução diária**: Mantém CRM sempre atualizado
2. 📊 **Cobertura total**: Processa **todas** as inadimplências
3. 🎯 **Organização inteligente**: Distribui por stages automaticamente
4. 💰 **Impacto direto**: Base para recuperação de receita
5. 📈 **Visão completa**: Permite análise de todo funil de cobrança
6. ⚡ **Priorização**: Stages permitem foco correto da equipe

---

## 🔗 Links Úteis

- 📖 [Documentação Técnica Completa](./README.md)
- 📊 [Fluxo Visual Detalhado](./fluxo.md)
- ⚙️ [Configuração Serverless](../../serverless.yml)
- 💻 [Código do Schedule](../../src/schedules/sync.ts)
- 🔧 [Código do Worker](../../src/workers/sync.ts)
- 🧩 [Lógica de Negócio](../../src/use-cases/sync-worker.ts)

---

## 📞 Suporte

Para dúvidas ou problemas:

1. Consulte a [documentação técnica completa](./README.md)
2. Verifique os logs no CloudWatch
3. Analise o [diagrama de fluxo](./fluxo.md)
4. Valide os critérios SQL da query
5. Confirme regras especiais (tickets, faturas, etc.)

---

**💡 Dica**: Esta é a automação central do sistema! Monitore-a diariamente para garantir sincronização completa e correta das inadimplências. O sucesso da cobrança depende dela!

**⚠️ Atenção**: Se o Sync falhar, toda a operação de cobrança é comprometida. Priorize sempre sua estabilidade e performance!
