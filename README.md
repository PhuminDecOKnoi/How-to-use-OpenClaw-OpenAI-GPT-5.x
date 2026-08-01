# How to Use OpenClaw with OpenAI GPT-5.x

![Documentation](https://img.shields.io/badge/type-documentation-blue)
![OpenClaw](https://img.shields.io/badge/platform-OpenClaw-purple)
![OpenAI](https://img.shields.io/badge/provider-OpenAI-black)
![Language](https://img.shields.io/badge/language-Thai%20%7C%20English-lightgrey)
![License](https://img.shields.io/badge/license-MIT-green)
![Security](https://img.shields.io/badge/security-first-critical)
![Sources](https://img.shields.io/badge/sources-point--level%20linked-success)

> คู่มือมาตรฐานสำหรับติดตั้ง กำหนดค่า ตรวจสอบสถานะ และใช้งาน **OpenClaw** ร่วมกับ **OpenAI GPT-5.x / GPT-5.6** เพื่อสร้าง AI Agent สำหรับ Dashboard, Telegram, Web Search, File Workflow และ Cron Automation อย่างเป็นระบบ ปลอดภัย ควบคุมต้นทุนได้ และมีแหล่งอ้างอิงแบบ point-level

---

## Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Who This Repository Is For](#who-this-repository-is-for)
- [Core Use Cases](#core-use-cases)
- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [GPT-5.6 Operating Standard](#gpt-56-operating-standard)
- [Source Reference Standard](#source-reference-standard)
- [Documentation Map](#documentation-map)
- [Operating Standards](#operating-standards)
- [Security Baseline](#security-baseline)
- [Troubleshooting First Steps](#troubleshooting-first-steps)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

**OpenClaw** ทำหน้าที่เป็น AI-agent gateway ระหว่างผู้ใช้ ช่องทางสื่อสาร เครื่องมือภายนอก และ model provider เช่น OpenAI โดย repository นี้จัดทำขึ้นเป็น **practical operating guide** สำหรับผู้ใช้ที่ต้องการนำ OpenClaw ไปใช้งานจริงหรือใช้สอนใน workshop

แนวทางของ repository นี้เน้น 6 เรื่องหลัก:

1. ติดตั้งและตรวจสอบ OpenClaw ให้พร้อมใช้งาน
2. เชื่อม OpenAI provider อย่างปลอดภัย
3. กำหนด model, fallback, alias และ gateway อย่างเป็นระบบ
4. ใช้งาน Dashboard, Telegram, Web Search, File Workflow และ Cron Automation
5. ควบคุม API key, token, cost, rate limit และ context overflow
6. อ้างอิงแหล่งข้อมูลภายนอกแบบ point-level ตามมาตรฐานใน [`docs/references.md`](docs/references.md)

---

## Repository Structure

```text
.
├── README.md
├── LICENSE
├── CHANGELOG.md
├── CONTRIBUTING.md
├── assets/
│   └── openclaw-openai-gpt5x-workflow-dark.svg
├── docs/
│   ├── architecture.md
│   ├── installation.md
│   ├── model-configuration.md
│   ├── openai-gpt56-operating-standard.md
│   ├── cost-control.md
│   ├── security.md
│   ├── troubleshooting.md
│   ├── cron-automation.md
│   ├── references.md
│   └── external-research-july-2026.md
├── examples/
│   ├── config/
│   │   └── .env.example
│   └── prompts/
│       └── daily-brief.md
└── .github/
    └── PULL_REQUEST_TEMPLATE.md
```

| Path | Purpose |
|---|---|
| `README.md` | หน้าแรกของ repository และแผนที่การใช้งานทั้งหมด |
| `assets/` | ภาพประกอบและ workflow diagram ที่ต้องการ render คงที่บน GitHub |
| `docs/` | เอกสารปฏิบัติการแยกตามหัวข้อ เพื่อให้อ่านง่ายและ maintain ได้ |
| `docs/references.md` | source registry และมาตรฐานการใส่ inline reference links |
| `examples/config/.env.example` | ตัวอย่าง environment variables แบบไม่มี secret จริง |
| `examples/prompts/` | ตัวอย่าง prompt สำหรับงานสอน งานทดลอง และ automation |
| `.github/PULL_REQUEST_TEMPLATE.md` | Template ตรวจคุณภาพก่อน merge |
| `CHANGELOG.md` | บันทึกการเปลี่ยนแปลงของ repository |
| `CONTRIBUTING.md` | แนวทางการปรับปรุงเอกสารและส่ง PR |

---

## Who This Repository Is For

| Audience | Use Case |
|---|---|
| ผู้เริ่มต้นใช้ AI Agent | ติดตั้ง OpenClaw และทดสอบ provider/model |
| IT instructors / trainers | ใช้เป็นคู่มือสอนหรือ workshop handout |
| Developers | ใช้เป็น runbook สำหรับตั้งค่า gateway, model และ automation |
| Knowledge workers | ใช้ AI Agent ช่วยสรุป ค้นหา จัดหมวด และสร้างรายงาน |
| Operations / governance users | วางมาตรฐาน security, token hygiene, source handling และ cost control |

---

## Core Use Cases

| Use Case | Description | Recommended Control |
|---|---|---|
| Daily Brief | สรุปข่าวหรือข้อมูลรายวัน | ใช้ model tier ที่คุมต้นทุน, จำกัด source และ output |
| Document Summary | สรุปไฟล์หรือเอกสาร | แบ่งไฟล์เป็นส่วนย่อยก่อนประมวลผล |
| Classification | จัดหมวดข้อมูลหรือเงื่อนไข | ใช้ prompt ที่มี output format ชัดเจน |
| Web Search Agent | ค้นข้อมูลล่าสุดและสรุปพร้อมแหล่งอ้างอิง | เปิด web provider, จำกัด query, ตรวจ source ทุกครั้ง |
| Cron Automation | ตั้งงานอัตโนมัติรายวัน/รายสัปดาห์ | ใช้ isolated session, output budget และ verified model route |
| Telegram Agent | สนทนากับ agent ผ่าน Telegram | ป้องกัน token และ reset session เมื่อ context ใหญ่เกินไป |
| Legal / Audit-style Analysis | วิเคราะห์ที่ต้องใช้เหตุผลและอ้างอิง source | ใช้ model tier ที่เหมาะกับ reasoning และห้ามละเลย source basis |
| Coding / Review | ช่วย coding, refactor, review, debug | ใช้ model ที่เหมาะกับ reasoning และต้องมี verification/test step |

---

## Architecture

<p align="center">
  <img src="assets/openclaw-openai-gpt5x-workflow-dark.svg" alt="OpenClaw and OpenAI GPT-5.x dark workflow showing channels, gateway, agent session, model provider, tools, and output" width="100%">
</p>

> The README uses an SVG workflow instead of an inline Mermaid block so the **dark background renders consistently on GitHub main**.

อ่านรายละเอียดเพิ่มเติมได้ที่ [`docs/architecture.md`](docs/architecture.md)

---

## Quick Start

```bash
# ตรวจสอบว่า OpenClaw ถูกติดตั้งและเรียกใช้งานได้
openclaw --version
openclaw doctor

# ตรวจสอบสถานะ gateway ก่อนเริ่มใช้งานจริง
openclaw gateway status

# เชื่อม OpenAI provider ผ่าน OpenClaw
openclaw models auth login --provider openai

# ตรวจรายการ model ที่บัญชีและ provider รองรับจริง
openclaw models list --provider openai

# ตรวจสถานะ model และทดสอบ probe
openclaw models status
openclaw models status --probe

# เปิด Dashboard เพื่อดูสถานะระบบผ่าน UI
openclaw dashboard
```

> อย่า hardcode model ID จากเอกสารนี้โดยไม่ตรวจ catalog ปัจจุบันก่อน ให้ใช้ผลจาก `openclaw models list --provider openai` เป็นแหล่งอ้างอิงก่อนตั้งค่า model ทุกครั้ง. [OC-MODELS]

---

## GPT-5.6 Operating Standard

มาตรฐานใหม่ของ repository นี้คือ **verification-first GPT-5.6 workflow**:

```text
Verify catalog → choose model tier → set primary → set fallback → restart gateway → probe → document result
```

| Tier strategy | Use for | Read more | Source |
|---|---|---|---|
| Sol / high reasoning | งานวิเคราะห์ลึก, coding, audit-style reasoning | [`docs/openai-gpt56-operating-standard.md`](docs/openai-gpt56-operating-standard.md) | [OA-GPT56] |
| Terra / balanced | งานเอกสาร งานสอน งาน professional ทั่วไป | [`docs/model-configuration.md`](docs/model-configuration.md) | [OA-GPT56] |
| Luna / cost-sensitive | งานสั้น งาน cron งาน classification ปริมาณมาก | [`docs/cost-control.md`](docs/cost-control.md) | [OA-GPT56] [NEWS-REUTERS] |
| Security baseline | tool access, prompt injection, secret leakage | [`docs/security.md`](docs/security.md) | [SEC-PRISM] |

> Pricing and availability can change. Always verify provider catalog, account access, quota, and current pricing before production use. [OC-MODELS] [NEWS-REUTERS]

---

## Source Reference Standard

This repository uses **point-level source links**. Put source markers immediately after the claim, command, model-route rule, pricing warning, or security-control statement that the source supports.

Example:

```markdown
OpenClaw uses the `openai/*` provider namespace for OpenAI model references. [OC-OPENAI]
```

Primary source registry:

- [`docs/references.md`](docs/references.md)
- [`docs/external-research-july-2026.md`](docs/external-research-july-2026.md)

Source priority:

```text
Official docs → Commands, provider routes, model availability wording
OpenAI docs → GPT-5.6 family / access information
News → Date-bound pricing-change signal only
Security research → Threat model and controls
Community/blog posts → Anecdotal implementation caution only
```

---

## Documentation Map

| Document | Description |
|---|---|
| [`docs/architecture.md`](docs/architecture.md) | อธิบายภาพรวม gateway, provider, session, tools และ output flow |
| [`docs/installation.md`](docs/installation.md) | ขั้นตอนติดตั้ง ตรวจสอบ และเปิด Dashboard |
| [`docs/model-configuration.md`](docs/model-configuration.md) | แนวทางตั้งค่า OpenAI provider, GPT-5.6 route, model, fallback และ alias พร้อม source links |
| [`docs/openai-gpt56-operating-standard.md`](docs/openai-gpt56-operating-standard.md) | มาตรฐานปฏิบัติ OpenClaw + OpenAI GPT-5.6 จากข้อมูลภายนอก July 2026 |
| [`docs/cost-control.md`](docs/cost-control.md) | แนวทางควบคุมต้นทุน model tier, output budget, tool calls และ cron |
| [`docs/security.md`](docs/security.md) | API key, token hygiene, agent threat model, tool permissions และ log sanitization |
| [`docs/troubleshooting.md`](docs/troubleshooting.md) | อาการผิดพลาดที่พบบ่อยและวิธีตรวจทีละขั้น |
| [`docs/cron-automation.md`](docs/cron-automation.md) | แนวทางออกแบบ scheduled job แบบ cost-safe |
| [`docs/references.md`](docs/references.md) | source registry และมาตรฐานการวาง link อ้างอิง ณ จุดใช้งาน |
| [`docs/external-research-july-2026.md`](docs/external-research-july-2026.md) | บันทึกแหล่งข้อมูลภายนอก เหตุผลการปรับมาตรฐาน และ point-level citation decisions |
| [`openclaw_openai_gpt_5_x_lesson_th.md`](openclaw_openai_gpt_5_x_lesson_th.md) | บทเรียนภาษาไทยที่สอดคล้องกับ README และ source-reference standard |
| [`examples/prompts/daily-brief.md`](examples/prompts/daily-brief.md) | prompt ตัวอย่างสำหรับ daily brief |
| [`examples/config/.env.example`](examples/config/.env.example) | environment variable template แบบไม่มีข้อมูลลับ |

---

## Operating Standards

### Code Fence Standard

| Block Type | Fence | Use For |
|---|---|---|
| Shell command | `bash` | คำสั่ง terminal ที่ผู้ใช้สามารถ copy ไป run ได้ |
| Terminal output | `console` | ตัวอย่างผลลัพธ์จาก terminal |
| Prompt/template | `markdown` or `text` | prompt, checklist, template, policy text |
| Diagram | `mermaid` | architecture, workflow, sequence, decision flow |
| Fixed visual | `svg` | workflow ที่ต้องการควบคุมพื้นหลัง/สีให้ render เหมือนกันบน GitHub |

### Shell Comment Standard

```bash
# ใช้ comment ภาษาไทยแบบปกติเพื่ออธิบายเหตุผลของคำสั่ง
openclaw models status --probe
```

ไม่ใช้ marker ลักษณะ debug เช่น `XXX`, `TODO` หรือคำที่ทำให้เอกสารดูไม่เป็นทางการ เว้นแต่เป็นงาน backlog จริง

---

## Security Baseline

ห้าม commit หรือแสดงข้อมูลต่อไปนี้ใน repository, screenshot, slide, chat หรือ shared terminal:

```text
API keys
Gateway tokens
Telegram bot tokens
Passwords
Session tokens
.env files with real values
Auth profiles
Logs containing secrets
```

ใช้ [`examples/config/.env.example`](examples/config/.env.example) เพื่อสอนรูปแบบตัวแปรเท่านั้น และใช้ [`docs/security.md`](docs/security.md) เป็น checklist ก่อนเผยแพร่เอกสารหรือ log. [OC-OPENAI] [SEC-PRISM]

---

## Troubleshooting First Steps

```bash
# ตรวจสุขภาพ OpenClaw เบื้องต้น
openclaw doctor

# ตรวจสถานะ gateway
openclaw gateway status

# ตรวจ provider และ model
openclaw models auth list --provider openai
openclaw models status --probe

# ตรวจ logs เมื่อยังไม่พบสาเหตุ
openclaw logs --follow
```

ดูรายละเอียดเพิ่มเติมใน [`docs/troubleshooting.md`](docs/troubleshooting.md)

---

## Roadmap

- [ ] Add a brand/title banner for README
- [ ] Add workshop slide outline
- [ ] Add hands-on lab worksheet
- [ ] Add model-selection checklist
- [ ] Add Thai/English glossary for AI-agent operations
- [ ] Add example workflows for Telegram, Web Search, File Summary, and Cron
- [ ] Add periodic source review process for OpenClaw/OpenAI model updates

---

## Contributing

อ่านแนวทางใน [`CONTRIBUTING.md`](CONTRIBUTING.md) ก่อนเปิด PR

มาตรฐานสำคัญ:

- ใช้ branch แยกจาก `main`
- ไม่ commit secrets หรือไฟล์ `.env` จริง
- ใช้ code fence language tag ให้ถูกต้อง
- ใส่ comment ภาษาไทยใน code block เมื่อเป็นสื่อสอน
- ใส่ source marker ณ จุดที่กล่าวอ้างข้อมูลภายนอก
- เปิด PR แบบ draft เพื่อ review ก่อน merge

---

## License

This repository is released under the MIT License. See [`LICENSE`](LICENSE) for details.

---

## Reference Links

[OC-OPENAI]: https://docs.openclaw.ai/providers/openai
[OC-MODELS]: https://docs.openclaw.ai/cli/models
[OA-GPT56]: https://openai.com/index/gpt-5-6/
[NEWS-REUTERS]: https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/
[SEC-PRISM]: https://arxiv.org/abs/2603.11853
