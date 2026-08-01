# OpenAI GPT-5.6 Operating Standard for OpenClaw

> มาตรฐานปฏิบัติสำหรับใช้งาน OpenClaw ร่วมกับ OpenAI GPT-5.x / GPT-5.6 โดยอ้างอิง public sources ที่ตรวจในช่วง July 2026 และออกแบบให้ใช้จริงในงานสอน งานเอกสาร งาน automation และ AI-agent operations

---

## Source Baseline

Reviewed date: `2026-08-01`  
Research window: `July 2026`  
Primary sources:

- OpenClaw OpenAI provider documentation: `https://docs.openclaw.ai/providers/openai`
- OpenClaw model providers documentation: `https://docs.openclaw.ai/concepts/model-providers`
- OpenClaw models CLI documentation: `https://docs.openclaw.ai/cli/models`
- OpenAI GPT-5.6 announcement: `https://openai.com/index/gpt-5-6/`
- OpenAI Help Center preview note: `https://help.openai.com/en/articles/20001325-a-preview-of-gpt-56-sol-terra-and-luna`

> Treat this document as an operating guide, not as a permanent source of pricing or model availability. Always verify model catalog and pricing before production use.

---

## Core Rule

```text
Do not hardcode GPT-5.x model IDs, pricing, or access assumptions without checking the live OpenClaw/OpenAI catalog first.
```

Use OpenClaw CLI verification before setting production defaults:

```bash
# ตรวจ provider และ model catalog ที่บัญชีนี้ใช้งานได้จริง
openclaw models auth list --provider openai
openclaw models list --provider openai
openclaw models status --probe
```

---

## Provider Route Standard

OpenClaw uses the OpenAI provider namespace:

```text
openai/*
```

Recommended operating pattern:

| Purpose | Model reference pattern | Rule |
|---|---|---|
| Direct OpenAI API usage | `openai/<model-id>` | Use API-key auth and verify available models first |
| GPT-5.6 default route | `openai/gpt-5.6` | Verify whether this resolves to the expected tier for the current account/runtime |
| GPT-5.6 Sol route | `openai/gpt-5.6-sol` | Use for complex reasoning only after catalog verification |
| GPT-5.6 Terra route | `openai/gpt-5.6-terra` | Use only if present in `openclaw models list --provider openai` |
| GPT-5.6 Luna route | `openai/gpt-5.6-luna` | Use only if present in `openclaw models list --provider openai` |
| Fallback route | `openai/<fallback-model-id>` | Use a deliberately selected fallback, not an accidental downgrade |

---

## Authentication Patterns

| Pattern | Recommended for | Notes |
|---|---|---|
| API-key auth | Server, automation, API billing, CI-like workflows | Store key outside repository; never commit real `.env` |
| OpenAI/Codex OAuth auth | Subscription-oriented usage and Codex-style runtime | Use named profiles where possible |
| Device-code auth | Headless machines, remote terminal, workshop setup | Useful when browser login is not available |

Example commands:

```bash
# Login through the default OpenAI auth flow
openclaw models auth login --provider openai

# Login with API key method when supported
openclaw models auth login --provider openai --method api-key

# Login from a headless environment
openclaw models auth login --provider openai --device-code

# Keep separate profiles for training, demo, and production accounts
openclaw models auth login --provider openai --profile-id training-demo
```

---

## Model Tier Strategy

| Tier | Practical use | Avoid using for |
|---|---|---|
| GPT-5.6 Sol | Deep reasoning, coding, legal/audit-style analysis, complex agents | High-volume cron jobs or routine summaries when cost matters |
| GPT-5.6 Terra | Balanced documentation, teaching, analysis, general professional tasks | Tasks that require maximum reasoning or ultra-low-cost high-volume work |
| GPT-5.6 Luna | Lightweight summaries, classification, bulk routine tasks, cron | High-stakes reasoning, complex code repair, long-context legal/audit analysis |

> The exact model route and tier availability may differ by account, auth method, region, runtime policy, and OpenAI rollout status. Verify before use.

---

## Recommended Defaults by Use Case

| Use case | Primary strategy | Fallback strategy | Extra control |
|---|---|---|---|
| Teaching demo | Terra or Luna | Smaller verified model | Cap output length and keep examples short |
| Daily brief | Luna | Terra | Limit sources/results and require source summary |
| Document summary | Terra | Luna for short docs | Chunk large files and avoid full-file reads |
| Coding assistant | Sol or Terra | Terra | Require tests or explicit verification steps |
| Legal/audit-style analysis | Sol | Terra only for lower-risk drafts | Require citations and preserve source basis |
| Cron automation | Luna or Terra | Disable fallback for risky jobs | Use isolated session and strict output budget |
| Web-search agent | Terra | Luna for light discovery | Require source list and recency check |

---

## Production Configuration Pattern

```bash
# 1) ตรวจ catalog ก่อนเสมอ
openclaw models list --provider openai

# 2) ตั้ง primary model จาก model ID ที่ตรวจพบจริง
openclaw models set "openai/<verified-primary-model-id>"

# 3) ล้าง fallback เดิมก่อนตั้งค่าใหม่
openclaw models fallbacks clear

# 4) ตั้ง fallback อย่างตั้งใจ ไม่ใช่ปล่อยให้ downgrade เอง
openclaw models fallbacks add "openai/<verified-fallback-model-id>"

# 5) ตรวจสถานะหลังตั้งค่า
openclaw gateway restart
openclaw models status --probe
```

---

## Alias Standard

Use aliases to make workshops and documentation easier to follow:

```bash
# Alias สำหรับงาน reasoning หนัก
openclaw models aliases add gpt-reasoning "openai/<verified-sol-or-terra-model-id>"

# Alias สำหรับงาน daily/cron ที่ต้องคุมต้นทุน
openclaw models aliases add gpt-economy "openai/<verified-luna-or-terra-model-id>"

# ตรวจ alias ก่อนใช้จริง
openclaw models aliases list
```

---

## Governance Controls

Before using GPT-5.x in OpenClaw production workflows, confirm:

- [ ] Model is visible in `openclaw models list --provider openai`
- [ ] Auth profile is correct for the intended account
- [ ] No real API key or token is stored in the repository
- [ ] Fallback model is explicitly selected
- [ ] Cron jobs use isolated sessions
- [ ] Prompt/output budgets are defined
- [ ] File workflows use chunking for large files
- [ ] Web-search workflows require sources
- [ ] Logs are sanitized before sharing
- [ ] Pricing and quota have been checked on the provider side

---

## Failure Handling

| Symptom | First action | Follow-up |
|---|---|---|
| Model not found | Run `openclaw models list --provider openai` | Replace stale model ID |
| Auth failure | Run `openclaw models auth list --provider openai` | Re-login or rotate API key |
| Unexpected runtime | Check provider/model runtime policy | Pin runtime only when needed |
| Cost spike | Stop cron/web jobs first | Reduce model tier, output budget, and tool calls |
| Poor answer quality | Check prompt, source basis, and model tier | Upgrade model or narrow scope |
| Context overflow | Start new isolated session | Split files/tasks into smaller batches |

---

## Documentation Rule

All README/docs examples should use placeholders unless the model route is explicitly verified:

```text
Good: openai/<verified-model-id>
Good: openai/gpt-5.6-sol, if verified in the live catalog
Avoid: hardcoding a model route as universally available
Avoid: hardcoding pricing without a date and source
```
