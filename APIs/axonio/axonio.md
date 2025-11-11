# Axônio API

**Tagline:** "Conduzindo dados estruturados do Medula"

## 1. Contexto e Objetivo

### 1.1. Contexto

A Segmedic possui um data hub interno chamado **Medula**, que agrega informações de diversas bases de dados. Dentro do Medula, a base **Sinapse** oferece visões estruturadas e consolidadas, cruzando dados de diferentes fontes. A Axônio API tem como principal função expor esses dados estruturados do Sinapse, e secundariamente, metadados controlados do próprio Medula.

### 1.2. Objetivo

O objetivo da Axônio API é criar uma **camada de acesso unificada, segura e escalável** aos dados do Medula, pronta para consumo por aplicações internas e externas. A API padronizará aspectos cruciais como autenticação, paginação, filtros, seleção/expansão de campos, idempotência e telemetria, garantindo consistência e facilidade de uso.

## 2. Identidade e Versionamento

* **Nome:** Axônio API
* **Tagline:** “Conduzindo dados estruturados do Medula”
* **Base URL (exemplo):** `https://api.segmedic.com/axonio/v1`
* **Namespace:** `/axonio/v1/…`
* **Header Opcional de Versão:** `Axonio-Version: 1`
* **Política de Versionamento:** Versionamento via path (`/v1`). Futuras versões serão introduzidas com novos paths (`/v2`, etc.). A política de depreciação será documentada no roadmap, com aviso prévio para migração.

## 3. Stack Técnica

* **Linguagem/Framework:** TypeScript + NestJS (Node 20+)
* **Padrões:** ESLint, Prettier, Husky + lint-staged (pre-commit)
* **Build:** Dockerfile multi-stage (produção)
* **Execução:** PM2 ou node (em container)
* **Hospedagem Alvo:** AWS EC2
* **Dependências Recomendadas:**
  * **Config/Env:** `@nestjs/config`, `dotenv`
  * **Validação/DTO:** `class-validator`, `class-transformer`, `ValidationPipe` global
  * **Documentação:** `@nestjs/swagger` + OpenAPI 3.0
  * **Segurança:** `helmet`, `rate-limiting` (`@nestjs/throttler`), CORS configurável
  * **Logs:** `pino` ou `winston` + `pino-http`/`winston-http`
  * **Observabilidade:** OpenTelemetry (HTTP tracing, métricas básicas, exporter OTLP)
  * **Idempotência:** Middleware/Interceptor com cabeçalho `Idempotency-Key` e store (em memória, substituível por Redis)
  * **Request ID/Correlation:** Interceptor que injeta `X-Request-Id`/`Correlation-Id`
* **Acesso a Dados:**
  * Utiliza um padrão de repositórios e serviços para abstrair a origem dos dados.
  * **Assunção:** PostgreSQL para exemplos (via Prisma).
  * **Flexibilidade:** O README detalhará como alternar para conectores de DW/lake (ex.: Athena, BigQuery, etc.) através da implementação de novos repositórios. O código não conterá lógica específica de fornecedor de banco de dados diretamente nos serviços.

## 4. Segurança e Autenticação

* **Autenticação:** OAuth2 Client Credentials com Bearer JWT.
* **Verificação do Token:** Validação de `iss` (issuer), `aud` (audience) e `exp` (expiração) do token. Suporte a JWKS (JSON Web Key Set) para obtenção de chaves públicas (URL configurável via variável de ambiente).
* **Autorização:** Placeholder para autorização baseada em roles (claim "scope" ou "roles" no JWT) com um guard simples.
* **Headers e Práticas:**
  * `Authorization: Bearer <token>`
  * `Helmet` ativado para proteção contra vulnerabilidades web comuns.
  * CORS restrito por variáveis de ambiente (origens permitidas).
  * **Rate Limit Padrão:** 100 requisições por minuto por cliente (retornar `429 Too Many Requests` quando exceder).
  * Conteúdo sensível (ex: senhas, PII) logado apenas de forma anonimizada ou omitido.

## 5. Contratos da API

### 5.1. Padrão de Paginação (Cursor-based)

* **Query Parameters:**
  * `page_size` (integer): Número de itens por página (default: 100, máximo: 1000).
  * `cursor` (string): Cursor para a próxima página, obtido da resposta anterior.
* **Resposta Inclui:**

    ```json
    {
      "data": [...],
      "pagination": {
        "page_size": 100,
        "next_cursor": "eyJpZCI6IjEyMyIsImRhdGUiOiIyMDI0LTAxLTAxIn0=" // Base64 encoded string ou null
      }
    }
    ```

### 5.2. Filtragem e Ordenação

* **Filtragem:**
  * Sintaxe: `filter[<campo>]=<valor>`
  * Exemplos:
    * `filter[origem]=erp`
    * `filter[data_gte]=2024-01-01` (maior ou igual)
    * `filter[status_in]=ativo,pendente` (valores múltiplos)
* **Ordenação:**
  * Sintaxe: `sort=<campo>` ou `sort=-<campo>` (para ordem decrescente)
  * Exemplos:
    * `sort=atualizado_em` (crescente)
    * `sort=-atualizado_em` (decrescente)

### 5.3. Seleção de Campos e Expansão

* **Seleção de Campos:**
  * Sintaxe: `fields[<entidade>]=<campo1>,<campo2>`
  * Exemplo: `fields[entidade]=id,nome,categoria`
* **Expansão de Relacionamentos:**
  * Sintaxe: `include=<relacao1>,<relacao2>`
  * Exemplo: `include=relacoes,metadados`

### 5.4. Idempotência

* **Cabeçalho:** `Idempotency-Key: <UUID>` para requisições `POST` e `PUT` que criam ou alteram recursos.
* **Comportamento:** Se a mesma `Idempotency-Key` for reutilizada dentro da janela configurada (ex: 10 minutos), a API retornará o mesmo resultado da primeira requisição bem-sucedida, sem reprocessar a operação.

### 5.5. Padrão de Erros

Todos os erros da API seguirão um padrão consistente:

```json
{
  "error": {
    "code": "STRING_CODE",         // Código de erro legível por máquina (ex: "BAD_REQUEST", "UNAUTHORIZED")
    "message": "Descrição humana",  // Mensagem de erro detalhada
    "details": { ... },             // Detalhes adicionais do erro (ex: erros de validação)
    "hint": "Sugestão de correção", // Dica para o desenvolvedor corrigir o problema
    "doc_url": "https://dev.segmedic.com/axonio/docs#rota", // Link para a documentação específica do erro
    "trace_id": "uuid/request-id"   // ID único da requisição para rastreamento nos logs
  }
}
