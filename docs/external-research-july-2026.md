# External Research Notes: OpenClaw + OpenAI GPT-5.x in July 2026

> Research note สำหรับบันทึกแหล่งข้อมูลภายนอกที่ใช้ปรับปรุง repository นี้ ไม่ใช่ replacement ของ official docs และไม่ใช่ข้อมูลราคา/availability แบบถาวร

---

## Research Metadata

| Field | Value |
|---|---|
| Research window | July 2026 |
| Reviewed on | 2026-08-01 |
| Topic | OpenClaw usage with OpenAI GPT-5.x / GPT-5.6 |
| Repository impact | Model strategy, cost control, security baseline, README documentation map, Thai lesson update |
| Source registry | [`docs/references.md`](references.md) |

---

## Source Reliability Tiers

| Tier | Source type | How to use |
|---|---|---|
| A | Official OpenClaw docs and OpenAI docs/help center | Use as primary reference for commands, provider routes, model family, access caveats, and availability wording |
| B | Reuters / Axios / Business Insider pricing news | Use as signal that pricing changed; verify current pricing before production |
| C | Security research / practitioner security reporting | Use for threat-model and control design; separate research claims from product commands |
| D | Community posts / Reddit / blogs | Use only as anecdotal implementation signals; do not treat as canonical |
| E | Unverified AI-generated content | Do not use as a repository source |

---

## Source Log

| Source ID | Source | Link | Supported takeaway | Repo action |
|---|---|---|---|---|
| OC-OPENAI | OpenClaw OpenAI provider docs | <https://docs.openclaw.ai/providers/openai> | OpenAI is addressed through the OpenClaw OpenAI provider namespace and includes provider/runtime/auth guidance | Added provider-route and authentication standards |
| OC-MODELS | OpenClaw Models CLI docs | <https://docs.openclaw.ai/cli/models> | Documents `models auth`, provider login, profile handling, device-code, model listing, aliases, and fallbacks | Added CLI verification and auth examples |
| OC-PROVIDERS | OpenClaw model providers docs | <https://docs.openclaw.ai/concepts/model-providers> | Documents `provider/model` refs, provider runtime split, `openai/*`, GPT-5.6 route examples, and model listing verification | Added verification-first rule and route examples |
| OA-GPT56 | OpenAI GPT-5.6 announcement | <https://openai.com/index/gpt-5-6/> | GPT-5.6 family includes Sol, Terra, and Luna tiers; available through OpenAI surfaces subject to rollout/access | Added tier strategy section |
| OA-GPT56-HELP | OpenAI Help Center GPT-5.6 preview | <https://help.openai.com/en/articles/20001325-a-preview-of-gpt-56-sol-terra-and-luna> | Describes access through API/Codex depending on organization approval | Added access caveat wording |
| NEWS-REUTERS | Reuters July 30 pricing news | <https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/> | Indicates Terra/Luna pricing changed near end of July 2026 | Added pricing verification warning |
| NEWS-AXIOS | Axios July 30 pricing news | <https://www.axios.com/2026/07/30/openai-cuts-prices-gpt-terra-luna5> | Confirms July 30 pricing-change news cycle | Added warning not to hardcode pricing |
| NEWS-BI | Business Insider July 2026 pricing article | <https://www.businessinsider.com/openai-price-cuts-gpt-terra-luna-2026-7> | Provides additional reporting on GPT-5.6 Terra/Luna pricing changes | Added cost-control review rule |
| SEC-PRISM | OpenClaw PRISM security research | <https://arxiv.org/abs/2603.11853> | Tool-augmented agents introduce risks including prompt injection, unsafe tool execution, credential leakage, path/tool controls, and audit/security controls | Added agent security threat model |
| NEWS-SECURITY | TechRadar security article | <https://www.techradar.com/pro/here-are-the-openclaw-security-risks-you-should-know-about> | Practitioner-facing warning about OpenClaw security risks, config exposure, and cost limits | Added secondary security caution |
| THW-HANDSON | Tom's Hardware hands-on article | <https://www.tomshardware.com/tech-industry/artificial-intelligence/setting-up-openclaw-isnt-as-straightforward-as-the-internet-wants-you-to-think-running-local-ai-on-humble-hardware> | Hands-on caution that setup/automation can be non-trivial and should be verified | Added implementation caution only where appropriate |

---

## Point-Level Citation Decisions

| Claim location | Source marker used | Why |
|---|---|---|
| Provider route `openai/*` | [OC-OPENAI], [OC-PROVIDERS] | Official product documentation supports provider route and provider/model reference pattern |
| `openclaw models auth login --provider openai` | [OC-MODELS] | Official CLI docs support auth command patterns |
| `openclaw models list --provider openai` verification rule | [OC-MODELS], [OC-PROVIDERS] | Official CLI/model-provider docs support live catalog verification |
| GPT-5.6 Sol/Terra/Luna tier strategy | [OA-GPT56], [OA-GPT56-HELP] | OpenAI official announcement and Help Center support model family/access caveats |
| Pricing must be rechecked | [NEWS-REUTERS], [NEWS-AXIOS], [NEWS-BI] | News sources support July 30 pricing-change signal; not permanent pricing source |
| Tool-call / prompt-injection / credential controls | [SEC-PRISM], [NEWS-SECURITY] | Security research/practitioner reporting support risk-control rationale |

---

## Decisions Adopted in This Repository

1. Use `openai/*` as the OpenClaw provider namespace pattern. [OC-OPENAI] [OC-PROVIDERS]
2. Treat GPT-5.6 Sol/Terra/Luna as a model-tier strategy, not as a hardcoded availability guarantee. [OA-GPT56] [OA-GPT56-HELP]
3. Require `openclaw models list --provider openai` before setting any production model. [OC-MODELS]
4. Keep all model IDs in examples as `<verified-model-id>` unless the route has been verified. [OC-MODELS]
5. Add cost-control guidance for cron, web search, file workflows, and bulk tasks. [NEWS-REUTERS] [SEC-PRISM]
6. Add security controls for agent tools, prompt injection, token hygiene, and unsafe tool execution. [SEC-PRISM]
7. Keep pricing language date-bound and verification-first. [NEWS-REUTERS]

---

## Recommended Source-Handling Rule

```text
Official docs decide commands and canonical configuration.
News sources only flag market/pricing changes.
Security research supports risk controls.
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

---

## Reference Links

[OC-OPENAI]: https://docs.openclaw.ai/providers/openai
[OC-MODELS]: https://docs.openclaw.ai/cli/models
[OC-PROVIDERS]: https://docs.openclaw.ai/concepts/model-providers
[OA-GPT56]: https://openai.com/index/gpt-5-6/
[OA-GPT56-HELP]: https://help.openai.com/en/articles/20001325-a-preview-of-gpt-56-sol-terra-and-luna
[NEWS-REUTERS]: https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/
[NEWS-AXIOS]: https://www.axios.com/2026/07/30/openai-cuts-prices-gpt-terra-luna5
[NEWS-BI]: https://www.businessinsider.com/openai-price-cuts-gpt-terra-luna-2026-7
[SEC-PRISM]: https://arxiv.org/abs/2603.11853
[NEWS-SECURITY]: https://www.techradar.com/pro/here-are-the-openclaw-security-risks-you-should-know-about
[THW-HANDSON]: https://www.tomshardware.com/tech-industry/artificial-intelligence/setting-up-openclaw-isnt-as-straightforward-as-the-internet-wants-you-to-think-running-local-ai-on-humble-hardware
