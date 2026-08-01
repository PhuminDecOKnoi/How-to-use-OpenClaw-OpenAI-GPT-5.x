# Security Guide

> เอกสารนี้เป็น checklist สำหรับป้องกัน API key, token, config และ log ไม่ให้รั่วไหลระหว่างใช้ OpenClaw + OpenAI

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

## Log Sanitization Checklist

ก่อนแชร์ log ให้ลบหรือ mask ข้อมูลเหล่านี้:

- API keys
- Tokens
- Chat IDs
- User IDs
- Local file paths ที่มีชื่อบุคคลหรือข้อมูลลูกค้า
- Prompt ที่มีข้อมูลลับ
- Output ที่มีข้อมูลส่วนบุคคล

---

## Rotation Rule

ถ้าสงสัยว่า key/token อาจรั่วไหล ให้ทำทันที:

1. Revoke หรือ rotate key เดิม
2. สร้าง key ใหม่
3. ตรวจ repository history และ logs
4. อัปเดตเครื่องหรือ secret manager ที่ใช้งานจริง
5. บันทึก incident note เพื่อป้องกันซ้ำ
