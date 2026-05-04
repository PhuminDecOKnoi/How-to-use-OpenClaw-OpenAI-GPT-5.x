# OpenClaw + OpenAI / GPT 5.x

คู่มือสรุปสาระสำคัญสำหรับติดตั้งและใช้งาน OpenClaw ร่วมกับ OpenAI / GPT 5.x เพื่อสร้าง AI Agent สำหรับงานทั่วไป เช่น สรุปข่าว สรุปเอกสาร เขียนอีเมล จัดหมวดข้อมูล และตั้งงานอัตโนมัติผ่าน Cron / Telegram

---

## 1. แนวคิดหลัก

```text
User / Telegram / Dashboard
        ↓
OpenClaw Gateway
        ↓
AI Agent
        ↓
OpenAI GPT 5.x
        ↓
Tools / Files / Web Search / Cron
```

OpenClaw คือระบบ AI Agent ที่รันบนเครื่องผู้ใช้ โดยเชื่อมต่อกับ Model Provider เช่น OpenAI และเครื่องมือเสริม เช่น Web Search, Files, Cron, Telegram และ Dashboard

---

## 2. Model Strategy ที่แนะนำ

```text
Primary   = openai/gpt-5.4-mini
Fallback  = openai/gpt-5.4-nano
```

| งาน | Model แนะนำ |
|---|---|
| งานทั่วไป / วิเคราะห์ / draft | `openai/gpt-5.4-mini` |
| งานเบา / Cron / classification | `openai/gpt-5.4-nano` |
| งานรายวัน | `openai/gpt-5.4-nano` |
| งานรายเดือน / รายงานละเอียด | `openai/gpt-5.4-mini` |

---

## 3. ติดตั้ง OpenClaw

### Installer Script

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### npm

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

ตรวจสอบ:

```bash
openclaw --version
openclaw doctor
openclaw gateway status
```

---

## 4. เปิด Dashboard / Restart Gateway

เปิด Dashboard:

```bash
openclaw dashboard
```

หรือ:

```bash
open http://127.0.0.1:18789
```

Restart Gateway:

```bash
openclaw gateway restart
```

> หมายเหตุ: ไม่มีคำสั่ง `openclaw restart` ให้ใช้ `openclaw gateway restart`

---

## 5. เชื่อม OpenAI API Key

```bash
openclaw models auth login --provider openai
```

ตรวจสอบ:

```bash
openclaw models status
openclaw models status --probe
```

ห้ามส่ง API Key / Token / Password / Secret ในแชตหรือเอกสารสาธารณะ

---

## 6. ตั้ง GPT เป็น Model หลัก

```bash
openclaw models set openai/gpt-5.4-mini

openclaw models fallbacks clear
openclaw models fallbacks add openai/gpt-5.4-nano

openclaw gateway restart
openclaw models status --probe
```

ผลที่ควรเห็น:

```text
Default   : openai/gpt-5.4-mini
Fallbacks : openai/gpt-5.4-nano
Probe     : ok
```

---

## 7. ตั้ง Alias

```bash
openclaw models aliases add gpt-mini openai/gpt-5.4-mini
openclaw models aliases add gpt-nano openai/gpt-5.4-nano
openclaw models aliases add GPT openai/gpt-5.4-mini
```

ตรวจสอบ:

```bash
openclaw models aliases list
```

---

## 8. ตั้งค่า Web Search

ใช้เมื่อ Agent ต้องค้นข้อมูลล่าสุดจากเว็บ

```bash
openclaw configure --section web
openclaw gateway restart
```

| Provider | เหมาะกับ |
|---|---|
| DuckDuckGo | ทดสอบเร็ว ไม่ต้องใช้ API Key |
| Brave | ใช้งานจริง เสถียรกว่า |
| Gemini Search | ต้องการ citation / grounding |

---

## 9. Cron Automation

ดูรายการ Cron:

```bash
openclaw cron list
```

รัน Cron:

```bash
openclaw cron run "<job-id>"
```

ดูประวัติ:

```bash
openclaw cron runs --id "<job-id>"
```

ปิด / เปิดงาน:

```bash
openclaw cron disable "<job-id>"
openclaw cron enable "<job-id>"
```

แก้ Model ของ Cron:

```bash
openclaw cron edit "<job-id>" --model openai/gpt-5.4-nano
```

---

## 10. ตัวอย่าง Cron: Daily News Brief

```bash
MSG=$(cat <<'EOF2'
ทำ Daily News Brief แบบสั้น

ค้นข่าวทั่วไปที่สำคัญใน 24 ชั่วโมงล่าสุด
จำกัดไม่เกิน 3 ข่าว
สรุปเป็นภาษาไทย
ข่าวละไม่เกิน 4 บรรทัด
ท้ายข้อความให้ถามว่า “ต้องการรายละเอียดข่าวใดเพิ่มเติมหรือไม่”
EOF2
)

openclaw cron add \
  --name "daily-general-news-brief" \
  --cron "0 8 * * *" \
  --tz "Asia/Bangkok" \
  --session isolated \
  --announce \
  --channel telegram \
  --to "<telegram-chat-id>" \
  --model openai/gpt-5.4-nano \
  --message "$MSG"
```

---

## 11. การใช้งานไฟล์เบื้องต้น

สร้างโฟลเดอร์:

```bash
mkdir -p "$HOME/AI-Agent-Lab/input"
mkdir -p "$HOME/AI-Agent-Lab/output"
```

อ่านไฟล์:

```bash
cat "$HOME/AI-Agent-Lab/input/sample.txt"
head -80 "$HOME/AI-Agent-Lab/input/sample.txt"
```

เขียนไฟล์ Markdown:

```bash
cat <<'EOF2' > "$HOME/AI-Agent-Lab/output/summary.md"
# Summary

This is a sample summary.
EOF2
```

Backup ก่อนแก้ไฟล์:

```bash
cp "$HOME/AI-Agent-Lab/output/summary.md" \
   "$HOME/AI-Agent-Lab/output/summary.backup.$(date +%Y%m%d-%H%M%S).md"
```

---

## 12. Prompt Pattern ที่แนะนำ

```text
บทบาท:
คุณคือ...

งาน:
ทำอะไร

ข้อมูล:
ข้อมูลที่ต้องใช้

ข้อจำกัด:
ความยาว / ห้ามทำอะไร / ใช้แหล่งใด

รูปแบบผลลัพธ์:
หัวข้อ / ตาราง / JSON / bullet
```

ตัวอย่าง:

```text
สรุปรายงานการประชุมต่อไปนี้
- ตอบภาษาไทย
- ไม่เกิน 500 คำ
- แยก Action Items
- ถ้าข้อมูลไม่พอ ให้ระบุว่า “ข้อมูลไม่เพียงพอ”
```

---

## 13. ควบคุมค่าใช้จ่าย

```text
ใช้ nano กับงานเบา
ใช้ mini กับงานละเอียด
จำกัด output
จำกัดจำนวนผลลัพธ์
ไม่อ่านไฟล์ยาวทั้งฉบับถ้าไม่จำเป็น
ไม่รัน Cron ซ้ำถี่
```

---

## 14. Rate Limit

ถ้าเจอ:

```text
Rate limit reached
```

ให้ทำ:

```bash
sleep 90
openclaw cron list
openclaw cron runs --id "<job-id>"
```

แนวทางลดปัญหา:

```text
ลด prompt
ลด output
ลดจำนวน tool call
แยก Cron ไม่ให้รันติดกัน
อย่ากด run ซ้ำหลายครั้ง
```

---

## 15. Context Overflow

เกิดเมื่อ:

```text
prompt + chat history + tool input + output budget > context limit
```

วิธีแก้:

```text
ใช้ /new ใน Telegram
ลด prompt
จำกัด output
ไม่อ่าน PDF เต็ม
ไม่ค้นหลายเว็บพร้อมกัน
แยกงานเป็น Discovery → Analysis → Record
```

---

## 16. Telegram Recovery

ถ้า Agent ค้างหรือขึ้น Something went wrong ให้ส่งใน Telegram:

```text
/new
```

แล้วทดสอบด้วยข้อความสั้น:

```text
ตรวจสถานะสั้น ๆ
```

---

## 17. Debug Commands

```bash
openclaw gateway status
openclaw models status --probe
openclaw cron list
openclaw cron runs --id "<job-id>"
openclaw logs --help
openclaw logs --follow
```

---

## 18. Security Checklist

```text
[ ] ไม่เปิดเผย API Key
[ ] ไม่เปิดเผย Telegram Bot Token
[ ] ไม่เปิดเผย Gateway Token
[ ] Backup ก่อนแก้ config
[ ] ตรวจ logs ก่อนส่งต่อ
[ ] ใช้คำสั่ง rm / sudo / chmod อย่างระมัดระวัง
```

Backup config:

```bash
cp "$HOME/.openclaw/openclaw.json" \
   "$HOME/.openclaw/openclaw.backup.$(date +%Y%m%d-%H%M%S).json"
```

---

## 19. Troubleshooting สั้น ๆ

| Error | สาเหตุ | วิธีแก้ |
|---|---|---|
| `401` | API Key ผิด | Login provider ใหม่ |
| `402` | Credit ไม่พอ | เติม credit / ลด token |
| `rate limit` | ใช้ token ต่อนาทีเกิน | รอ / ลด prompt |
| `context overflow` | prompt ใหญ่เกิน | ลด context / ใช้ `/new` |
| `web_search disabled` | ยังไม่ตั้ง Web Search | `openclaw configure --section web` |
| `unknown command restart` | ใช้คำสั่งผิด | ใช้ `openclaw gateway restart` |

---

## 20. Command Cheat Sheet

```bash
# System
openclaw doctor
openclaw gateway status
openclaw gateway restart
openclaw dashboard

# Models
openclaw models list --provider openai
openclaw models status
openclaw models status --probe
openclaw models set openai/gpt-5.4-mini
openclaw models fallbacks clear
openclaw models fallbacks add openai/gpt-5.4-nano
openclaw models aliases list

# Auth
openclaw models auth login --provider openai

# Cron
openclaw cron list
openclaw cron run "<job-id>"
openclaw cron runs --id "<job-id>"
openclaw cron disable "<job-id>"
openclaw cron enable "<job-id>"
openclaw cron edit "<job-id>" --model openai/gpt-5.4-nano

# Web Search
openclaw configure --section web
openclaw gateway restart

# Logs
openclaw logs --help
openclaw logs --follow
```

---

## สรุป

OpenClaw + OpenAI / GPT 5.x เหมาะสำหรับสร้าง AI Agent ใช้งานทั่วไป เช่น สรุปข่าว สรุปไฟล์ เขียนอีเมล จัดหมวดข้อมูล ทำรายงาน และตั้ง Automation

ค่าที่แนะนำ:

```text
Primary   = openai/gpt-5.4-mini
Fallback  = openai/gpt-5.4-nano
Cron เบา  = openai/gpt-5.4-nano
งานละเอียด = openai/gpt-5.4-mini
```

หลักปฏิบัติ:

```text
ตรวจสถานะก่อนแก้
backup ก่อนเปลี่ยน config
ไม่เปิดเผย secret
ใช้ model ให้เหมาะกับงาน
จำกัด prompt/output
ไม่รัน Cron ซ้ำถี่
ใช้ /new เมื่อ session ค้าง
```
