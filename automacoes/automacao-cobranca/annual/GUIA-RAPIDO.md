# Automação: Annual Renewal - Guia Rápido

## 🎯 Resumo Executivo

Esta automação processa **planos anuais que precisam ser renovados**, identificando clientes cujo último pagamento ocorreu no mesmo mês do ano anterior, criando registros no RD Station CRM para ação comercial proativa de renovação.

### Quando executa?

⏰ **Todo dia 1º de cada mês às 3h UTC** (Meia-noite horário de Brasília)

### O que faz?

1. Busca planos anuais com último pagamento até o ano anterior
2. Filtra apenas os que pagaram no mesmo mês do ano anterior
3. Cria/atualiza deals no RD Station CRM
4. Permite ação comercial **proativa** antes do vencimento

---

## 📊 Critérios de Seleção

### Incluídos ✅

- Plano: Contém `ANUAL` no nome
- Fatura: Status **diferente** de `CANCELED`
- Último pagamento: **Até o ano anterior**
- Mês do pagamento: **Igual ao mês atual**
- Status da assinatura: **Diferente** de `CANCELED`

### Excluídos ❌

- Planos que não são anuais
- Faturas canceladas
- Assinaturas canceladas
- Pagamentos que não foram no mesmo mês do ano anterior

### Exemplo Prático 💡

**Cenário**: Hoje é **1º de outubro de 2025**

- ✅ **Incluído**: Último pagamento em **outubro de 2024**
- ❌ **Excluído**: Último pagamento em **setembro de 2024**
- ❌ **Excluído**: Último pagamento em **outubro de 2025** (já renovado)

---

## 🔄 Fluxo Simplificado

```text
Cron Job (Dia 1º às 3h)
    ↓
Busca planos anuais (pagamento ≤ ano anterior)
    ↓
Filtra por mês (mesmo mês do ano anterior)
    ↓
Filtra status ativo (não cancelado)
    ↓
Envia para fila SQS
    ↓
Worker processa individualmente
    ↓
Cria/Atualiza deal no RD Station
```

---

## 📋 Informações no CRM

### Dados do Deal

- **Assinatura**: `{id}-anual`
- **Titular**: Nome, email, telefones
- **Tipo**: Baseado no payment_type
- **Stage**: Primeiro estágio do funil de renovação anual
- **Classificação**: Baseada em score e tipo

### Campos Customizados

- ⏱️ **Tempo Inadimplente**: Dias de atraso (se houver fatura aberta)
- 🆔 **CPF**: Do titular
- 💳 **Forma de Pagamento**: CREDIT_CARD, TICKETS, etc.
- 📦 **Plano**: Nome do plano anual contratado
- ✅ **Status**: OK, BLOCKED, etc.
- 📊 **Score**: Do cartão (se disponível)
- 🚫 **Código/Erro**: Retorno ABECS (se aplicável)

### Anotações

- 🔗 Link direto para o backoffice
- 📄 Lista de faturas abertas (se houver)

---

## ⚙️ Configurações Técnicas

### Schedule Lambda

- **Timeout**: 15 minutos (900s)
- **Memória**: 168 MB
- **Handler**: `src/schedules/annualRennewal.handler`
- **Trigger**: Todo dia 1º do mês às 3h UTC

### Worker Lambda

- **Timeout**: 3 minutos (180s)
- **Memória**: 168 MB
- **Handler**: `src/workers/annualRennewal.handler`
- **Concorrência**: Máximo 2 execuções simultâneas
- **Batch Size**: 1 mensagem por vez

### Fila SQS

- **Delay**: 30 segundos
- **Concorrência Máxima**: 2
- **Processamento**: Sequencial com 5s entre registros

---

## 📅 Lógica de Renovação

### Como Funciona a Identificação

**Passo 1**: Busca ampla

- Recupera todos os planos anuais
- Filtra último pagamento até ano anterior

**Passo 2**: Filtro mensal

- Compara mês do último pagamento com mês atual
- Apenas registros com mesmo mês passam

**Passo 3**: Filtro de status

- Remove assinaturas canceladas
- Envia para processamento

### Exemplo Detalhado

| Data Atual | Busca Pagamentos Até | Filtra Mês | Resultado |
|------------|---------------------|------------|-----------|
| 1º out/2025 | 2024 | Outubro | Renovações out/2025 |
| 1º jan/2025 | 2024 | Janeiro | Renovações jan/2025 |
| 1º dez/2025 | 2024 | Dezembro | Renovações dez/2025 |

---

## 📈 Monitoramento

### Logs de Sucesso

```text
Deal processado: {id_do_deal}
```

### Logs de Aviso

```text
Falta dados para a assinatura {id}
Falha ao processar lead para a assinatura {id}
```

### Logs de Erro

```text
Erro ao processar assinatura {id}: {detalhes}
```

---

## 🔍 Troubleshooting

### Problema: Plano não apareceu no CRM

**Verifique**:

- ✅ Plano tem "ANUAL" no nome?
- ✅ Último pagamento foi no mesmo mês do ano anterior?
- ✅ Status da assinatura não é CANCELED?
- ✅ Fatura não está cancelada?
- ✅ Logs indicam processamento bem-sucedido?

### Problema: Muitos deals criados no mesmo mês

**Causa provável**: Vários clientes renovam no mesmo mês

**Ação**: Normal! Isso indica renovações concentradas em determinado período

### Problema: Nenhum deal criado no mês

**Possíveis causas**:

1. Nenhum cliente pagou neste mês no ano anterior
2. Todos os planos desse mês foram cancelados
3. Erro na execução do schedule

**Verificar**:

- Logs do CloudWatch
- Query manual no banco de dados
- Histórico de execuções da Lambda

### Problema: Deal criado no mês errado

**Causa provável**: Timezone UTC vs local

**Verificar**: Execução acontece após meia-noite UTC

---

## 🎯 Diferenças vs Tickets Renewal

| Aspecto | Annual Renewal | Tickets Renewal |
|---------|----------------|-----------------|
| **Quando** | Dia 1º do mês | Toda segunda-feira |
| **Objetivo** | Renovação proativa | Cobrança retroativa |
| **Período** | Mesmo mês ano anterior | Últimos 4 meses |
| **Tipo Plano** | Anuais | Carnês |
| **Identificador** | `{id}-anual` | `{id}-ticket` |

---

## 💡 Vantagens da Abordagem

- ⏰ **Antecipação**: Identifica renovações no início do mês
- 🎯 **Precisão**: Foca apenas nos planos do mês corrente
- 📊 **Previsibilidade**: Execução mensal consistente
- 🔄 **Proatividade**: Permite ação antes do vencimento
- 💼 **Comercial**: Tempo para negociar renovação

---

## 🔗 Links Úteis

- 📖 [Documentação Técnica Completa](./README.md)
- 📊 [Fluxo Visual Detalhado](./fluxo.md)
- ⚙️ [Configuração Serverless](../../serverless.yml)
- 💻 [Código do Schedule](../../src/schedules/annualRennewal.ts)
- 🔧 [Código do Worker](../../src/workers/annualRennewal.ts)
- 🧩 [Lógica de Negócio](../../src/use-cases/annualRennewal-worker.ts)

---

## 📞 Suporte

Para dúvidas ou problemas:

1. Consulte a [documentação técnica completa](./README.md)
2. Verifique os logs no CloudWatch
3. Analise o [diagrama de fluxo](./fluxo.md)
4. Valide os utilitários de data (`src/@utils/date.ts`)

---

**💡 Dica**: Esta automação é ideal para manter relacionamento comercial proativo, permitindo preparar propostas de renovação com antecedência e evitar churn!
