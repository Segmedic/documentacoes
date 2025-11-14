# Guia da API - Segmedic Totem

## O que é este sistema?

O Segmedic Totem é uma API que permite que pacientes realizem agendamentos médicos de forma autônoma através de totems de autoatendimento, similar a um caixa eletrônico bancário.

---

## Principais Funcionalidades

### 1. 📋 Agendamentos

**O que faz:** Permite criar, consultar e gerenciar consultas médicas.

**Fluxo básico:**
1. Paciente informa CPF
2. Sistema busca dados do paciente
3. Paciente escolhe especialidade
4. Paciente escolhe médico
5. Paciente escolhe data e horário
6. Sistema confirma agendamento

**Principais ações:**
- ✅ Agendar uma consulta
- 📅 Consultar agendamentos existentes
- ❌ Cancelar um agendamento
- 🔄 Reagendar uma consulta
- 🎫 Gerar senha de atendimento

---

### 2. 👥 Cadastro de Pacientes

**O que faz:** Gerencia informações dos pacientes.

**Informações necessárias:**
- Nome completo
- CPF
- Data de nascimento
- Telefone
- E-mail
- Tipo de atendimento (particular ou convênio)

**Principais ações:**
- ➕ Cadastrar novo paciente
- 🔍 Buscar paciente por CPF
- ✏️ Atualizar dados cadastrais

---

### 3. 💰 Pagamentos

**O que faz:** Processa pagamentos das consultas.

**Formas de pagamento aceitas:**
- 💳 Cartão de crédito
- 📱 PIX
- 💵 Dinheiro (registrado no sistema)

**Fluxo de pagamento:**
1. Sistema calcula valor da consulta
2. Paciente escolhe forma de pagamento
3. Sistema processa pagamento
4. Sistema emite comprovante
5. Sistema libera senha de atendimento

---

### 4. 🏥 Convênios Médicos

**O que faz:** Valida se o paciente possui plano de saúde ativo.

**Verificações realizadas:**
- ✅ Paciente possui convênio?
- ✅ Convênio está ativo?
- ✅ Convênio cobre a especialidade?
- ✅ Possui carência?

**Convênio ClubFlex:**
- Sistema de assinatura mensal
- Desconto em consultas
- Verificação automática de elegibilidade

---

### 5. 👨‍⚕️ Profissionais e Especialidades

**O que faz:** Mostra médicos disponíveis e especialidades.

**Informações disponíveis:**
- Nome do médico
- Especialidade
- CRM
- Unidades de atendimento
- Horários disponíveis

**Especialidades exemplo:**
- Cardiologia
- Ortopedia
- Pediatria
- Clínica Geral
- Dermatologia

---

### 6. 🏢 Unidades de Atendimento

**O que faz:** Mostra locais disponíveis para atendimento.

**Informações de cada unidade:**
- Nome da clínica
- Endereço completo
- Telefone
- Horário de funcionamento
- Especialidades disponíveis

---

## Fluxo Completo de Uso

### Cenário 1: Paciente com Convênio

```
1. Paciente digita CPF no totem
2. Sistema verifica se CPF está cadastrado
3. Sistema verifica elegibilidade no convênio
4. Paciente escolhe especialidade
5. Sistema mostra médicos disponíveis
6. Paciente escolhe médico e horário
7. Sistema confirma agendamento
8. Sistema gera senha de atendimento
9. Sistema imprime comprovante
```

### Cenário 2: Paciente Particular (sem convênio)

```
1. Paciente digita CPF no totem
2. Sistema verifica se CPF está cadastrado
   - Se não: paciente preenche cadastro
3. Paciente escolhe especialidade
4. Sistema mostra médicos e valores
5. Paciente escolhe médico e horário
6. Sistema mostra valor da consulta
7. Paciente escolhe forma de pagamento
8. Sistema processa pagamento
9. Sistema confirma agendamento
10. Sistema gera senha de atendimento
11. Sistema imprime comprovante
```

### Cenário 3: Paciente já tem consulta marcada

```
1. Paciente digita CPF no totem
2. Sistema mostra consultas agendadas
3. Paciente escolhe opção:
   - Fazer check-in (chegada)
   - Reimprimir senha
   - Cancelar consulta
   - Agendar nova consulta
```

---

## Benefícios do Sistema

### Para Pacientes
- ⚡ Agilidade no atendimento
- 🕐 Disponível 24/7
- 📱 Não precisa ligar para clínica
- 🎫 Emissão imediata de senha
- 💳 Pagamento na hora

### Para Clínica
- 📊 Redução de filas
- 💼 Menos carga no atendimento
- 📈 Melhor gestão de agenda
- 💰 Recebimento automático
- 📋 Registro de todas as interações

---

## Integrações Externas

O sistema se comunica com outros sistemas para funcionar:

### 1. **Feegow** (Sistema de Gestão)
- Gerencia agendamentos
- Armazena cadastro de pacientes
- Controla agenda dos médicos
- Emite notas fiscais

### 2. **ClubFlex** (Convênio)
- Verifica elegibilidade
- Consulta status do plano
- Valida benefícios

### 3. **Itaú** (Pagamentos PIX)
- Gera QR Code PIX
- Verifica status de pagamento
- Confirma recebimento

---

## Dados Importantes

### Informações do Agendamento
- **ID do Agendamento**: Número único que identifica a consulta
- **Data e Hora**: Quando será a consulta
- **Médico**: Quem irá atender
- **Valor**: Quanto custa a consulta
- **Status**: Situação atual (Agendado, Cancelado, Finalizado)

### Informações do Paciente
- **ID do Paciente**: Número único no sistema
- **CPF**: Documento de identificação
- **Dados de Contato**: Telefone e e-mail
- **Convênio**: Se possui e qual

### Informações de Pagamento
- **Forma de Pagamento**: Como foi pago
- **Valor**: Quanto foi cobrado
- **Status**: Se foi aprovado ou não
- **Comprovante**: Número da transação

---

## Status dos Agendamentos

| Status | Significado |
|--------|-------------|
| **Agendado** | Consulta confirmada e aguardando data |
| **Confirmado** | Paciente confirmou presença |
| **Em Atendimento** | Paciente está sendo atendido |
| **Finalizado** | Consulta concluída |
| **Cancelado** | Consulta foi cancelada |
| **Faltou** | Paciente não compareceu |

---

## Tipos de Atendimento

### Particular
- Paciente paga direto pela consulta
- Valores conforme tabela da clínica
- Pagamento no ato do agendamento

### Convênio
- Paciente usa plano de saúde
- Sistema verifica elegibilidade
- Cobrança conforme contrato do convênio

### ClubFlex
- Sistema de assinatura mensal
- Desconto em consultas
- Verificação automática de benefícios

---

## Segurança e Privacidade

- 🔒 Todos os dados são criptografados
- 👤 Acesso controlado por autenticação
- 📝 Todos as ações são registradas (auditoria)
- 🗑️ Dados sensíveis não são impressos
- ⚠️ Conformidade com LGPD (Lei Geral de Proteção de Dados)

---

## Relatórios Disponíveis

A API permite gerar relatórios gerenciais:

- 📊 Total de agendamentos por período
- 💰 Receita por forma de pagamento
- 👥 Novos pacientes cadastrados
- 📈 Especialidades mais procuradas
- ⏱️ Horários de maior movimento
- ❌ Taxa de cancelamentos
- 📱 Uso do totem vs. atendimento telefônico

---

## Perguntas Frequentes

### Como funciona o cancelamento?
O paciente pode cancelar até X horas antes da consulta (conforme política da clínica). O sistema registra o motivo e libera o horário para outros pacientes.

### E se o pagamento falhar?
O sistema tenta novamente. Se persistir o erro, oferece outras formas de pagamento ou permite agendar sem pagar (conforme política da clínica).

### O que acontece se o paciente não aparecer?
O sistema marca como "Faltou" após X minutos do horário. Isso pode gerar bloqueio temporário para novos agendamentos (conforme política).

### Como funciona a fila de espera?
Quando paciente chega, o totem gera uma senha. O sistema gerencia a ordem de atendimento baseado em prioridade e ordem de chegada.

### Paciente pode agendar para outra pessoa?
Sim, basta ter o CPF da pessoa. Útil para pais agendando para filhos menores.

---

## Glossário

- **Totem**: Terminal de autoatendimento (máquina que o paciente usa)
- **Agendamento**: Marcação de uma consulta
- **Procedimento**: Tipo de atendimento (consulta, exame, etc.)
- **Unidade**: Clínica ou local de atendimento
- **Senha**: Número para controle da fila de atendimento
- **Convênio**: Plano de saúde
- **Elegibilidade**: Verificação se paciente pode usar o convênio
- **Invoice**: Nota fiscal
- **PIX**: Sistema de pagamento instantâneo
- **Check-in**: Confirmação de chegada do paciente

---

## Contato e Suporte

Para dúvidas sobre o sistema:
- 📧 E-mail: suporte@segmedic.com.br
- 📞 Telefone: (11) 1234-5678
- 🌐 Site: www.segmedic.com.br
