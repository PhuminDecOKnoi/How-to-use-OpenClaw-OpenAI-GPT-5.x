# Installation Guide

> ขั้นตอนติดตั้ง ตรวจสอบ และเปิดใช้งาน OpenClaw เบื้องต้น

---

## Requirements

- macOS, Linux หรือ Windows/WSL2 ที่ใช้งาน terminal ได้
- Node.js และ npm
- Internet connection
- OpenAI account/API access
- สิทธิ์ในการติดตั้ง package บนเครื่อง

---

## Install OpenClaw

### Option A: Installer Script

```bash
# ติดตั้งผ่าน installer script ของ OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Option B: npm

```bash
# ติดตั้ง OpenClaw เป็น global package
npm install -g openclaw@latest

# เริ่ม onboarding และติดตั้ง daemon/service
openclaw onboard --install-daemon
```

---

## Verify Installation

```bash
# ตรวจสอบ version และสุขภาพระบบ
openclaw --version
openclaw doctor

# ตรวจ gateway ว่าทำงานหรือไม่
openclaw gateway status
```

Expected output example:

```console
Gateway: running
Dashboard: http://127.0.0.1:18789/
Connectivity probe: ok
```

---

## Open Dashboard

```bash
# เปิด Dashboard ผ่าน command ของ OpenClaw
openclaw dashboard
```

macOS local URL option:

```bash
# เปิด local dashboard ผ่าน browser โดยตรง
open http://127.0.0.1:18789
```

---

## Restart Gateway

```bash
# restart gateway หลังเปลี่ยน provider/model/tool configuration
openclaw gateway restart
```
