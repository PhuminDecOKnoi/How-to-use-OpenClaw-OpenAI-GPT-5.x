# Security Guide

> เอกสารนี้เป็น checklist สำหรับป้องกัน API key, token, config, logs และ tool workflows ไม่ให้รั่วไหลหรือถูกใช้งานผิดพลาดระหว่างใช้ OpenClaw + OpenAI GPT-5.x

---

## Never Commit These Values

```text
API keys
OpenAI keys
Gateway tokens
Telegram bot tokens
Passwords
Session tokens
.env files with real values
Auth profiles
Logs containing secrets
Customer data
Private file paths
Unredacted prompt/output logs
```

---

## Use Environment Variables Safely

```bash
# ใช้ environment variable เฉพาะในเครื่อง local หรือ secret manager
export OPENAI_API_KEY="<replace-with-real-key-only-on-your-machine>"
```

> อย่าใส่ key จริงใน README, issue, PR, screenshot, slide หรือ chat สาธารณะ

---

## Use `.env.example` Only for Templates

ไฟล์ตัวอย่างควรมีเฉพาะ placeholder เช่น:

```bash
# ตัวอย่างเท่านั้น ไม่ใช่ key จริง
OPENAI_API_KEY="replace-me"
OPENCLAW_ENV="development"
```

---

## Agent Threat Model

OpenClaw-style AI agents can connect models, tools, files, web search, logs, external apps, and automation. This creates a broader attack surface than a simple chatbot.

| Threat | Risk | Mitigation |
|---|---|---|
| Prompt injection | External content may instruct the agent to ignore rules or reveal data | Treat external text as untrusted; require source review and tool limits |
| Credential leakage | Keys/tokens may appear in prompts, logs, files, screenshots, or examples | Mask secrets and rotate exposed credentials immediately |
| Unsafe tool execution | Agent may call tools beyond the intended task scope | Limit tools per workflow and require explicit operator approval for risky actions |
| Skill/plugin poisoning | A workflow or prompt template may include hidden malicious instructions | Review templates before reuse and store trusted templates in version control |
| Over-broad file access | Agent may read files/folders unrelated to the task | Use explicit file paths and chunking; avoid whole-folder reads by default |
| Cron amplification | A recurring job can multiply cost or repeated mistakes | Use isolated sessions, low-risk model tier, and strict output/tool budgets |
| Log exposure | Logs may contain prompts, user data, or provider responses | Sanitize before sharing and avoid uploading raw logs publicly |

---

## Tool Permission Standard

| Workflow | Default permission |
|---|---|
| Web search | Allowed only when current/external information is needed |
| File reading | Limit to named files or clearly scoped folders |
| File writing | Use branch/PR workflow for repository changes |
| Cron automation | Require output limit, model route, and owner |
| External integrations | Use least privilege and avoid broad account scopes |
| Logs | Read targeted ranges and redact before sharing |

---

## Log Sanitization Checklist

ก่อนแชร์ log ให้ลบหรือ mask ข้อมูลเหล่านี้:

- API keys
- Tokens
- Chat IDs
- User IDs
- Local file paths ที่มีชื่อบุคคลหรือข้อมูลลูกค้า
- Prompt ที่มีข้อมูลลับ
- Output ที่มีข้อมูลส่วนบุคคล
- Provider request IDs if they can reveal account context
- Full raw payloads unless the reviewer really needs them

---

## Production Security Checklist

- [ ] No real `.env` files committed
- [ ] No API keys/tokens in README/docs/examples
- [ ] Auth profiles are not exported into the repository
- [ ] Logs are sanitized before sharing
- [ ] Tool access is limited per workflow
- [ ] Cron jobs use isolated sessions
- [ ] External sources are treated as untrusted input
- [ ] Fallback model is deliberately selected
- [ ] Model pricing/quota is reviewed before recurring jobs
- [ ] Human approval is required for repository write/merge actions

---

## Rotation Rule

ถ้าสงสัยว่า key/token อาจรั่วไหล ให้ทำทันที:

1. Revoke หรือ rotate key เดิม
2. สร้าง key ใหม่
3. ตรวจ repository history และ logs
4. อัปเดตเครื่องหรือ secret manager ที่ใช้งานจริง
5. บันทึก incident note เพื่อป้องกันซ้ำ

---

## Incident Note Template

```markdown
# Security Incident Note

Date:
Detected by:
Affected workflow:
Potential exposure:
Immediate action taken:
Credential rotation status:
Repository/log cleanup status:
Prevention measure:
Owner:
```
