# Troubleshooting Guide

> คู่มือแก้ปัญหาเบื้องต้นสำหรับ OpenClaw + OpenAI workflow

---

## First Checks

```bash
# ตรวจสุขภาพระบบ OpenClaw
openclaw doctor

# ตรวจสถานะ gateway
openclaw gateway status

# ตรวจ provider authentication
openclaw models auth list

# ตรวจ model และ probe
openclaw models status
openclaw models status --probe
```

---

## Common Issues

| Symptom | Likely Cause | Recommended Action |
|---|---|---|
| `401` or authentication error | API key missing, expired, or not loaded | Re-login provider and verify environment variables |
| Gateway not running | Service stopped or failed to start | Run `openclaw gateway status` then restart gateway |
| Unknown model | Model ID is wrong or stale | Run `openclaw models list --provider openai` and update model ref |
| Context overflow | Prompt, history, files, or output too large | Start new session and split task into smaller parts |
| Rate limit | Too many requests or provider limit | Wait, reduce workload, or switch model strategy |
| Web search disabled | Web provider not configured | Configure web provider before using web search tools |
| Cron output too long | Automation prompt is too broad | Limit output, results, and tool usage |

---

## Restart Flow

```bash
# restart gateway หลังแก้ configuration
openclaw gateway restart

# ตรวจซ้ำหลัง restart
openclaw gateway status
openclaw models status --probe
```

---

## Log Review

```bash
# ดู log แบบต่อเนื่องเพื่อหาจุดผิดพลาด
openclaw logs --follow
```

ก่อนส่ง log ให้ผู้อื่นตรวจ ต้อง sanitize ตาม `docs/security.md` ทุกครั้ง
