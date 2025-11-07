# Automações RC - Resumo Executivo

## Visão Geral

Este documento apresenta um resumo simples e direto sobre o **motivo** e **objetivo** de cada automação implementada no sistema.

---

## 📋 Índice de Automações

1. [Consultas](#-consultas)
2. [Convênios](#-convênios)
3. [Exames](#-exames)
4. [Propostas](#-propostas)
5. [Propostas RC](#-propostas-rc)

---

## 📅 Consultas

### Motivo

Recuperar leads de pacientes que **agendaram consulta mas ainda não foram atendidos**.

### Objetivo

Criar oportunidades comerciais no RD Station para a equipe de relacionamento entrar em contato e:

- Confirmar presença na consulta
- Oferecer procedimentos complementares
- Realizar upsell de serviços

### Fonte de Dados

Banco de dados MySQL - tabela `site_leads` (agendamentos do site)

### Quando Executa

1x por dia (04:00 UTC) - busca agendamentos do dia anterior

### Diferencial

Foco em pacientes que **já demonstraram interesse** ao agendar, aumentando taxa de conversão.

---

## 🏥 Convênios

### Motivo

Identificar em **tempo real** pacientes que foram atendidos com **convênio médico**.

### Objetivo

Criar oportunidades para a equipe comercial oferecer:

- Planos de saúde próprios
- Migração para plano particular
- Serviços adicionais não cobertos pelo convênio

### Fonte de Dados

API Feegow - atendimentos realizados

### Quando Executa

A cada **15 minutos** (96x por dia) - monitoramento contínuo

### Diferencial

Única automação com **alta frequência** para captura imediata de oportunidades quentes.

---

## 🔬 Exames

### Motivo

Recuperar leads de pacientes que **solicitaram exames mas não agendaram**.

### Objetivo

Criar oportunidades para a equipe de relacionamento:

- Ajudar no agendamento dos exames
- Esclarecer dúvidas sobre procedimentos
- Oferecer pacotes de exames

### Fonte de Dados

Banco de dados MySQL - tabela `patient` (exames solicitados)

### Quando Executa

1x por dia (04:00 UTC) - busca solicitações do dia anterior

### Diferencial

Foco em pacientes que **têm necessidade identificada** mas não concluíram o processo.

---

## 💼 Propostas

### Motivo

Processar propostas comerciais de **procedimentos diversos** vindas da clínica.

### Objetivo

Criar deals estruturados no RD Station com:

- Distribuição equilibrada entre equipe (5 atendentes)
- Informações completas do paciente
- Lista de procedimentos orçados
- Valores para análise comercial

### Fonte de Dados

API Feegow - endpoint `proposals()`

### Quando Executa

1x por dia (04:00 UTC) - busca propostas do dia anterior

### Diferencial

**Validação de qualidade**: apenas propostas acima de R$ 65 e com telefone válido.

---

## 🎯 Propostas RC

### Motivo

Processar propostas comerciais **específicas da unidade RC** com regras diferenciadas.

### Objetivo

Criar deals no RD Station com **filtro inteligente** que:

- Remove procedimentos já agendados
- Evita duplicação de esforço comercial
- Foca apenas no que o paciente ainda precisa agendar

### Fonte de Dados

API Feegow - endpoint `proposalsRc()` (específico RC)

### Quando Executa

1x por dia (04:00 UTC) - busca propostas RC do dia anterior

### Diferencial

**Único com filtro de procedimentos**: elimina propostas onde todos os itens já foram agendados, otimizando o trabalho da equipe.

---

## 📊 Comparação Rápida

| Automação | Frequência | Fonte | Objetivo Principal |
|-----------|------------|-------|-------------------|
| **Consultas** | 1x/dia | MySQL | Recuperar pacientes agendados |
| **Convênios** | 96x/dia | API Feegow | Capturar atendimentos com convênio |
| **Exames** | 1x/dia | MySQL | Recuperar solicitações não agendadas |
| **Propostas** | 1x/dia | API Feegow | Processar orçamentos gerais |
| **Propostas RC** | 1x/dia | API Feegow | Processar orçamentos RC filtrados |

---

## 🎯 Impacto Geral

### Para o Negócio

- ✅ **Aumento de conversão** através de follow-up ativo
- ✅ **Redução de no-show** em consultas e exames
- ✅ **Identificação de oportunidades** de upsell
- ✅ **Otimização do tempo** da equipe comercial

### Para a Equipe

- ✅ Leads qualificados e organizados no CRM
- ✅ Informações completas para abordagem
- ✅ Distribuição equilibrada de trabalho
- ✅ Foco em oportunidades reais (filtros inteligentes)

### Para o Paciente

- ✅ Acompanhamento proativo do atendimento
- ✅ Lembretes e confirmações
- ✅ Esclarecimento de dúvidas
- ✅ Facilidade para agendar procedimentos

---

## 📁 Documentação Detalhada

Para informações técnicas completas, arquitetura e fluxos visuais, consulte as pastas:

- [`/docs/consultas/`](./consultas/) - Documentação técnica + fluxo visual
- [`/docs/convenios/`](./convenios/) - Documentação técnica + fluxo visual
- [`/docs/exam/`](./exam/) - Documentação técnica + fluxo visual
- [`/docs/proposals/`](./proposals/) - Documentação técnica + fluxo visual
- [`/docs/proposals-rc/`](./proposals-rc/) - Documentação técnica + fluxo visual

---

## 🔗 Links Úteis

- **Repositório:** [Segmedic/automacoes-rc](https://github.com/Segmedic/automacoes-rc)
- **RD Station:** Plataforma de CRM para gestão de leads
- **Feegow:** Sistema de gestão clínica e agendamentos

---

**Última atualização:** Novembro 2025
