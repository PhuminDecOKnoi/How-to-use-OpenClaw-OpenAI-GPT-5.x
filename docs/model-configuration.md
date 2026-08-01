# Model Configuration Guide

> แนวทางเชื่อม OpenAI provider, ตรวจ model catalog, ตั้งค่า primary model, fallback และ alias อย่างปลอดภัย สำหรับ OpenClaw + OpenAI GPT-5.x / GPT-5.6

---

## Read First

สำหรับมาตรฐานเต็มของ GPT-5.6 ให้ดูเอกสารนี้ก่อน:

- [`docs/openai-gpt56-operating-standard.md`](openai-gpt56-operating-standard.md)
- [`docs/cost-control.md`](cost-control.md)
- [`docs/external-research-july-2026.md`](external-research-july-2026.md)

---

## Login OpenAI Provider

```bash
# เชื่อม OpenAI provider ผ่าน OpenClaw
openclaw models auth login --provider openai
```

Alternative login patterns:

```bash
# ใช้ API-key method เมื่อรองรับในสภาพแวดล้อมนั้น
openclaw models auth login --provider openai --method api-key

# ใช้ device-code สำหรับเครื่อง headless หรือ remote terminal
openclaw models auth login --provider openai --device-code

# แยก profile สำหรับ training/demo/production
openclaw models auth login --provider openai --profile-id training-demo
```

---

## Check Model Availability

```bash
# ตรวจรายการ model ที่ provider และ account รองรับจริงในขณะนั้น
openclaw models list --provider openai

# ตรวจสถานะการตั้งค่า model ปัจจุบัน
openclaw models status

# ทดสอบเรียก model แบบ probe
openclaw models status --probe
```

> ใช้ model ID จากผลลัพธ์ของ `openclaw models list --provider openai` ก่อนตั้งค่าเสมอ อย่าอ้างอิงชื่อ model จากเอกสารเก่าโดยไม่ตรวจสอบ

---

## July 2026 GPT-5.6 Route Guidance

OpenClaw ใช้ provider namespace แบบ:

```text
openai/*
```

Route ที่อาจพบในเอกสารหรือ catalog ช่วง July 2026:

| Route pattern | Use only when | Notes |
|---|---|---|
| `openai/gpt-5.6` | ปรากฏใน catalog ของบัญชีจริง | ใช้เป็น route แบบรวม/ค่าเริ่มต้นได้เมื่อ verify แล้ว |
| `openai/gpt-5.6-sol` | ปรากฏใน catalog และงานต้องการ reasoning สูง | เหมาะกับงานยาก เช่น coding, audit-style reasoning, complex agents |
| `openai/gpt-5.6-terra` | ปรากฏใน catalog | เหมาะกับงานสมดุล quality/cost |
| `openai/gpt-5.6-luna` | ปรากฏใน catalog | เหมาะกับงานเร็ว เบา ปริมาณมาก และ cron |
| `openai/<verified-fallback-model-id>` | ตรวจแล้วว่า fallback เหมาะกับงาน | ไม่ควรปล่อยให้ downgrade โดยไม่ตั้งใจ |

> อย่าคัดลอก route จากตารางนี้ไปใช้ production โดยไม่ตรวจด้วย CLI ก่อน เพราะ access, rollout, account type และ auth method อาจไม่เหมือนกัน

---

## Set Primary Model

```bash
# ตั้ง primary model โดยแทน <verified-primary-model-id> ด้วย model ที่ตรวจพบจริง
openclaw models set "openai/<verified-primary-model-id>"
```

---

## Set Fallback Model

```bash
# ล้าง fallback เดิมก่อน เพื่อป้องกันการเรียก model ที่ไม่ต้องการ
openclaw models fallbacks clear

# ตั้ง fallback model สำหรับกรณี primary model ไม่พร้อมใช้งาน
openclaw models fallbacks add "openai/<verified-fallback-model-id>"
```

Fallback policy:

| Workload | Fallback rule |
|---|---|
| High-risk legal/audit analysis | Use fallback only if it preserves reasoning quality and source fidelity |
| Cron/daily summary | Use lower-cost fallback if output remains short and low-risk |
| Coding/review | Prefer Terra/Sol-class fallback, then require tests or verification |
| Public content generation | Use fallback only after checking tone and quality |

---

## Create Aliases

```bash
# สร้าง alias เพื่อให้เรียก model ได้ง่ายขึ้นในการสอนหรือ demo
openclaw models aliases add gpt-reasoning "openai/<verified-sol-or-terra-model-id>"
openclaw models aliases add gpt-economy "openai/<verified-luna-or-terra-model-id>"
openclaw models aliases add gpt-fallback "openai/<verified-fallback-model-id>"

# ตรวจรายการ alias ทั้งหมด
openclaw models aliases list
```

---

## Apply and Verify

```bash
# restart gateway หลังแก้ configuration
openclaw gateway restart

# probe เพื่อยืนยันว่าค่าใหม่ใช้งานได้จริง
openclaw models status --probe
```

---

## Model Strategy Checklist

- ใช้ model ID ที่ตรวจจาก catalog ปัจจุบัน
- แยก primary/fallback ให้ชัดเจน
- ใช้ Sol/Terra เฉพาะงานที่ต้องการ reasoning หรือคุณภาพสูง
- ใช้ Terra/Luna กับงานทั่วไป งาน cron และงานที่ต้องคุมต้นทุน
- จำกัด output และ tool calls เมื่อควบคุม cost
- ตรวจ pricing/quota ก่อนเปิด production หรือ scheduled jobs
- ทดสอบ probe หลังเปลี่ยน model ทุกครั้ง
- บันทึก model route และ auth profile ที่ใช้จริงใน runbook ภายใน

---

## Production Note

For July 2026 GPT-5.6 workflows, use this sequence:

```text
Verify catalog → choose model tier → set primary → set fallback → restart gateway → probe → document result
```
