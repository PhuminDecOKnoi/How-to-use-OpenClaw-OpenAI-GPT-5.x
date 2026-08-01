# Cost Control Guide

> แนวทางควบคุมต้นทุนเมื่อใช้งาน OpenClaw ร่วมกับ OpenAI GPT-5.x / GPT-5.6 โดยเฉพาะงานที่มี tool calls, web search, file workflow และ cron automation

---

## Source Basis

- GPT-5.6 Sol/Terra/Luna เป็น tier strategy ที่อ้างอิงจาก OpenAI official GPT-5.6 materials. [OA-GPT56] [OA-GPT56-HELP]
- ข่าวราคา July 30, 2026 จาก Reuters/Axios/Business Insider ใช้เป็นสัญญาณว่า pricing เปลี่ยนได้ จึงต้องตรวจราคาปัจจุบันก่อน production. [NEWS-REUTERS] [NEWS-AXIOS] [NEWS-BI]
- Tool-augmented agents มีความเสี่ยงด้าน tool use, credential leakage, prompt injection และ auditability จึงต้องคุม tool-call budget ควบคู่กับ cost. [SEC-PRISM]

---

## Cost-Control Principle

```text
Pick the lowest-cost verified model tier that can still complete the task safely and accurately.
```

Do not choose the largest reasoning model for every job. Match the model to workload risk, depth, and volume. Pricing and model availability can change, so verify current provider pricing before production use. [NEWS-REUTERS]

---

## Model Tier Policy

| Workload | Preferred tier | Reason | Source |
|---|---|---|---|
| One-off complex reasoning | Sol or Terra | Accuracy and reasoning quality matter more than cost | [OA-GPT56] |
| Routine professional writing | Terra | Balanced quality and cost | [OA-GPT56] |
| Daily brief / cron summary | Luna or Terra | Recurring jobs can accumulate cost quickly | [NEWS-REUTERS] |
| Bulk classification | Luna | Short output and high volume | [OA-GPT56] |
| Code repair / architecture review | Sol or Terra | More reasoning and verification needed | [OA-GPT56] |
| Legal/audit-style analysis | Sol or Terra | Source fidelity and reasoning quality are important | [OA-GPT56] |

> Always verify the actual route and pricing before production use. Public prices and tier availability can change after publication. [NEWS-REUTERS] [NEWS-AXIOS]

---

## Output Budget Standard

Every prompt used in production or cron should include an output budget. Output limits reduce token cost and reduce uncontrolled agent expansion risk. [NEWS-REUTERS] [SEC-PRISM]

Example:

```markdown
Constraints:
- Keep the answer under 500 words.
- Return no more than 5 bullet points.
- Use sources only when available.
- If information is insufficient, say so clearly.
```

---

## Tool-Call Budget Standard

For agent workflows, uncontrolled tool use can create cost and safety risk. Tool-augmented LLM agents need limits over tools, paths, outbound data, and audit trail. [SEC-PRISM]

| Tool/workflow | Control |
|---|---|
| Web search | Limit query count and require source summary |
| File reading | Chunk large files; do not read entire folders by default |
| Cron | Use isolated sessions and shorter outputs |
| Logs | Read targeted log ranges first |
| Multi-step agents | Require explicit stop condition |

---

## Cron Cost Guardrail

Before enabling a scheduled job, document:

- Frequency
- Model route
- Maximum expected output length
- Tool access allowed
- Source count or file count limit
- Fallback behavior
- Owner responsible for review

Recurring jobs can multiply cost quickly, especially when combined with web search, file reads, and retries. [NEWS-REUTERS] [SEC-PRISM]

Example command pattern:

```bash
# ใช้ model ที่ตรวจแล้วและจำกัด prompt ให้สั้นสำหรับ cron
openclaw cron add \
  --name "daily-brief-cost-safe" \
  --cron "0 8 * * *" \
  --tz "Asia/Bangkok" \
  --session isolated \
  --model "openai/<verified-economy-model-id>" \
  --message "$MSG"
```

---

## Prompt Caching Note

GPT-5.6 public materials describe platform-level improvements and model behavior that can affect cost/performance planning. Treat caching and billing assumptions as provider-side behavior that must be verified in the current account before relying on savings. [OA-GPT56] [NEWS-REUTERS]

Recommended practice:

- Keep reusable instruction blocks stable.
- Put frequently changing user data after stable instructions.
- Avoid rewriting the whole prompt template for every scheduled run.
- Still verify provider-side cache behavior and billing before relying on savings.

---

## Production Review Checklist

- [ ] Model route verified with `openclaw models list --provider openai` [OC-MODELS]
- [ ] Pricing checked on current OpenAI/provider billing page [NEWS-REUTERS]
- [ ] Prompt has output limit [NEWS-REUTERS]
- [ ] Tool call count is limited [SEC-PRISM]
- [ ] File scope is limited [SEC-PRISM]
- [ ] Cron frequency is justified [NEWS-REUTERS]
- [ ] Fallback does not silently downgrade high-risk tasks [OC-MODELS]
- [ ] Logs are reviewed after first production run [SEC-PRISM]
- [ ] Budget owner is defined

---

## Stop Conditions

Stop or pause the workflow if any of these occur:

- Unexpected cost spike
- Repeated provider errors
- Model route changes or disappears from catalog
- Output quality drops after model/tier update
- Cron job runs longer than expected
- Agent starts reading broader file scope than intended
- Logs contain secrets or personal data

---

## Reference Links

[OC-MODELS]: https://docs.openclaw.ai/cli/models
[OA-GPT56]: https://openai.com/index/gpt-5-6/
[OA-GPT56-HELP]: https://help.openai.com/en/articles/20001325-a-preview-of-gpt-56-sol-terra-and-luna
[NEWS-REUTERS]: https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/
[NEWS-AXIOS]: https://www.axios.com/2026/07/30/openai-cuts-prices-gpt-terra-luna5
[NEWS-BI]: https://www.businessinsider.com/openai-price-cuts-gpt-terra-luna-2026-7
[SEC-PRISM]: https://arxiv.org/abs/2603.11853
