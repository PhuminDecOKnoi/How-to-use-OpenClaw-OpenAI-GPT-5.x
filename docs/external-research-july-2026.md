# External Research Notes: OpenClaw + OpenAI GPT-5.x in July 2026

> Research note สำหรับบันทึกแหล่งข้อมูลภายนอกที่ใช้ปรับปรุง repository นี้ ไม่ใช่ replacement ของ official docs และไม่ใช่ข้อมูลราคา/availability แบบถาวร

---

## Research Metadata

| Field | Value |
|---|---|
| Research window | July 2026 |
| Reviewed on | 2026-08-01 |
| Topic | OpenClaw usage with OpenAI GPT-5.x / GPT-5.6 |
| Repository impact | Model strategy, cost control, security baseline, README documentation map |

---

## Source Reliability Tiers

| Tier | Source type | How to use |
|---|---|---|
| A | Official OpenClaw docs and OpenAI docs/help center | Use as primary reference for commands, provider routes, model family, and availability wording |
| B | Reuters / Axios / Business Insider pricing news | Use as signal that pricing changed; verify current pricing before production |
| C | Community posts / Reddit / blogs | Use only as anecdotal implementation signals; do not treat as canonical |
| D | Unverified AI-generated content | Do not use as a repository source |

---

## Source Log

| Source | URL | Supported takeaway | Repo action |
|---|---|---|---|
| OpenClaw OpenAI provider docs | `https://docs.openclaw.ai/providers/openai` | OpenAI is addressed through the OpenClaw OpenAI provider namespace and includes provider/runtime/auth guidance | Added provider-route and authentication standards |
| OpenClaw model providers docs | `https://docs.openclaw.ai/concepts/model-providers` | Verify account/model availability with `openclaw models list --provider openai`; OpenAI route-specific model behavior may vary by setup | Added verification-first rule |
| OpenClaw models CLI docs | `https://docs.openclaw.ai/cli/models` | Documents `models auth`, provider login, profile handling, device-code, and model listing patterns | Added CLI verification and auth examples |
| OpenAI GPT-5.6 announcement | `https://openai.com/index/gpt-5-6/` | GPT-5.6 family includes Sol, Terra, and Luna tiers; available through OpenAI surfaces subject to rollout/access | Added tier strategy section |
| OpenAI Help Center GPT-5.6 preview | `https://help.openai.com/en/articles/20001325-a-preview-of-gpt-56-sol-terra-and-luna` | Describes Sol as flagship, Terra as lower-cost strong option, and Luna as fastest/cost-efficient option | Added model-tier use-case mapping |
| Reuters July 30 pricing news | `https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/` | Indicates Terra/Luna pricing changed near end of July 2026 | Added pricing verification warning |
| Axios July 30 pricing news | `https://www.axios.com/2026/07/30/openai-cuts-prices-gpt-terra-luna5` | Confirms July 30 pricing-change news cycle | Added warning not to hardcode pricing |
| Business Insider July 2026 pricing article | `https://www.businessinsider.com/openai-price-cuts-gpt-terra-luna-2026-7` | Provides additional reporting on GPT-5.6 Terra/Luna pricing changes | Added cost-control review rule |
| OpenClaw security analysis on arXiv | `https://arxiv.org/abs/2603.11619` | Autonomous agents with tools, memory, external apps, and automation increase attack surface | Added agent security threat model |

---

## Decisions Adopted in This Repository

1. Use `openai/*` as the OpenClaw provider namespace pattern.
2. Treat GPT-5.6 Sol/Terra/Luna as a model-tier strategy, not as a hardcoded availability guarantee.
3. Require `openclaw models list --provider openai` before setting any production model.
4. Keep all model IDs in examples as `<verified-model-id>` unless the route has been verified.
5. Add cost-control guidance for cron, web search, file workflows, and bulk tasks.
6. Add security controls for agent tools, prompt injection, token hygiene, and unsafe tool execution.
7. Keep pricing language date-bound and verification-first.

---

## Recommended Source-Handling Rule

```text
Official docs decide commands and canonical configuration.
News sources only flag market/pricing changes.
Community sources can suggest issues to test, but cannot decide standards.
```

---

## Open Questions for Future Maintenance

- Has OpenClaw changed provider route behavior for GPT-5.6 after July 2026?
- Are Sol, Terra, and Luna all visible in the target account's `openclaw models list --provider openai` output?
- Are there new model aliases or default model policies in OpenClaw docs?
- Has OpenAI updated pricing after July 30, 2026?
- Do production accounts use API-key auth, OAuth/Codex auth, or both?
- Should this repository add a hands-on lab for model selection and cost testing?
