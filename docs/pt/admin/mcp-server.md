# Servidor MCP Administrativo

O EpidBot expõe o **subsistema administrativo a agentes externos** (Claude Code, Cursor, orquestradores customizados, etc.) por meio de um servidor [Model Context Protocol](https://modelcontextprotocol.io). O agente pode consultar análises — usuários, planos, uso de tokens, adoção de funcionalidades, métricas de campanhas — e executar ações administrativas: criar campanhas de e-mail marketing, criar anúncios no app, enviar e-mails, atualizar perfis de usuários e mais.

O servidor é **opcional** (desativado por padrão), autenticado com **chaves de API administrativas dedicadas**, e toda ação de escrita é registrada em um **log de auditoria**.

## Visão geral

| | |
|---|---|
| Transporte | MCP **streamable HTTP** (JSON-RPC + SSE) |
| Endpoint | `/api/v1/mcp` (funciona com ou sem barra final) |
| Autenticação | cabeçalho `X-API-Key: mk_…` (ou `Authorization: Bearer mk_…`) |
| Versão do protocolo | MCP 2025-06-18 (via SDK oficial `mcp`) |
| Estado padrão | **Desativado** — ative com `mcp.enabled: true` no `config.yaml` |
| Ferramentas | 24 tools: 14 somente leitura, 10 ações (sem exclusões, por design) |
| Implementação | `epidbot/mcp/` — FastMCP + Starlette, montado por `create_app()` |

As ferramentas **reutilizam os handlers do FastAPI diretamente**, então validações, salvaguardas de plano e regras de ciclo de vida (ex.: campanhas só são editáveis em `draft`) se comportam de forma idêntica à API REST e à interface web.

```text
Agente externo ──MCP (streamable HTTP)──► NGINX (/api/) ──► FastAPI :8123
                                                 │
                                        /api/v1/mcp
                                                 │
                                     ┌───────────┴───────────┐
                                     │  middleware de auth   │  chave mk_ + papel admin vigente
                                     │  FastMCP (24 tools)   │  → handlers/serviços
                                     │  gravador de auditoria│  → admin_audit_log
                                     └───────────────────────┘
```

## Ativando o Servidor

Adicione uma seção `mcp:` ao `config.yaml` e reinicie a API:

```yaml
mcp:
  enabled: true
  path: /api/v1/mcp                # caminho de montagem (padrão)
  server_name: epidbot-admin       # nome anunciado no handshake MCP
  dns_rebinding_protection: true   # checagens de Host/Origin (recomendado)
  allowed_hosts:                   # cabeçalhos Host aceitos — OBRIGATÓRIO em produção
    - kwar-ai.com.br
    - localhost
    - 127.0.0.1
```

!!! warning "Produção: configure `mcp.allowed_hosts`"
    O SDK MCP traz proteção contra DNS rebinding que rejeita qualquer requisição cujo cabeçalho `Host` não esteja em `allowed_hosts` (**HTTP 421**). O NGINX encaminha o domínio público real, então ambientes de produção **precisam** listá-lo — caso contrário toda requisição falha com `421 Misdirected Request`. O padrão permite apenas `localhost` / `127.0.0.1` (para desenvolvimento local com o MCP Inspector).

Desativar (`mcp.enabled: false`, o padrão) pula a montagem por completo; o restante do EpidBot não é afetado.

## Chaves de API (`mk_`)

O acesso MCP usa chaves **separadas das chaves de API comuns `ek_`**:

- Armazenadas na tabela `api_keys` com `scope = 'admin_mcp'`
- Prefixo `mk_` + 64 caracteres hex; a chave completa é exibida **uma única vez** na criação
- O dono deve ser **administrador** — e o papel de admin é revalidado no banco de dados em **toda requisição**: rebaixar um admin revoga o acesso MCP dele instantaneamente, sem tocar na chave
- Chaves `ek_` comuns são rejeitadas pelo servidor MCP, e chaves `mk_` não funcionam nos endpoints REST comuns — são domínios de confiança separados

### Gerenciando Chaves (endpoints REST administrativos)

Autentique com um **JWT** de admin (o mesmo do painel administrativo web):

```bash
# Emite uma chave (copie o valor da resposta — não é recuperável depois)
curl -X POST https://seu-host/api/v1/admin/mcp-keys \
  -H "Authorization: Bearer $ADMIN_JWT" \
  -H "Content-Type: application/json" \
  -d '{"name": "growth-agent"}'
# → {"id": 3, "name": "growth-agent", "key": "mk_9f2c..."}

# Lista chaves (apenas metadados)
curl https://seu-host/api/v1/admin/mcp-keys \
  -H "Authorization: Bearer $ADMIN_JWT"

# Revoga
curl -X DELETE https://seu-host/api/v1/admin/mcp-keys/3 \
  -H "Authorization: Bearer $ADMIN_JWT"
```

Nomes duplicados são rejeitados (`400`). A revogação vale a partir da próxima requisição do agente.

## Conectando um Agente

Aponte o cliente MCP para `https://seu-host/api/v1/mcp` com o cabeçalho da chave. Exemplo em um agente Python usando o SDK oficial:

```python
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async def main():
    async with streamablehttp_client(
        "https://seu-host/api/v1/mcp",
        headers={"X-API-Key": "mk_9f2c..."},
    ) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()
            tools = await session.list_tools()
            result = await session.call_tool("get_user_stats", {})
            print(result.content[0].text)
```

Clientes desktop (Claude Desktop, Cursor, …) usam a configuração JSON equivalente:

```json
{
  "mcpServers": {
    "epidbot-admin": {
      "type": "http",
      "url": "https://seu-host/api/v1/mcp",
      "headers": { "X-API-Key": "mk_9f2c..." }
    }
  }
}
```

## Modelo de Segurança

- **Autenticação**: toda requisição — incluindo `initialize` e `tools/list` — passa pelo middleware de autenticação; não há descoberta anônima de capacidades.
- **Autorização**: `scope='admin_mcp'` **e** papel `admin` vigente do dono da chave.
- **Portão de confirmação**: ferramentas que alcançam usuários reais — `send_campaign`, `send_user_message` — recusam execução sem `confirm=true`. A recusa é explícita e auditada.
- **Sem exclusões**: a superfície de ferramentas exclui deliberadamente operações de delete (usuários, anúncios, campanhas, templates, grupos). Desativação (`is_active=false`) está disponível onde existe; deletes reais ficam no painel administrativo web.
- **Trilha de auditoria**: toda chamada de ferramenta de escrita grava uma linha na tabela `admin_audit_log` — autor, chave, ação, alvo, parâmetros e resultado (`ok` / `refused` / `error`).

```sql
SELECT created_at, actor_username, key_prefix, action, target_type, target_id, result
FROM admin_audit_log
ORDER BY created_at DESC
LIMIT 50;
```

## Referência de Ferramentas

Todas as ferramentas retornam JSON. Erros aparecem como resultados de erro MCP com o status HTTP de origem, ex.: `HTTP 404: Campaign not found`.

### Somente leitura — análises e inventário

#### `get_user_stats`

Visão geral da base de usuários: totais, ativos/verificados, cadastros e logins nos últimos 30 dias, distribuição por plano e idioma, e os principais países (resolvidos com as mesmas regras de inferência da tabela administrativa).

#### `list_users`

Listagem paginada com busca opcional ou filtro por grupo.

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `search` | str | `""` | Correspondência em nome de usuário ou e-mail |
| `page` | int | `1` | Página |
| `per_page` | int | `50` | Linhas por página (máx. 200) |
| `group_id` | int? | `null` | Restringe a um grupo |

Cada linha inclui `plan`, `role`, `is_active`, `language` e o `country` resolvido com proveniência — `country_source` (`user` / `chat` / `email` / `language`) e `country_confidence` (`high` / `medium` / `low`).

#### `get_usage_analytics`

Séries mensais de tokens/consultas/usuários ativos e média de consultas por usuário por grupo.

| Parâmetro | Tipo | Padrão |
|-----------|------|--------|
| `months` | int | `12` (1–36) |

#### `get_feature_usage`

Uso agregado de funcionalidades (chamadas de ferramentas do agente + recursos do produto).

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `days` | int | `30` | Janela |
| `group_id` | int? | `null` | Restringe a um grupo (`0` = sem grupo) |

#### `get_admin_config`

Visão somente leitura da configuração ativa de provedor/modelo.

#### `list_groups`

Todos os grupos com contagem de membros.

#### `get_job_status`

Consulta um job em segundo plano iniciado via MCP.

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| `job_id` | str | O id `job_<hex>` retornado por uma ação assíncrona |

Retorna `status` (`pending` / `processing` / `completed` / `failed`), `result` ou `error`. Jobs expiram uma hora após a conclusão.

### Somente leitura — marketing

#### `list_email_templates`

Todos os templates (id, nome, assunto, idioma, datas).

#### `preview_email_template`

Renderiza um template com dados de exemplo para revisar exatamente o que os destinatários receberão.

| Parâmetro | Tipo | Padrão |
|-----------|------|--------|
| `template_id` | int | — |
| `username` | str | `"John Doe"` |
| `plan` | str | `"free"` |

Variáveis de template: `{{username}}`, `{{plan}}`, `{{email}}`, `{{base_url}}`, `{{unsubscribe_url}}`.

#### `list_campaigns`

Todas as campanhas com `status` (`draft` / `sending` / `completed` / `failed`), filtro de audiência, variantes de idioma e contagens de destinatários.

#### `get_campaign_metrics`

Métricas de entrega por campanha: enviados / abertos / clicados, taxa de abertura, CTR, distribuição por plano, conversões, cancelamentos e receita atribuída.

#### `get_campaigns_overview`

Agregados mensais de marketing (e-mails, aberturas, cliques, taxas, receita, usuários pagantes).

#### `preview_campaign_audience`

Estima quantos usuários um filtro alcançaria, com amostra de 10 usuários. **Sempre execute antes de criar ou enviar uma campanha.**

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `plans` | str[] | `[]` | `free` / `pro` / `max` / `team`; vazio = todos |
| `group_ids` | int[] | `[]` | Segmentação por grupo |
| `include_groupless` | bool | `false` | Incluir também usuários sem grupo |
| `usage_levels` | str[] | `[]` | `power` / `regular` / `light` / `dormant` (quartis de uso) |
| `languages` | str[] | `[]` | Idiomas da interface: `en`, `pt-BR`, `es`, `zh-CN`, `ar` |
| `verified_only` | bool | `true` | Excluir e-mails não verificados |
| `created_after` | str? | `null` | Data ISO mínima de cadastro |
| `last_login_before` | str? | `null` | Data ISO — segmentar usuários inativos |

Descadastrados e inativos são sempre excluídos automaticamente.

#### `list_ads`

Todos os banners do app com segmentação, peso, janela de exibição e contagens de impressões/cliques.

### Ações — marketing (sem exclusões)

#### `create_email_template`

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `name` | str | — | Nome de exibição |
| `subject` | str | — | Assunto (Jinja permitido) |
| `html_body` | str | — | Corpo HTML com Jinja |
| `language` | str | `"all"` | `all` ou um de `en`, `pt-BR`, `es`, `zh-CN`, `ar` |

#### `update_email_template`

| Parâmetro | Tipo |
|-----------|------|
| `template_id` | int |
| `name`, `subject`, `html_body`, `language` | str opcionais — apenas os informados mudam |

#### `create_campaign`

Cria uma campanha em status **draft** (nada é enviado).

| Parâmetro | Tipo | Padrão |
|-----------|------|--------|
| `name` | str | — |
| `template_id` | int? | `null` — template padrão |
| `subject_override` | str? | `null` |
| campos de audiência | — | mesmos de `preview_campaign_audience` |
| `template_variants` | mapa locale → template_id | `{}` — entrega por idioma |

#### `update_campaign`

Edita uma campanha em **draft** (apenas os campos informados mudam). **Informar qualquer campo de audiência substitui o filtro inteiro** — rode `preview_campaign_audience` novamente para validar o alcance.

#### `create_ad`

Cria um banner exibido no app para usuários dos planos segmentados.

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `title_en`, `body_en` | str | — | Conteúdo em inglês (obrigatório) |
| `title_pt`, `body_pt` | str? | `null` | Variante em português |
| `cta_label` | str | `"Upgrade"` | Rótulo do botão |
| `cta_url` | str | `"/settings#plan"` | Destino do botão |
| `target_plans` | str[] | `["free"]` | Subconjunto de `free` / `pro` / `max` |
| `weight` | int | `10` | Frequência de exibição (1–100) |
| `is_active` | bool | `true` | |
| `starts_at`, `ends_at` | str? | `null` | Janela em datetime ISO |

#### `update_ad`

Atualiza um anúncio (apenas os campos informados mudam). Desative com `is_active=false` em vez de excluir.

#### `send_campaign` 🔒 `confirm`

Inicia o envio de uma campanha em **draft** para sua audiência como job em segundo plano.

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| `campaign_id` | int | Deve estar em `draft` |
| `confirm` | bool | **Deve ser `true`** — e-mails reais para usuários reais |

Retorna um `job_id` para `get_job_status`. Rastreamento por destinatário (pixel de abertura, redirect de clique, link de descadastro) e seleção de template por idioma idênticos ao envio pelo painel web.

#### `send_user_message` 🔒 `confirm`

Envia um e-mail direto para um usuário no template padrão do EpidBot.

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| `user_id` | int | Destinatário |
| `subject` | str | Máx. 200 caracteres |
| `body` | str | Texto simples (máx. 10 000 caracteres) |
| `confirm` | bool | **Deve ser `true`** |

### Ações — perfis

#### `refresh_user_profile`

Reconstrói o perfil de um usuário (interesses, totais de uso, país inferido do chat, …) a partir das sessões recentes.

| Parâmetro | Tipo | Padrão |
|-----------|------|--------|
| `user_id` | int | — |
| `use_llm` | bool | `true` — use `false` só para contagens |

Seguro por design: campos preenchidos pelo usuário nunca são sobrescritos por atualizações automáticas.

#### `refresh_all_profiles`

Mesma semântica do ciclo agendado de perfis, para todos os usuários ativos nos últimos 7 dias.

## Fluxo Típico: Criando uma Campanha

```text
1. get_user_stats()                        → quem está na plataforma
2. preview_campaign_audience(plans=["free"], languages=["pt-BR"])
                                           → estimativa de alcance + amostra
3. create_email_template(...)              → rascunho do texto
4. preview_email_template(template_id)     → revisar a renderização
5. create_campaign(name, template_id, audience)
6. update_campaign(...)                    → iterar em draft
7. send_campaign(campaign_id, confirm=true)
                                           → retorna job_id
8. get_job_status(job_id)                  → aguardar conclusão
9. get_campaign_metrics(campaign_id)       → aberturas, cliques, receita
```

## Solução de Problemas

| Sintoma | Causa | Correção |
|---------|-------|----------|
| `401 Invalid, revoked, or non-admin MCP key` | Chave revogada, valor errado ou dono rebaixado | Reemita a chave; verifique se o dono ainda é `role='admin'` |
| `401 Missing API key` | Cliente sem o cabeçalho | Configure `X-API-Key: mk_…` no cliente |
| `421 Misdirected Request` | `Host` fora de `mcp.allowed_hosts` | Adicione o domínio público a `mcp.allowed_hosts` e reinicie |
| `Refused: send_campaign(...) was NOT executed` | Portão de confirmação | Re-invoque com `confirm=true` após revisar a audiência |
| `HTTP 400: Cannot send campaign with status 'sending'` | Salvaguarda de ciclo de vida | Só campanhas em `draft` podem ser enviadas; aguarde o job atual |
| Job 404 | Job expirou | Jobs ficam disponíveis por 1 hora após a conclusão |
| Tools listadas mas chamadas travam | Session manager parado | Inicie a app pelo lifespan normal (systemd/docker), não por uma `Starlette` solitária |

## Limitações

- Sem operações de exclusão nem gestão de usuários/planos — essas ações ficam no painel administrativo web (estenda deliberadamente se necessário).
- Jobs em segundo plano expiram após 1 hora; as *métricas* de campanha são permanentes, mas o `get_job_status` de um envio concluído não.
- Requer cliente compatível com o transporte streamable-HTTP (qualquer cliente MCP de meados de 2025 em diante).
