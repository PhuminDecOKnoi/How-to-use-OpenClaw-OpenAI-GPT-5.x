# Cron Automation Guide

> แนวทางออกแบบ scheduled AI-agent workflow แบบควบคุมต้นทุน ความเสี่ยง และคุณภาพผลลัพธ์ สำหรับ OpenClaw + OpenAI GPT-5.x

---

## Related Standards

อ่านประกอบก่อนเปิด cron production:

- [`docs/openai-gpt56-operating-standard.md`](openai-gpt56-operating-standard.md)
- [`docs/cost-control.md`](cost-control.md)
- [`docs/security.md`](security.md)

---

## Cost-Safe Cron Principles

- ใช้ prompt ที่สั้นและชัดเจน
- จำกัดจำนวนผลลัพธ์
- จำกัดความยาวคำตอบ
- ใช้ session แยกสำหรับ scheduled job
- หลีกเลี่ยงการอ่านไฟล์ใหญ่ทั้งไฟล์
- ตรวจ log หลังเริ่มใช้งานครั้งแรก
- หลีกเลี่ยง schedule ถี่เกินจำเป็น
- ใช้ model route ที่ตรวจแล้ว ไม่ใช้ชื่อ model จากความจำ
- ตรวจ pricing/quota ก่อนเปิด recurring job

---

## Model Selection for Cron

| Cron type | Preferred model tier | Reason |
|---|---|---|
| Daily lightweight brief | Luna or Terra | Output สั้นและรันซ้ำทุกวัน |
| Weekly executive summary | Terra | ต้องการคุณภาพมากขึ้นแต่ยังต้องคุมต้นทุน |
| Coding/diagnostic cron | Terra or Sol | ต้องการ reasoning และ verification |
| Legal/audit monitoring | Terra or Sol | ต้องการ source fidelity และลดความเสี่ยงในการสรุปผิด |
| Bulk classification | Luna | งานสั้น ปริมาณมาก และตรวจด้วย rule ได้ |

> Cron ที่มีความเสี่ยงสูงควร disable fallback หรือใช้ fallback ที่มีคุณภาพใกล้เคียงเท่านั้น

---

## Preflight Verification

```bash
# ตรวจว่า OpenAI provider ใช้งานได้
openclaw models auth list --provider openai

# ตรวจ model catalog ของ account นี้
openclaw models list --provider openai

# ตรวจ model และ gateway ก่อนเปิด cron
openclaw models status --probe
openclaw gateway status
```

---

## Example Prompt Variable

```bash
# เก็บ prompt ในตัวแปร MSG เพื่อให้ command อ่านง่ายและ maintain ได้
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

---

## Add Cron Job

```bash
# เพิ่ม cron job โดยใช้ model ที่ตรวจแล้วว่าเหมาะกับงาน daily brief
openclaw cron add \
  --name "daily-lightweight-brief" \
  --cron "0 8 * * *" \
  --tz "Asia/Bangkok" \
  --session isolated \
  --model "openai/<verified-economy-model-id>" \
  --message "$MSG"
```

---

## Inspect Cron Jobs

```bash
# ดูรายการ cron job ทั้งหมด
openclaw cron list

# ทดสอบ run เฉพาะ job ที่เลือก
openclaw cron run "<job-id>"

# ดูประวัติการ run ของ job
openclaw cron runs --id "<job-id>"
```

---

## Review Checklist

- [ ] Prompt จำกัด scope แล้ว
- [ ] Output จำกัดความยาวแล้ว
- [ ] ใช้ model ที่เหมาะกับต้นทุนและความเสี่ยง
- [ ] Model route ตรวจจาก catalog แล้ว
- [ ] Pricing/quota ตรวจแล้วก่อน production
- [ ] ไม่อ่านไฟล์ใหญ่ทั้งไฟล์
- [ ] ไม่ใช้ข้อมูลลับจริง
- [ ] จำกัด tool calls แล้ว
- [ ] ใช้ isolated session
- [ ] ตรวจ logs หลัง run แล้ว
- [ ] มี owner รับผิดชอบ cron job

---

## Stop / Pause Conditions

หยุดหรือ pause cron ทันทีเมื่อพบ:

- ค่าใช้จ่ายสูงผิดปกติ
- provider error ซ้ำ
- model route หายจาก catalog
- output ยาวกว่าที่กำหนด
- tool calls เกิน scope
- logs มี secrets หรือข้อมูลส่วนบุคคล
- agent สรุปข้อมูลโดยไม่มี source ทั้งที่งานต้องการ source
