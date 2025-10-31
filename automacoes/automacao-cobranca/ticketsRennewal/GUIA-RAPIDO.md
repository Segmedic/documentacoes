# Automação: Tickets Renewal - Guia Rápido

## 🎯 Resumo Executivo

Esta automação processa **assinaturas com pagamento via carnê** que possuem faturas abertas, criando registros no RD Station CRM para acompanhamento comercial.

### Quando executa?

⏰ **Toda segunda-feira às 3h UTC** (Meia-noite horário de Brasília)

### O que faz?

1. Busca assinaturas elegíveis no banco DNA
2. Filtra apenas as com última data de pagamento ≤ 4 meses
3. Cria/atualiza deals no RD Station CRM
4. Adiciona informações detalhadas para ação comercial

---

## 📊 Critérios de Seleção

### Incluídos ✅

- Tipo de pagamento: `TICKET` ou `TICKETS`
- Status: **Diferente** de `CANCELED`
- Plano: Contém `CARNÊ` no nome
- Faturas abertas: **Até 1 fatura**
- Último pagamento: **Há 4 meses ou menos**

### Excluídos ❌

- Assinaturas canceladas
- Mais de 1 fatura aberta
- Último pagamento há mais de 4 meses
- Planos que não são de carnê

---

## 🔄 Fluxo Simplificado

``` bash
Cron Job (Segundas 3h)
    ↓
Busca assinaturas no banco
    ↓
Filtra por período (≤ 4 meses)
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

- **Assinatura**: `{id}-ticket`
- **Titular**: Nome, email, telefones
- **Tipo**: Carnê/Boleto
- **Stage**: Primeiro estágio do funil
- **Classificação**: Baseada em score e tipo

### Campos Customizados

- ⏱️ **Tempo Inadimplente**: Dias de atraso
- 🆔 **CPF**: Do titular
- 💳 **Forma de Pagamento**: TICKET/TICKETS
- 📦 **Plano**: Nome do plano contratado
- ✅ **Status**: OK, BLOCKED, etc.
- 📊 **Score**: Do cartão (se disponível)
- 🚫 **Código/Erro**: Retorno ABECS (se aplicável)

### Anotações

- 🔗 Link direto para o backoffice
- 📄 Lista de faturas abertas

---

## ⚙️ Configurações Técnicas

### Schedule Lambda

- **Timeout**: 15 minutos (900s)
- **Memória**: 168 MB
- **Handler**: `src/schedules/ticketsRennewal.handler`

### Worker Lambda

- **Timeout**: 3 minutos (180s)
- **Memória**: 168 MB
- **Handler**: `src/workers/ticketsRennewal.handler`
- **Concorrência**: Máximo 2 execuções simultâneas
- **Batch Size**: 1 mensagem por vez

### Fila SQS

- **Delay**: 30 segundos
- **Concorrência Máxima**: 2
- **Processamento**: Sequencial com 5s entre registros

---

## 📈 Monitoramento

### Logs de Sucesso

``` bash
Deal processado: {id_do_deal}
```

### Logs de Aviso

``` bash
Falta dados para a assinatura {id}
Falha ao processar lead para a assinatura {id}
```

### Logs de Erro

``` bash
Erro ao processar assinatura {id}: {detalhes}
```

---

## 🔍 Troubleshooting

### Problema: Assinatura não apareceu no CRM

**Verifique**:

- ✅ Assinatura atende aos critérios SQL?
- ✅ Último pagamento há 4 meses ou menos?
- ✅ Logs indicam processamento bem-sucedido?
- ✅ Deal pode ter sido criado anteriormente?

### Problema: Dados incompletos no CRM

**Possíveis causas**:

- ❌ Titular sem email ou telefone
- ❌ Plano não encontrado
- ❌ Faturas sem dados completos

### Problema: Worker com timeout

**Ações**:

1. Verificar latência da API do RD Station
2. Aumentar timeout do worker se necessário
3. Verificar quantidade de anotações sendo criadas

---

## 🔗 Links Úteis

- 📖 [Documentação Técnica Completa](./README.md)
- 📊 [Fluxo Visual Detalhado](./fluxo.md)
- ⚙️ [Configuração Serverless](../../serverless.yml)
- 💻 [Código do Schedule](../../src/schedules/ticketsRennewal.ts)
- 🔧 [Código do Worker](../../src/workers/ticketsRennewal.ts)
- 🧩 [Lógica de Negócio](../../src/use-cases/ticketRennewal-worker.ts)

---

## 📞 Suporte

Para dúvidas ou problemas:

1. Consulte a [documentação técnica completa](./README.md)
2. Verifique os logs no CloudWatch
3. Analise o [diagrama de fluxo](./fluxo.md)

---

**💡 Dica**: Visualize o diagrama do fluxo.md no GitHub para uma compreensão visual completa da automação!
