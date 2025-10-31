# Guia Rápido - Automação Update

## 📋 Resumo Executivo

A automação **Update** é responsável por sincronizar o status dos deals no **RD Station CRM** com a situação real das assinaturas no banco de dados DNA. Diferente das outras automações (que sincronizam Banco → CRM), esta funciona na **direção inversa** (CRM → Banco), realizando uma "limpeza" nos pipelines do CRM.

### Características Principais

- **Execução**: Diária às **22h UTC** (19h BRT)
- **Direção**: CRM → Banco de Dados (inversa)
- **Objetivo**: Marcar como LOST deals regularizados ou no último dia do mês
- **Pipelines**: Todos os 4 pipelines (1-60, 61-180, 181-365, 366+ dias)
- **Volume**: Até 9.800 deals por execução (49 páginas × 4 pipelines)

---

## 🎯 Critérios de Processamento

### 1. Critério Principal: Último Dia do Mês

| Critério | Ação | Motivo |
|----------|------|--------|
| É o último dia do mês | Marca deal como **LOST** | Fechamento mensal obrigatório |
| Não é o último dia do mês | Verifica critério 2 | Continua validação |

**Exemplo:**

``` bash
Data: 31/01/2024 → Todos os deals são marcados como LOST
Data: 15/01/2024 → Avalia critério 2
```

### 2. Critério Secundário: Regularização

| Situação | Faturas Abertas | Ação | Motivo |
|----------|-----------------|------|--------|
| Cliente regularizado | ❌ Não tem | Marca como **LOST** | Cliente não está mais inadimplente |
| Cliente inadimplente | ✅ Tem | Mantém **OPEN** | Continua no funil de cobrança |

**Exemplo:**

``` bash
Assinatura 12345:
- Tinha 3 faturas abertas
- Cliente pagou todas
- Agora: 0 faturas abertas
→ Deal 67890 marcado como LOST
```

---

## 🔄 Fluxo Simplificado

``` bash
┌─────────────────────────────────────────────────────────────┐
│ 1. COLETA (Schedule - 22h UTC)                              │
├─────────────────────────────────────────────────────────────┤
│ • Busca todos os deals dos 4 pipelines do RD Station        │
│ • Pipeline 1-60 dias     (até 2.450 deals)                  │
│ • Pipeline 61-180 dias   (até 2.450 deals)                  │
│ • Pipeline 181-365 dias  (até 2.450 deals)                  │
│ • Pipeline 366+ dias     (até 2.450 deals)                  │
│ • Delays: 2s entre páginas, 3s entre pipelines              │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. EXTRAÇÃO                                                  │
├─────────────────────────────────────────────────────────────┤
│ • Extrai subscription ID do nome da organização             │
│ • Exemplo: "Assinatura 12345" → ID: 12345                   │
│ • Usa regex para pegar apenas os números                    │
│ • Cria UpdateDTO: {subscription, dealId}                    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. ENFILEIRAMENTO                                            │
├─────────────────────────────────────────────────────────────┤
│ • Envia cada UpdateDTO para fila SQS "update"               │
│ • Processamento assíncrono pelo Worker                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. VALIDAÇÃO (Worker)                                        │
├─────────────────────────────────────────────────────────────┤
│ • Busca assinatura no banco DNA                             │
│ • Busca faturas abertas da assinatura                       │
│ • Verifica se é último dia do mês                           │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. DECISÃO                                                   │
├─────────────────────────────────────────────────────────────┤
│ É último dia do mês?                                         │
│   ✅ SIM → Marca como LOST no RD Station                    │
│   ❌ NÃO → Verifica faturas abertas                         │
│                                                              │
│ Tem faturas abertas?                                         │
│   ✅ SIM → Mantém deal OPEN                                 │
│   ❌ NÃO → Marca como LOST (regularizado)                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Exemplos Práticos

### Exemplo 1: Último Dia do Mês

**Contexto:**

- Data: 30/04/2024 (último dia do mês)
- Deal ID: 67890
- Subscription: 12345
- Status no CRM: OPEN
- Faturas abertas: 2

**Processamento:**

``` bash
1. Schedule coleta o deal do pipeline
2. Extrai subscription ID: 12345
3. Worker valida: É 30/04/2024 (último dia)
4. Resultado: Marca como LOST imediatamente
5. Status final: LOST (independente das faturas)
```

### Exemplo 2: Cliente Regularizado

**Contexto:**

- Data: 15/03/2024 (meio do mês)
- Deal ID: 45678
- Subscription: 67890
- Status no CRM: OPEN
- Faturas abertas: 0 (cliente pagou tudo)

**Processamento:**

``` bash
1. Schedule coleta o deal do pipeline
2. Extrai subscription ID: 67890
3. Worker valida: Não é último dia
4. Verifica faturas: 0 faturas abertas
5. Resultado: Marca como LOST (regularizado)
6. Status final: LOST
```

### Exemplo 3: Cliente Ainda Inadimplente

**Contexto:**

- Data: 20/02/2024 (meio do mês)
- Deal ID: 23456
- Subscription: 34567
- Status no CRM: OPEN
- Faturas abertas: 3

**Processamento:**

``` bash
1. Schedule coleta o deal do pipeline
2. Extrai subscription ID: 34567
3. Worker valida: Não é último dia
4. Verifica faturas: 3 faturas abertas
5. Resultado: Mantém OPEN
6. Status final: OPEN (continua no funil)
```

---

## ⚙️ Configuração Rápida

### Variáveis de Ambiente Essenciais

```bash
# RD Station API
RD_STATION_TOKEN=seu_token_aqui
RD_STAGE_1_60=id_do_pipeline_1_60
RD_STAGE_61_180=id_do_pipeline_61_180
RD_STAGE_181_365=id_do_pipeline_181_365
RD_STAGE_366=id_do_pipeline_366_mais

# AWS
AWS_REGION=sa-east-1
SQS_UPDATE_QUEUE_URL=url_da_fila_update

# Banco de Dados
DB_HOST=host_do_mysql
DB_USER=usuario
DB_PASSWORD=senha
DB_DATABASE=dna
```

### CloudWatch Cron Expression

```yaml
events:
  - schedule:
      rate: cron(0 22 * * ? *)  # Todos os dias às 22h UTC
      enabled: true
```

---

## 🔍 Monitoramento

### Métricas Importantes

| Métrica | Local | O que Observar |
|---------|-------|----------------|
| **Deals coletados** | CloudWatch Logs (Schedule) | Volume total de deals nos 4 pipelines |
| **Taxa de envio** | CloudWatch Logs (Schedule) | Quantos UpdateDTOs foram enviados para SQS |
| **Deals marcados LOST** | CloudWatch Logs (Worker) | Quantos deals foram fechados |
| **Erros de validação** | CloudWatch Logs (Worker) | Problemas com subscription ID inválido |
| **Mensagens na fila** | AWS SQS Console | Acúmulo de mensagens não processadas |

### Logs de Sucesso

```javascript
// Schedule
"✅ Update schedule completed. Deals collected: 850, Sent to queue: 850"

// Worker
"✅ Deal 67890 marked as LOST - Last day of month"
"✅ Deal 45678 marked as LOST - Regularized (0 open invoices)"
"✅ Deal 23456 kept OPEN - 3 open invoices found"
```

---

## 🆘 Troubleshooting Rápido

### Problema 1: Deals não sendo marcados como LOST no último dia

**Sintomas:**

- É último dia do mês
- Deals continuam OPEN no CRM

**Causas Possíveis:**

``` bash
1. Erro na detecção de último dia:
   → Verificar timezone (UTC vs BRT)
   → Confirmar lógica de isLastDayOfMonth()

2. Erro na API do RD Station:
   → Verificar token expirado
   → Conferir logs de erro no Worker
```

**Solução:**

```bash
# Verificar logs do Worker
aws logs tail /aws/lambda/automacao-cobranca-prod-update-worker --follow

# Testar manualmente
curl -X PUT "https://api.rd.services/platform/deals/{deal_id}" \
  -H "Authorization: Bearer $RD_STATION_TOKEN" \
  -d '{"deal": {"win": false}}'
```

### Problema 2: Cliente regularizado não sendo removido

**Sintomas:**

- Cliente pagou todas as faturas
- Deal continua OPEN no CRM

**Causas Possíveis:**

``` bash
1. Delay na atualização do banco:
   → Automação Sync ainda não processou o pagamento
   → Faturas ainda marcadas como abertas

2. Problema na query de faturas:
   → SQL não está retornando faturas corretas
   → Filtro de status incorreto
```

**Solução:**

```sql
-- Verificar faturas da assinatura
SELECT * FROM invoices 
WHERE subscription_id = 12345 
AND status IN ('pending', 'overdue')
ORDER BY due_date;

-- Se retornar vazio, deal deveria ser LOST
```

### Problema 3: Volume muito alto de deals

**Sintomas:**

- Schedule demora muito (timeout)
- Mais de 9.800 deals nos pipelines

**Causas Possíveis:**

``` bash
1. Limite de paginação atingido:
   → 49 páginas × 50 deals × 4 pipelines = 9.800
   → Deals além desse limite não são processados

2. Rate limiting muito conservador:
   → 2s entre páginas + 3s entre pipelines
```

**Solução:**

```typescript
// Ajustar limites no Schedule
const MAX_PAGES_PER_STAGE = 60; // Aumentar de 49 para 60
const DELAY_BETWEEN_PAGES = 1500; // Reduzir de 2000 para 1500ms
```

### Problema 4: Subscription ID não encontrado

**Sintomas:**

- Log: "Subscription ID is invalid"
- UpdateDTO não é enviado para fila

**Causas Possíveis:**

``` bash
1. Nome da organização em formato diferente:
   → Esperado: "Assinatura 12345"
   → Encontrado: "Contrato XYZ" (sem números)

2. Regex não capturando números:
   → Organização sem ID numérico
```

**Solução:**

```typescript
// Verificar nome da organização no RD
console.log('Organization name:', deal.organization.name);

// Ajustar regex se necessário
const subscriptionId = deal.organization.name
  .match(/\d+/)?.[0];
```

---

## 🔗 Relação com Automação Sync

A automação Update trabalha em **complemento** com a automação Sync:

| Aspecto | Sync | Update |
|---------|------|--------|
| **Direção** | Banco → CRM | CRM → Banco |
| **Horário** | 03h UTC | 22h UTC |
| **Objetivo** | Criar/atualizar deals | Limpar deals regularizados |
| **Frequência** | Diária | Diária |
| **Critério** | Faturas abertas | Sem faturas OU último dia |
| **Ação** | CREATE / UPDATE deal | LOST deal |

### Ciclo Completo

``` bash
DIA 1:
03h → Sync cria/atualiza deals com inadimplências
22h → Update remove deals regularizados

DIA 2:
03h → Sync adiciona novos inadimplentes
22h → Update remove os que pagaram

DIA 30 (último dia):
03h → Sync adiciona novos inadimplentes
22h → Update marca TODOS como LOST (fechamento mensal)

DIA 31 (início do mês):
03h → Sync cria novos deals do zero
22h → Update mantém apenas inadimplentes ativos
```

---

## 📈 Análise de Volume

### Estimativa de Processamento

| Pipeline | Deals Médios | Tempo Estimado |
|----------|--------------|----------------|
| 1-60 dias | 800 deals | ~2 minutos |
| 61-180 dias | 500 deals | ~1.5 minutos |
| 181-365 dias | 300 deals | ~1 minuto |
| 366+ dias | 200 deals | ~1 minuto |
| **Total** | **1.800 deals** | **~6 minutos** |

### Picos de Volume

- **Último dia do mês**: TODOS os deals são processados
- **Início do mês**: Volume reduzido (Sync acabou de criar)
- **Meio do mês**: Volume médio (apenas regularizados)

---

## 🎓 Dicas e Best Practices

1. **Monitorar no último dia do mês**
   - É quando o maior volume é processado
   - Confirmar que todos os deals foram fechados

2. **Coordenar com Sync**
   - Update roda depois do Sync (22h vs 03h)
   - Garante que dados estejam atualizados

3. **Validar subscription IDs**
   - Garantir padronização no nome da organização
   - Formato: "Assinatura {ID}"

4. **Revisar deals LOST**
   - Analisar se clientes realmente regularizaram
   - Evitar falsos positivos

5. **Alertas automáticos**
   - Configurar alarmes para timeout do Schedule
   - Notificar se nenhum deal for processado

---

## 📚 Documentação Relacionada

- [Documentação Técnica Completa](./README.md)
- [Fluxo Visual (Mermaid)](./fluxo.md)
- [Automação Sync](../sync/README.md)
- [Arquitetura Geral](../README.md)

---

**Última atualização:** $(date +%Y-%m-%d)
**Versão:** 1.0.0
