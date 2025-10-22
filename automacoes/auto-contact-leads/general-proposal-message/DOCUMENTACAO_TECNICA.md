# Documentação Técnica - General Proposal Message

## 📋 Visão Geral

A automação **General Proposal Message** é responsável por enviar mensagens via WhatsApp para leads do pipeline de Propostas Gerais. O sistema busca negócios criados no dia atual no CRM RD Station e envia mensagens automáticas através da plataforma Escallo.

## 🎯 Objetivo

Automatizar o processo de comunicação via WhatsApp com clientes que entraram no pipeline de propostas gerais, garantindo contato imediato no mesmo dia da criação do negócio. Este pipeline atende demandas diversas que não se enquadram especificamente em propostas laboratoriais.

## 🔄 Fluxo de Execução

### 1. Trigger (Gatilho)

- **Tipo**: Scheduled (Cron Job)
- **Agendamento**: `cron(0 12 * * ? *)`
  - Executa diariamente às 12h UTC (09h BRT)
- **Frequência**: Todos os dias da semana

### 2. Entrada de Dados

```typescript
{
  deal_pipeline_id: string // ID do pipeline de propostas gerais
}
```

**Pipeline Configurado**:

- **ID**: `647e4cdab66552000db15fd4`
- **Chave Campanha**: `3b685d99-e284-45fd-b739-1d3207fb5c8e`

### 3. Processamento

#### 3.1 Validações Iniciais

- Valida se o `deal_pipeline_id` foi fornecido
- Verifica se existe uma chave de campanha mapeada para o pipeline
- Registra erro em caso de pipeline inválido

#### 3.2 Busca de Negócios (GetAllDealsUseCase)

**Repositório**: `RdCrmRcTeamRepository`

**Processo**:

1. Busca negócios criados **no dia atual**
2. Realiza paginação (até 50 páginas)
3. Limita 200 deals por página
4. Coleta informações de contato:
   - Nome do contato
   - Telefone (normalizado com `0` no início)
5. Remove duplicatas usando Set de telefones
6. Aguarda 2 segundos entre requisições (rate limiting)

**Filtros Aplicados**:

- `created_at_period`: true
- `start_date`: Data atual (YYYY-MM-DD)
- `end_date`: Data atual (YYYY-MM-DD)
- `win`: null (apenas negócios não ganhos)

#### 3.3 Envio de Mensagens (SendMessageWhatsapp)

**Repositório**: `EscalloWhatsappRepository`

**Processo**:

1. Valida se existem contatos para enviar
2. Formata os contatos no padrão Escallo
3. Envia payload com configurações:
   - `expiraLista`: 60 minutos
   - `cancelarPendentes`: false
   - `chaveExterna`: identificador da campanha
4. Cada contato inclui:
   - Telefone normalizado
   - Nome do cliente

**Endpoint**: `POST /campanha/texto/lista`

## 🗂️ Estrutura do Pipeline

### Pipeline: Propostas Gerais

- **ID**: `647e4cdab66552000db15fd4`
- **Chave Campanha**: `3b685d99-e284-45fd-b739-1d3207fb5c8e`
- **Descrição**: Pipeline de propostas gerais e diversas
- **Execução**: Diária às 12h UTC
- **Público-alvo**: Clientes com demandas variadas de propostas

## 🏗️ Arquitetura

### Camadas

#### Controllers

- `campaign-send-message.ts`: Handler principal da Lambda (compartilhado com laboratory-proposal-message)

#### Use Cases

- `GetAllDealsUseCase`: Busca negócios no CRM
- `SendMessageWhatsapp`: Envia mensagens via WhatsApp

#### Repositories

- `RdCrmRcTeamRepository`: Integração com RD Station CRM (time RC)
- `EscalloWhatsappRepository`: Integração com Escallo WhatsApp

#### Factories

- `makeGetAllDealsByStageUseCase`: Cria instância do caso de uso de busca
- `makeSendMessageWhatsappUseCase`: Cria instância do caso de uso de envio

## 🔑 Variáveis de Ambiente

```bash
CRM_TOKEN_RC_TEAM                # Token de autenticação RD Station CRM (RC Team)
TOKEN_ESCALLO                    # Token de autenticação Escallo
KEY_CAMPAIGN_PROPOSTAS_GERAIS    # Chave da campanha de propostas gerais
NODE_ENV                         # Ambiente de execução
```

## 📊 Configuração AWS Lambda

- **Runtime**: Node.js 22.x
- **Timeout**: 900 segundos (15 minutos)
- **Memory**: 128 MB
- **Region**: us-east-1
- **Stage**: prod

## ⚠️ Tratamento de Erros

### Erros Possíveis

1. **LeadsNotFoundError**: Nenhum negócio encontrado no pipeline
2. **SendMessageError**: Falha ao enviar mensagens via WhatsApp
3. **Erro de Rede**: Falhas na comunicação com APIs externas
4. **Pipeline Inválido**: ID de pipeline não encontrado ou sem chave mapeada

### Comportamento em Caso de Erro

- Logs detalhados são registrados no CloudWatch
- Execução encerrada graciosamente quando não há leads
- SendMessageError lançado quando lista de contatos está vazia
- Paginação interrompida em caso de falha na página

## 🔍 Logs e Monitoramento

### Logs Gerados

- Evento da campanha recebido
- Pipeline ID e chave da campanha
- Quantidade de leads encontrados
- Respostas das APIs externas (RD Station e Escallo)
- Erros de validação e processamento

### CloudWatch

Todos os logs são enviados automaticamente para o CloudWatch Logs da AWS.

## 📈 Características Especiais

### Código Compartilhado

Esta automação utiliza o **mesmo handler** (`campaign-send-message.ts`) que a automação `laboratory-proposal-message`, diferenciando-se apenas pelo:

1. **Pipeline ID diferente**
2. **Chave de campanha diferente**
3. **Público-alvo distinto**

### Diferenças em Relação ao Campaign Dialer Call

1. **Filtro Temporal**: Busca apenas negócios criados no dia atual
2. **Sem Iteração de Stages**: Não itera por estágios do pipeline
3. **Canal de Comunicação**: WhatsApp em vez de telefone
4. **Expiração Mais Curta**: Lista expira em 60 minutos vs 720 minutos
5. **Não Cancela Pendentes**: `cancelarPendentes: false`
6. **Token Diferente**: Usa `CRM_TOKEN_RC_TEAM` em vez de `CRM_TOKEN_CHARGES_TEAM`

### Diferenças em Relação ao Laboratory Proposal Message

| Aspecto | General Proposal | Laboratory Proposal |
|---------|------------------|---------------------|
| **Pipeline ID** | `647e4cdab66552000db15fd4` | `65391eec1e66020013a4a869` |
| **Chave Campanha** | `3b685d99-e284-45fd-b739-1d3207fb5c8e` | `b6f7016d-dfe1-4acf-a4ba-853bf900c264` |
| **Tipo de Proposta** | Gerais/Diversas | Laboratoriais |
| **Handler** | `campaign-send-message.handler` | `campaign-send-message.handler` |
| **Frequência** | Diária às 12h UTC | Diária às 12h UTC |

### Vantagens da Abordagem

- ✅ Contato imediato com leads recém-criados
- ✅ Comunicação via WhatsApp (maior taxa de resposta)
- ✅ Processamento diário automático
- ✅ Remoção de duplicatas por telefone
- ✅ Código reutilizável entre diferentes pipelines
- ✅ Configuração simples via variáveis de ambiente

## 🚀 Melhorias Futuras

- [ ] Adicionar retry automático em caso de falha
- [ ] Implementar métricas de taxa de entrega por pipeline
- [ ] Adicionar templates de mensagem personalizados por tipo de proposta
- [ ] Implementar A/B testing de mensagens
- [ ] Adicionar notificações em caso de erro
- [ ] Adicionar validação de números de WhatsApp
- [ ] Otimizar paginação com processamento paralelo
- [ ] Implementar cache para reduzir chamadas à API
- [ ] Adicionar dashboard de monitoramento em tempo real
- [ ] Implementar rate limiting adaptativo

## 📝 Observações Importantes

1. O sistema busca apenas negócios criados na data de execução
2. Remove automaticamente telefones duplicados
3. A normalização de telefone adiciona `0` no início
4. Há um delay de 2 segundos entre requisições para respeitar rate limits
5. A lista de contatos expira após 60 minutos no Escallo
6. Contatos pendentes **não são cancelados** a cada nova execução
7. Usa o token do time RC (Relacionamento com Cliente)
8. Compartilha a mesma implementação com laboratory-proposal-message
9. Diferenciação ocorre apenas na configuração do pipeline

## 🔗 Dependências Principais

- **axios**: Cliente HTTP para requisições
- **RD Station CRM API**: Fonte de dados de negócios
- **Escallo API**: Sistema de envio de mensagens WhatsApp

## 📚 Referências

- [Documentação RD Station CRM](https://developers.rdstation.com/pt-BR/crm)
- [Documentação Escallo](https://escallo.com.br)
- [AWS Lambda](https://docs.aws.amazon.com/lambda/)
- [Serverless Framework](https://www.serverless.com/framework/docs)

## 🔄 Fluxo de Dados Resumido

```text
AWS Lambda (Cron Diário)
    ↓
RD Station CRM (Busca deals do dia)
    ↓
Normalização e Deduplicação
    ↓
Escallo WhatsApp API (Envia mensagens)
    ↓
Cliente recebe mensagem no WhatsApp
```

## 🎯 Casos de Uso

Esta automação é ideal para:

- 📋 Propostas comerciais gerais
- 💼 Negociações diversas
- 🤝 Oportunidades variadas
- 📞 Primeiro contato rápido via WhatsApp
- ⚡ Resposta imediata a novos leads

## 🔐 Segurança

- Tokens armazenados como variáveis de ambiente
- Comunicação via HTTPS com APIs externas
- Logs não expõem informações sensíveis
- Rate limiting para prevenir abuso
- Validação de entrada de dados
