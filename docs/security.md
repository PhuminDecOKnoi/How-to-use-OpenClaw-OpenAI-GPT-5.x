# Security Guide

> เอกสารนี้เป็น checklist สำหรับป้องกัน API key, token, config, logs และ tool workflows ไม่ให้รั่วไหลหรือถูกใช้งานผิดพลาดระหว่างใช้ OpenClaw + OpenAI GPT-5.x

---

## Source Basis

- OpenClaw OpenAI provider documentation supports OpenAI auth/profile/provider context. [OC-OPENAI]
- OpenClaw-style tool-augmented agents have documented risks around prompt injection, unsafe tool execution, credential leakage, path/tool controls, and audit controls. [SEC-PRISM]
- Practitioner security reporting also emphasizes config/credential exposure and spending limits for OpenClaw-style setups. [NEWS-SECURITY]

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

API keys, auth profiles, and provider credentials must not be stored in public repository files or screenshots. [OC-OPENAI] [NEWS-SECURITY]

---

## Use Environment Variables Safely

```bash
# ใช้ environment variable เฉพาะในเครื่อง local หรือ secret manager
export OPENAI_API_KEY="<replace-with-real-key-only-on-your-machine>"
```

> อย่าใส่ key จริงใน README, issue, PR, screenshot, slide หรือ chat สาธารณะ. [OC-OPENAI]

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

OpenClaw-style AI agents can connect models, tools, files, web search, logs, external apps, and automation. This creates a broader attack surface than a simple chatbot. [SEC-PRISM]

| Threat | Risk | Mitigation | Source |
|---|---|---|---|
| Prompt injection | External content may instruct the agent to ignore rules or reveal data | Treat external text as untrusted; require source review and tool limits | [SEC-PRISM] |
| Credential leakage | Keys/tokens may appear in prompts, logs, files, screenshots, or examples | Mask secrets and rotate exposed credentials immediately | [SEC-PRISM] [NEWS-SECURITY] |
| Unsafe tool execution | Agent may call tools beyond the intended task scope | Limit tools per workflow and require explicit operator approval for risky actions | [SEC-PRISM] |
| Skill/plugin poisoning | A workflow or prompt template may include hidden malicious instructions | Review templates before reuse and store trusted templates in version control | [SEC-PRISM] |
| Over-broad file access | Agent may read files/folders unrelated to the task | Use explicit file paths and chunking; avoid whole-folder reads by default | [SEC-PRISM] |
| Cron amplification | A recurring job can multiply cost or repeated mistakes | Use isolated sessions, low-risk model tier, and strict output/tool budgets | [SEC-PRISM] |
| Log exposure | Logs may contain prompts, user data, or provider responses | Sanitize before sharing and avoid uploading raw logs publicly | [SEC-PRISM] |

---

## Tool Permission Standard

| Workflow | Default permission | Source |
|---|---|---|
| Web search | Allowed only when current/external information is needed | [SEC-PRISM] |
| File reading | Limit to named files or clearly scoped folders | [SEC-PRISM] |
| File writing | Use branch/PR workflow for repository changes | [SEC-PRISM] |
| Cron automation | Require output limit, model route, and owner | [SEC-PRISM] |
| External integrations | Use least privilege and avoid broad account scopes | [SEC-PRISM] |
| Logs | Read targeted ranges and redact before sharing | [SEC-PRISM] |

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

Log sanitization and outbound secret-pattern controls are part of the security-control approach for tool-augmented agents. [SEC-PRISM]

---

## Production Security Checklist

- [ ] No real `.env` files committed [OC-OPENAI]
- [ ] No API keys/tokens in README/docs/examples [OC-OPENAI]
- [ ] Auth profiles are not exported into the repository [OC-OPENAI]
- [ ] Logs are sanitized before sharing [SEC-PRISM]
- [ ] Tool access is limited per workflow [SEC-PRISM]
- [ ] Cron jobs use isolated sessions [SEC-PRISM]
- [ ] External sources are treated as untrusted input [SEC-PRISM]
- [ ] Fallback model is deliberately selected [OC-MODELS]
- [ ] Model pricing/quota is reviewed before recurring jobs [NEWS-REUTERS]
- [ ] Human approval is required for repository write/merge actions

---

## Rotation Rule

ถ้าสงสัยว่า key/token อาจรั่วไหล ให้ทำทันที:

1. Revoke หรือ rotate key เดิม
2. สร้าง key ใหม่
3. ตรวจ repository history และ logs
4. อัปเดตเครื่องหรือ secret manager ที่ใช้งานจริง
5. บันทึก incident note เพื่อป้องกันซ้ำ

Credential rotation is a practical response when keys or profiles may have been exposed. [NEWS-SECURITY]

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

---

## Reference Links

[OC-OPENAI]: https://docs.openclaw.ai/providers/openai
[OC-MODELS]: https://docs.openclaw.ai/cli/models
[NEWS-REUTERS]: https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/
[SEC-PRISM]: https://arxiv.org/abs/2603.11853
[NEWS-SECURITY]: https://www.techradar.com/pro/here-are-the-openclaw-security-risks-you-should-know-about
