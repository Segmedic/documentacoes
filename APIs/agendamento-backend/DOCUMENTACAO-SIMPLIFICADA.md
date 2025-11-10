# Sistema de Agendamento Online - Guia Simplificado

## 🎯 Visão Geral

O Sistema de Agendamento Online é uma API que permite aos pacientes agendar consultas médicas de forma digital, integrando-se com o sistema Feegow. A plataforma gerencia todo o fluxo desde a captura do interesse do paciente até a confirmação do agendamento.

---

## 🔄 Como Funciona

### Fluxo Completo do Agendamento

``` cURL
1. Paciente acessa o site
   ↓
2. Sistema cria um registro de interesse (Lead)
   ↓
3. Paciente escolhe convênio e especialidade
   ↓
4. Sistema busca horários disponíveis
   ↓
5. Paciente escolhe data e horário
   ↓
6. Sistema cadastra ou busca dados do paciente
   ↓
7. Agendamento é confirmado
   ↓
8. Email de confirmação é enviado
```

---

## 📊 Principais Funcionalidades

### 1. **Gestão de Leads** 🎯

**O que faz:** Registra o interesse do paciente em agendar uma consulta.

**Quando usar:**

- Quando um paciente inicia o processo de agendamento
- Para rastrear todas as etapas até a conclusão
- Para análise de conversão e abandono de agendamentos

**Informações armazenadas:**

- Nome do paciente
- Email e telefone
- Convênio escolhido
- Especialidade de interesse
- Data de início do processo

---

### 2. **Cadastro de Pacientes** 👤

**O que faz:** Busca ou cria o cadastro do paciente no sistema.

**Quando usar:**

- Antes de confirmar qualquer agendamento
- Para verificar se o paciente já existe no sistema
- Para atualizar dados cadastrais

**Informações necessárias:**

- Nome completo
- CPF
- Data de nascimento
- Email
- Telefone
- Gênero

---

### 3. **Consulta de Convênios** 🏥

**O que faz:** Lista todos os planos de saúde aceitos.

**Quando usar:**

- Na primeira etapa do agendamento
- Para validar se o convênio do paciente é aceito
- Para exibir opções de planos disponíveis

**Informações fornecidas:**

- Nome do convênio
- ID para integração
- Status de disponibilidade

---

### 4. **Busca de Especialidades** 🩺

**O que faz:** Mostra todas as especialidades médicas disponíveis.

**Quando usar:**

- Após o paciente selecionar o convênio
- Para filtrar profissionais e horários
- Para direcionar o paciente ao especialista correto

**Exemplos de especialidades:**

- Cardiologia
- Pediatria
- Ginecologia
- Ortopedia
- E outras 30+ especialidades

---

### 5. **Disponibilidade de Horários** 📅

**O que faz:** Busca datas e horários disponíveis para agendamento.

**Quando usar:**

- Após paciente selecionar especialidade e profissional
- Para mostrar agenda dos próximos 30 dias
- Para permitir remarcação de consultas

**Filtros disponíveis:**

- Período (data início e fim)
- Especialidade médica
- Profissional específico
- Unidade de atendimento
- Convênio e plano

---

### 6. **Criação de Agendamentos** ✅

**O que faz:** Confirma e registra a consulta no sistema.

**Quando usar:**

- Após paciente escolher data e horário
- Para bloquear o horário na agenda
- Para gerar confirmação por email

**Informações registradas:**

- Dados do paciente
- Profissional selecionado
- Data e horário
- Unidade de atendimento
- Convênio e procedimento
- Observações especiais

---

### 7. **Gerenciamento de Agendamentos** 🔄

**O que faz:** Permite alterar ou cancelar consultas agendadas.

**Quando usar:**

**Atualização:**

- Remarcar consulta
- Alterar observações
- Atualizar status

**Cancelamento:**

- Paciente não pode comparecer
- Profissional indisponível
- Clínica precisa reagendar

**Motivos de cancelamento:**

- Por solicitação do paciente
- Por solicitação do profissional
- Por necessidade da clínica

---

### 8. **Comunicação por Email** 📧

**O que faz:** Envia emails automáticos aos pacientes.

**Tipos de email:**

1. **Confirmação de Consulta**
   - Enviado após agendamento bem-sucedido
   - Contém data, horário e local
   - Inclui dados do profissional

2. **Resultados de Exames**
   - Notifica quando exames estão prontos
   - Fornece link para acesso

3. **Pesquisa de Satisfação (NPS)**
   - Enviada após consulta realizada
   - Coleta feedback do atendimento

---

### 9. **Verificação Clubflex** 🏃

**O que faz:** Verifica se o paciente tem benefício Clubflex.

**Quando usar:**

- Durante processo de agendamento
- Para aplicar descontos especiais
- Para validar elegibilidade

**Informações verificadas:**

- Status de ativação
- Tipo de plano
- Benefícios disponíveis

---

### 10. **Upload de Documentos** 📎

**O que faz:** Permite envio de pedidos médicos e documentos.

**Quando usar:**

- Paciente tem pedido médico para anexar
- Necessário enviar documentação prévia
- Para consultas que requerem autorização

**Tipos de arquivo aceitos:**

- PDF
- Imagens (JPG, PNG)
- Documentos escaneados

---

### 11. **Verificação de Elegibilidade (Nuria - não funcionamento)** 🏥

**O que faz:** Valida elegibilidade do paciente junto ao convênio.

**Quando usar:**

- Antes de confirmar agendamento
- Para verificar cobertura do plano
- Para validar dados da carteirinha

**Validações realizadas:**

- Carteirinha ativa
- Cobertura do procedimento
- Dados cadastrais do titular

---

## 📈 Métricas e Monitoramento

### Indicadores Importantes

1. **Taxa de Conversão**
   - Leads criados vs agendamentos confirmados
   - Abandono em cada etapa do funil

2. **Volume de Agendamentos**
   - Total por dia/semana/mês
   - Por especialidade
   - Por convênio

3. **Cancelamentos**
   - Taxa de cancelamento
   - Motivos mais comuns
   - Tempo médio até cancelamento

4. **Satisfação (NPS)**
   - Score de satisfação
   - Feedback dos pacientes
   - Pontos de melhoria

---

## 🔐 Segurança e Privacidade

### Proteção de Dados

- **Dados Pessoais:** Todos os dados são protegidos conforme LGPD
- **CPF:** Armazenado de forma segura e criptografada
- **Integração Segura:** Comunicação com Feegow via API autenticada
- **Arquivos:** Upload protegido com validação de tipo e tamanho

---

## 🚨 Cenários de Uso Comuns

### Cenário 1: Novo Paciente Agendando Primeira Consulta

```cURL
1. Sistema cria Lead
2. Paciente seleciona convênio (ex: Bradesco)
3. Paciente escolhe especialidade (ex: Cardiologia)
4. Sistema mostra horários disponíveis
5. Paciente escolhe data/horário
6. Sistema cadastra novo paciente
7. Agendamento é confirmado
8. Email de confirmação enviado
```

### Cenário 2: Paciente Existente Reagendando

```cURL
1. Sistema cria Lead
2. Paciente seleciona convênio e especialidade
3. Sistema busca paciente por CPF (já existe)
4. Paciente escolhe novo horário
5. Agendamento é confirmado
6. Email de confirmação enviado
```

### Cenário 3: Cancelamento de Consulta (ainda em breve)

```cURL
1. Sistema localiza agendamento
2. Paciente solicita cancelamento
3. Sistema registra motivo
4. Horário fica disponível novamente
5. Email de cancelamento enviado (opcional)
```

---

## 🎨 Integração com Frontend

### Páginas Típicas

1. **Página Inicial**
   - Seleção de convênio
   - Início do Lead

2. **Escolha de Especialidade**
   - Lista de especialidades
   - Busca por nome

3. **Seleção de Horário**
   - Calendário com disponibilidade
   - Filtros por profissional/unidade

4. **Dados do Paciente**
   - Formulário de cadastro
   - Busca por CPF

5. **Confirmação**
   - Resumo do agendamento
   - Botão confirmar
   - Opção de anexar pedido médico

6. **Sucesso**
   - Confirmação visual
   - Detalhes do agendamento
   - Opções de adicionar à agenda

---

## 📞 Suporte

### Para Questões Técnicas

- Verifique a documentação técnica completa (API-REFERENCE.md)
- Consulte o diagrama de fluxo (FLUXO-API.md)
- Entre em contato com a equipe de desenvolvimento

### Para Questões de Negócio

- Acompanhe métricas no dashboard
- Revise relatórios de agendamentos
- Analise feedbacks dos pacientes

---

## 📚 Glossário

**Lead**: Registro de interesse do paciente em agendar uma consulta

**Convênio**: Plano de saúde ou seguro médico

**Procedimento**: Consulta ou exame específico

**Especialidade**: Área médica (Cardiologia, Pediatria, etc.)

**Unidade**: Local físico de atendimento

**Elegibilidade**: Validação se o paciente tem direito ao atendimento

**NPS**: Net Promoter Score - métrica de satisfação do cliente

**Clubflex**: Programa de benefícios para pacientes

**Feegow**: Sistema de gestão clínica integrado

---

## ✅ Checklist de Sucesso

### Para um Agendamento Completo

- [x] Lead criado com sucesso
- [x] Convênio validado
- [x] Especialidade selecionada
- [x] Horário disponível encontrado
- [x] Paciente cadastrado ou localizado
- [x] Elegibilidade confirmada (se aplicável)
- [x] Agendamento registrado no sistema
- [x] Email de confirmação enviado
- [x] Paciente recebeu todas as informações

---

*Última atualização: 10 de novembro de 2025*
