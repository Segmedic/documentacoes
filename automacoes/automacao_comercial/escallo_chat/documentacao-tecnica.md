# Documentação Técnica - Escallo Chat

## 📋 Visão Geral

A automação de **Escallo Chat** tem como objetivo capturar conversas por chat atendidas por agentes comerciais no sistema Escallo, gerando leads qualificados para o CRM. Diferente da automação de ligação, o chat trabalha apenas com agentes comerciais, sem distinção de filas para origem do lead.

## 🎯 Objetivo

Capturar conversas de chat que:

- Foram atendidas por **agentes comerciais** cadastrados
- Possuem dados válidos de cliente (nome e contato)
- Possuem agente atribuído
- Não são duplicadas (mesmo número de contato)

## 🔧 Tecnologias e Dependências

### Serviços Externos

- **Escallo**: Sistema de atendimento multicanal (chat, WhatsApp, redes sociais)
- **AWS SQS**: Fila para envio de leads

### APIs Escallo

- **Report 087**: Relatório de atendimentos por chat/mensagens

### Constantes

- `AGENTES_ESCALLO_RD`: Mapeamento de IDs de agentes comerciais (mesma lista da ligação)
- `FILAS`: Filas do Chat Ação Record (informativo, não usado no código)

## 📊 Fluxo de Execução

### 1. Inicialização

```typescript
handler()
```

- Cria cliente Escallo
- Define data de hoje para busca

### 2. Busca de Dados

Busca relatório 087 do dia atual:

- **Report 087**: Atendimentos via chat/mensagens

### 3. Processamento de Registros

Para cada registro de chat:

#### 3.1. Verificação de Agente

Verifica se o agente está na lista de **agentes comerciais** cadastrados no `AGENTES_ESCALLO_RD`.

**Nota:** Diferente da ligação, aqui NÃO há verificação de fila. Apenas agentes comerciais são aceitos.

#### 3.2. Validações de Continuação

Continua para próximo registro se:

- ❌ Número de contato já foi processado (duplicata)
- ❌ Não possui ID de agente
- ❌ Não é agente comercial

#### 3.3. Validação de Lead

Chama função `isLead()` que valida:

- ✅ Possui nome do agente (não vazio)
- ✅ Possui nome do cliente (não vazio)
- ✅ Possui valor do contato (telefone/email não vazio)

#### 3.4. Origem Única

```typescript
origin = "escallo_chat"
```

**Importante:** Diferente da ligação, há apenas UMA origem. As filas do Chat Ação Record existem mas não alteram a origem do lead.

### 4. Criação e Envio do Lead

- Cria `LeadEvent` com origem "escallo_chat"
- Payload é o registro completo (Registro87)
- Envia para fila SQS
- Adiciona número à lista de processados (evita duplicatas)

## 📦 Estrutura de Dados

### Registro87 (Escallo)

```typescript
{
  id: number,                           // ID do atendimento
  dataHoraInicial: string,              // Data/hora início
  dataHoraFinal: string,                // Data/hora fim
  clienteContato: {                     // Dados do cliente
    id: string,
    nome: string,                       // Nome do cliente
    tipo: string,                       // Tipo de contato
    valor: string                       // Telefone/email/WhatsApp
  },
  midiaSocial: {                        // Canal de atendimento
    // WhatsApp, Facebook, Instagram, etc
  },
  protocolo: string,                    // Número do protocolo
  direcao: string,                      // Entrada ou Saída
  direcaoFormatado: string,            
  situacao: string,                     // Status da conversa
  situacaoFormatado: string,
  status: string,
  statusFormatado: string,
  agente: {                             // Dados do agente
    id: string,
    nome: string | null,
    codigo: string | null
  },
  motivoInicial: {                      // Motivo inicial
    // Dados do motivo
  },
  motivo: {                             // Motivo detalhado
    // Dados do motivo
  },
  classificacao: {                      // Classificação
    // Dados da classificação
  },
  duracaoContato: string,               // Duração do atendimento
  qtdeMensagens: number,                // Quantidade de mensagens
  atendimentos: {                       // Informações de atendimento
    // Dados do atendimento
  }
}
```

### LeadEvent

```typescript
{
  origin: "escallo_chat",
  payload: Registro87
}
```

## 🔍 Critérios de Filtragem

### Agentes Comerciais

Os mesmos 20 agentes da automação de ligação:

- 334, 261, 412, 409, 321 (Gleice Carvalho)
- 339 (Pedro Santos), 352 (Caroline Saravel)
- 456 (Keismi Galvão), 459 (Karolyn Nascimento)
- 431 (Daniel Evangelista), 345 (Patrick Araujo)
- 446 (Carina Oliveira), 519 (Rafaela Lima)
- 525 (Livia Bastos), 555 (Ana Beatriz)
- 552 (Thuane Luize), 546 (Luiz Felipe)
- 314 (André Rodrigues), 573 (Maria Clara)
- 585 (Samara Oliveira)

### Validação de Lead

```typescript
function isLead(registro87) {
  return !(
    agente.nome == "" ||
    clienteContato.nome == "" ||
    clienteContato.valor == ""
  )
}
```

**Requisitos:**

- ✅ Deve ter nome do agente
- ✅ Deve ter nome do cliente
- ✅ Deve ter valor de contato (telefone/email)

### Deduplicação

- Array `numbers` mantém valores de contato já processados
- Mesmo contato não gera múltiplos leads
- Usa `clienteContato.valor` como chave única

## ⚙️ Configurações

### Período de Busca

- **Data**: Hoje (data atual)
- **Relatório**: 087 (atendimentos por chat)

### Origem de Lead

- **escallo_chat**: Única origem para todos os chats

## 🎯 Diferenças vs Escallo Ligação

| Característica | Escallo Chat | Escallo Ligação |
|----------------|--------------|-----------------|
| Report | 087 | 086 + 002 |
| Áudio | ❌ Não | ✅ Sim |
| Filas afetam origem | ❌ Não | ✅ Sim (Record) |
| Agentes | Apenas comerciais | Comerciais OU Fila Record |
| Origem | 1 (escallo_chat) | 2 (normal/record) |
| Direção validada | ❌ Não | ✅ Sim (só Entrada) |
| Canal | Chat/WhatsApp/Social | Telefone |
| Deduplicação | clienteContato.valor | origem |

## 🚨 Tratamento de Erros

- Não possui try/catch explícito
- Erros serão propagados para o runtime
- Não há garantia de destruição de conexões

## 📝 Observações Importantes

1. **Apenas Agentes Comerciais**: Diferente da ligação, não aceita outros agentes mesmo com fila específica
2. **Filas Declaradas mas Não Usadas**: Constante `FILAS` existe mas não influencia o código
3. **Uma Única Origem**: Não diferencia Chat Ação Record de chats normais
4. **Deduplicação por Contato**: Usa `clienteContato.valor` (pode ser telefone, email, WhatsApp)
5. **Sem Validação de Direção**: Aceita tanto entrada quanto saída
6. **Multicanal**: Pode incluir WhatsApp, Facebook, Instagram, etc.
7. **Quantidade de Mensagens**: Payload inclui `qtdeMensagens`
8. **Duração**: Payload inclui `duracaoContato`

## 🔄 Dependências de Outros Módulos

- `src/escallo/client.ts`: Cliente da API Escallo
- `src/aws/sqs.ts`: Envio para fila
- `src/rd/constants.ts`: Mapeamento de agentes

## 🎯 Comparação com Todas as Automações

| Característica | Escallo Chat | Escallo Ligação | Leads Amanhã | Leads Convênios | Recuperação Agd | Retenção B2B |
|----------------|--------------|-----------------|--------------|-----------------|-----------------|--------------|
| Fonte | Escallo Chat | Escallo Call | Feegow | Medula | Feegow | ClubFlex + Medula |
| Período | Hoje | Hoje | D+1 | D-1 | D-1 até D+15 | Último mês |
| Canal | Chat/Social | Telefone | Web | Presencial | Web | N/A |
| Áudio | Não | Sim | Não | Não | Não | Não |
| Validação Direção | Não | Sim | Não | Não | Não | Não |
| Origens | 1 | 2 | 1 | 1 | 1 | 1 |
| Deduplicação | Contato | Telefone | CPF | CPF | CPF+Tel | CPF |

## 📊 Canais de Atendimento

O Report 087 pode incluir conversas de diversos canais:

- 💬 Chat Web
- 📱 WhatsApp
- 📘 Facebook Messenger
- 📷 Instagram Direct
- 💼 LinkedIn Messages
- 📧 Email
- Outros canais integrados ao Escallo

## 🎯 Casos de Uso Comercial

Esta automação é útil para:

1. **Lead Digital**: Capturar contatos de canais digitais
2. **Atendimento Multicanal**: Integrar diversos canais em um funil
3. **Rastreamento**: Protocolo e histórico de mensagens
4. **Qualificação**: Apenas agentes comerciais garantem qualidade
5. **Follow-up**: Duração e quantidade de mensagens indicam interesse

## 🔍 Simplificações vs Ligação

A automação de chat é **mais simples** que a de ligação:

1. ❌ Não busca áudios (Report 002)
2. ❌ Não valida direção (aceita entrada e saída)
3. ❌ Não diferencia por fila (origem única)
4. ❌ Não aceita agentes não-comerciais em nenhuma condição
5. ✅ Código mais enxuto e direto

## 🎯 Próximos Passos (Sugestões)

1. Implementar diferenciação por fila (usar constante `FILAS`)
2. Adicionar validação de direção (só entrada ou ambas?)
3. Adicionar try/catch para tratamento de erros
4. Implementar logging estruturado
5. Adicionar métricas por canal (WhatsApp vs Chat vs Social)
6. Considerar criar origem específica para Chat Ação Record
7. Adicionar validação de formato do valor de contato
8. Implementar retry logic para falhas
9. Adicionar timestamp de processamento
10. Integrar com classificação para segmentação
11. Adicionar limite de tempo (só chats recentes?)
12. Validar se `qtdeMensagens` > 0 antes de processar
