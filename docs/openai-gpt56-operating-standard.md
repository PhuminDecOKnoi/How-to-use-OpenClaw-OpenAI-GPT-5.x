# OpenAI GPT-5.6 Operating Standard for OpenClaw

> มาตรฐานปฏิบัติสำหรับใช้งาน OpenClaw ร่วมกับ OpenAI GPT-5.x / GPT-5.6 โดยอ้างอิง public sources ที่ตรวจในช่วง July 2026 และออกแบบให้ใช้จริงในงานสอน งานเอกสาร งาน automation และ AI-agent operations

---

## Source Baseline

Reviewed date: `2026-08-01`  
Research window: `July 2026`  
Source registry: [`docs/references.md`](references.md)

Primary source families:

- OpenClaw OpenAI provider documentation: [OC-OPENAI]
- OpenClaw Models CLI documentation: [OC-MODELS]
- OpenClaw model providers documentation: [OC-PROVIDERS]
- OpenAI GPT-5.6 announcement: [OA-GPT56]
- OpenAI Help Center GPT-5.6 preview note: [OA-GPT56-HELP]
- July 30, 2026 pricing-change reporting: [NEWS-REUTERS]
- Agent security research for OpenClaw-style tool agents: [SEC-PRISM]

> Treat this document as an operating guide, not as a permanent source of pricing or model availability. Always verify model catalog, account access, quota, and pricing before production use. [OC-MODELS] [NEWS-REUTERS]

---

## Core Rule

```text
Do not hardcode GPT-5.x model IDs, pricing, quota, or access assumptions without checking the live OpenClaw/OpenAI catalog first.
```

Use OpenClaw CLI verification before setting production defaults. The `models` CLI is the supported place to inspect auth profiles, model lists, model status, fallbacks, and aliases. [OC-MODELS]

```bash
# ตรวจ provider auth profile ของ OpenAI
openclaw models auth list --provider openai

# ตรวจ model catalog ที่บัญชีนี้เห็นจริง
openclaw models list --provider openai

# ตรวจสถานะ model และ probe ก่อนใช้งานจริง
openclaw models status --probe
```

---

## Provider Route Standard

OpenClaw uses the OpenAI provider namespace `openai/*` for OpenAI model references. This route pattern is documented by the OpenClaw OpenAI provider and model-provider documentation. [OC-OPENAI] [OC-PROVIDERS]

```text
openai/*
```

Recommended operating pattern:

| Purpose | Model reference pattern | Rule | Source |
|---|---|---|---|
| Direct OpenAI API usage | `openai/<model-id>` | Use API-key auth and verify available models first | [OC-OPENAI] |
| GPT-5.6 default route | `openai/gpt-5.6` | Verify whether this resolves to the expected tier for the current account/runtime | [OC-PROVIDERS] |
| GPT-5.6 Sol route | `openai/gpt-5.6-sol` | Use for complex reasoning only after catalog verification | [OA-GPT56] |
| GPT-5.6 Terra route | `openai/gpt-5.6-terra` | Use only if present in `openclaw models list --provider openai` | [OA-GPT56] [OC-MODELS] |
| GPT-5.6 Luna route | `openai/gpt-5.6-luna` | Use only if present in `openclaw models list --provider openai` | [OA-GPT56] [OC-MODELS] |
| Fallback route | `openai/<fallback-model-id>` | Use a deliberately selected fallback, not an accidental downgrade | [OC-MODELS] |

---

## Authentication Patterns

OpenClaw documents OpenAI provider auth through the `openai` provider ID and Models CLI auth commands. [OC-OPENAI] [OC-MODELS]

| Pattern | Recommended for | Notes | Source |
|---|---|---|---|
| API-key auth | Server, automation, API billing, CI-like workflows | Store key outside repository; never commit real `.env` | [OC-OPENAI] |
| OpenAI/Codex OAuth auth | Subscription-oriented usage and Codex-style runtime | Use named profiles where possible | [OC-OPENAI] |
| Device-code auth | Headless machines, remote terminal, workshop setup | Useful when browser login is not available | [OC-MODELS] |

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

OpenAI describes GPT-5.6 as a model family with Sol, Terra, and Luna access through supported OpenAI surfaces, subject to approval and rollout. [OA-GPT56] [OA-GPT56-HELP]

| Tier | Practical use | Avoid using for | Source |
|---|---|---|---|
| GPT-5.6 Sol | Deep reasoning, coding, legal/audit-style analysis, complex agents | High-volume cron jobs or routine summaries when cost matters | [OA-GPT56] |
| GPT-5.6 Terra | Balanced documentation, teaching, analysis, general professional tasks | Tasks that require maximum reasoning or ultra-low-cost high-volume work | [OA-GPT56] |
| GPT-5.6 Luna | Lightweight summaries, classification, bulk routine tasks, cron | High-stakes reasoning, complex code repair, long-context legal/audit analysis | [OA-GPT56] |

> The exact model route and tier availability may differ by account, auth method, region, runtime policy, and OpenAI rollout status. Verify before use. [OA-GPT56-HELP] [OC-MODELS]

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

The cost-sensitive recommendations above are strategy guidance. They must be checked against current model availability and pricing before production. [OC-MODELS] [NEWS-REUTERS]

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

The commands above are based on OpenClaw Models CLI patterns for listing, setting, probing, aliases, and fallbacks. [OC-MODELS]

---

## Alias Standard

Use aliases to make workshops and documentation easier to follow. Alias handling belongs to the OpenClaw Models CLI. [OC-MODELS]

```bash
# Alias สำหรับงาน reasoning หนัก
openclaw models aliases add gpt-reasoning "openai/<verified-sol-or-terra-model-id>"

# Alias สำหรับงานสมดุลคุณภาพ/ต้นทุน
openclaw models aliases add gpt-balanced "openai/<verified-terra-model-id>"

# Alias สำหรับงาน daily/cron ที่ต้องคุมต้นทุน
openclaw models aliases add gpt-economy "openai/<verified-luna-or-terra-model-id>"

# ตรวจ alias ก่อนใช้จริง
openclaw models aliases list
```

---

## Governance Controls

Before using GPT-5.x in OpenClaw production workflows, confirm:

- [ ] Model is visible in `openclaw models list --provider openai` [OC-MODELS]
- [ ] Auth profile is correct for the intended account [OC-MODELS]
- [ ] No real API key or token is stored in the repository [OC-OPENAI]
- [ ] Fallback model is explicitly selected [OC-MODELS]
- [ ] Cron jobs use isolated sessions and output budgets [NEWS-REUTERS]
- [ ] Prompt/output budgets are defined [NEWS-REUTERS]
- [ ] File workflows use chunking for large files
- [ ] Web-search workflows require sources
- [ ] Logs are sanitized before sharing [SEC-PRISM]
- [ ] Pricing and quota have been checked on the provider side [NEWS-REUTERS]

---

## Failure Handling

| Symptom | First action | Follow-up | Source |
|---|---|---|---|
| Model not found | Run `openclaw models list --provider openai` | Replace stale model ID | [OC-MODELS] |
| Auth failure | Run `openclaw models auth list --provider openai` | Re-login or rotate API key | [OC-MODELS] |
| Unexpected runtime | Check provider/model runtime policy | Pin runtime only when needed | [OC-PROVIDERS] |
| Cost spike | Stop cron/web jobs first | Reduce model tier, output budget, and tool calls | [NEWS-REUTERS] |
| Poor answer quality | Check prompt, source basis, and model tier | Upgrade model or narrow scope | [OA-GPT56] |
| Context overflow | Start new isolated session | Split files/tasks into smaller batches | [SEC-PRISM] |

---

## Reference Links

[OC-OPENAI]: https://docs.openclaw.ai/providers/openai
[OC-MODELS]: https://docs.openclaw.ai/cli/models
[OC-PROVIDERS]: https://docs.openclaw.ai/concepts/model-providers
[OA-GPT56]: https://openai.com/index/gpt-5-6/
[OA-GPT56-HELP]: https://help.openai.com/en/articles/20001325-a-preview-of-gpt-56-sol-terra-and-luna
[NEWS-REUTERS]: https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/
[SEC-PRISM]: https://arxiv.org/abs/2603.11853
