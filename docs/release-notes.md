# Release Notes

Highlights from recent EpidBot releases. For the full change history see the [repository commits](https://github.com/Deeplearn-PeD/EpiDBot/commits/main).

## October 2026 (v2.8)

### Admin MCP Server for External Agents

EpidBot now ships a **Model Context Protocol server** that lets an external agent query the administrative subsystem and execute admin actions — designing marketing campaigns, creating ads, sending emails, refreshing user profiles — from any MCP-capable client.

- Streamable-HTTP endpoint at `/api/v1/mcp`, opt-in via `mcp.enabled` in `config.yaml`
- Dedicated `mk_` admin keys (`scope='admin_mcp'`) with the owner's admin role re-checked on every request — demoting an admin revokes MCP access instantly
- 24 tools: read-only analytics plus campaign/ad/email actions; real-email actions require an explicit `confirm=true`
- Every mutation is written to a new `admin_audit_log` table (actor, key, action, target, result)
- DNS-rebinding protection on by default — production must list its domain in `mcp.allowed_hosts`

👉 Full guide: [Admin MCP Server](admin/mcp-server.md)

### User Country Inference in the Admin Panel

The **Country** column in the admin users table now shows the best available localization estimate per user instead of only the (mostly empty) self-reported profile field:

1. **Confirmed** — the user-entered profile country (high confidence)
2. **Chat** — a country inferred by the LLM profiling pipeline from chat history (institutions, cities, municipalities), stored in a separate `country_llm` column so user data stays sacred
3. **Email** — the country-code TLD of the email domain (`@fiocruz.br` → Brazil, `.ac.uk` → UK; free-mail providers ignored)
4. **Language** — the UI language, only for unambiguous locales (`pt-BR`, `zh-CN`)

The table renders a colored provenance dot with a tooltip, and the "top countries" stats chart aggregates with the same rules. Backfill by running **Admin → Profiles → update** (or wait for the scheduled profile cycle).

### Edge TTS: Free Neural Voices by Default

The default text-to-speech engine is now **Microsoft Edge neural voices** (`edge-tts`) — free, no API key. Voice selection is **language-aware**: the server detects the message language and picks a matching neural voice from a curated catalog covering all interface languages (pt-BR, English, Spanish, Chinese, Arabic).

- OpenAI and local Piper engines remain one-click fallbacks in the admin panel
- Gift (group-invite) accounts included; STT/dictation is unchanged

### Group Invites: Plans Now Revert on Time

Fixed a bug where accounts created through **admin group invites** kept their gifted plan past the expected date:

- The invited plan duration is now stored per invite (`plan_duration_days`, default 30) instead of a hardcoded interval, and the admin form exposes it explicitly
- Gifted plans **no longer receive the 7-day grace period** — they revert to free exactly at expiry (paid subscriptions keep the grace window, since real payments deserve the benefit of the doubt)
- If the user subscribes themselves, renewal works normally and grace applies again

### Also in this release

- Admin MCP key management endpoints (`POST/GET/DELETE /admin/mcp-keys`)
- Per-invite plan duration badge and form field in the admin UI (all 5 UI languages)
- Email-country badges and tooltips in the admin users table (5 languages)
- `admin_audit_log` table for traceability of MCP-triggered actions

## September 2026

- **Group invites with gifted subscriptions**: admins can create invite links (in the Groups tab) that add redeemers to a group and gift a chosen plan (Pro/Max/Team) — with a configurable link validity, usage cap and now a configurable plan duration
- **Subscription expiry automation**: expired paid plans downgrade to free automatically after a grace period, with a notification email
