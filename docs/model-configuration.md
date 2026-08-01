# Model Configuration Guide

> แนวทางเชื่อม OpenAI provider, ตรวจ model catalog, ตั้งค่า primary model, fallback และ alias อย่างปลอดภัย สำหรับ OpenClaw + OpenAI GPT-5.x / GPT-5.6

---

## Read First

สำหรับมาตรฐานเต็มของ GPT-5.6 ให้ดูเอกสารนี้ก่อน:

- [`docs/openai-gpt56-operating-standard.md`](openai-gpt56-operating-standard.md)
- [`docs/cost-control.md`](cost-control.md)
- [`docs/references.md`](references.md)
- [`docs/external-research-july-2026.md`](external-research-july-2026.md)

---

## Source Basis

- OpenClaw ใช้ OpenAI provider ID `openai` และ model route pattern `openai/*` สำหรับ OpenAI model references. [OC-OPENAI]
- คำสั่ง `openclaw models auth`, `models list`, `models set`, `fallbacks`, และ `aliases` อยู่ในกลุ่ม OpenClaw Models CLI. [OC-MODELS]
- GPT-5.6 Sol/Terra/Luna เป็น model-family/tier ที่อ้างอิงจาก OpenAI official announcement และ Help Center preview. [OA-GPT56] [OA-GPT56-HELP]

---

## Login OpenAI Provider

```bash
# เชื่อม OpenAI provider ผ่าน OpenClaw
openclaw models auth login --provider openai
```

ใช้คำสั่ง login ผ่าน OpenClaw Models CLI และ provider ID `openai`. [OC-MODELS] [OC-OPENAI]

Alternative examples:

```bash
# ใช้ API-key auth method เมื่อ environment รองรับ
openclaw models auth login --provider openai --method api-key

# ใช้ device-code login สำหรับเครื่อง remote/headless
openclaw models auth login --provider openai --device-code

# ใช้ profile แยกสำหรับ training/demo/production
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

> ใช้ model ID จากผลลัพธ์ของ `openclaw models list --provider openai` ก่อนตั้งค่าเสมอ อย่าอ้างอิงชื่อ model จากเอกสารเก่าโดยไม่ตรวจสอบ. [OC-MODELS] [OC-PROVIDERS]

---

## GPT-5.6 Route Strategy

| Tier / route pattern | Use for | Verification rule | Source |
|---|---|---|---|
| `openai/gpt-5.6` | Default GPT-5.6 route pattern | Check account/runtime behavior before production | [OC-PROVIDERS] |
| `openai/gpt-5.6-sol` | Reasoning-heavy work | Use only if visible in live catalog | [OA-GPT56] [OC-MODELS] |
| `openai/gpt-5.6-terra` | Balanced professional work | Use only if visible in live catalog | [OA-GPT56] [OC-MODELS] |
| `openai/gpt-5.6-luna` | Lightweight/cost-sensitive work | Use only if visible in live catalog | [OA-GPT56] [OC-MODELS] |
| `openai/<fallback-model-id>` | Recovery/fallback | Select deliberately; do not allow silent downgrade | [OC-MODELS] |

---

## Set Primary Model

```bash
# ตั้ง primary model โดยแทน <verified-primary-model-id> ด้วย model ที่ตรวจพบจริง
openclaw models set "openai/<verified-primary-model-id>"
```

The `models set` pattern is part of OpenClaw Models CLI. [OC-MODELS]

---

## Set Fallback Model

```bash
# ล้าง fallback เดิมก่อน เพื่อป้องกันการเรียก model ที่ไม่ต้องการ
openclaw models fallbacks clear

# ตั้ง fallback model สำหรับกรณี primary model ไม่พร้อมใช้งาน
openclaw models fallbacks add "openai/<verified-fallback-model-id>"
```

Fallbacks should be explicit and verified. [OC-MODELS]

---

## Create Aliases

```bash
# สร้าง alias สำหรับงาน reasoning หนัก
openclaw models aliases add gpt-reasoning "openai/<verified-sol-or-terra-model-id>"

# สร้าง alias สำหรับงานสมดุลคุณภาพ/ต้นทุน
openclaw models aliases add gpt-balanced "openai/<verified-terra-model-id>"

# สร้าง alias สำหรับงานเบา/cron/cost-sensitive
openclaw models aliases add gpt-economy "openai/<verified-luna-or-terra-model-id>"

# ตรวจรายการ alias ทั้งหมด
openclaw models aliases list
```

Alias commands are managed through OpenClaw Models CLI. [OC-MODELS]

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

- [ ] ใช้ model ID ที่ตรวจจาก live catalog ปัจจุบัน [OC-MODELS]
- [ ] แยก primary/fallback ให้ชัดเจน [OC-MODELS]
- [ ] เลือก Sol/Terra/Luna ตาม risk, reasoning depth, cost, volume [OA-GPT56]
- [ ] ตรวจ account/organization approval เมื่อใช้ GPT-5.6 preview/rollout access [OA-GPT56-HELP]
- [ ] ใช้ model tier ที่ต่ำที่สุดที่ยังทำงานได้ปลอดภัยและแม่นยำ [NEWS-REUTERS]
- [ ] จำกัด output และ tool calls เมื่อควบคุม cost [NEWS-REUTERS]
- [ ] ทดสอบ probe หลังเปลี่ยน model ทุกครั้ง [OC-MODELS]

---

## Reference Links

[OC-OPENAI]: https://docs.openclaw.ai/providers/openai
[OC-MODELS]: https://docs.openclaw.ai/cli/models
[OC-PROVIDERS]: https://docs.openclaw.ai/concepts/model-providers
[OA-GPT56]: https://openai.com/index/gpt-5-6/
[OA-GPT56-HELP]: https://help.openai.com/en/articles/20001325-a-preview-of-gpt-56-sol-terra-and-luna
[NEWS-REUTERS]: https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/
