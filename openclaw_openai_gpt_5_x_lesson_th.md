# บทเรียน: การใช้งาน OpenClaw + OpenAI / GPT 5.x

> เวอร์ชันเอกสาร: v1.0  
> รูปแบบ: บทเรียนสำหรับสอน / แชร์ / ใช้เป็นคู่มือปฏิบัติ  
> กลุ่มเป้าหมาย: ผู้เริ่มต้นถึงระดับปฏิบัติการ  
> ตัวอย่างในเอกสารนี้เป็น “งานทั่วไป” ไม่อิงงานเฉพาะของผู้ใช้

---

## ภาพรวมบทเรียน

บทเรียนนี้อธิบายการติดตั้ง ตั้งค่า และใช้งาน **OpenClaw** ร่วมกับ **OpenAI / GPT 5.x** เพื่อสร้าง AI Agent ที่สามารถทำงานผ่าน Dashboard, Terminal, Telegram, Cron Automation, Web Search และไฟล์ในเครื่องได้อย่างปลอดภัย

เมื่อเรียนจบ ผู้เรียนควรทำได้ดังนี้

1. เข้าใจโครงสร้าง OpenClaw + OpenAI
2. ติดตั้งและเปิด OpenClaw Gateway ได้
3. เชื่อม OpenAI API Key กับ OpenClaw ได้
4. ตั้งค่า GPT 5.x เป็น Primary / Fallback Model ได้
5. ตรวจสอบสถานะ model, gateway, cron และ logs ได้
6. สร้าง automation ด้วย cron job ได้
7. ใช้ GPT ให้เหมาะกับงานทั่วไป เช่น สรุปข่าว สรุปไฟล์ สร้างรายงาน เตือนงาน และ draft เอกสาร
8. ควบคุมค่าใช้จ่ายและลดปัญหา rate limit / context overflow ได้
9. แก้ปัญหาพื้นฐานได้อย่างเป็นระบบ

---

# บทที่ 1: OpenClaw คืออะไร

## 1.1 ความหมาย

OpenClaw คือระบบ AI Agent ที่รันบนเครื่องของผู้ใช้ ทำหน้าที่เชื่อมต่อระหว่างผู้ใช้, ช่องทางสนทนา, model AI, เครื่องมือเสริม และงาน automation

โครงสร้างพื้นฐาน:

```text
User
 ↓
Dashboard / Telegram / Terminal
 ↓
OpenClaw Gateway
 ↓
AI Agent Session
 ↓
OpenAI GPT 5.x
 ↓
Tools เช่น Web Search, Files, Cron, Browser, Telegram
```

## 1.2 จุดเด่นของ OpenClaw

| ความสามารถ | คำอธิบาย |
|---|---|
| Local Gateway | มี gateway บนเครื่องผู้ใช้ |
| Dashboard | ควบคุม agent ผ่าน browser |
| Model Provider | เชื่อมต่อ OpenAI หรือ provider อื่นได้ |
| Cron Automation | ตั้งงานให้รันอัตโนมัติได้ |
| Telegram Channel | ส่งผลลัพธ์ไป Telegram ได้ |
| File Operation | อ่าน/เขียนไฟล์ สร้างโฟลเดอร์ จัดการ workspace ได้ |
| Tool Use | เรียกใช้ web search, browser, script หรือเครื่องมืออื่นตาม config |

---

# บทที่ 2: OpenAI / GPT 5.x คืออะไรในบริบท OpenClaw

## 2.1 ความหมายของ GPT 5.x

ในบทเรียนนี้ คำว่า **GPT 5.x** หมายถึง model ตระกูล GPT รุ่นใหม่ที่ใช้ผ่าน OpenAI API หรือผ่าน model reference ที่ OpenClaw รองรับ เช่น

```text
openai/gpt-5.4-mini
openai/gpt-5.4-nano
openai/gpt-5.4
openai/gpt-5.x
```

หมายเหตุ: ชื่อ model จริงขึ้นอยู่กับบัญชี, provider, OpenClaw version และ model catalog ที่ติดตั้งอยู่ ควรตรวจด้วยคำสั่ง:

```bash
openclaw models list --provider openai
```

## 2.2 หลักการเลือก model

| ประเภทงาน | Model ที่เหมาะ |
|---|---|
| งานทั่วไป / chat / สรุป / draft | GPT mini |
| งานเบา / cron / classification | GPT nano |
| งานซับซ้อน / reasoning / วิเคราะห์เอกสารยาว | GPT full / mini ตามงบประมาณ |
| งานทดลอง / demo | nano หรือ mini |
| งาน production สำคัญ | mini เป็น primary และ nano เป็น fallback |

## 2.3 แนวคิด Primary + Fallback

แนวทางแนะนำสำหรับระบบทั่วไป:

```text
Primary   = GPT mini
Fallback  = GPT nano
```

เหตุผล:

```text
GPT mini = คุณภาพดี เหมาะกับงานส่วนใหญ่
GPT nano = ประหยัด เหมาะเป็น fallback และงานเบา
```

---

# บทที่ 3: เตรียมเครื่องก่อนติดตั้ง

## 3.1 ตรวจ Node.js และ npm

เปิด Terminal แล้วรัน:

```bash
node --version
npm --version
```

ถ้ายังไม่มี Node.js สามารถติดตั้งด้วย Homebrew:

```bash
brew install node
```

## 3.2 ตรวจ shell

```bash
echo $SHELL
```

ส่วนใหญ่บน macOS จะเป็น:

```text
/bin/zsh
```

## 3.3 สร้างโฟลเดอร์สำหรับงาน AI Agent

ตัวอย่างทั่วไป:

```bash
mkdir -p "$HOME/AI-Agent-Lab"
mkdir -p "$HOME/AI-Agent-Lab/input"
mkdir -p "$HOME/AI-Agent-Lab/output"
mkdir -p "$HOME/AI-Agent-Lab/templates"
mkdir -p "$HOME/AI-Agent-Lab/logs"
```

ตรวจผล:

```bash
ls -la "$HOME/AI-Agent-Lab"
```

---

# บทที่ 4: ติดตั้ง OpenClaw

## 4.1 ติดตั้งด้วย Installer Script

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

## 4.2 ติดตั้งด้วย npm

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

## 4.3 ตรวจว่า OpenClaw ติดตั้งสำเร็จ

```bash
openclaw --version
openclaw doctor
openclaw gateway status
```

ผลที่ต้องการ:

```text
Runtime: running
Connectivity probe: ok
Dashboard: http://127.0.0.1:18789/
```

---

# บทที่ 5: เปิด Gateway และ Dashboard

## 5.1 Start / Restart Gateway

คำสั่ง restart ที่ถูกต้อง:

```bash
openclaw gateway restart
```

ไม่ใช่:

```bash
openclaw restart
```

## 5.2 เปิด Dashboard

```bash
openclaw dashboard
```

หรือ:

```bash
open http://127.0.0.1:18789
```

## 5.3 ตรวจสถานะ Gateway

```bash
openclaw gateway status
```

จุดที่ควรดู:

| จุดตรวจ | ความหมาย |
|---|---|
| `Runtime: running` | Gateway ทำงานอยู่ |
| `Connectivity probe: ok` | CLI ติดต่อ gateway ได้ |
| `Listening: 127.0.0.1:18789` | เปิด port แล้ว |
| `Dashboard URL` | เปิดหน้า control UI ได้ |

---

# บทที่ 6: สร้างและเชื่อม OpenAI API Key

## 6.1 หลักความปลอดภัย

ห้ามส่งข้อมูลต่อไปนี้ใน chat, screenshot หรือเอกสารสาธารณะ:

```text
OpenAI API key
Telegram bot token
Gateway token
Password
.env file
OAuth token
Secret key
```

## 6.2 Login OpenAI provider ใน OpenClaw

```bash
openclaw models auth login --provider openai
```

จากนั้นวาง API key ใน Terminal เท่านั้น

## 6.3 ตรวจ auth

```bash
openclaw models status
```

ควรเห็นข้อมูลลักษณะ:

```text
openai effective=profiles ... api_key=1
```

## 6.4 Probe model

```bash
openclaw models status --probe
```

ถ้าผ่านควรเห็นสถานะ `ok`

---

# บทที่ 7: ตั้ง GPT 5.x เป็น Primary Model

## 7.1 ตรวจ model ที่มีในบัญชี

```bash
openclaw models list --provider openai
```

## 7.2 ตั้ง model หลัก

ตัวอย่าง:

```bash
openclaw models set openai/gpt-5.4-mini
```

## 7.3 ตั้ง fallback

```bash
openclaw models fallbacks clear
openclaw models fallbacks add openai/gpt-5.4-nano
```

## 7.4 Restart และตรวจซ้ำ

```bash
openclaw gateway restart
openclaw models status
openclaw models status --probe
```

ผลที่ต้องการ:

```text
Default   : openai/gpt-5.4-mini
Fallbacks : openai/gpt-5.4-nano
Probe     : ok
```

---

# บทที่ 8: การตั้ง Alias ให้เรียก model ง่าย

## 8.1 เพิ่ม alias

```bash
openclaw models aliases add gpt-mini openai/gpt-5.4-mini
openclaw models aliases add gpt-nano openai/gpt-5.4-nano
openclaw models aliases add GPT openai/gpt-5.4-mini
```

## 8.2 ตรวจ alias

```bash
openclaw models aliases list
```

## 8.3 แก้ alias ที่ผิด

```bash
openclaw models aliases remove GPT
openclaw models aliases add GPT openai/gpt-5.4-mini
```

ถ้า error ว่า `Alias not found` แปลว่าไม่มี alias นั้นอยู่แล้ว ไม่ใช่ปัญหาใหญ่

---

# บทที่ 9: ออกแบบ Model Strategy สำหรับงานทั่วไป

## 9.1 Strategy ที่แนะนำ

```text
Primary: GPT mini
Fallback: GPT nano
```

## 9.2 แยกตามรูปแบบงาน

| งานทั่วไป | Model แนะนำ | เหตุผล |
|---|---|---|
| สรุปข้อความสั้น | GPT nano | ประหยัดและเร็ว |
| สรุปเอกสารระดับกลาง | GPT mini | คุณภาพดีกว่า |
| เขียนอีเมล | GPT mini หรือ nano | ขึ้นกับความสำคัญ |
| สร้างรายงาน | GPT mini | คุมโครงสร้างได้ดี |
| จัดหมวดข้อมูล | GPT nano | ใช้ token น้อย |
| วิเคราะห์ปัญหา | GPT mini | reasoning ดีกว่า |
| Cron รายวัน | GPT nano | ลดค่าใช้จ่าย |
| Cron รายเดือน | GPT mini | คุณภาพสำคัญกว่า |

---

# บทที่ 10: Prompt Design สำหรับ GPT 5.x

## 10.1 โครงสร้าง Prompt ที่ดี

```text
บทบาท:
คุณคือ...

งาน:
ทำอะไร

ข้อมูล:
ให้ข้อมูลที่จำเป็น

ข้อจำกัด:
ห้ามทำอะไร / จำกัดความยาว / ใช้แหล่งใด

รูปแบบผลลัพธ์:
ต้องตอบเป็นหัวข้อ ตาราง JSON หรือ bullet
```

## 10.2 ตัวอย่าง Prompt: สรุปรายงานการประชุม

```text
บทบาท:
คุณคือผู้ช่วยสรุปรายงานการประชุม

งาน:
สรุปข้อความประชุมด้านล่างให้เป็น executive summary

ข้อจำกัด:
- ตอบภาษาไทย
- ไม่เกิน 500 คำ
- แยก Action Items ให้ชัด
- ถ้าข้อมูลไม่พอ ให้ระบุว่า “ข้อมูลไม่เพียงพอ”

รูปแบบ:
1) สรุปภาพรวม
2) ประเด็นสำคัญ
3) Action Items
4) ความเสี่ยง/ข้อควรติดตาม
```

## 10.3 ตัวอย่าง Prompt: เขียนอีเมลธุรกิจ

```text
เขียนอีเมลภาษาไทยแบบสุภาพถึงลูกค้า
วัตถุประสงค์: แจ้งเลื่อนกำหนดส่งงานจากวันศุกร์เป็นวันจันทร์
น้ำเสียง: มืออาชีพ กระชับ รับผิดชอบ
ความยาว: ไม่เกิน 180 คำ
ให้มี subject ด้วย
```

## 10.4 ตัวอย่าง Prompt: จัดหมวด ticket support

```text
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

---

# บทที่ 11: ใช้ Web Search กับงานทั่วไป

## 11.1 ตั้งค่า Web Search

```bash
openclaw configure --section web
```

ตัวเลือกทั่วไป:

| Provider | เหมาะกับ |
|---|---|
| DuckDuckGo | ทดสอบเร็ว ไม่ต้องใช้ key |
| Brave | ใช้งานจริง เสถียรกว่า |
| Gemini Search | ต้องการ grounding/citation |

หลังตั้งค่า:

```bash
openclaw gateway restart
```

## 11.2 ตัวอย่างงาน: Daily General News Brief

Prompt ตัวอย่าง:

```text
ค้นข่าวเทคโนโลยีสำคัญใน 24 ชั่วโมงล่าสุด
จำกัดไม่เกิน 3 ข่าว
สรุปข่าวละไม่เกิน 4 บรรทัด
ระบุผลกระทบต่อคนทำงานทั่วไป
ส่งผลเป็นภาษาไทย
ถ้า web_search ใช้งานไม่ได้ ให้แจ้งข้อจำกัดชัดเจน
```

## 11.3 ข้อควรระวัง

```text
- อย่าค้นหลายเว็บเกินไปใน cron เดียว
- จำกัดผลลัพธ์ไม่เกิน 3–5 รายการ
- อย่าให้เปิด PDF หรือ full text ถ้าไม่จำเป็น
- ระบุว่าแหล่งใดเป็นแหล่งหลัก แหล่งใดใช้ cross-check
```

---

# บทที่ 12: การใช้งานไฟล์ทั่วไป

## 12.1 สร้างโฟลเดอร์

```bash
mkdir -p "$HOME/AI-Agent-Lab/input"
mkdir -p "$HOME/AI-Agent-Lab/output"
mkdir -p "$HOME/AI-Agent-Lab/archive"
```

## 12.2 อ่านไฟล์ text

```bash
cat "$HOME/AI-Agent-Lab/input/sample.txt"
```

อ่านเฉพาะต้นไฟล์:

```bash
head -80 "$HOME/AI-Agent-Lab/input/sample.txt"
```

## 12.3 เขียนไฟล์ Markdown

```bash
cat <<'EOF' > "$HOME/AI-Agent-Lab/output/summary.md"
# Summary

This is a sample summary generated by OpenClaw.
EOF
```

## 12.4 Append ไม่เขียนทับ

```bash
cat <<'EOF' >> "$HOME/AI-Agent-Lab/output/summary.md"

## Additional Notes
- New item added.
EOF
```

## 12.5 Backup ก่อนแก้ไฟล์

```bash
cp "$HOME/AI-Agent-Lab/output/summary.md" \
   "$HOME/AI-Agent-Lab/output/summary.backup.$(date +%Y%m%d-%H%M%S).md"
```

---

# บทที่ 13: Cron Automation พื้นฐาน

## 13.1 Cron คืออะไร

Cron คือระบบตั้งเวลารันงานอัตโนมัติ เช่น

```text
ทุกวัน 08:00 ส่งข่าวสรุป
ทุกวันศุกร์ 17:00 สรุปงานประจำสัปดาห์
ทุกเดือนวันที่ 1 สร้างรายงานค่าใช้จ่าย
```

## 13.2 คำสั่งดู cron

```bash
openclaw cron list
```

## 13.3 สร้าง cron ตัวอย่าง: Daily News Brief

```bash
MSG=$(cat <<'EOF'
ทำ Daily News Brief แบบสั้น

ค้นข่าวทั่วไปที่สำคัญใน 24 ชั่วโมงล่าสุด
จำกัดไม่เกิน 3 ข่าว
สรุปเป็นภาษาไทย
ข่าวละไม่เกิน 4 บรรทัด
ท้ายข้อความให้ระบุว่า “ต้องการรายละเอียดข่าวใดเพิ่มเติมหรือไม่”
EOF
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

## 13.4 Run test

```bash
openclaw cron list
openclaw cron run "<job-id>"
openclaw cron runs --id "<job-id>"
```

## 13.5 Disable / Enable

```bash
openclaw cron disable "<job-id>"
openclaw cron enable "<job-id>"
```

---

# บทที่ 14: ตัวอย่าง Automation ทั่วไป

## 14.1 Daily Personal Brief

```text
ทุกวัน 07:00
ส่งสรุป:
- สภาพอากาศโดยย่อ
- ข่าวสำคัญ 3 เรื่อง
- งานที่ควรทำวันนี้
- คำแนะนำสั้น ๆ
```

Model แนะนำ:

```text
openai/gpt-5.4-nano
```

## 14.2 Weekly Project Summary

```text
ทุกวันศุกร์ 17:00
อ่านบันทึกในโฟลเดอร์ project-notes
สรุป:
1) งานที่เสร็จแล้ว
2) งานที่ยังค้าง
3) ความเสี่ยง
4) แผนสัปดาห์ถัดไป
```

Model แนะนำ:

```text
openai/gpt-5.4-mini
```

## 14.3 Monthly Expense Summary

```text
ทุกวันที่ 1 เวลา 09:00
อ่านไฟล์ CSV ค่าใช้จ่าย
สรุปยอดรวม แยกหมวดหมู่ และข้อสังเกต
เขียนผลเป็น Markdown และ CSV
```

Model แนะนำ:

```text
openai/gpt-5.4-mini
```

## 14.4 Customer Feedback Classifier

```text
ทุกวัน 18:00
อ่าน feedback ใหม่
จัดหมวดเป็น:
- Complaint
- Feature Request
- Praise
- Bug
- Other
บันทึกเป็น CSV
```

Model แนะนำ:

```text
openai/gpt-5.4-nano
```

## 14.5 Study Assistant

```text
ทุกคืน 20:00
ส่งคำถามทบทวน 5 ข้อจากบทเรียนล่าสุด
ไม่เฉลยทันที
ให้ผู้เรียนตอบก่อน
```

Model แนะนำ:

```text
openai/gpt-5.4-nano
```

---

# บทที่ 15: การควบคุมค่าใช้จ่าย

## 15.1 หลักคิด

ค่าใช้จ่ายขึ้นกับ:

```text
input tokens + output tokens + tool usage + จำนวนครั้งที่เรียก model
```

## 15.2 ใช้ model ให้เหมาะกับงาน

| งาน | ใช้ model |
|---|---|
| งานสั้น / classification | nano |
| งานทั่วไป | mini |
| งานยาว / วิเคราะห์ | mini หรือรุ่นใหญ่ตามความจำเป็น |

## 15.3 ลด token ด้วยวิธีง่าย ๆ

```text
- จำกัดจำนวนรายการผลลัพธ์
- จำกัดจำนวนคำ
- อย่าให้ model อ่านไฟล์ยาวทั้งฉบับ
- แยกงานใหญ่เป็นหลายขั้นตอน
- ไม่ให้ cron ทำงานหนักเกินไป
```

## 15.4 ตัวอย่าง prompt ประหยัด token

```text
สรุปข้อความต่อไปนี้ไม่เกิน 150 คำ
แสดงเฉพาะ:
1) ใจความสำคัญ
2) สิ่งที่ต้องทำต่อ
ไม่ต้องอธิบายเพิ่ม
```

---

# บทที่ 16: Rate Limit และการแก้ปัญหา

## 16.1 Rate Limit คืออะไร

Rate limit คือข้อจำกัดการใช้งาน API เช่น

```text
RPM = requests per minute
TPM = tokens per minute
RPD = requests per day
TPD = tokens per day
```

## 16.2 ตัวอย่าง error

```text
Rate limit reached for gpt-5.4-nano on tokens per min
Please try again in 24s
```

## 16.3 วิธีแก้

```text
1) รอ 60–90 วินาที
2) อย่ากด run ซ้ำ
3) ลด prompt/output
4) ลดจำนวน tool call
5) แยก cron ไม่ให้รันติดกัน
6) ใช้ model ที่มี limit เหมาะกว่า
```

## 16.4 คำสั่งที่ควรใช้

```bash
sleep 90
openclaw cron list
openclaw cron runs --id "<job-id>"
```

---

# บทที่ 17: Context Overflow และวิธีป้องกัน

## 17.1 Context Overflow คืออะไร

เกิดเมื่อข้อมูลรวมทั้งหมดใหญ่เกิน context ของ model:

```text
prompt + chat history + tool input + output budget > context limit
```

## 17.2 วิธีแก้ทันที

```text
- ใช้ /new ใน Telegram
- ลด prompt
- จำกัด output
- ไม่เปิด PDF เต็ม
- ไม่ค้นหลายเว็บพร้อมกัน
- แยกงานเป็น Discovery → Analysis → Record
```

## 17.3 Micro-light Prompt Pattern

```text
ค้นแบบเบามาก
ใช้ web_search ไม่เกิน 1 ครั้ง
รายงานไม่เกิน 2 รายการ
ตอบไม่เกิน 350 คำ
ห้ามใช้ตาราง
ห้ามอ่าน PDF
ถามก่อนวิเคราะห์ต่อ
```

---

# บทที่ 18: Telegram Channel

## 18.1 ใช้ Telegram เพื่ออะไร

```text
- รับผลสรุปรายวัน
- แจ้งเตือนงาน cron
- สั่ง agent แบบสั้น
- ตรวจสถานะระบบ
```

## 18.2 เริ่ม session ใหม่

ใน Telegram ส่ง:

```text
/new
```

ใช้เมื่อ:

```text
- agent ตอบไม่ออก
- session ยาวมาก
- context overflow
- เปลี่ยน model แล้ว
- เจอ Something went wrong
```

## 18.3 ข้อความทดสอบ

```text
สวัสดี ตรวจสถานะสั้น ๆ ให้หน่อย
```

---

# บทที่ 19: Logs และการ Debug

## 19.1 ดู logs

```bash
openclaw logs --help
openclaw logs --follow
```

ถ้าเวอร์ชันรองรับ:

```bash
openclaw logs --tail 200
```

## 19.2 ดู cron runs

```bash
openclaw cron runs --id "<job-id>"
```

## 19.3 วิธีอ่าน error เบื้องต้น

| Error | สาเหตุ | วิธีแก้ |
|---|---|---|
| `401` | auth/API key ผิด | login provider ใหม่ |
| `402` | credit ไม่พอ | เติม credit / ลด token |
| `rate limit` | ใช้ TPM/RPM เกิน | รอ / ลด prompt |
| `context overflow` | prompt/tool ใหญ่เกิน | ลดขนาดงาน |
| `web_search disabled` | ยังไม่ตั้ง web provider | configure web |
| `couldn't generate response` | model/tool/session error | ดู runs/logs เพิ่ม |

---

# บทที่ 20: Security Best Practices

## 20.1 ไม่เผยแพร่ secret

```text
API key
Bot token
Gateway token
.env
Auth profile
Password
```

## 20.2 ก่อนส่ง log ให้ตรวจ secret

```bash
grep -RniE "sk-|token|api[_-]?key|secret|password" "$HOME/.openclaw" --exclude-dir=node_modules
```

## 20.3 Backup ก่อนแก้ config

```bash
cp "$HOME/.openclaw/openclaw.json" \
   "$HOME/.openclaw/openclaw.backup.$(date +%Y%m%d-%H%M%S).json"
```

---

# บทที่ 21: Rollback Plan

## 21.1 หา backup ล่าสุด

```bash
ls -lt "$HOME/.openclaw" | grep openclaw.backup
```

## 21.2 Restore config

```bash
cp "$HOME/.openclaw/openclaw.backup.YYYYMMDD-HHMMSS.json" \
   "$HOME/.openclaw/openclaw.json"

openclaw gateway restart
openclaw models status --probe
```

## 21.3 ปิด cron ที่ error ก่อน rollback

```bash
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
[ ] OpenAI API key login แล้ว
[ ] GPT mini ตั้งเป็น primary
[ ] GPT nano ตั้งเป็น fallback
[ ] models status --probe ผ่าน
[ ] Telegram ใช้งานได้
[ ] Web search provider ตั้งค่าแล้ว
[ ] Cron test ผ่าน
```

## 22.2 Troubleshooting Checklist

```text
[ ] ตรวจ gateway status
[ ] ตรวจ models status --probe
[ ] ตรวจ cron list
[ ] ตรวจ cron runs
[ ] ตรวจ web_search
[ ] ตรวจ Telegram target
[ ] ตรวจ rate limit / credit
[ ] ใช้ /new ถ้า session ค้าง
[ ] disable job ที่ error ซ้ำ
[ ] backup ก่อนแก้ config
```

---

# บทที่ 23: แบบฝึกหัด

## แบบฝึกหัดที่ 1: ตั้ง GPT Primary

ให้ผู้เรียนรัน:

```bash
openclaw models set openai/gpt-5.4-mini
openclaw models fallbacks clear
openclaw models fallbacks add openai/gpt-5.4-nano
openclaw gateway restart
openclaw models status --probe
```

แล้วตอบว่า:

```text
Default model คืออะไร
Fallback คืออะไร
Probe ผ่านหรือไม่
```

## แบบฝึกหัดที่ 2: สร้าง Daily Brief

ให้สร้าง cron ที่ส่งสรุปข่าวทั่วไปทุกวัน 08:00 โดยใช้ GPT nano

## แบบฝึกหัดที่ 3: เขียนไฟล์ Markdown

ให้สร้างไฟล์:

```text
~/AI-Agent-Lab/output/daily-summary.md
```

และเขียนหัวข้อ:

```text
# Daily Summary
```

## แบบฝึกหัดที่ 4: วิเคราะห์ error

ให้ผู้เรียนจับคู่ error กับวิธีแก้:

| Error | วิธีแก้ |
|---|---|
| rate limit | รอ / ลด prompt |
| context overflow | ลด context / ใช้ /new |
| web_search disabled | configure web |
| 401 | login provider ใหม่ |
| unknown command restart | ใช้ gateway restart |

---

# ภาคผนวก: Command Cheat Sheet

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

OpenClaw + OpenAI / GPT 5.x เหมาะสำหรับสร้าง AI Agent ที่ใช้งานจริงได้ในหลายบริบท เช่น สรุปข่าว สรุปเอกสาร จัดหมวดข้อมูล ทำรายงาน เขียนอีเมล ตั้ง reminder และสร้าง workflow อัตโนมัติ

แนวทางที่แนะนำสำหรับผู้เริ่มต้นและงานทั่วไป:

```text
Primary   = GPT mini
Fallback  = GPT nano
Cron เบา  = GPT nano
งานละเอียด = GPT mini
```

หลักสำคัญ:

```text
ตรวจสถานะก่อนแก้
backup ก่อนเปลี่ยน config
ไม่เปิดเผย secret
ใช้ model ให้เหมาะกับงาน
จำกัด prompt/output
ไม่รัน cron ซ้ำถี่
ใช้ /new เมื่อ session ค้าง
```

**End of Lesson**

