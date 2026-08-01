# Source Reference Registry

> Central reference registry for this repository. Use these source IDs for point-level citations inside README, docs, lessons, and workshop material.

---

## Citation Standard

Use inline source markers immediately after the claim, command pattern, model-policy rule, or security/cost recommendation that relies on an external source.

Example:

```markdown
OpenClaw uses the `openai/*` provider route for OpenAI model references. [OC-OPENAI]
```

Rules:

1. Put the source marker close to the claim it supports.
2. Prefer official documentation for commands, provider routes, model availability, and configuration behavior.
3. Use news sources only for date-bound market or pricing-change signals.
4. Use security research for threat-model and control recommendations.
5. Do not use community posts as canonical source material unless clearly labeled as anecdotal.

---

## Primary Source IDs

| ID | Source | Link | Use for |
|---|---|---|---|
| OC-OPENAI | OpenClaw OpenAI provider documentation | <https://docs.openclaw.ai/providers/openai> | OpenAI provider ID, `openai/*` route, direct API-key auth, ChatGPT/Codex OAuth, OpenAI route behavior |
| OC-MODELS | OpenClaw Models CLI documentation | <https://docs.openclaw.ai/cli/models> | `openclaw models auth`, `models list`, `models set`, aliases, fallbacks, device-code login, profiles |
| OC-PROVIDERS | OpenClaw Model providers documentation | <https://docs.openclaw.ai/concepts/model-providers> | Provider/model reference pattern, provider runtime split, OpenAI setup defaults, GPT-5.6 route examples |
| OA-GPT56 | OpenAI GPT-5.6 announcement | <https://openai.com/index/gpt-5-6/> | GPT-5.6 Sol/Terra/Luna family, API/Codex/ChatGPT availability, official model-family positioning |
| OA-GPT56-HELP | OpenAI Help Center GPT-5.6 preview | <https://help.openai.com/en/articles/20001325-a-preview-of-gpt-56-sol-terra-and-luna> | Access caveats, organization approval, API/Codex access statements |
| NEWS-REUTERS | Reuters July 30, 2026 pricing report | <https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/> | Date-bound signal that GPT-5.6 Terra/Luna pricing changed and production pricing must be rechecked |
| NEWS-AXIOS | Axios July 30, 2026 pricing report | <https://www.axios.com/2026/07/30/openai-cuts-prices-gpt-terra-luna5> | Cross-check for July 30 pricing-change news cycle |
| NEWS-BI | Business Insider July 2026 pricing report | <https://www.businessinsider.com/openai-price-cuts-gpt-terra-luna-2026-7> | Additional pricing-change context; use only as secondary news support |
| SEC-PRISM | OpenClaw PRISM security research | <https://arxiv.org/abs/2603.11853> | Prompt injection, unsafe tool execution, credential leakage, path/tool controls, audit/security controls for OpenClaw-style agents |
| NEWS-SECURITY | TechRadar security article | <https://www.techradar.com/pro/here-are-the-openclaw-security-risks-you-should-know-about> | Practitioner-facing security caution, config/credential exposure, spending limits; secondary source only |
| THW-HANDSON | Tom's Hardware OpenClaw hands-on article | <https://www.tomshardware.com/tech-industry/artificial-intelligence/setting-up-openclaw-isnt-as-straightforward-as-the-internet-wants-you-to-think-running-local-ai-on-humble-hardware> | Non-canonical implementation caution; use only as anecdotal operational signal |

---

## Recommended Source Placement

| Topic in repo | Preferred source marker |
|---|---|
| OpenAI provider namespace `openai/*` | [OC-OPENAI] or [OC-PROVIDERS] |
| `openclaw models auth login --provider openai` | [OC-MODELS] |
| `openclaw models list --provider openai` | [OC-MODELS], [OC-PROVIDERS] |
| GPT-5.6 Sol/Terra/Luna model family | [OA-GPT56], [OA-GPT56-HELP] |
| Pricing or cost-change warning | [NEWS-REUTERS], optional [NEWS-AXIOS] / [NEWS-BI] |
| Prompt injection / unsafe tool execution / credential leakage | [SEC-PRISM] |
| Practitioner security cautions | [NEWS-SECURITY] |
| Hardware/local setup caution | [THW-HANDSON] |

---

## Maintenance Notes

- Review this file whenever OpenClaw or OpenAI changes model names, provider behavior, pricing, or authentication flow.
- Keep URLs as direct links, not short links.
- If a source disappears or becomes stale, mark it as deprecated rather than silently deleting it.
- Do not cite sources that do not directly support the nearby claim.
