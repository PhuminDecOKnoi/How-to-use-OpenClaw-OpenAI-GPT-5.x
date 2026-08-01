# Cost Control Guide

> แนวทางควบคุมต้นทุนเมื่อใช้งาน OpenClaw ร่วมกับ OpenAI GPT-5.x / GPT-5.6 โดยเฉพาะงานที่มี tool calls, web search, file workflow และ cron automation

---

## Cost-Control Principle

```text
Pick the lowest-cost verified model tier that can still complete the task safely and accurately.
```

Do not choose the largest reasoning model for every job. Match the model to workload risk, depth, and volume.

---

## Model Tier Policy

| Workload | Preferred tier | Reason |
|---|---|---|
| One-off complex reasoning | Sol or Terra | Accuracy and reasoning quality matter more than cost |
| Routine professional writing | Terra | Balanced quality and cost |
| Daily brief / cron summary | Luna or Terra | Recurring jobs can accumulate cost quickly |
| Bulk classification | Luna | Short output and high volume |
| Code repair / architecture review | Sol or Terra | More reasoning and verification needed |
| Legal/audit-style analysis | Sol or Terra | Source fidelity and reasoning quality are important |

> Always verify the actual route and pricing before production use. Public prices and tier availability can change after publication.

---

## Output Budget Standard

Every prompt used in production or cron should include an output budget.

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

For agent workflows, uncontrolled tool use can create cost and safety risk.

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

GPT-5.6 public materials describe more predictable prompt-caching behavior. In OpenClaw workflows, this matters when repeated jobs reuse stable instruction blocks, templates, or system context.

Recommended practice:

- Keep reusable instruction blocks stable.
- Put frequently changing user data after stable instructions.
- Avoid rewriting the whole prompt template for every scheduled run.
- Still verify provider-side cache behavior and billing before relying on savings.

---

## Production Review Checklist

- [ ] Model route verified with `openclaw models list --provider openai`
- [ ] Pricing checked on current OpenAI/provider billing page
- [ ] Prompt has output limit
- [ ] Tool call count is limited
- [ ] File scope is limited
- [ ] Cron frequency is justified
- [ ] Fallback does not silently downgrade high-risk tasks
- [ ] Logs are reviewed after first production run
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
