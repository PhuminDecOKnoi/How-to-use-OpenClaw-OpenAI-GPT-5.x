# บทเรียน: การใช้งาน OpenClaw + OpenAI GPT-5.x / GPT-5.6

> เวอร์ชันเอกสาร: v1.1  
> สถานะ: ปรับให้สอดคล้องกับ `README.md` และมาตรฐาน GPT-5.6 Operating Standard  
> รูปแบบ: บทเรียนภาษาไทยสำหรับสอน / workshop / self-study / runbook  
> กลุ่มเป้าหมาย: ผู้เริ่มต้นถึงระดับปฏิบัติการ  
> หลักสำคัญ: **ตรวจสอบ model catalog ก่อนตั้งค่าเสมอ ไม่ hardcode model ID / ราคา / สิทธิ์การใช้งานจากความจำ**

---

## แผนที่เอกสารที่ควรอ่านประกอบ

| เอกสาร | ใช้เพื่อ |
|---|---|
| [`README.md`](README.md) | ภาพรวม repo, architecture, quick start และ documentation map |
| [`docs/architecture.md`](docs/architecture.md) | เข้าใจ flow: Channel → Gateway → Agent Session → Model → Tools → Output |
| [`docs/installation.md`](docs/installation.md) | ติดตั้ง OpenClaw และตรวจ gateway/dashboard |
| [`docs/model-configuration.md`](docs/model-configuration.md) | ตั้ง OpenAI provider, primary model, fallback และ alias |
| [`docs/openai-gpt56-operating-standard.md`](docs/openai-gpt56-operating-standard.md) | มาตรฐาน OpenClaw + OpenAI GPT-5.6 จากข้อมูลภายนอก July 2026 |
| [`docs/cost-control.md`](docs/cost-control.md) | ควบคุมต้นทุน model tier, output budget, tool calls และ cron |
| [`docs/security.md`](docs/security.md) | token hygiene, agent threat model, tool permissions และ log sanitization |
| [`docs/cron-automation.md`](docs/cron-automation.md) | ออกแบบ scheduled job แบบ cost-safe และ risk-aware |
| [`docs/external-research-july-2026.md`](docs/external-research-july-2026.md) | บันทึกแหล่งข้อมูลภายนอกและเหตุผลการปรับมาตรฐาน |

---

## ผลลัพธ์การเรียนรู้

เมื่อเรียนจบบทเรียนนี้ ผู้เรียนควรสามารถ:

1. อธิบายบทบาทของ OpenClaw ในฐานะ AI-agent gateway/orchestration layer ได้
2. อธิบายความสัมพันธ์ระหว่าง OpenClaw, OpenAI provider, GPT-5.x / GPT-5.6 และ tools ได้
3. ติดตั้งและตรวจสอบ OpenClaw Gateway / Dashboard ได้
4. Login OpenAI provider ผ่าน OpenClaw ได้อย่างปลอดภัย
5. ตรวจ model catalog ด้วย `openclaw models list --provider openai` ก่อนตั้งค่าได้
6. เลือก model tier แบบ Sol / Terra / Luna ให้เหมาะกับงาน ความเสี่ยง และต้นทุนได้
7. ตั้ง primary model, fallback model และ alias โดยใช้ `<verified-model-id>` ได้
8. ออกแบบ prompt, web search, file workflow และ cron automation แบบจำกัด scope ได้
9. ใช้ security checklist เพื่อป้องกัน API key, token, logs และ prompt/output leakage ได้
10. แก้ปัญหาเบื้องต้น เช่น auth error, model not found, rate limit, context overflow และ cron failure ได้

---

# บทที่ 1: OpenClaw คืออะไร

## 1.1 ความหมาย

OpenClaw คือระบบ AI Agent / Gateway ที่ทำหน้าที่เชื่อมต่อระหว่างผู้ใช้ ช่องทางสื่อสาร model provider เครื่องมือเสริม และงาน automation

ใน repository นี้ให้มอง OpenClaw เป็น **ตัวกลางควบคุมงาน** ไม่ใช่ตัว model โดยตรง

```text
User / Trainer / Operator
        ↓
Channel Layer: Dashboard / Telegram / CLI
        ↓
OpenClaw Gateway
        ↓
Agent Session
        ↓
OpenAI GPT-5.x / GPT-5.6 Provider
        ↓
Tools Layer: Web Search / Files / Cron / Logs
        ↓
Output: Summary / Report / Action-ready response
```

ดูภาพ workflow แบบ dark SVG ได้ใน [`README.md`](README.md) และ [`docs/architecture.md`](docs/architecture.md)

## 1.2 จุดเด่นที่ควรสอนผู้เรียน

| ความสามารถ | คำอธิบาย | จุดควบคุม |
|---|---|---|
| Gateway | จัดการ routing, session, provider และ tools | ตรวจ `openclaw gateway status` |
| Dashboard | ใช้ browser ควบคุม agent และตรวจสถานะ | เปิดด้วย `openclaw dashboard` |
| Model Provider | เชื่อม OpenAI ผ่าน provider namespace `openai/*` | ตรวจ catalog ก่อนตั้งค่า |
| Tool Use | ใช้ web search, files, cron, logs หรือ integration อื่น | จำกัด scope และ permission |
| Cron Automation | ตั้งงานอัตโนมัติรายวัน/รายสัปดาห์ | ใช้ isolated session และ output budget |
| Security | ป้องกัน key/token/logs รั่วไหล | ใช้ checklist จาก `docs/security.md` |

---

# บทที่ 2: OpenAI GPT-5.x / GPT-5.6 ในบริบท OpenClaw

## 2.1 คำจำกัดความในบทเรียนนี้

ในบทเรียนนี้คำว่า **GPT-5.x / GPT-5.6** หมายถึง model family/tier ของ OpenAI ที่ใช้งานผ่าน OpenClaw provider namespace:

```text
openai/*
```

ตัวอย่าง pattern ที่ใช้ในเอกสารนี้:

```text
openai/<verified-model-id>
openai/gpt-5.6
openai/gpt-5.6-sol
openai/gpt-5.6-terra
openai/gpt-5.6-luna
```

> หมายเหตุสำคัญ: ตัวอย่าง route ข้างต้นเป็น operating pattern ตามเอกสาร repo นี้ ผู้ใช้ต้องตรวจว่าบัญชี/region/runtime ของตนเห็น model จริงหรือไม่ด้วยคำสั่ง `openclaw models list --provider openai` ก่อนตั้งค่า production ทุกครั้ง

## 2.2 เปลี่ยนจาก mini/nano เป็น Sol/Terra/Luna

บทเรียนเวอร์ชันก่อนหน้าเคยใช้แนวคิด `mini/nano` และตัวอย่าง `gpt-5.4-mini/nano` ซึ่งไม่สอดคล้องกับ README ล่าสุดแล้ว

มาตรฐานใหม่ของ repo นี้คือ:

| Tier | เหมาะกับ | หลีกเลี่ยงเมื่อ |
|---|---|---|
| GPT-5.6 Sol | งาน reasoning หนัก, coding, legal/audit-style analysis, complex agents | งาน cron ปริมาณมากหรืองานสั้นที่ต้องคุมต้นทุน |
| GPT-5.6 Terra | งานเอกสาร งานสอน งาน professional ทั่วไป และ web/search summary | งานที่ต้องการ reasoning สูงสุดหรือ ultra-low-cost volume |
| GPT-5.6 Luna | งานสั้น งาน classification งาน cron งานสรุปเบา | งาน high-stakes reasoning หรือวิเคราะห์เอกสารซับซ้อน |

## 2.3 กฎหลักของบทเรียน

```text
Verify catalog → choose model tier → set primary → set fallback → restart gateway → probe → document result
```

ผู้เรียนต้องจำว่า:

```text
ห้าม hardcode model ID, ราคา, quota หรือสิทธิ์การใช้งาน โดยไม่ตรวจ catalog/account access ก่อน
```

---

# บทที่ 3: เตรียมเครื่องก่อนติดตั้ง

## 3.1 ตรวจ Node.js และ npm

```bash
# ตรวจ Node.js version
node --version

# ตรวจ npm version
npm --version
```

ถ้ายังไม่มี Node.js บน macOS สามารถติดตั้งด้วย Homebrew:

```bash
# ติดตั้ง Node.js ผ่าน Homebrew
brew install node
```

## 3.2 ตรวจ shell

```bash
# ตรวจ shell ปัจจุบัน
 echo $SHELL
```

ตัวอย่างผลลัพธ์ที่พบบ่อยบน macOS:

```console
/bin/zsh
```

## 3.3 สร้าง workspace สำหรับ workshop

```bash
# สร้าง workspace สำหรับทดลอง AI Agent
mkdir -p "$HOME/AI-Agent-Lab/input"
mkdir -p "$HOME/AI-Agent-Lab/output"
mkdir -p "$HOME/AI-Agent-Lab/templates"
mkdir -p "$HOME/AI-Agent-Lab/logs"
mkdir -p "$HOME/AI-Agent-Lab/archive"

# ตรวจโครงสร้าง workspace
ls -la "$HOME/AI-Agent-Lab"
```

---

# บทที่ 4: ติดตั้งและตรวจ OpenClaw

## 4.1 ติดตั้งด้วย Installer Script

```bash
# ติดตั้ง OpenClaw ด้วย installer script
curl -fsSL https://openclaw.ai/install.sh | bash
```

## 4.2 ติดตั้งด้วย npm

```bash
# ติดตั้ง OpenClaw เป็น global package
npm install -g openclaw@latest

# เริ่ม onboarding และติดตั้ง daemon/service
openclaw onboard --install-daemon
```

## 4.3 ตรวจการติดตั้ง

```bash
# ตรวจ version และสุขภาพระบบ
openclaw --version
openclaw doctor

# ตรวจสถานะ gateway
openclaw gateway status
```

ตัวอย่างผลลัพธ์ที่ต้องการ:

```console
Gateway: running
Dashboard: http://127.0.0.1:18789/
Connectivity probe: ok
```

---

# บทที่ 5: Gateway และ Dashboard

## 5.1 Restart Gateway

```bash
# restart gateway หลังเปลี่ยน provider/model/tool configuration
openclaw gateway restart
```

> ไม่ใช้ `openclaw restart` หากคำสั่งนั้นไม่ได้รองรับใน version ที่ใช้งาน

## 5.2 เปิด Dashboard

```bash
# เปิด Dashboard ผ่าน OpenClaw CLI
openclaw dashboard
```

macOS local URL option:

```bash
# เปิด local dashboard ผ่าน browser โดยตรง
open http://127.0.0.1:18789
```

## 5.3 จุดตรวจที่ควรอธิบายใน workshop

| จุดตรวจ | ความหมาย |
|---|---|
| Gateway running | OpenClaw service พร้อมรับงาน |
| Connectivity probe ok | CLI ติดต่อ gateway ได้ |
| Dashboard URL | UI พร้อมใช้งาน |
| Logs | ใช้ตรวจปัญหาเมื่อ command/probe fail |

---

# บทที่ 6: เชื่อม OpenAI Provider อย่างปลอดภัย

## 6.1 หลักความปลอดภัยก่อน login

ห้ามเผยแพร่ข้อมูลต่อไปนี้ใน README, issue, PR, screenshot, slide, chat หรือ shared terminal:

```text
OpenAI API key
Gateway token
Telegram bot token
OAuth token
Password
.env file with real values
Auth profile
Raw logs containing secrets
Customer/student/private data
```

## 6.2 Login OpenAI provider

```bash
# เชื่อม OpenAI provider ผ่าน OpenClaw
openclaw models auth login --provider openai
```

รูปแบบอื่นที่อาจใช้ได้ตาม environment:

```bash
# Login ด้วย API key method เมื่อรองรับ
openclaw models auth login --provider openai --method api-key

# Login จากเครื่อง headless หรือ remote terminal
openclaw models auth login --provider openai --device-code

# แยก profile สำหรับ training/demo/production
openclaw models auth login --provider openai --profile-id training-demo
```

## 6.3 ตรวจ auth profile

```bash
# ตรวจ OpenAI auth profile ที่ใช้งานได้
openclaw models auth list --provider openai
```

## 6.4 Probe provider/model

```bash
# ตรวจสถานะ model และทดสอบการเรียกใช้งาน
openclaw models status
openclaw models status --probe
```

ถ้าพบ error ให้ดูบทที่ 18 และ [`docs/troubleshooting.md`](docs/troubleshooting.md)

---

# บทที่ 7: ตรวจ Model Catalog และเลือก Tier

## 7.1 ตรวจ model catalog ก่อนเสมอ

```bash
# ตรวจ model ที่ OpenAI provider และ account นี้ใช้งานได้จริง
openclaw models list --provider openai
```

ให้ผู้เรียนบันทึกผลลัพธ์ที่เห็นจริง เช่น:

```text
Available OpenAI routes found in this account:
- openai/<verified-model-id-1>
- openai/<verified-model-id-2>
- openai/<verified-model-id-3>
```

## 7.2 Decision Rule สำหรับเลือก tier

| คำถาม | ถ้าใช่ | Model tier ที่ควรพิจารณา |
|---|---|---|
| งานต้องใช้ reasoning ลึกหรือ high-stakes หรือไม่ | ใช่ | Sol หรือ Terra |
| งานเป็นเอกสาร/สรุป/งานสอนทั่วไปหรือไม่ | ใช่ | Terra |
| งานสั้น รันซ้ำ หรือปริมาณมากหรือไม่ | ใช่ | Luna หรือ Terra |
| งานเป็น cron ที่รันทุกวันหรือไม่ | ใช่ | Luna/Terra พร้อม output budget |
| งานเกี่ยวกับกฎหมาย/audit/source fidelity หรือไม่ | ใช่ | Sol/Terra พร้อม citation/source check |

## 7.3 ห้ามใช้ชื่อ model จากความจำ

ไม่ควรสอนแบบนี้:

```bash
# ไม่แนะนำ: hardcode model โดยไม่ตรวจ catalog
openclaw models set openai/gpt-5.6-sol
```

ควรสอนแบบนี้:

```bash
# แนะนำ: ตรวจ catalog ก่อน แล้วแทนค่าด้วย model ที่พบจริง
openclaw models list --provider openai
openclaw models set "openai/<verified-primary-model-id>"
```

---

# บทที่ 8: ตั้ง Primary Model และ Fallback

## 8.1 ตั้ง primary model

```bash
# ตั้ง primary model จาก route ที่ตรวจพบจริง
openclaw models set "openai/<verified-primary-model-id>"
```

ตัวอย่างแนวคิด:

```text
งานสอนทั่วไป      → primary อาจเป็น Terra
งาน reasoning หนัก → primary อาจเป็น Sol
งาน cron เบา      → primary อาจเป็น Luna หรือ Terra
```

## 8.2 ตั้ง fallback อย่างตั้งใจ

```bash
# ล้าง fallback เดิมก่อน เพื่อป้องกัน fallback ที่ไม่ตั้งใจ
openclaw models fallbacks clear

# ตั้ง fallback ด้วย model ที่ตรวจพบจริง
openclaw models fallbacks add "openai/<verified-fallback-model-id>"
```

## 8.3 Fallback policy

| งาน | Fallback ที่เหมาะ | หมายเหตุ |
|---|---|---|
| งานสอน/demo | model ที่ถูกกว่าได้ | ยอมรับคุณภาพลดลงเล็กน้อย |
| งานเอกสารทั่วไป | tier ใกล้เคียงหรือเล็กกว่า | ต้องตรวจ output |
| งาน legal/audit/coding สำคัญ | tier ใกล้เคียงเท่านั้น | หลีกเลี่ยง silent downgrade |
| Cron high-risk | อาจ disable fallback | ลดความเสี่ยงของ output คุณภาพต่ำ |

## 8.4 Restart และ probe

```bash
# restart gateway เพื่อให้ configuration ใหม่มีผล
openclaw gateway restart

# ตรวจสถานะและ probe
openclaw models status
openclaw models status --probe
```

ตัวอย่างบันทึกผลสำหรับ workshop:

```text
Primary model: openai/<verified-primary-model-id>
Fallback model: openai/<verified-fallback-model-id>
Probe result: ok / failed
Checked by:
Checked at:
```

---

# บทที่ 9: Alias Standard สำหรับการสอน

## 9.1 เหตุผลที่ควรใช้ alias

Alias ทำให้ผู้เรียนเข้าใจง่ายขึ้น โดยไม่ต้องจำ model ID จริงทุกครั้ง และช่วยให้เอกสาร workshop ใช้ซ้ำได้

## 9.2 Alias ที่แนะนำ

```bash
# Alias สำหรับงาน reasoning หนัก
openclaw models aliases add gpt-reasoning "openai/<verified-sol-or-terra-model-id>"

# Alias สำหรับงานสมดุล คุณภาพ/ต้นทุน
openclaw models aliases add gpt-balanced "openai/<verified-terra-model-id>"

# Alias สำหรับงานเบา/cron/cost-sensitive
openclaw models aliases add gpt-economy "openai/<verified-luna-or-terra-model-id>"

# ตรวจ alias ทั้งหมด
openclaw models aliases list
```

## 9.3 แก้ alias ที่ผิด

```bash
# ลบ alias ที่ตั้งผิด แล้วเพิ่มใหม่
openclaw models aliases remove gpt-economy
openclaw models aliases add gpt-economy "openai/<verified-luna-or-terra-model-id>"
```

ถ้าเจอ `Alias not found` ให้ถือว่า alias นั้นยังไม่มี ไม่ใช่ปัญหาร้ายแรง

---

# บทที่ 10: Model Strategy สำหรับงานจริง

## 10.1 Strategy ตาม README ล่าสุด

| Use case | Tier strategy | Control |
|---|---|---|
| Daily Brief | Luna/Terra | จำกัด source และ output |
| Document Summary | Terra | chunk ไฟล์ ไม่อ่านทั้งไฟล์ยาวโดยไม่จำเป็น |
| Classification | Luna | output เป็น JSON/table และจำกัดคำอธิบาย |
| Web Search Agent | Terra/Luna | จำกัด query และตรวจ source |
| Cron Automation | Luna/Terra | isolated session + output budget |
| Telegram Agent | Luna/Terra | reset session เมื่อ context ใหญ่ |
| Legal / Audit-style Analysis | Sol/Terra | ต้อง preserve source basis และ citation |
| Coding / Review | Sol/Terra | ต้องมี verification/test step |

## 10.2 หลักเลือก model แบบมืออาชีพ

```text
ใช้ model ใหญ่เมื่อความเสี่ยงสูง
ใช้ model กลางเมื่อความสมดุลสำคัญ
ใช้ model เล็กเมื่อเป็นงานสั้น/ซ้ำ/ปริมาณมาก
ตรวจผลลัพธ์ทุกครั้งก่อนใช้จริง
```

---

# บทที่ 11: Prompt Design สำหรับ GPT-5.x / GPT-5.6

## 11.1 โครงสร้าง prompt ที่ควรสอน

```text
Role:
ระบุบทบาทของ AI

Task:
ระบุงานที่ต้องทำ

Context:
ให้ข้อมูลที่จำเป็นเท่านั้น

Constraints:
จำกัดความยาว, source, tool use, format, language

Output format:
ระบุรูปแบบผลลัพธ์ เช่น table, JSON, bullet, summary

Quality rule:
ถ้าข้อมูลไม่พอ ให้แจ้งว่าไม่พอ อย่าเดา
```

## 11.2 ตัวอย่าง prompt: executive summary

```markdown
Role:
คุณคือผู้ช่วยสรุปรายงานการประชุมสำหรับผู้บริหาร

Task:
สรุปข้อความประชุมด้านล่างให้เป็น executive summary

Constraints:
- ตอบภาษาไทย
- ไม่เกิน 500 คำ
- แยก Action Items ให้ชัด
- ถ้าข้อมูลไม่พอ ให้ระบุว่า “ข้อมูลไม่เพียงพอ”
- ห้ามเพิ่มข้อเท็จจริงที่ไม่มีในข้อมูล

Output format:
1) สรุปภาพรวม
2) ประเด็นสำคัญ
3) Action Items
4) ความเสี่ยง/ข้อควรติดตาม
```

## 11.3 ตัวอย่าง prompt: classification แบบควบคุม output

```markdown
จัดหมวดข้อความต่อไปนี้เป็นหนึ่งในหมวด:
Billing, Technical, Account, Feature Request, Other

ให้ตอบเป็น JSON เท่านั้น:
{
  "category": "",
  "confidence": 0-1,
  "reason": ""
}

ข้อความ:
"ฉันล็อกอินไม่ได้หลังจากเปลี่ยนเบอร์โทรศัพท์"
```

## 11.4 Output budget

ทุก prompt สำหรับ production หรือ cron ควรมี output budget เช่น:

```text
- ตอบไม่เกิน 500 คำ
- แสดงไม่เกิน 5 bullet points
- ไม่ใช้ตารางถ้าไม่จำเป็น
- ถ้า source ไม่พอ ให้แจ้งข้อจำกัด
```

---

# บทที่ 12: ใช้ Web Search อย่างปลอดภัย

## 12.1 ตั้งค่า Web Search

```bash
# ตั้งค่า web provider ตาม environment ที่ใช้งาน
openclaw configure --section web

# restart gateway หลังตั้งค่า
openclaw gateway restart
```

## 12.2 Prompt ตัวอย่าง: Daily Technology Brief

```markdown
Task:
ค้นข่าวเทคโนโลยีสำคัญใน 24 ชั่วโมงล่าสุด

Constraints:
- จำกัดไม่เกิน 3 ข่าว
- สรุปข่าวละไม่เกิน 4 บรรทัด
- ระบุแหล่งที่มาเมื่อมี source
- ถ้า web search ใช้งานไม่ได้ ให้แจ้งข้อจำกัด
- อย่าอ่าน PDF เต็มฉบับถ้าไม่จำเป็น

Output:
1) ข่าวสำคัญ
2) ผลกระทบต่อผู้ใช้งานทั่วไป
3) Source list
```

## 12.3 Web Search Guardrails

```text
จำกัดจำนวน query
จำกัดจำนวน source
ตรวจวันเผยแพร่และวันที่เกิดเหตุ
แยก official source ออกจาก news/community source
ไม่ใช้ community post เป็น source หลักสำหรับ config/commands
```

---

# บทที่ 13: File Workflow สำหรับงานสรุปเอกสาร

## 13.1 สร้าง folder งาน

```bash
# สร้าง folder สำหรับ input/output/archive
mkdir -p "$HOME/AI-Agent-Lab/input"
mkdir -p "$HOME/AI-Agent-Lab/output"
mkdir -p "$HOME/AI-Agent-Lab/archive"
```

## 13.2 อ่านไฟล์แบบจำกัด scope

```bash
# อ่านเฉพาะต้นไฟล์ ไม่อ่านทั้งไฟล์ยาวโดยไม่จำเป็น
head -80 "$HOME/AI-Agent-Lab/input/sample.txt"
```

## 13.3 เขียน Markdown output

```bash
# สร้างไฟล์ Markdown output
cat <<'EOF' > "$HOME/AI-Agent-Lab/output/summary.md"
# Summary

This is a sample summary generated through a controlled OpenClaw workflow.
EOF
```

## 13.4 Backup ก่อนแก้ไฟล์

```bash
# backup ก่อนแก้ไขไฟล์สำคัญ
cp "$HOME/AI-Agent-Lab/output/summary.md" \
   "$HOME/AI-Agent-Lab/output/summary.backup.$(date +%Y%m%d-%H%M%S).md"
```

## 13.5 หลัก chunking

```text
ถ้าไฟล์ยาว ให้แบ่งเป็นส่วนย่อย
สรุปทีละส่วน
บันทึก finding/source basis
ค่อยทำ synthesis ตอนท้าย
```

---

# บทที่ 14: Cron Automation แบบ Cost-safe

## 14.1 Cron คืออะไร

Cron คือระบบตั้งเวลารันงานอัตโนมัติ เช่น:

```text
ทุกวัน 08:00 ส่งสรุปข่าว
ทุกวันศุกร์ 17:00 สรุปงานประจำสัปดาห์
ทุกเดือนวันที่ 1 สร้างรายงานค่าใช้จ่าย
```

## 14.2 Preflight ก่อนเปิด cron

```bash
# ตรวจ provider/model/gateway ก่อนเปิด cron
openclaw models auth list --provider openai
openclaw models list --provider openai
openclaw models status --probe
openclaw gateway status
```

## 14.3 สร้าง prompt variable

```bash
# เก็บ prompt ในตัวแปร MSG เพื่อให้อ่านง่ายและแก้ไขง่าย
MSG=$(cat <<'EOF'
Create a lightweight daily brief.

Scope:
- Use only important items.
- Limit the result to three items.
- Keep the response under 500 words.
- Do not read full PDFs.
- If information is insufficient, say so clearly.

Controls:
- Do not include secrets, tokens, or private data.
- Do not expand the scope beyond the requested brief.
- Use sources only when available.

Output format:
1) Overall status
2) Key items
3) Short impact notes
4) Sources, if available
EOF
)
```

## 14.4 เพิ่ม cron job

```bash
# ใช้ model ที่ตรวจแล้วว่าเหมาะกับงาน recurring/cost-sensitive
openclaw cron add \
  --name "daily-lightweight-brief" \
  --cron "0 8 * * *" \
  --tz "Asia/Bangkok" \
  --session isolated \
  --model "openai/<verified-economy-model-id>" \
  --message "$MSG"
```

## 14.5 ตรวจ cron

```bash
# ดูรายการ cron job ทั้งหมด
openclaw cron list

# ทดสอบ run เฉพาะ job
openclaw cron run "<job-id>"

# ดูประวัติการ run
openclaw cron runs --id "<job-id>"
```

## 14.6 Disable / Enable

```bash
# ปิด job ที่มีปัญหาก่อนแก้ไข
openclaw cron disable "<job-id>"

# เปิดใช้งานเมื่อแก้ไขแล้ว
openclaw cron enable "<job-id>"
```

---

# บทที่ 15: ตัวอย่าง Automation ตาม Use Case

## 15.1 Daily Personal Brief

```text
ทุกวัน 07:00
ส่งสรุป:
- ข่าวสำคัญไม่เกิน 3 เรื่อง
- งานที่ควรทำวันนี้
- ความเสี่ยง/ข้อควรติดตาม
- source ถ้ามี
```

Model strategy:

```text
Luna/Terra หรือ <verified-economy-model-id>
```

## 15.2 Weekly Project Summary

```text
ทุกวันศุกร์ 17:00
อ่าน notes ที่กำหนดไว้เท่านั้น
สรุป:
1) งานที่เสร็จแล้ว
2) งานที่ยังค้าง
3) ความเสี่ยง
4) แผนสัปดาห์ถัดไป
```

Model strategy:

```text
Terra หรือ <verified-balanced-model-id>
```

## 15.3 Legal / Audit-style Review

```text
สรุป finding หรือ issue โดยต้อง preserve source basis
แยก:
1) Facts from source
2) Analysis
3) Risk
4) Recommended action
5) Evidence to request
```

Model strategy:

```text
Sol/Terra หรือ <verified-reasoning-model-id>
```

Control:

```text
ห้ามสรุปเกิน source
ต้องระบุข้อจำกัดเมื่อ source ไม่พอ
ต้องมี human review ก่อนใช้จริง
```

## 15.4 Customer Feedback Classifier

```text
ทุกวัน 18:00
อ่าน feedback ใหม่เฉพาะไฟล์ที่กำหนด
จัดหมวดเป็น:
- Complaint
- Feature Request
- Praise
- Bug
- Other
บันทึกเป็น CSV
```

Model strategy:

```text
Luna หรือ <verified-economy-model-id>
```

---

# บทที่ 16: Cost Control

## 16.1 หลักคิด

ต้นทุนขึ้นกับ:

```text
input tokens + output tokens + model tier + tool calls + frequency + retries
```

## 16.2 Policy สั้น ๆ

```text
Pick the lowest-cost verified model tier that can still complete the task safely and accurately.
```

## 16.3 เลือก tier ตาม workload

| Workload | Preferred tier | เหตุผล |
|---|---|---|
| One-off complex reasoning | Sol/Terra | ความถูกต้องสำคัญกว่าต้นทุน |
| Routine professional writing | Terra | สมดุลคุณภาพและต้นทุน |
| Daily brief / cron summary | Luna/Terra | งานรันซ้ำ ต้นทุนสะสมเร็ว |
| Bulk classification | Luna | output สั้น ปริมาณมาก |
| Code review / architecture | Sol/Terra | ต้อง reasoning และ verify |
| Legal/audit analysis | Sol/Terra | source fidelity สำคัญ |

## 16.4 วิธีลดต้นทุน

```text
จำกัด output
จำกัดจำนวน source/query
จำกัด tool calls
chunk ไฟล์ใหญ่
ใช้ isolated session สำหรับ cron
หยุด workflow เมื่อ cost spike
ตรวจ pricing/quota ก่อน production
```

---

# บทที่ 17: Rate Limit และ Context Overflow

## 17.1 Rate Limit

Rate limit คือข้อจำกัดของ provider เช่น:

```text
RPM = requests per minute
TPM = tokens per minute
RPD = requests per day
TPD = tokens per day
```

แนวทางแก้:

```text
1) หยุด run ซ้ำทันที
2) รอช่วงเวลาที่เหมาะสม
3) ลด prompt/output
4) ลด tool calls
5) แยก cron ไม่ให้ชนกัน
6) ตรวจ quota/billing/provider status
```

## 17.2 Context Overflow

เกิดเมื่อข้อมูลรวมทั้งหมดใหญ่เกิน context ของ model:

```text
prompt + chat history + tool input + file content + output budget > context limit
```

แนวทางแก้:

```text
ลด prompt
เริ่ม session ใหม่
chunk ไฟล์ใหญ่
จำกัด output
ไม่เปิด PDF/full text โดยไม่จำเป็น
แยกงานเป็น Discovery → Analysis → Record
```

## 17.3 Micro-light Prompt Pattern

```text
ค้นแบบเบามาก
ใช้ web search ไม่เกิน 1 ครั้ง
รายงานไม่เกิน 2 รายการ
ตอบไม่เกิน 350 คำ
ห้ามอ่าน PDF
ถ้าข้อมูลไม่พอ ให้แจ้งข้อจำกัด
```

---

# บทที่ 18: Troubleshooting

## 18.1 First Checks

```bash
# ตรวจสุขภาพ OpenClaw เบื้องต้น
openclaw doctor

# ตรวจ gateway
openclaw gateway status

# ตรวจ auth/model
openclaw models auth list --provider openai
openclaw models list --provider openai
openclaw models status --probe
```

## 18.2 Error Table

| Error / Symptom | สาเหตุที่เป็นไปได้ | วิธีแก้แรก |
|---|---|---|
| `401` | auth/API key ผิดหรือหมดอายุ | login provider ใหม่ / rotate key |
| `402` | credit/billing ไม่พอ | ตรวจ billing / ลด token / ลด model tier |
| `model not found` | route stale หรือ account ไม่มี model | run `models list` ใหม่ |
| `rate limit` | เกิน RPM/TPM/quota | รอ / ลด prompt / ลด tool calls |
| `context overflow` | prompt/history/file/tool ใหญ่เกิน | chunk / new session / ลด output |
| `web_search disabled` | ยังไม่ได้ตั้ง web provider | configure web แล้ว restart |
| cron output ยาวเกิน | prompt กว้างเกิน | เพิ่ม output budget |
| tool scope กว้างเกิน | agent อ่าน/ค้นมากกว่าที่ควร | จำกัด tool/file/source scope |

## 18.3 Logs

```bash
# ดู help ของ log command
openclaw logs --help

# ติดตาม log แบบต่อเนื่อง
openclaw logs --follow
```

ก่อนแชร์ log ให้ sanitize ตาม [`docs/security.md`](docs/security.md)

---

# บทที่ 19: Telegram Channel

## 19.1 ใช้ Telegram เพื่ออะไร

```text
รับผลสรุปรายวัน
รับแจ้งเตือน cron
สั่ง agent แบบสั้น
ตรวจสถานะระบบ
```

## 19.2 เริ่ม session ใหม่

ใน Telegram ส่ง:

```text
/new
```

ใช้เมื่อ:

```text
session ยาวเกิน
context overflow
เปลี่ยน model แล้ว
agent ตอบผิดปกติ
งานใหม่ไม่เกี่ยวกับงานเดิม
```

## 19.3 Security สำหรับ Telegram

```text
ห้ามโพสต์ bot token
ห้ามใส่ gateway token ใน chat
อย่า forward raw logs ที่มี secret
จำกัด channel/recipient สำหรับ cron notification
```

---

# บทที่ 20: Security Best Practices

## 20.1 Agent Threat Model

| Threat | Risk | Mitigation |
|---|---|---|
| Prompt injection | external content สั่ง agent ให้ ignore rules หรือ reveal data | treat external text as untrusted |
| Credential leakage | key/token หลุดใน prompt/log/screenshot | mask/rotate ทันที |
| Unsafe tool execution | agent ใช้ tool เกิน scope | limit tools per workflow |
| Skill/plugin poisoning | template มี hidden malicious instruction | review template ก่อน reuse |
| Over-broad file access | agent อ่านไฟล์ไม่เกี่ยวข้อง | ใช้ explicit path และ chunking |
| Cron amplification | job ซ้ำทำให้ cost/error เพิ่ม | isolated session + strict budget |
| Log exposure | logs มีข้อมูลส่วนบุคคลหรือ secrets | sanitize ก่อนแชร์ |

## 20.2 Secret scan เบื้องต้น

```bash
# ค้น pattern เสี่ยงใน folder OpenClaw local ก่อนแชร์ log/config
 grep -RniE "sk-|token|api[_-]?key|secret|password" "$HOME/.openclaw" --exclude-dir=node_modules
```

## 20.3 Incident Note Template

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

# บทที่ 21: Rollback Plan

## 21.1 Backup ก่อนแก้ config

```bash
# backup config ก่อนแก้ไข
cp "$HOME/.openclaw/openclaw.json" \
   "$HOME/.openclaw/openclaw.backup.$(date +%Y%m%d-%H%M%S).json"
```

## 21.2 Restore config

```bash
# restore config จาก backup ที่เลือก
cp "$HOME/.openclaw/openclaw.backup.YYYYMMDD-HHMMSS.json" \
   "$HOME/.openclaw/openclaw.json"

# restart และ probe หลัง restore
openclaw gateway restart
openclaw models status --probe
```

## 21.3 ปิด cron ที่ error ก่อน rollback

```bash
# ปิด cron ที่ทำงานผิดพลาดก่อนแก้ config
openclaw cron disable "<job-id>"
```

---

# บทที่ 22: Workshop Checklist

## 22.1 Installation Checklist

```text
[ ] Node.js พร้อม
[ ] OpenClaw ติดตั้งแล้ว
[ ] Gateway running
[ ] Dashboard เปิดได้
[ ] OpenAI provider login แล้ว
[ ] Auth profile ตรวจแล้ว
[ ] Model catalog ตรวจด้วย models list แล้ว
[ ] Primary model ตั้งจาก verified route แล้ว
[ ] Fallback model ตั้งอย่างตั้งใจแล้ว
[ ] models status --probe ผ่าน
[ ] Security checklist ผ่าน
[ ] Web search provider ตั้งค่าแล้วเมื่อจำเป็น
[ ] Cron test ผ่านเมื่อมี automation
```

## 22.2 GPT-5.6 Verification Checklist

```text
[ ] เห็น model route ใน openclaw models list --provider openai
[ ] เลือก tier ตามงาน ไม่ใช่เลือก model ใหญ่เสมอ
[ ] ไม่ hardcode ราคา/availability
[ ] ตรวจ pricing/quota ก่อน production
[ ] มี output budget
[ ] มี tool-call budget
[ ] มี fallback policy
[ ] มี human review สำหรับงาน high-risk
```

## 22.3 Troubleshooting Checklist

```text
[ ] ตรวจ gateway status
[ ] ตรวจ auth list
[ ] ตรวจ models list
[ ] ตรวจ models status --probe
[ ] ตรวจ cron list/runs
[ ] ตรวจ web provider
[ ] ตรวจ rate limit / credit / quota
[ ] ใช้ new session ถ้า context ค้าง
[ ] disable cron ที่ error ซ้ำ
[ ] backup ก่อนแก้ config
```

---

# บทที่ 23: แบบฝึกหัด

## แบบฝึกหัดที่ 1: ตรวจ catalog และเลือก primary

ให้ผู้เรียนรัน:

```bash
openclaw models auth list --provider openai
openclaw models list --provider openai
```

แล้วตอบ:

```text
1) พบ model route ใดบ้าง
2) งานที่กำลังทำควรใช้ Sol, Terra หรือ Luna เพราะอะไร
3) route ใดเหมาะเป็น primary
4) route ใดเหมาะเป็น fallback
```

## แบบฝึกหัดที่ 2: ตั้ง primary/fallback แบบ verified route

```bash
openclaw models set "openai/<verified-primary-model-id>"
openclaw models fallbacks clear
openclaw models fallbacks add "openai/<verified-fallback-model-id>"
openclaw gateway restart
openclaw models status --probe
```

ให้บันทึก:

```text
Primary:
Fallback:
Probe result:
Reason for tier selection:
```

## แบบฝึกหัดที่ 3: สร้าง Daily Brief แบบ cost-safe

ให้ผู้เรียนสร้าง cron ที่:

```text
รันทุกวัน 08:00
ใช้ isolated session
ใช้ <verified-economy-model-id>
จำกัดไม่เกิน 3 รายการ
ตอบไม่เกิน 500 คำ
ไม่อ่าน PDF เต็มฉบับ
```

## แบบฝึกหัดที่ 4: วิเคราะห์ error

| Error | วิธีแก้ที่ควรตอบ |
|---|---|
| model not found | ตรวจ `openclaw models list --provider openai` |
| rate limit | รอ / ลด prompt / ลด tool calls |
| context overflow | chunk / new session / ลด output |
| web_search disabled | configure web แล้ว restart |
| 401 | login provider ใหม่ / rotate key |
| cost spike | pause cron / ลด tier / ตรวจ pricing/quota |

---

# ภาคผนวก: Command Cheat Sheet

```bash
# System
openclaw doctor
openclaw gateway status
openclaw gateway restart
openclaw dashboard

# Auth
openclaw models auth login --provider openai
openclaw models auth login --provider openai --method api-key
openclaw models auth login --provider openai --device-code
openclaw models auth list --provider openai

# Models
openclaw models list --provider openai
openclaw models status
openclaw models status --probe
openclaw models set "openai/<verified-primary-model-id>"
openclaw models fallbacks clear
openclaw models fallbacks add "openai/<verified-fallback-model-id>"
openclaw models aliases list
openclaw models aliases add gpt-reasoning "openai/<verified-sol-or-terra-model-id>"
openclaw models aliases add gpt-balanced "openai/<verified-terra-model-id>"
openclaw models aliases add gpt-economy "openai/<verified-luna-or-terra-model-id>"

# Cron
openclaw cron list
openclaw cron run "<job-id>"
openclaw cron runs --id "<job-id>"
openclaw cron disable "<job-id>"
openclaw cron enable "<job-id>"
openclaw cron edit "<job-id>" --model "openai/<verified-economy-model-id>"

# Web Search
openclaw configure --section web
openclaw gateway restart

# Logs
openclaw logs --help
openclaw logs --follow

# Files
mkdir -p "$HOME/AI-Agent-Lab/output"
cat "file.txt"
head -80 "file.txt"
cat <<'EOF' > "output.md"
# Title
EOF
```

---

# สรุปบทเรียน

OpenClaw + OpenAI GPT-5.x / GPT-5.6 เหมาะสำหรับสร้าง AI Agent ที่ใช้งานจริงได้ในหลายบริบท เช่น สรุปข่าว สรุปเอกสาร จัดหมวดข้อมูล ทำรายงาน วิเคราะห์เชิงเหตุผล ตั้ง automation และสร้าง workflow ที่เชื่อมกับ tools ได้

แนวทางใหม่ของ repo นี้ไม่ใช่การจำชื่อ model แต่คือ:

```text
Verify catalog → choose model tier → set primary → set fallback → restart gateway → probe → document result
```

หลักสำคัญที่ต้องจำ:

```text
ตรวจ catalog ก่อนตั้ง model
ไม่ hardcode model ID/ราคา/availability
ใช้ Sol/Terra/Luna ตามความเสี่ยงและปริมาณงาน
จำกัด prompt/output/tool calls
ใช้ isolated session สำหรับ cron
ไม่เปิดเผย secret
sanitize logs ก่อนแชร์
ใช้ human review สำหรับงาน high-risk
```

**End of Lesson**
