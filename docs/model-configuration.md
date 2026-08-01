# Model Configuration Guide

> แนวทางเชื่อม OpenAI provider, ตรวจ model catalog, ตั้งค่า primary model, fallback และ alias อย่างปลอดภัย

---

## Login OpenAI Provider

```bash
# เชื่อม OpenAI provider ผ่าน OpenClaw
openclaw models auth login --provider openai
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

## Set Primary Model

```bash
# ตั้ง primary model โดยแทน <model-id> ด้วย model ที่ตรวจพบจริง
openclaw models set "openai/<model-id>"
```

---

## Set Fallback Model

```bash
# ล้าง fallback เดิมก่อน เพื่อป้องกันการเรียก model ที่ไม่ต้องการ
openclaw models fallbacks clear

# ตั้ง fallback model สำหรับกรณี primary model ไม่พร้อมใช้งาน
openclaw models fallbacks add "openai/<fallback-model-id>"
```

---

## Create Aliases

```bash
# สร้าง alias เพื่อให้เรียก model ได้ง่ายขึ้นในการสอนหรือ demo
openclaw models aliases add gpt-primary "openai/<model-id>"
openclaw models aliases add gpt-fallback "openai/<fallback-model-id>"

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
- ใช้ model ขนาดเล็กกับงานเบาและ cron
- ใช้ model ที่เหมาะกับ reasoning เมื่องานต้องการวิเคราะห์
- จำกัด output และ tool calls เมื่อควบคุม cost
- ทดสอบ probe หลังเปลี่ยน model ทุกครั้ง
