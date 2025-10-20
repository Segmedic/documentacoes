# Documentação - Automações Comerciais

## 📋 Visão Geral

Sistema de automações para captura e qualificação de leads comerciais através de múltiplas fontes de dados. As automações processam diferentes tipos de eventos e enviam leads qualificados para o CRM via AWS SQS.

## 🎯 Arquitetura Geral

``` mermaid
flowchart TB
    subgraph Fontes["Fontes de Dados"]
        Feegow[(Feegow - Sistema Médico)]
        Escallo[(Escallo - Call Center)]
        ClubFlex[(ClubFlex - Clube)]
        Medula[(Medula - Data Warehouse)]
    end
    
    subgraph Automacoes["Automações"]
        A1[Leads Amanhã]
        A2[Leads Convênios]
        A3[Recuperação Agendamento]
        A4[Retenção B2B]
        A5[Escallo Ligação]
        A6[Escallo Chat]
    end
    
    subgraph Processamento["Processamento"]
        Queue[(AWS SQS)]
        CRM[CRM RD Station]
    end
    
    Feegow --> A1
    Feegow --> A3
    Medula --> A2
    Medula --> A4
    ClubFlex --> A4
    Escallo --> A5
    Escallo --> A6
    
    A1 --> Queue
    A2 --> Queue
    A3 --> Queue
    A4 --> Queue
    A5 --> Queue
    A6 --> Queue
    
    Queue --> CRM
    
    style A1 fill:#e1e4e8
    style A2 fill:#e1e4e8
    style A3 fill:#e1e4e8
    style A4 fill:#e1e4e8
    style A5 fill:#e1e4e8
    style A6 fill:#e1e4e8
    style Queue fill:#d1d5da
    style CRM fill:#d1d5da
```

## 📊 Automações Disponíveis

### 1️⃣ [Leads de Amanhã](./leads_amanha/)

**Objetivo:** Capturar agendamentos do dia seguinte sem convênio

- **Período:** D+1 (próximas 24h)
- **Fonte:** Feegow
- **Validações:** CRM + ClubFlex + Tabelas válidas
- **Origem:** `leads_amanha`

📄 [Documentação Técnica](./leads_amanha/documentacao-tecnica.md) | 📊 [Fluxo Visual](./leads_amanha/fluxo-visual.md)

---

### 2️⃣ [Leads de Convênios](./leads_convenios/)

**Objetivo:** Capturar atendimentos com convênio para ações de upsell

- **Período:** D-1 (ontem)
- **Fonte:** Medula (Data Warehouse)
- **Validações:** Convênio específico + Atendimento realizado
- **Origem:** `leads_convenios`

📄 [Documentação Técnica](./leads_convenios/documentacao-tecnica.md) | 📊 [Fluxo Visual](./leads_convenios/fluxo-visual.md)

---

### 3️⃣ [Recuperação de Agendamento](./recuperacao_agendamento/)

**Objetivo:** Recuperar carrinhos abandonados de agendamento

- **Período:** D-1 até D+15
- **Fonte:** Feegow + Banco Interno
- **Validações:** Especialidade + CRM + ClubFlex
- **Origem:** `recuperacao_agendamento`

📄 [Documentação Técnica](./recuperacao_agendamento/documentacao-tecnica.md) | 📊 [Fluxo Visual](./recuperacao_agendamento/fluxo-visual.md)

---

### 4️⃣ [Retenção de Ex-Colaboradores B2B](./retencao_b2b/)

**Objetivo:** Reconquistar ex-dependentes de planos empresariais

- **Período:** Último mês
- **Fonte:** ClubFlex + Medula
- **Validações:** Status REMOVED + Enriquecimento de dados
- **Origem:** `retencao_excolaboradores_b2b`

📄 [Documentação Técnica](./retencao_b2b/documentacao-tecnica.md) | 📊 [Fluxo Visual](./retencao_b2b/fluxo-visual.md)

---

### 5️⃣ [Escallo Ligação](./escallo_ligacao/)

**Objetivo:** Capturar ligações telefônicas de entrada com agentes comerciais

- **Período:** Hoje
- **Fonte:** Escallo (Reports 086 + 002)
- **Validações:** Agente comercial OU Fila Ação Record + Áudio
- **Origens:** `escallo_ligacao` / `escallo_ligacao_record`

📄 [Documentação Técnica](./escallo_ligacao/documentacao-tecnica.md) | 📊 [Fluxo Visual](./escallo_ligacao/fluxo-visual.md)

---

### 6️⃣ [Escallo Chat](./escallo_chat/)

**Objetivo:** Capturar atendimentos por chat/WhatsApp/redes sociais

- **Período:** Hoje
- **Fonte:** Escallo (Report 087)
- **Validações:** Apenas agentes comerciais
- **Origem:** `escallo_chat`

📄 [Documentação Técnica](./escallo_chat/documentacao-tecnica.md) | 📊 [Fluxo Visual](./escallo_chat/fluxo-visual.md)

---

## 📊 Tabela Comparativa

| Automação | Período | Fonte Principal | Complexidade | Validações | Enriquecimento |
|-----------|---------|-----------------|--------------|------------|----------------|
| **Leads Amanhã** | D+1 | Feegow | ⭐⭐⭐ | CRM + ClubFlex | Não |
| **Leads Convênios** | D-1 | Medula | ⭐⭐ | Convênio | Não |
| **Recuperação Agd** | D-1 a D+15 | Feegow + BD | ⭐⭐⭐⭐ | Especialidade + CRM + ClubFlex | Não |
| **Retenção B2B** | Último mês | ClubFlex + Medula | ⭐⭐⭐⭐⭐ | Status + Histórico | Sim (2 sistemas) |
| **Escallo Ligação** | Hoje | Escallo | ⭐⭐⭐ | Agente/Fila + Direção | Sim (áudio) |
| **Escallo Chat** | Hoje | Escallo | ⭐⭐ | Agente comercial | Não |

## 🔄 Fluxo de Dados Geral

``` mermaid
sequenceDiagram
    participant Fonte as Fonte de Dados
    participant Auto as Automação
    participant Valid as Validações
    participant SQS as AWS SQS
    participant CRM as CRM
    
    Fonte->>Auto: 1. Buscar dados do período
    Auto->>Auto: 2. Processar registros
    Auto->>Valid: 3. Aplicar filtros
    Valid-->>Auto: 4. Leads qualificados
    Auto->>Auto: 5. Deduplicação
    Auto->>SQS: 6. Enviar LeadEvent
    SQS->>CRM: 7. Processar no CRM
```

## 🎯 Tipos de Lead (origin)

``` mermaid
mindmap
  root((Lead Origins))
    Agendamentos
      leads_amanha
      recuperacao_agendamento
    Atendimentos
      leads_convenios
    Retenção
      retencao_excolaboradores_b2b
    Escallo Ligação
      escallo_ligacao
      escallo_ligacao_record
    Escallo Chat
      escallo_chat
```

## 📦 Estrutura de Dados Unificada

Todas as automações enviam para a fila no formato:

```typescript
{
  origin: string,  // Identificador da automação
  payload: any     // Dados específicos de cada tipo
}
```

### Origens Disponíveis

- `leads_amanha`
- `leads_convenios`
- `recuperacao_agendamento`
- `retencao_excolaboradores_b2b`
- `escallo_ligacao`
- `escallo_ligacao_record`
- `escallo_chat`

## 🔍 Validações Comuns

### 1. Verificação CRM (Medula)

Utilizada em: Leads Amanhã, Recuperação Agendamento

- Verifica se telefone já possui oportunidade ativa
- Evita duplicação de leads no CRM

### 2. Verificação ClubFlex

Utilizada em: Leads Amanhã, Recuperação Agendamento, Retenção B2B

- Verifica se CPF é membro ClubFlex
- Filtra clientes que já possuem benefício

### 3. Agentes Comerciais

Utilizada em: Escallo Ligação, Escallo Chat

- 20 agentes mapeados em `AGENTES_ESCALLO_RD`
- Garante qualidade do lead comercial

## ⚙️ Configuração e Execução

### Variáveis de Ambiente

``` bash
# AWS SQS
AWS_SQS_QUEUE_URL=<url-da-fila>

# Feegow
FEEGOW_API_KEY=<api-key>

# Escallo
ESCALLO_API_KEY=<api-key>

# Medula (PostgreSQL)
MEDULA_CONNECTION_STRING=<connection-string>

# ClubFlex (MySQL)
CLUBFLEX_CONNECTION_STRING=<connection-string>
```

### Estrutura de Pastas

``` folder
src/
├── schedules/              # Handlers das automações
│   ├── leads_amanha.ts
│   ├── leads_convenios.ts
│   ├── recuperacao_agendamento.ts
│   ├── retencao_excolaboradores_b2b.ts
│   ├── escallo_ligacao.ts
│   └── escallo_chat.ts
├── aws/                    # Integração AWS SQS
├── feegow/                 # Cliente Feegow
├── escallo/                # Cliente Escallo
├── medula/                 # Repositório Medula
├── clubflex/               # Repositório ClubFlex
└── utils/                  # Utilitários
```

## 📈 Métricas Recomendadas

### Por Automação

- Total de leads enviados
- Total de leads filtrados
- Taxa de conversão (enviados/filtrados)
- Tempo de execução

### Por Fonte

- Feegow: Volume de agendamentos
- Escallo: Volume de atendimentos (ligação + chat)
- ClubFlex: Taxa de remoção
- Medula: Leads com histórico

### Por Validação

- Leads bloqueados por CRM
- Leads bloqueados por ClubFlex
- Leads sem dados válidos

## 🚨 Tratamento de Erros

Cada automação possui seu próprio tratamento:

- ✅ **Com try/catch:** Retenção B2B
- ⚠️ **Sem try/catch:** Demais automações (propagam erros)

**Recomendação:** Implementar try/catch e logging estruturado em todas.

## 🔐 Segurança

- Connection strings não devem estar hardcoded
- Usar variáveis de ambiente ou AWS Secrets Manager
- SSL configurado para conexões de banco

## 🎯 Próximos Passos

### Melhorias Sugeridas

1. **Logging Estruturado**
   - Implementar Winston ou similar
   - Logs com contexto e rastreabilidade

2. **Métricas e Monitoramento**
   - CloudWatch Metrics
   - Alertas para falhas
   - Dashboard de volume

3. **Retry Logic**
   - Implementar retry para falhas temporárias
   - Dead Letter Queue (DLQ)

4. **Testes**
   - Testes unitários
   - Testes de integração
   - Testes E2E

5. **Documentação**
   - ✅ Documentação técnica (completa)
   - ✅ Fluxos visuais (completo)
   - ⏳ Runbooks operacionais
   - ⏳ Guia de troubleshooting

## 📚 Recursos Adicionais

### Sistemas Externos

- [Feegow](https://www.feegow.com/) - Sistema de gestão médica
- [Escallo](https://escallo.com.br/) - Call center e atendimento
- [RD Station](https://www.rdstation.com/) - CRM e automação de marketing

### Documentação Relacionada

- [AWS SQS](https://aws.amazon.com/sqs/)
- [Serverless Framework](https://www.serverless.com/)
- [TypeScript](https://www.typescriptlang.org/)

## 🤝 Contribuindo

Para adicionar uma nova automação:

1. Criar handler em `src/schedules/`
2. Implementar lógica de filtros
3. Enviar para SQS com `origin` única
4. Criar documentação em `docs/<nome>/`
5. Atualizar este README

## 📞 Suporte

Para dúvidas ou problemas:

- Time de Desenvolvimento
- Time Comercial
- Documentação técnica específica de cada automação

---

**Última atualização:** Outubro 2025  
**Versão:** 1.0  
**Automações:** 6 ativas
