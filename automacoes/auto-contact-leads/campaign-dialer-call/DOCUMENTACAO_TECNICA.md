# Documentação Técnica - Campaign Dialer Call

## 📋 Visão Geral

A automação **Campaign Dialer Call** é responsável por realizar campanhas de discagem automática para leads do time de cobranças, organizados por tempo de inadimplência. O sistema busca negócios (deals) no CRM RD Station e envia os contatos para o sistema de discagem Escallo.

## 🎯 Objetivo

Automatizar o processo de contato telefônico com clientes inadimplentes, segmentados por período de atraso, garantindo que sejam realizadas tentativas de contato sistemáticas através do discador automático.

## 🔄 Fluxo de Execução

### 1. Trigger (Gatilho)

- **Tipo**: Scheduled (Cron Job)
- **Agendamento**: Varia conforme o pipeline
  - **Pipelines 1-60, 61-180, 181-365 dias**: `cron(0 12 1,3,5,7,9,11,13,15,17,19,21,23,25,27,29,31 * ? *)`
    - Dias ímpares do mês às 12h UTC
  - **Pipeline +366 dias**: `cron(0 12 3,7,14,21 * ? *)`
    - Dias 3, 7, 14 e 21 de cada mês às 12h UTC
- **Proteção**: Não executa aos domingos

### 2. Entrada de Dados

``` typescript
{
  deal_pipeline_id: string // ID do pipeline de negócios
}
```

### 3. Processamento

#### 3.1 Validações Iniciais

- Verifica se não é domingo
- Valida se o `deal_pipeline_id` foi fornecido
- Verifica se existe uma chave de campanha mapeada para o pipeline

#### 3.2 Busca de Negócios (GetAllDealsUseCase)

**Repositório**: `RdCrmChargeTeamRepository`

**Processo**:

1. Itera por todos os estágios (stages) do pipeline
2. Para cada estágio, realiza paginação (até 50 páginas)
3. Limita 200 deals por página
4. Coleta informações de contato:
   - Nome do contato
   - Telefone (normalizado com `0` no início)
5. Remove duplicatas usando Set de telefones
6. Aguarda 2 segundos entre requisições (rate limiting)

**Estágios por Pipeline**:

Cada pipeline possui 6 estágios:

- Novos
- [Sistema] Contato 1
- [Manual] Sem Conexão
- [Sistema] Contato 2
- [Manual] Sem Resposta
- [Sistema] Resgate

#### 3.3 Envio para Discador (DialerLeads)

**Repositório**: `EscalloDialerRepository`

**Processo**:

1. Formata os contatos no padrão Escallo
2. Envia payload com configurações:
   - `expiraLista`: 720 minutos (12 horas)
   - `cancelarPendentes`: true
   - `chaveExterna`: identificador da campanha
3. Cada contato inclui:
   - Telefone normalizado
   - Nome do cliente
   - Variáveis personalizadas (nome do paciente)

**Endpoint**: `POST /campanha/telefonia/confirmacaoAgenda`

## 🗂️ Estrutura de Pipelines

### Pipeline: 1-60 Dias

- **ID**: `66db5321b075c70026b57949`
- **Chave Campanha**: `1_60DIAS`
- **Descrição**: Clientes com 1 a 60 dias de atraso

### Pipeline: 61-180 Dias

- **ID**: `66db64909885e10023aee3d5`
- **Chave Campanha**: `61_180DIAS`
- **Descrição**: Clientes com 61 a 180 dias de atraso

### Pipeline: 181-365 Dias

- **ID**: `66db64b0f64a1f001a00d2e5`
- **Chave Campanha**: `181_365DIAS`
- **Descrição**: Clientes com 181 a 365 dias de atraso

### Pipeline: +366 Dias

- **ID**: `66db64bebdef0d0020f400a5`
- **Chave Campanha**: `+365DIAS`
- **Descrição**: Clientes com mais de 366 dias de atraso

## 🏗️ Arquitetura

### Camadas

#### Controllers

- `campaign-dialer-call.ts`: Handler principal da Lambda

#### Use Cases

- `GetAllDealsUseCase`: Busca negócios no CRM
- `DialerLeads`: Envia contatos para o discador

#### Repositories

- `RdCrmChargeTeamRepository`: Integração com RD Station CRM
- `EscalloDialerRepository`: Integração com Escallo Dialer

#### Factories

- `makeGetAllDealsByStageUseCase`: Cria instância do caso de uso de busca
- `makeDialerLeadsUseCase`: Cria instância do caso de uso de discagem

## 🔑 Variáveis de Ambiente

```bash
CRM_TOKEN_CHARGES_TEAM    # Token de autenticação RD Station CRM
TOKEN_ESCALLO             # Token de autenticação Escallo
NODE_ENV                  # Ambiente de execução
```

## 📊 Configuração AWS Lambda

- **Runtime**: Node.js 22.x
- **Timeout**: 900 segundos (15 minutos)
- **Memory**: 128 MB
- **Region**: us-east-1

## ⚠️ Tratamento de Erros

### Erros Possíveis

1. **LeadsNotFoundError**: Nenhum negócio encontrado no pipeline
2. **DialingError**: Falha ao enviar contatos para o discador
3. **Erro de Rede**: Falhas na comunicação com APIs externas

### Comportamento em Caso de Erro

- Logs detalhados são registrados no CloudWatch
- Erros não bloqueiam completamente a execução
- Paginação interrompida em caso de falha na página

## 🔍 Logs e Monitoramento

### Logs Gerados

- Evento da campanha recebido
- Validação de domingo
- Pipeline ID e chave da campanha
- Quantidade de leads encontrados
- Respostas das APIs externas

### CloudWatch

Todos os logs são enviados automaticamente para o CloudWatch Logs da AWS.

## 🚀 Melhorias Futuras

- [ ] Adicionar retry automático em caso de falha
- [ ] Implementar métricas de sucesso/falha
- [ ] Adicionar notificações em caso de erro
- [ ] Otimizar paginação com processamento paralelo
- [ ] Implementar cache para reduzir chamadas à API

## 📝 Observações Importantes

1. O sistema remove automaticamente telefones duplicados
2. A normalização de telefone adiciona `0` no início
3. Há um delay de 2 segundos entre requisições para respeitar rate limits
4. A lista de contatos expira após 12 horas no Escallo
5. Contatos pendentes são cancelados a cada nova execução

## 🔗 Dependências Principais

- **axios**: Cliente HTTP para requisições
- **RD Station CRM API**: Fonte de dados de negócios
- **Escallo API**: Sistema de discagem automática

## 📚 Referências

- [Documentação RD Station CRM](https://developers.rdstation.com/pt-BR/crm)
- [Documentação Escallo](https://escallo.com.br)
- [AWS Lambda](https://docs.aws.amazon.com/lambda/)
- [Serverless Framework](https://www.serverless.com/framework/docs)
