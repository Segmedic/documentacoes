# Comparativo das Automações

## 📊 Visão Geral Consolidada

Este documento apresenta uma visão comparativa das 4 automações do sistema de cobrança, destacando semelhanças, diferenças e complementaridade.

---

## 🔄 Fluxo Cronológico Diário

``` bash
┌─────────────────────────────────────────────────────────────┐
│ TIMELINE DIÁRIA (UTC)                                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  00h ──────────────────────────────────────────────────────  │
│                                                              │
│  03h ──► 🔥 SYNC (Diário)                                   │
│            └─ Sincroniza TODAS inadimplências (Banco → CRM) │
│                                                              │
│  03h ──► 🎫 TICKETS RENEWAL (Segunda-feira)                 │
│            └─ Carnês com faturas ≤ 4 meses (Banco → CRM)    │
│                                                              │
│  03h ──► 📅 ANNUAL RENEWAL (Dia 1º do mês)                  │
│            └─ Planos anuais a renovar (Banco → CRM)         │
│                                                              │
│  12h ──────────────────────────────────────────────────────  │
│                                                              │
│  22h ──► 🔄 UPDATE (Diário)                                 │
│            └─ Limpa deals regularizados (CRM → Banco)       │
│                                                              │
│  24h ──────────────────────────────────────────────────────  │
└─────────────────────────────────────────────────────────────┘
```

---

## 📋 Tabela Comparativa Detalhada

| Característica | Sync 🔥 | Tickets Renewal 🎫 | Annual 📅 | Update 🔄 |
|----------------|---------|---------------------|-----------|-----------|
| **Frequência** | Diária | Semanal (Segunda) | Mensal (Dia 1º) | Diária |
| **Horário UTC** | 3h | 3h | 3h | 22h |
| **Direção Dados** | Banco → CRM | Banco → CRM | Banco → CRM | CRM → Banco |
| **Criticidade** | 🔴 Crítica | 🟠 Alta | 🟡 Média | 🟢 Suporte |
| **Objetivo** | Sincronizar inadimplentes | Alertar carnês vencidos | Renovação proativa | Limpar regularizados |
| **Escopo** | Todos inadimplentes | Apenas carnês | Apenas anuais | Todos pipelines CRM |
| **Volume Médio** | 2.000-3.000 | 200-400 | 50-100 | 1.500-2.000 |
| **Pipelines CRM** | 4 stages | 4 stages | Separado | 4 stages |
| **Filtro Principal** | Faturas abertas | Últimos 4 meses | Mesmo mês ano anterior | Sem faturas abertas |
| **Ação CRM** | CREATE/UPDATE | CREATE/UPDATE | CREATE | LOST |
| **Rate Limiting** | 100ms/req | Batch 30 | 100ms/req | 2s/3s |
| **Dependências** | MySQL + RD API | MySQL + RD API | MySQL + RD API | MySQL + RD API |
| **Queue SQS** | sync | ticketsRennewal | annualRennewal | update |
| **Batch Size** | 1 | 1 | 1 | 1 |

---

## 🎯 Matriz de Critérios

### Critérios de Seleção de Assinaturas

| Automação | Critério Principal | Critérios Secundários | Exclusões |
|-----------|-------------------|----------------------|-----------|
| **Sync** | Tem faturas abertas | Status ativo | - |
| **Tickets** | Carnê + Faturas abertas | Último pagamento ≤ 4 meses | Planos anuais |
| **Annual** | Plano anual + Data renovação | Pagamento no mesmo mês ano anterior | Carnês |
| **Update** | CRM → Sem faturas abertas | Último dia do mês | - |

### Critérios de Pipeline (Stage CRM)

| Stage | Dias de Atraso | Sync | Tickets | Annual | Update |
|-------|----------------|------|---------|--------|--------|
| **1** | 1-60 dias | ✅ Sim | ✅ Sim | ➖ N/A | ✅ Verifica |
| **2** | 61-180 dias | ✅ Sim | ✅ Sim | ➖ N/A | ✅ Verifica |
| **3** | 181-365 dias | ✅ Sim | ✅ Sim | ➖ N/A | ✅ Verifica |
| **4** | 366+ dias | ✅ Sim | ✅ Sim | ➖ N/A | ✅ Verifica |

---

## 🔗 Relacionamento Entre Automações

### Fluxo de Complementaridade

``` mermaid
---
title: Relacionamento Entre Automações
---
graph LR
    DB[(Banco DNA)]
    CRM[RD Station CRM]
    
    DB -->|03h - Sync| CRM
    DB -->|03h - Tickets| CRM
    DB -->|03h - Annual| CRM
    CRM -->|22h - Update| DB
    
    style DB fill:#d4d4d4,stroke:#666
    style CRM fill:#e0e0e0,stroke:#666
```

### Ciclo Completo

1. **03h - Sync** 🔥
   - Identifica TODOS os inadimplentes no banco
   - Cria/atualiza deals no CRM
   - Organiza por tempo de atraso (4 stages)
   - **Resultado**: CRM atualizado com todas inadimplências

2. **03h - Tickets Renewal** 🎫 (Segundas)
   - Identifica carnês com últimos 4 meses de pagamento
   - Complementa Sync com foco em carnês
   - **Resultado**: Alertas específicos para carnês

3. **03h - Annual Renewal** 📅 (Dia 1º)
   - Identifica planos anuais a renovar
   - Ação proativa (antes do vencimento)
   - **Resultado**: Oportunidade de renovação antecipada

4. **22h - Update** 🔄
   - Busca deals no CRM
   - Verifica se clientes regularizaram
   - Remove do CRM (marca LOST)
   - **Resultado**: CRM limpo e atualizado

---

## 📊 Análise de Volume e Performance

### Volume de Processamento Mensal

| Automação | Execuções/Mês | Assinaturas/Execução | Total/Mês | Tempo/Execução |
|-----------|---------------|----------------------|-----------|----------------|
| **Sync** | 30 | 2.500 | 75.000 | ~8 min |
| **Tickets** | 4 | 300 | 1.200 | ~3 min |
| **Annual** | 1 | 80 | 80 | ~2 min |
| **Update** | 30 | 1.800 | 54.000 | ~6 min |
| **TOTAL** | - | - | **130.280** | - |

### Picos de Volume

| Período | Automação Afetada | Motivo | Ação |
|---------|-------------------|--------|------|
| Início do mês | Sync, Annual | Novos vencimentos + renovações | Monitorar timeout |
| Segundas-feiras | Sync, Tickets | Duas automações simultâneas | Verificar rate limit |
| Final do mês | Update | Fechamento mensal obrigatório | Validar 100% LOST |
| Feriados | Todas | Atraso acumulado de processamento | Alertas duplicados |

---

## 🎨 Estratégias de Segmentação

### Sync: Segmentação por Tempo

``` bash
1-60 dias    → Cobrança suave
61-180 dias  → Cobrança moderada
181-365 dias → Cobrança intensa
366+ dias    → Cobrança jurídica
```

### Tickets: Segmentação por Método

``` bash
Carnê + Últimos 4 meses → Alerta renovação ticket
Outras formas           → Ignorado
```

### Annual: Segmentação por Ciclo

``` bash
Mesmo mês ano anterior → Oportunidade renovação
Outros meses           → Ignorado
```

### Update: Segmentação por Status

``` bash
Sem faturas abertas    → Remover do CRM
Com faturas abertas    → Manter no funil
Último dia do mês      → Remover TODOS
```

---

## 🚨 Tratamento de Conflitos

### Cenário 1: Cliente com Múltiplos Critérios

**Situação:**

- Assinatura #12345
- Plano anual
- Paga via carnê
- 3 faturas abertas (90 dias de atraso)

**Processamento:**

| Automação | Avalia? | Resultado |
|-----------|---------|-----------|
| Sync | ✅ Sim | Cria/atualiza deal no stage 61-180 |
| Tickets | ✅ Sim | Cria/atualiza deal adicional para carnê |
| Annual | ❌ Não | Não está no mês de renovação |
| Update | ✅ Sim | Mantém deal OPEN (tem faturas) |

**Resultado Final:** 2 deals ativos (1 Sync + 1 Tickets)

### Cenário 2: Cliente Regulariza Durante o Dia

**Timeline:**

- 03h: Sync cria deal (3 faturas abertas)
- 10h: Cliente paga todas as faturas
- 22h: Update verifica e marca LOST

**Processamento:**

``` bash
03h  → Sync: CREATE deal #67890
10h  → Cliente paga → 0 faturas abertas
22h  → Update: LOST deal #67890
```

**Resultado Final:** Deal removido do CRM (correto)

### Cenário 3: Último Dia do Mês

**Situação:**

- 30/04/2024 (último dia)
- Sync rodou às 03h (criou 2.500 deals)
- Update roda às 22h

**Processamento:**

``` bash
03h  → Sync: 2.500 deals criados/atualizados
22h  → Update: TODOS os 2.500 deals → LOST
```

**Resultado Final:** CRM zerado (fechamento mensal)

---

## 🛠️ Configuração Consolidada

### Variáveis de Ambiente Comuns

```bash
# RD Station API
RD_STATION_TOKEN=token_compartilhado
RD_STAGE_1_60=pipeline_1_60_shared
RD_STAGE_61_180=pipeline_61_180_shared
RD_STAGE_181_365=pipeline_181_365_shared
RD_STAGE_366=pipeline_366_shared

# Pipelines Específicos
RD_ANNUAL_STAGE=pipeline_annual_separado

# AWS
AWS_REGION=sa-east-1
SQS_SYNC_QUEUE_URL=url_sync
SQS_TICKETS_QUEUE_URL=url_tickets
SQS_ANNUAL_QUEUE_URL=url_annual
SQS_UPDATE_QUEUE_URL=url_update

# Banco de Dados
DB_HOST=mysql_host
DB_USER=db_user
DB_PASSWORD=db_pass
DB_DATABASE=dna
```

### CloudWatch Cron Expressions

```yaml
sync:
  schedule: cron(0 3 * * ? *)      # Diário às 3h
  
ticketsRennewal:
  schedule: cron(0 3 ? * MON *)    # Segundas às 3h
  
annualRennewal:
  schedule: cron(0 3 1 * ? *)      # Dia 1º às 3h
  
update:
  schedule: cron(0 22 * * ? *)     # Diário às 22h
```

---

## 📈 Monitoramento Integrado

### Dashboards CloudWatch

#### Dashboard: Visão Geral Automações

| Métrica | Sync | Tickets | Annual | Update |
|---------|------|---------|--------|--------|
| **Execuções/dia** | 1 | 0.14 | 0.03 | 1 |
| **Duração média** | 8 min | 3 min | 2 min | 6 min |
| **Taxa sucesso** | 99.5% | 99.8% | 100% | 99.2% |
| **Deals/execução** | 2.500 | 300 | 80 | 1.800 |

#### Alertas Configurados

| Alerta | Condição | Ação |
|--------|----------|------|
| **Timeout Schedule** | Duração > 15 min | Email + Slack |
| **Fila parada** | Msgs > 1000 por 30 min | Email + PagerDuty |
| **Taxa erro alta** | Erros > 5% | Email + Slack |
| **Nenhum deal processado** | Deals = 0 | Email (exceto feriados) |

---

## 🎓 Best Practices

### 1. Ordem de Execução

✅ **Correto:**

``` bash
03h → Sync, Tickets, Annual (Banco → CRM)
22h → Update (CRM → Banco)
```

❌ **Incorreto:**

``` bash
03h → Update (limparia antes de popular)
22h → Sync (dados do dia anterior)
```

### 2. Coordenação de Updates

- Sync sempre roda ANTES do Update
- Update limpa apenas o que Sync criou
- Fechamento mensal (último dia) tem prioridade

### 3. Monitoramento Proativo

- Verificar logs após cada execução
- Comparar volumes com média histórica
- Alertas para desvios > 20%

### 4. Tratamento de Feriados

- Sync: executa normalmente (volume maior dia seguinte)
- Tickets: pode acumular para segunda seguinte
- Annual: OK (só dia 1º)
- Update: executa normalmente

---

## 🔍 Troubleshooting Integrado

### Problema: Volume de deals no CRM crescendo indefinidamente

**Diagnóstico:**

``` bash
1. Verificar execução do Update (22h)
2. Confirmar critérios de regularização
3. Validar query de faturas abertas
```

**Solução:**

- Update deve rodar diariamente
- No último dia do mês: LOST em 100% dos deals
- Início do mês: CRM deve estar vazio

### Problema: Deals duplicados no CRM

**Diagnóstico:**

``` bash
1. Verificar organization_id único
2. Confirmar sincronização Sync vs Tickets/Annual
3. Validar lógica CREATE vs UPDATE
```

**Solução:**

- Organization ID deve ser único por assinatura
- Sync: campo custom_fields.subscription
- Tickets/Annual: seguem mesma lógica

### Problema: Clientes reclamando de cobrança após pagamento

**Diagnóstico:**

``` bash
1. Verificar delay Sync (D+1)
2. Confirmar Update às 22h removeu deal
3. Validar status faturas no banco
```

**Solução:**

- Sync tem delay de 1 dia (normal)
- Update limpa no mesmo dia (22h)
- Máximo de exposição: 19h (03h → 22h)

---

## 📚 Documentação Individual

- [Sync - README](./sync/README.md) | [Guia Rápido](./sync/GUIA-RAPIDO.md)
- [Tickets Renewal - README](./ticketsRennewal/README.md) | [Guia Rápido](./ticketsRennewal/GUIA-RAPIDO.md)
- [Annual Renewal - README](./annual/README.md) | [Guia Rápido](./annual/GUIA-RAPIDO.md)
- [Update - README](./update/README.md) | [Guia Rápido](./update/GUIA-RAPIDO.md)

---

**Última atualização:** 2024
**Versão:** 1.0.0
