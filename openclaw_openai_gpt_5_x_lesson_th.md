# บทเรียน: การใช้งาน OpenClaw + OpenAI GPT-5.x / GPT-5.6

> เวอร์ชันเอกสาร: v1.2  
> สถานะ: ปรับให้สอดคล้องกับ `README.md`, GPT-5.6 Operating Standard และ source-reference standard  
> รูปแบบ: บทเรียนภาษาไทยสำหรับสอน / workshop / self-study / runbook  
> กลุ่มเป้าหมาย: ผู้เริ่มต้นถึงระดับปฏิบัติการ  
> หลักสำคัญ: **ตรวจสอบ model catalog ก่อนตั้งค่าเสมอ ไม่ hardcode model ID / ราคา / สิทธิ์การใช้งานจากความจำ** [OC-MODELS]

---

## แผนที่เอกสารที่ควรอ่านประกอบ

| เอกสาร | ใช้เพื่อ |
|---|---|
| [`README.md`](README.md) | ภาพรวม repo, architecture, quick start และ documentation map |
| [`docs/references.md`](docs/references.md) | source registry และมาตรฐานการใส่ link อ้างอิง |
| [`docs/architecture.md`](docs/architecture.md) | เข้าใจ flow: Channel → Gateway → Agent Session → Model → Tools → Output |
| [`docs/installation.md`](docs/installation.md) | ติดตั้ง OpenClaw และตรวจ gateway/dashboard |
| [`docs/model-configuration.md`](docs/model-configuration.md) | ตั้ง OpenAI provider, primary model, fallback และ alias |
| [`docs/openai-gpt56-operating-standard.md`](docs/openai-gpt56-operating-standard.md) | มาตรฐาน OpenClaw + OpenAI GPT-5.6 จากข้อมูลภายนอก July 2026 |
| [`docs/cost-control.md`](docs/cost-control.md) | ควบคุมต้นทุน model tier, output budget, tool calls และ cron |
| [`docs/security.md`](docs/security.md) | token hygiene, agent threat model, tool permissions และ log sanitization |
| [`docs/cron-automation.md`](docs/cron-automation.md) | ออกแบบ scheduled job แบบ cost-safe และ risk-aware |
| [`docs/external-research-july-2026.md`](docs/external-research-july-2026.md) | บันทึกแหล่งข้อมูลภายนอกและเหตุผลการปรับมาตรฐาน |

---

## รูปแบบการอ้างอิงในบทเรียนนี้

บทเรียนนี้ใช้ inline source markers ณ จุดที่กล่าวถึง command, provider route, model tier, pricing/cost และ security risk เช่น:

```markdown
OpenClaw ใช้ provider namespace `openai/*` สำหรับ OpenAI model references. [OC-OPENAI]
```

Source marker ทุกตัวมี link definition อยู่ท้ายไฟล์ และมี registry กลางที่ [`docs/references.md`](docs/references.md)

---

## ผลลัพธ์การเรียนรู้

เมื่อเรียนจบบทเรียนนี้ ผู้เรียนควรสามารถ:

1. อธิบายบทบาทของ OpenClaw ในฐานะ AI-agent gateway/orchestration layer ได้
2. อธิบายความสัมพันธ์ระหว่าง OpenClaw, OpenAI provider, GPT-5.x / GPT-5.6 และ tools ได้
3. ติดตั้งและตรวจสอบ OpenClaw Gateway / Dashboard ได้
4. Login OpenAI provider ผ่าน OpenClaw ได้อย่างปลอดภัย [OC-MODELS]
5. ตรวจ model catalog ด้วย `openclaw models list --provider openai` ก่อนตั้งค่าได้ [OC-MODELS]
6. เลือก model tier แบบ Sol / Terra / Luna ให้เหมาะกับงาน ความเสี่ยง และต้นทุนได้ [OA-GPT56]
7. ตั้ง primary model, fallback model และ alias โดยใช้ `<verified-model-id>` ได้ [OC-MODELS]
8. ออกแบบ prompt, web search, file workflow และ cron automation แบบจำกัด scope ได้ [SEC-PRISM]
9. ใช้ security checklist เพื่อป้องกัน API key, token, logs และ prompt/output leakage ได้ [SEC-PRISM]
10. แก้ปัญหาเบื้องต้น เช่น auth error, model not found, rate limit, context overflow และ cron failure ได้

---

# บทที่ 1: OpenClaw คืออะไร

OpenClaw คือระบบ AI Agent / Gateway ที่เชื่อมต่อผู้ใช้ ช่องทางสื่อสาร model provider เครื่องมือเสริม และงาน automation โดยใน repository นี้ให้มอง OpenClaw เป็น **ตัวกลางควบคุมงาน** ไม่ใช่ตัว model โดยตรง

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

OpenClaw provider/model configuration ใช้แนวคิด model references แบบ `provider/model` และสำหรับ OpenAI ใช้ route pattern `openai/*`. [OC-PROVIDERS] [OC-OPENAI]

---

# บทที่ 2: OpenAI GPT-5.x / GPT-5.6 ในบริบท OpenClaw

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

OpenAI ระบุ GPT-5.6 เป็น model family ที่มี Sol, Terra และ Luna และ OpenClaw docs ระบุ OpenAI model refs ผ่าน `openai/*`; อย่างไรก็ตาม ผู้ใช้ต้องตรวจ model ที่บัญชีเห็นจริงก่อนตั้งค่า production. [OA-GPT56] [OC-OPENAI] [OC-MODELS]

## 2.1 เปลี่ยนจาก mini/nano เป็น Sol/Terra/Luna

บทเรียนเวอร์ชันก่อนหน้าเคยใช้แนวคิด `mini/nano` และตัวอย่าง `gpt-5.4-mini/nano` ซึ่งไม่สอดคล้องกับ README ล่าสุดแล้ว

มาตรฐานใหม่ของ repo นี้คือ:

| Tier | เหมาะกับ | หลีกเลี่ยงเมื่อ | Source |
|---|---|---|---|
| GPT-5.6 Sol | งาน reasoning หนัก, coding, legal/audit-style analysis, complex agents | งาน cron ปริมาณมากหรืองานสั้นที่ต้องคุมต้นทุน | [OA-GPT56] |
| GPT-5.6 Terra | งานเอกสาร งานสอน งาน professional ทั่วไป และ web/search summary | งานที่ต้องการ reasoning สูงสุดหรือ ultra-low-cost volume | [OA-GPT56] |
| GPT-5.6 Luna | งานสั้น งาน classification งาน cron งานสรุปเบา | งาน high-stakes reasoning หรือวิเคราะห์เอกสารซับซ้อน | [OA-GPT56] |

## 2.2 กฎหลักของบทเรียน

```text
Verify catalog → choose model tier → set primary → set fallback → restart gateway → probe → document result
```

ผู้เรียนต้องจำว่า:

```text
ห้าม hardcode model ID, ราคา, quota หรือสิทธิ์การใช้งาน โดยไม่ตรวจ catalog/account access ก่อน
```

กฎนี้อ้างอิงแนวทาง OpenClaw Models CLI และข้อเท็จจริงว่า access/pricing อาจเปลี่ยนได้. [OC-MODELS] [NEWS-REUTERS]

---

# บทที่ 3: เตรียมเครื่องก่อนติดตั้ง

## 3.1 ตรวจ Node.js และ npm

```bash
# ตรวจ Node.js version
node --version

# ตรวจ npm version
npm --version
```

## 3.2 สร้าง workspace สำหรับ workshop

```bash
# สร้างโฟลเดอร์แยก input/output/logs เพื่อลดความเสี่ยง file access กว้างเกินไป
mkdir -p "$HOME/AI-Agent-Lab/input"
mkdir -p "$HOME/AI-Agent-Lab/output"
mkdir -p "$HOME/AI-Agent-Lab/templates"
mkdir -p "$HOME/AI-Agent-Lab/logs"
```

การแยก workspace ช่วยจำกัด file scope และลดความเสี่ยง over-broad file access ใน agent workflow. [SEC-PRISM]

---

# บทที่ 4: ติดตั้งและตรวจ OpenClaw

```bash
# ติดตั้งด้วย installer script
curl -fsSL https://openclaw.ai/install.sh | bash

# หรือติดตั้งด้วย npm
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

หลังติดตั้งให้ตรวจ:

```bash
# ตรวจ version และสุขภาพระบบ
openclaw --version
openclaw doctor

# ตรวจสถานะ gateway
openclaw gateway status

# เปิด Dashboard
openclaw dashboard
```

หมายเหตุ: คำสั่งติดตั้งอาจเปลี่ยนตาม version ของ OpenClaw ให้ตรวจ `README.md` และ official docs ก่อนใช้จริงใน production workshop

---

# บทที่ 5: Login OpenAI Provider

OpenClaw ใช้ provider ID `openai` สำหรับ OpenAI auth และ model route. [OC-OPENAI]

```bash
# Login OpenAI provider ผ่าน OpenClaw Models CLI
openclaw models auth login --provider openai
```

ตัวเลือกเพิ่มเติม:

```bash
# ใช้ API-key method เมื่อรองรับ
openclaw models auth login --provider openai --method api-key

# ใช้ device-code สำหรับเครื่อง remote/headless
openclaw models auth login --provider openai --device-code

# ใช้ profile แยกสำหรับ training/demo/production
openclaw models auth login --provider openai --profile-id training-demo
```

คำสั่ง auth/profile/device-code อยู่ในกลุ่ม OpenClaw Models CLI. [OC-MODELS]

---

# บทที่ 6: ตรวจ Model Catalog ก่อนตั้งค่า

```bash
# ตรวจรายการ model ที่บัญชีเห็นจริง
openclaw models list --provider openai

# ตรวจสถานะ model ปัจจุบัน
openclaw models status

# ทดสอบ probe
openclaw models status --probe
```

อย่าใช้ชื่อ model จากความจำหรือจากเอกสารเก่าโดยไม่ตรวจ catalog ก่อน เพราะ model availability ขึ้นกับ account, organization, region, auth method และ rollout. [OC-MODELS] [OA-GPT56-HELP]

---

# บทที่ 7: ตั้ง Primary / Fallback Model

```bash
# ตั้ง primary model จาก model ID ที่ตรวจพบจริง
openclaw models set "openai/<verified-primary-model-id>"

# ล้าง fallback เดิมก่อนตั้งค่าใหม่
openclaw models fallbacks clear

# ตั้ง fallback อย่างตั้งใจ
openclaw models fallbacks add "openai/<verified-fallback-model-id>"

# restart และ probe
openclaw gateway restart
openclaw models status --probe
```

คำสั่ง model set/fallback/probe อ้างอิง OpenClaw Models CLI. [OC-MODELS]

แนวคิด fallback:

| งาน | Primary | Fallback | หมายเหตุ |
|---|---|---|---|
| งาน reasoning สูง | Sol/Terra | Terra หรือ disable fallback | หลีกเลี่ยง downgrade ในงานเสี่ยงสูง |
| งานเอกสารทั่วไป | Terra | Luna/Terra | คุมต้นทุนและคุณภาพ |
| งาน cron เบา | Luna/Terra | disable หรือ Luna | จำกัด output/tool calls |

---

# บทที่ 8: ตั้ง Alias สำหรับการสอน

```bash
# งาน reasoning หนัก
openclaw models aliases add gpt-reasoning "openai/<verified-sol-or-terra-model-id>"

# งานสมดุลคุณภาพ/ต้นทุน
openclaw models aliases add gpt-balanced "openai/<verified-terra-model-id>"

# งานเบา/cron/cost-sensitive
openclaw models aliases add gpt-economy "openai/<verified-luna-or-terra-model-id>"

# ตรวจ alias
openclaw models aliases list
```

Alias commands อยู่ใน OpenClaw Models CLI. [OC-MODELS]

---

# บทที่ 9: Model Strategy ตาม Use Case

| Use Case | Strategy | Control | Source |
|---|---|---|---|
| Daily Brief | Luna/Terra | จำกัด source/output | [OA-GPT56] [NEWS-REUTERS] |
| Document Summary | Terra | chunk ไฟล์ใหญ่ | [SEC-PRISM] |
| Classification | Luna | JSON/output format ชัดเจน | [OA-GPT56] |
| Web Search Agent | Terra | จำกัด query และต้องมี source summary | [SEC-PRISM] |
| Cron Automation | Luna/Terra | isolated session + output budget | [NEWS-REUTERS] [SEC-PRISM] |
| Legal / Audit-style Analysis | Sol/Terra | preserve source basis + human review | [OA-GPT56] |
| Coding / Review | Sol/Terra | ต้องมี verification/test step | [OA-GPT56] |

---

# บทที่ 10: Prompt Design แบบปลอดภัยและคุมต้นทุน

Prompt production ควรมี 5 ส่วน:

```text
Role:
Task:
Input scope:
Constraints:
Output format:
```

ตัวอย่าง:

```markdown
Role:
You are a concise AI operations assistant.

Task:
Create a lightweight daily brief.

Input scope:
Use only explicitly provided sources or enabled web search results.

Constraints:
- Keep the answer under 500 words.
- Return no more than 5 bullet points.
- Do not include secrets, tokens, or private data.
- If information is insufficient, say so clearly.

Output format:
1) Overall status
2) Key items
3) Impact notes
4) Sources, if available
```

Output budget และ tool-scope constraints ช่วยลด cost และลดความเสี่ยง agent ทำงานเกินขอบเขต. [NEWS-REUTERS] [SEC-PRISM]

---

# บทที่ 11: Web Search Workflow

หลักการ:

```text
จำกัด query → ตรวจ source → สรุปแบบสั้น → แยก fact/inference → ระบุข้อจำกัด
```

ข้อควรระวัง:

- อย่าค้นหลายเว็บเกินไปใน cron เดียว
- จำกัดผลลัพธ์ไม่เกิน 3–5 รายการ
- อย่าให้เปิด PDF หรือ full text ถ้าไม่จำเป็น
- ระบุ source ที่ใช้ทุกครั้งเมื่อข้อมูลเป็น current/external

External content must be treated as untrusted input because tool-augmented agents are exposed to prompt injection risk. [SEC-PRISM]

---

# บทที่ 12: File Workflow

```bash
# อ่านเฉพาะต้นไฟล์ก่อน เพื่อประเมินขนาดและโครงสร้าง
head -80 "$HOME/AI-Agent-Lab/input/sample.txt"

# เขียนผลลัพธ์ลง output folder เท่านั้น
cat <<'EOF' > "$HOME/AI-Agent-Lab/output/summary.md"
# Summary

This is a sample summary generated by OpenClaw.
EOF
```

ข้อควรระวัง:

```text
อย่าอ่านทั้ง folder โดยไม่จำเป็น
อย่าเปิดไฟล์ลับหรือไฟล์ลูกค้าทั้งหมดในครั้งเดียว
chunk ไฟล์ใหญ่ก่อนประมวลผล
backup ก่อนเขียนทับ
```

Over-broad file access เป็นหนึ่งในความเสี่ยงของ tool-augmented agents. [SEC-PRISM]

---

# บทที่ 13: Cron Automation

Cron jobs ต้องออกแบบแบบ cost-safe และ risk-aware เพราะงานรันซ้ำสามารถขยายต้นทุนและข้อผิดพลาดได้. [NEWS-REUTERS] [SEC-PRISM]

```bash
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
EOF
)

openclaw cron add \
  --name "daily-lightweight-brief" \
  --cron "0 8 * * *" \
  --tz "Asia/Bangkok" \
  --session isolated \
  --model "openai/<verified-economy-model-id>" \
  --message "$MSG"
```

ตรวจ job:

```bash
openclaw cron list
openclaw cron run "<job-id>"
openclaw cron runs --id "<job-id>"
```

---

# บทที่ 14: Cost Control

ต้นทุนขึ้นกับ:

```text
input tokens + output tokens + model tier + tool calls + frequency + retries
```

Policy:

```text
Pick the lowest-cost verified model tier that can still complete the task safely and accurately.
```

ข่าวราคา July 30, 2026 แสดงให้เห็นว่า pricing ของ Terra/Luna เปลี่ยนได้ ดังนั้น repository นี้จึงไม่ hardcode ราคาเป็นค่าถาวร. [NEWS-REUTERS] [NEWS-AXIOS] [NEWS-BI]

วิธีลดต้นทุน:

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

# บทที่ 15: Security Best Practices

ห้ามเผยแพร่:

```text
API key
Bot token
Gateway token
.env
Auth profile
Password
Raw logs with secrets
Private customer data
```

OpenClaw-style agents ที่ต่อ tools/files/web/logs มีความเสี่ยง prompt injection, credential leakage, unsafe tool execution และ log exposure จึงต้องใช้ least privilege และ sanitize logs. [SEC-PRISM]

ก่อนแชร์ log ให้ตรวจ secret:

```bash
grep -RniE "sk-|token|api[_-]?key|secret|password" "$HOME/.openclaw" --exclude-dir=node_modules
```

---

# บทที่ 16: Troubleshooting

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

| Error / Symptom | สาเหตุที่เป็นไปได้ | วิธีแก้แรก | Source |
|---|---|---|---|
| `401` | auth/API key ผิดหรือหมดอายุ | login provider ใหม่ / rotate key | [OC-MODELS] |
| `402` | credit/billing ไม่พอ | ตรวจ billing / ลด token / ลด model tier | [NEWS-REUTERS] |
| `model not found` | route stale หรือ account ไม่มี model | run `models list` ใหม่ | [OC-MODELS] |
| `rate limit` | เกิน RPM/TPM/quota | รอ / ลด prompt / ลด tool calls | [NEWS-REUTERS] |
| `context overflow` | prompt/history/file/tool ใหญ่เกิน | chunk / new session / ลด output | [SEC-PRISM] |
| `web_search disabled` | ยังไม่ได้ตั้ง web provider | configure web แล้ว restart | [SEC-PRISM] |
| cron output ยาวเกิน | prompt กว้างเกิน | เพิ่ม output budget | [NEWS-REUTERS] |

---

# บทที่ 17: Workshop Checklist

## Installation / Model Checklist

```text
[ ] Node.js พร้อม
[ ] OpenClaw ติดตั้งแล้ว
[ ] Gateway running
[ ] Dashboard เปิดได้
[ ] OpenAI provider login แล้ว
[ ] openclaw models list --provider openai ผ่าน
[ ] เลือก Sol/Terra/Luna ตาม workload แล้ว
[ ] ตั้ง primary เป็น <verified-primary-model-id>
[ ] ตั้ง fallback เป็น <verified-fallback-model-id> หรือ disable ตาม risk
[ ] models status --probe ผ่าน
[ ] ไม่มี secret ใน repo/docs/examples
```

## Cron Checklist

```text
[ ] Model route ตรวจจาก catalog แล้ว
[ ] Output budget ระบุแล้ว
[ ] Tool scope จำกัดแล้ว
[ ] ใช้ isolated session
[ ] Pricing/quota ตรวจแล้ว
[ ] Logs หลัง test run ไม่มี secret
[ ] มี owner รับผิดชอบ job
```

---

# บทที่ 18: แบบฝึกหัด

## แบบฝึกหัดที่ 1: ตรวจ catalog และตั้ง model

```bash
openclaw models auth list --provider openai
openclaw models list --provider openai
openclaw models set "openai/<verified-primary-model-id>"
openclaw models fallbacks clear
openclaw models fallbacks add "openai/<verified-fallback-model-id>"
openclaw gateway restart
openclaw models status --probe
```

ให้ผู้เรียนตอบ:

```text
Default model คืออะไร
Fallback คืออะไร
Probe ผ่านหรือไม่
เหตุผลที่เลือก tier นี้คืออะไร
```

## แบบฝึกหัดที่ 2: เลือก tier จากสถานการณ์

| สถานการณ์ | Tier ที่ควรเลือก | เหตุผล |
|---|---|---|
| Daily brief สั้นทุกเช้า | Luna/Terra | คุมต้นทุน recurring |
| วิเคราะห์ legal/audit finding | Sol/Terra | reasoning/source fidelity |
| Bulk classification 5,000 records | Luna | output สั้น ปริมาณมาก |
| Code architecture review | Sol/Terra | reasoning และ verification |

## แบบฝึกหัดที่ 3: Security review

ให้ผู้เรียนตรวจเอกสารตัวอย่างและ mark ว่ามีความเสี่ยงใด:

```text
API key exposed
raw log exposed
file scope too broad
prompt lacks output budget
cron has no owner
```

---

# สรุปบทเรียน

OpenClaw + OpenAI GPT-5.x / GPT-5.6 เหมาะสำหรับสร้าง AI Agent ที่ใช้งานจริงได้ในหลายบริบท เช่น สรุปข่าว สรุปเอกสาร จัดหมวดข้อมูล ทำรายงาน เขียนอีเมล ตรวจโค้ด ตั้ง cron และสร้าง workflow อัตโนมัติ

แนวทางที่แนะนำใน repository นี้คือ:

```text
Verify catalog first
Use openai/* route pattern
Choose Sol/Terra/Luna by workload risk and cost
Set explicit primary/fallback
Probe before production
Limit output/tool/file scope
Sanitize logs
Document sources
```

**End of Lesson**

---

## Reference Links

[OC-OPENAI]: https://docs.openclaw.ai/providers/openai
[OC-MODELS]: https://docs.openclaw.ai/cli/models
[OC-PROVIDERS]: https://docs.openclaw.ai/concepts/model-providers
[OA-GPT56]: https://openai.com/index/gpt-5-6/
[OA-GPT56-HELP]: https://help.openai.com/en/articles/20001325-a-preview-of-gpt-56-sol-terra-and-luna
[NEWS-REUTERS]: https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/
[NEWS-AXIOS]: https://www.axios.com/2026/07/30/openai-cuts-prices-gpt-terra-luna5
[NEWS-BI]: https://www.businessinsider.com/openai-price-cuts-gpt-terra-luna-2026-7
[SEC-PRISM]: https://arxiv.org/abs/2603.11853
