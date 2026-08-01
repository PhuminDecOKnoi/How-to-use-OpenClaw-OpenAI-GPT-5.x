# OpenClaw + OpenAI / GPT 5.x Operating Guide

![Documentation](https://img.shields.io/badge/type-documentation-blue)
![AI Agent](https://img.shields.io/badge/focus-AI%20Agent-purple)
![Security First](https://img.shields.io/badge/security-first-critical)
![Language](https://img.shields.io/badge/language-Thai%20%7C%20English-lightgrey)

> คู่มือมาตรฐานสำหรับติดตั้ง กำหนดค่า ตรวจสอบสถานะ และใช้งาน **OpenClaw** ร่วมกับ **OpenAI / GPT 5.x** เพื่อสร้าง AI Agent สำหรับงานประจำ งานสรุปข้อมูล งานค้นเว็บ งานไฟล์ งาน Telegram และงาน Automation ผ่าน Cron

---

## Table of Contents

- [Overview](#overview)
- [Use Cases](#use-cases)
- [Architecture](#architecture)
- [Requirements](#requirements)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Dashboard and Gateway](#dashboard-and-gateway)
- [OpenAI Model Configuration](#openai-model-configuration)
- [Model Strategy](#model-strategy)
- [Web Search](#web-search)
- [Cron Automation](#cron-automation)
- [File Workflow](#file-workflow)
- [Prompt Pattern](#prompt-pattern)
- [Cost and Rate-Limit Control](#cost-and-rate-limit-control)
- [Security Checklist](#security-checklist)
- [Troubleshooting](#troubleshooting)
- [Command Cheat Sheet](#command-cheat-sheet)
- [Recommended Repository Structure](#recommended-repository-structure)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

**OpenClaw** คือระบบ AI Agent ที่รันบนเครื่องผู้ใช้หรือเครื่อง server ส่วนตัว โดยทำหน้าที่เป็น gateway ระหว่างผู้ใช้ เครื่องมือภายนอก และ model provider เช่น OpenAI

Repository นี้เป็น **practical operating guide** สำหรับผู้ใช้ที่ต้องการนำ OpenClaw ไปใช้ในงานจริง โดยเน้นหลักการสำคัญ 5 เรื่อง:

1. ตั้งค่า model ให้ถูกต้อง
2. เชื่อม OpenAI API อย่างปลอดภัย
3. ใช้ Telegram / Dashboard / Cron อย่างเป็นระบบ
4. ควบคุม token, cost, rate limit และ context overflow
5. วางแนวปฏิบัติด้าน security ก่อนใช้งานจริง

---

## Use Cases

| Use Case | Description | Recommended Model |
|---|---|---|
| Daily Brief | สรุปข่าวหรือข้อมูลรายวัน | `nano` / lightweight model |
| Document Summary | สรุปไฟล์หรือเอกสาร | `mini` / reasoning model |
| Email Drafting | ร่างอีเมล ข้อความ และรายงาน | `mini` |
| Classification | จัดหมวดข้อมูล ตรวจเงื่อนไขเบื้องต้น | `nano` |
| Web Search Agent | ค้นข้อมูลล่าสุดจากเว็บพร้อมสรุป | `mini` |
| Cron Automation | ตั้งงานอัตโนมัติรายวัน / รายสัปดาห์ | `nano` หรือ `mini` ตามความซับซ้อน |
| Telegram Agent | ใช้งานผ่าน Telegram แบบสนทนา | `mini` เป็นค่าเริ่มต้น |

> หมายเหตุ: ชื่อ model จริงขึ้นอยู่กับ provider, account, region, version และรายการ model ที่ OpenClaw ตรวจพบในขณะใช้งาน ควรตรวจสอบด้วยคำสั่ง `openclaw models list --provider openai` ก่อนตั้งค่าเสมอ

---

## Architecture

```text
User / Telegram / Dashboard
        |
        v
OpenClaw Gateway
        |
        v
AI Agent Runtime
        |
        v
OpenAI / GPT Model Provider
        |
        v
Tools / Files / Web Search / Cron / Logs
```

### Component Responsibilities

| Component | Responsibility |
|---|---|
| User Interface | รับคำสั่งจากผู้ใช้ผ่าน Telegram, Dashboard หรือ CLI |
| OpenClaw Gateway | จัดการ request, routing, session และ tool access |
| AI Agent Runtime | ประมวลผล prompt, context, tools และ model response |
| Model Provider | ให้บริการ model สำหรับ reasoning, generation และ classification |
| Tools Layer | จัดการ web search, files, cron, logs และ integration อื่น ๆ |

---

## Requirements

ก่อนเริ่มต้น ควรเตรียมสิ่งต่อไปนี้:

- macOS, Linux หรือ Windows ที่รองรับ shell / terminal
- Node.js และ npm สำหรับการติดตั้งผ่าน npm
- OpenAI API Key หรือบัญชี model provider ที่ต้องการใช้
- Telegram Bot Token และ Chat ID หากต้องการใช้งานผ่าน Telegram
- Internet connection สำหรับติดตั้ง package และเรียก API
- ความเข้าใจพื้นฐานเกี่ยวกับ terminal, environment variables และ API key security

---

## Quick Start

```bash
# 1) ตรวจสอบ OpenClaw
openclaw --version
openclaw doctor

# 2) ตรวจสอบ Gateway
openclaw gateway status

# 3) Login OpenAI provider
openclaw models auth login --provider openai

# 4) ตรวจสอบ model
openclaw models list --provider openai
openclaw models status --probe

# 5) เปิด Dashboard
openclaw dashboard
```

---

## Installation

### Option 1: Installer Script

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Option 2: npm

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

### Verify Installation

```bash
openclaw --version
openclaw doctor
openclaw gateway status
```

หากคำสั่งใดไม่สำเร็จ ให้ตรวจสอบ path, permission, Node.js version และ network connection ก่อนดำเนินการต่อ

---

## Dashboard and Gateway

### Open Dashboard

```bash
openclaw dashboard
```

หรือเปิดผ่าน browser:

```bash
open http://127.0.0.1:18789
```

### Restart Gateway

```bash
openclaw gateway restart
```

> ไม่มีคำสั่ง `openclaw restart` ให้ใช้ `openclaw gateway restart` แทน

---

## OpenAI Model Configuration

### Login Provider

```bash
openclaw models auth login --provider openai
```

### Check Status

```bash
openclaw models status
openclaw models status --probe
```

### Set Default Model

```bash
openclaw models set openai/gpt-5.4-mini
```

### Set Fallback Model

```bash
openclaw models fallbacks clear
openclaw models fallbacks add openai/gpt-5.4-nano
```

### Restart and Probe

```bash
openclaw gateway restart
openclaw models status --probe
```

Expected result:

```text
Default   : openai/gpt-5.4-mini
Fallbacks : openai/gpt-5.4-nano
Probe     : ok
```

> ใช้ model ID ด้านบนเป็นตัวอย่างตามคู่มือนี้เท่านั้น หากบัญชีของคุณมีชื่อ model แตกต่างกัน ให้ใช้ชื่อที่แสดงจาก `openclaw models list --provider openai`

---

## Model Strategy

```text
Primary   = openai/gpt-5.4-mini
Fallback  = openai/gpt-5.4-nano
```

| Workload | Strategy |
|---|---|
| งานทั่วไป | ใช้ primary model |
| งานเบา / classification | ใช้ fallback หรือ nano model |
| Cron รายวัน | ใช้ model ขนาดเล็กเพื่อลด cost |
| รายงานละเอียด | ใช้ model ที่ reasoning ดีกว่า |
| งานค้นเว็บ | ใช้ model ที่สรุปและอ้างอิงได้ดี |
| งานไฟล์ยาว | แบ่งไฟล์เป็นส่วน ๆ ก่อนประมวลผล |

### Alias Setup

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

## Web Search

ใช้เมื่อ Agent ต้องค้นข้อมูลล่าสุดจากเว็บ เช่น ข่าว กฎหมาย ราคา version หรือข้อมูลที่อาจเปลี่ยนแปลงได้

```bash
openclaw configure --section web
openclaw gateway restart
```

| Provider | Suitable For |
|---|---|
| DuckDuckGo | ทดสอบเร็ว ไม่ต้องใช้ API key |
| Brave | ใช้งานจริง เสถียรกว่า |
| Gemini Search | งานที่ต้องการ grounding / citation |

Best practice:

- ระบุช่วงเวลาให้ชัดเจน เช่น วันนี้, 24 ชั่วโมงล่าสุด, ปี 2026
- จำกัดจำนวนผลลัพธ์เพื่อควบคุม token
- ให้ Agent แยกข้อเท็จจริงจากการวิเคราะห์
- ขอ citation เมื่อใช้ข้อมูลจากเว็บ

---

## Cron Automation

### List Jobs

```bash
openclaw cron list
```

### Run Job

```bash
openclaw cron run "<job-id>"
```

### View Runs

```bash
openclaw cron runs --id "<job-id>"
```

### Enable / Disable Job

```bash
openclaw cron disable "<job-id>"
openclaw cron enable "<job-id>"
```

### Edit Job Model

```bash
openclaw cron edit "<job-id>" --model openai/gpt-5.4-nano
```

### Example: Daily News Brief

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

## File Workflow

### Create Working Directories

```bash
mkdir -p "$HOME/AI-Agent-Lab/input"
mkdir -p "$HOME/AI-Agent-Lab/output"
```

### Read Input File

```bash
cat "$HOME/AI-Agent-Lab/input/sample.txt"
head -80 "$HOME/AI-Agent-Lab/input/sample.txt"
```

### Write Markdown Output

```bash
cat <<'EOF2' > "$HOME/AI-Agent-Lab/output/summary.md"
# Summary

This is a sample summary.
EOF2
```

### Backup Before Editing

```bash
cp "$HOME/AI-Agent-Lab/output/summary.md" \
   "$HOME/AI-Agent-Lab/output/summary.backup.$(date +%Y%m%d-%H%M%S).md"
```

---

## Prompt Pattern

ใช้โครงสร้าง prompt ต่อไปนี้เพื่อให้ Agent ทำงานชัดเจนและตรวจสอบได้:

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

Example:

```text
สรุปรายงานการประชุมต่อไปนี้
- ตอบภาษาไทย
- ไม่เกิน 500 คำ
- แยก Action Items
- ถ้าข้อมูลไม่พอ ให้ระบุว่า “ข้อมูลไม่เพียงพอ”
```

---

## Cost and Rate-Limit Control

### Cost Control

```text
[ ] ใช้ nano model กับงานเบา
[ ] ใช้ mini model กับงานซับซ้อน
[ ] จำกัด output length
[ ] จำกัดจำนวนผลลัพธ์จาก web search
[ ] ไม่อ่านไฟล์ยาวทั้งฉบับถ้าไม่จำเป็น
[ ] ไม่รัน Cron ซ้ำถี่เกินความจำเป็น
[ ] แยกงานใหญ่เป็นหลายขั้นตอน
```

### Rate Limit Recovery

ถ้าเจอข้อความลักษณะนี้:

```text
Rate limit reached
```

ให้ดำเนินการ:

```bash
sleep 90
openclaw cron list
openclaw cron runs --id "<job-id>"
```

แนวทางลดปัญหา:

- ลด prompt และ output
- ลดจำนวน tool call
- แยก Cron ไม่ให้รันติดกัน
- หลีกเลี่ยงการกด run ซ้ำหลายครั้ง

---

## Context Overflow

เกิดเมื่อขนาดข้อมูลรวมเกิน context limit ของ model:

```text
prompt + chat history + tool input + output budget > context limit
```

วิธีแก้:

```text
ใช้ /new ใน Telegram
ลด prompt
จำกัด output
ไม่อ่าน PDF เต็มฉบับในครั้งเดียว
ไม่ค้นหลายเว็บพร้อมกันโดยไม่จำเป็น
แยกงานเป็น Discovery → Analysis → Record
```

---

## Telegram Recovery

หาก Agent ค้าง หรือแสดงข้อความ `Something went wrong` ให้ส่งคำสั่งใน Telegram:

```text
/new
```

จากนั้นทดสอบด้วยข้อความสั้น:

```text
ตรวจสถานะสั้น ๆ
```

---

## Security Checklist

```text
[ ] ไม่เปิดเผย OpenAI API Key
[ ] ไม่เปิดเผย Telegram Bot Token
[ ] ไม่เปิดเผย Gateway Token
[ ] ไม่แนบ secret ใน GitHub, README, issue หรือ screenshot
[ ] Backup config ก่อนแก้ไข
[ ] ตรวจ logs ก่อนส่งต่อให้บุคคลอื่น
[ ] ใช้คำสั่ง rm / sudo / chmod อย่างระมัดระวัง
[ ] แยก environment ระหว่าง test และ production
[ ] จำกัดสิทธิ์ของ token เท่าที่จำเป็น
```

### Backup Config

```bash
cp "$HOME/.openclaw/openclaw.json" \
   "$HOME/.openclaw/openclaw.backup.$(date +%Y%m%d-%H%M%S).json"
```

---

## Troubleshooting

| Error / Symptom | Possible Cause | Recommended Action |
|---|---|---|
| `401` | API key ผิดหรือหมดอายุ | Login provider ใหม่ |
| `402` | Credit ไม่พอ | เติม credit หรือลด token/output |
| `rate limit` | ใช้ token ต่อนาทีเกิน | รอ / ลด prompt / ลด tool call |
| `context overflow` | prompt หรือไฟล์ใหญ่เกิน | ลด context / ใช้ `/new` / แยกงาน |
| `web_search disabled` | ยังไม่ได้ตั้งค่า web search | `openclaw configure --section web` |
| `unknown command restart` | ใช้คำสั่งผิด | ใช้ `openclaw gateway restart` |
| Dashboard เปิดไม่ได้ | Gateway ไม่ทำงานหรือ port ไม่พร้อม | `openclaw gateway status` แล้ว restart |
| Telegram ไม่ตอบ | session ค้างหรือ bot/channel ผิด | ใช้ `/new` และตรวจ token/chat ID |

---

## Command Cheat Sheet

```bash
# System
openclaw --version
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

## Recommended Repository Structure

```text
.
├── README.md                 # Main operating guide
├── docs/
│   ├── installation.md       # Detailed installation notes
│   ├── model-strategy.md     # Model selection and fallback strategy
│   ├── cron-recipes.md       # Cron automation examples
│   ├── telegram.md           # Telegram setup and recovery
│   ├── troubleshooting.md    # Error handling and diagnostics
│   └── security.md           # Secret handling and operational security
├── examples/
│   ├── prompts/
│   ├── cron/
│   └── scripts/
├── assets/
│   └── images/
├── CHANGELOG.md
├── CONTRIBUTING.md
└── LICENSE
```

> Repository นี้ยังสามารถขยายเป็น knowledge base เต็มรูปแบบได้ โดยแยกคู่มือย่อยออกจาก README เพื่อให้ดูแลง่ายขึ้น

---

## Roadmap

- [ ] เพิ่ม `docs/installation.md`
- [ ] เพิ่ม `docs/security.md`
- [ ] เพิ่ม `docs/cron-recipes.md`
- [ ] เพิ่มตัวอย่าง prompt สำหรับงานประจำ
- [ ] เพิ่มตัวอย่าง Telegram recovery workflow
- [ ] เพิ่ม CHANGELOG
- [ ] เพิ่ม CONTRIBUTING guideline
- [ ] เพิ่ม LICENSE file ตามนโยบายของ repository

---

## Contributing

แนวทางการปรับปรุง repository:

1. สร้าง branch ใหม่ก่อนแก้ไขเสมอ
2. ใช้ชื่อ branch ที่สื่อความหมาย เช่น `docs/update-readme`, `version/1.1-readme-standard`
3. แก้ไขเฉพาะไฟล์ที่เกี่ยวข้องกับงานนั้น
4. ใช้ commit message แบบชัดเจน เช่น `docs: update model configuration guide`
5. ตรวจสอบว่าไม่มี API key, token, password หรือ secret หลุดอยู่ในไฟล์
6. เปิด pull request เพื่อ review ก่อน merge เข้า `main`

---

## License

ยังไม่พบข้อมูล license ใน README เดิมโดยตรง ควรเพิ่มไฟล์ `LICENSE` เพื่อกำหนดเงื่อนไขการใช้งาน repository ให้ชัดเจนก่อนเผยแพร่หรือ reuse ในวงกว้าง

หากต้องการเปิดให้ใช้เพื่อการเรียนรู้และต่อยอดทั่วไป สามารถพิจารณาใช้ **MIT License** หรือ license อื่นตามวัตถุประสงค์ของเจ้าของ repository

---

## Summary

OpenClaw + OpenAI / GPT 5.x เหมาะสำหรับสร้าง AI Agent ใช้งานทั่วไป เช่น สรุปข่าว สรุปไฟล์ เขียนอีเมล จัดหมวดข้อมูล ทำรายงาน ค้นเว็บ และตั้ง Automation

ค่าที่แนะนำตามคู่มือนี้:

```text
Primary   = openai/gpt-5.4-mini
Fallback  = openai/gpt-5.4-nano
Cron เบา  = openai/gpt-5.4-nano
งานละเอียด = openai/gpt-5.4-mini
```

หลักปฏิบัติที่ควรยึดไว้:

```text
ตรวจสถานะก่อนแก้
backup ก่อนเปลี่ยน config
ไม่เปิดเผย secret
ใช้ model ให้เหมาะกับงาน
จำกัด prompt และ output
ไม่รัน Cron ซ้ำถี่
ใช้ /new เมื่อ session ค้าง
```

---

**Maintainer:** `PhuminDecOKnoi`  
**Repository:** `How-to-use-OpenClaw-OpenAI-GPT-5.x`  
**Document Version:** `v1.1`  
**Last Updated:** `2026-08-01`
