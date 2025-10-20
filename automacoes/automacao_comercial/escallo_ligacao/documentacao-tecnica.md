# Documentação Técnica - Escallo Ligação

## 📋 Visão Geral

A automação de **Escallo Ligação** tem como objetivo capturar ligações telefônicas de entrada (inbound) atendidas por agentes comerciais no sistema Escallo, gerando leads qualificados para o CRM com base em critérios específicos de fila e agente.

## 🎯 Objetivo

Capturar ligações telefônicas que:

- São de **entrada** (direção = "Entrada")
- Foram atendidas por **agentes comerciais** cadastrados
- Passaram por filas de atendimento específicas
- Não são duplicadas (mesmo número de origem)
- Possuem agente e origem válidos

## 🔧 Tecnologias e Dependências

### Serviços Externos

- **Escallo**: Sistema de call center e atendimento telefônico
- **AWS SQS**: Fila para envio de leads

### APIs Escallo

- **Report 086**: Relatório de ligações com detalhes de atendimento
- **Report 002**: Relatório com links de áudio das ligações

### Constantes

- `AGENTES_ESCALLO_RD`: Mapeamento de IDs de agentes comerciais
- `FILAS`: Filas específicas do Ação Record

## 📊 Fluxo de Execução

### 1. Inicialização

```typescript
handler()
```

- Cria cliente Escallo
- Define data de hoje para busca

### 2. Busca de Dados (Paralela)

Busca dois relatórios do dia atual:

- **Report 086**: Ligações com detalhes de atendimento
- **Report 002**: Áudios das ligações

### 3. Processamento de Registros

Para cada registro de ligação:

#### 3.1. Verificação de Fila

```typescript
filaAcaoRecord = ["Ação Record", "Transferencia Ação Record"]
```

Verifica se a ligação passou por fila do Ação Record.

#### 3.2. Verificação de Agente

Verifica se o agente está na lista de **agentes comerciais** cadastrados no `AGENTES_ESCALLO_RD`.

#### 3.3. Lógica de Filtragem

Continua para próximo registro se:

- ❌ Número de origem já foi processado (duplicata)
- ❌ Não atende a condição: `(agenteComercial OU (NÃO agenteComercial E filaAcaoRecord))`

**Tradução da condição:**

- Se é agente comercial → **Sempre processa**
- Se NÃO é agente comercial → **Só processa se for fila Ação Record**

#### 3.4. Busca de Áudio

Localiza o áudio correspondente usando `uniqueid` da ligação.

#### 3.5. Validação de Lead

Chama função `isLead()` que valida:

- ✅ Possui agente (não vazio)
- ✅ Possui origem/número (não vazio)
- ✅ Direção é "Entrada" (inbound)

#### 3.6. Definição de Origem

```typescript
origin = filaAcaoRecord 
  ? "escallo_ligacao_record" 
  : "escallo_ligacao"
```

Define origem baseado na fila de atendimento.

### 4. Criação e Envio do Lead

- Cria `LeadEvent` com origem apropriada
- Adiciona link do áudio ao payload
- Envia para fila SQS
- Adiciona número à lista de processados (evita duplicatas)

## 📦 Estrutura de Dados

### Registro86 (Escallo)

```typescript
{
  dataHoraInicial: string,        // Data/hora da ligação
  uniqueid: string,               // ID único da ligação
  primaryuuid: string,            // UUID primário
  origem: string,                 // Número de telefone do cliente
  destino: string,                // Número de destino
  agenteId: string,               // ID do agente
  agente: string,                 // Nome do agente
  filaAtendimentoId: string,      // ID da fila
  filaAtendimento: string,        // Nome da fila
  direcao: string,                // "Entrada" ou "Saída"
  fcr: string,                    // First Call Resolution
  classificacaoId: string | null, // ID da classificação
  classificacao: string | null,   // Classificação do atendimento
  observacao: string | null,      // Observações
  linkAudio?: string              // URL do áudio (adicionado)
}
```

### LeadEvent

```typescript
{
  origin: "escallo_ligacao" | "escallo_ligacao_record",
  payload: Registro86
}
```

## 🔍 Critérios de Filtragem

### Filas Aceitas (Ação Record)

```typescript
["Ação Record", "Transferencia Ação Record"]
```

### Agentes Comerciais

20 agentes mapeados em `AGENTES_ESCALLO_RD`:

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
function isLead(registro86) {
  return !(
    agente == "" ||
    origem == "" ||
    direcao != "Entrada"
  )
}
```

**Requisitos:**

- ✅ Deve ter agente atribuído
- ✅ Deve ter número de origem
- ✅ Deve ser ligação de entrada (inbound)

### Deduplicação

- Array `numbers` mantém números já processados
- Mesmo número não gera múltiplos leads

## ⚙️ Configurações

### Período de Busca

- **Data**: Hoje (data atual)
- **Relatórios**: 086 (ligações) e 002 (áudios)

### Origens de Lead

- **escallo_ligacao**: Ligações de filas normais com agentes comerciais
- **escallo_ligacao_record**: Ligações de filas Ação Record

## 🎯 Lógica de Decisão

### Tabela de Decisão

| Agente Comercial? | Fila Ação Record? | Processa? |
|-------------------|-------------------|-----------|
| ✅ Sim | ✅ Sim | ✅ Sim - Record |
| ✅ Sim | ❌ Não | ✅ Sim - Normal |
| ❌ Não | ✅ Sim | ✅ Sim - Record |
| ❌ Não | ❌ Não | ❌ Não |

### Condição Booleana

```typescript
agenteComercial || (!agenteComercial && filaAcaoRecord)
```

Simplificando:

```typescript
agenteComercial || filaAcaoRecord
```

## 🚨 Tratamento de Erros

- Não possui try/catch explícito
- Erros serão propagados para o runtime
- Não há garantia de destruição de conexões

## 📝 Observações Importantes

1. **Ligações Inbound Only**: Apenas ligações de entrada são processadas
2. **Deduplicação Simples**: Array de números processados na memória
3. **Áudio Opcional**: Link de áudio pode não existir
4. **Duas Origens**: Diferencia entre filas normais e Ação Record
5. **Agentes Cadastrados**: Lista fixa de agentes comerciais
6. **Mesma Data**: Busca apenas ligações do dia atual
7. **Matching de Áudio**: Usa uniqueid para correlacionar áudio com ligação

## 🔄 Dependências de Outros Módulos

- `src/escallo/client.ts`: Cliente da API Escallo
- `src/aws/sqs.ts`: Envio para fila
- `src/rd/constants.ts`: Mapeamento de agentes

## 🎯 Diferenças vs Outras Automações

| Característica | Escallo Ligação | Escallo Chat | Leads Amanhã |
|----------------|-----------------|--------------|--------------|
| Período | Hoje | Hoje | D+1 |
| Fonte | Escallo (Call Center) | Escallo (Chat) | Feegow |
| Tipo de Contato | Telefone | Chat | Agendamento |
| Direção | Inbound | Inbound/Outbound | N/A |
| Áudio | Sim | Não | Não |
| Filas Específicas | Ação Record | Nenhuma | N/A |
| Deduplicação | Array números | Array números | Set CPF |

## 📊 Fluxo de Dados

```event
Escallo Report 086 → Ligações do Dia
         ↓
Filtro: Agente Comercial OU Fila Ação Record
         ↓
Validação: isLead() (agente, origem, direção)
         ↓
Escallo Report 002 → Busca Áudio
         ↓
Deduplicação: números já processados
         ↓
AWS SQS → Lead Event
```

## 🎯 Casos de Uso Comercial

Esta automação é útil para:

1. **Rastreamento de Leads**: Capturar todos os contatos telefônicos
2. **Qualificação**: Apenas ligações atendidas por time comercial
3. **Auditoria**: Áudio disponível para revisão
4. **Follow-up**: Leads quentes de contato direto
5. **Análise**: Diferenciar entre filas normais e Ação Record

## 🎯 Próximos Passos (Sugestões)

1. Adicionar try/catch para tratamento de erros
2. Implementar logging estruturado
3. Adicionar métricas (volume por agente, por fila)
4. Considerar deduplicação por período (não apenas execução)
5. Validar se áudio está disponível antes de enviar
6. Adicionar timestamp de processamento
7. Implementar retry logic para falhas
8. Considerar cache para evitar reprocessamento
9. Adicionar validação de formato de número de telefone
10. Integrar com classificação de atendimento para segmentação
