# Cron Automation Guide

> แนวทางออกแบบ scheduled AI-agent workflow แบบควบคุมต้นทุน ความเสี่ยง และคุณภาพผลลัพธ์ สำหรับ OpenClaw + OpenAI GPT-5.x

---

## Related Standards

อ่านประกอบก่อนเปิด cron production:

- [`docs/openai-gpt56-operating-standard.md`](openai-gpt56-operating-standard.md)
- [`docs/cost-control.md`](cost-control.md)
- [`docs/security.md`](security.md)
- [`docs/references.md`](references.md)

---

## Source Basis

- Cron ที่ใช้ model route ต้องตรวจ live model catalog ผ่าน OpenClaw Models CLI ก่อนเสมอ. [OC-MODELS]
- GPT-5.6 Sol/Terra/Luna ควรถูกเลือกตามระดับ reasoning, risk, volume และ cost. [OA-GPT56]
- งาน recurring มีความเสี่ยง cost amplification เมื่อมี output ยาว, tool calls, web search, file reads หรือ retries. [NEWS-REUTERS] [SEC-PRISM]

---

## Cost-Safe Cron Principles

- ใช้ prompt ที่สั้นและชัดเจน
- จำกัดจำนวนผลลัพธ์
- จำกัดความยาวคำตอบ
- ใช้ session แยกสำหรับ scheduled job
- หลีกเลี่ยงการอ่านไฟล์ใหญ่ทั้งไฟล์
- ตรวจ log หลังเริ่มใช้งานครั้งแรก
- หลีกเลี่ยง schedule ถี่เกินจำเป็น
- ใช้ model route ที่ตรวจแล้ว ไม่ใช้ชื่อ model จากความจำ [OC-MODELS]
- ตรวจ pricing/quota ก่อนเปิด recurring job [NEWS-REUTERS]

---

## Model Selection for Cron

| Cron type | Preferred model tier | Reason | Source |
|---|---|---|---|
| Daily lightweight brief | Luna or Terra | Output สั้นและรันซ้ำทุกวัน | [OA-GPT56] [NEWS-REUTERS] |
| Weekly executive summary | Terra | ต้องการคุณภาพมากขึ้นแต่ยังต้องคุมต้นทุน | [OA-GPT56] |
| Coding/diagnostic cron | Terra or Sol | ต้องการ reasoning และ verification | [OA-GPT56] |
| Legal/audit monitoring | Terra or Sol | ต้องการ source fidelity และลดความเสี่ยงในการสรุปผิด | [OA-GPT56] |
| Bulk classification | Luna | งานสั้น ปริมาณมาก และตรวจด้วย rule ได้ | [OA-GPT56] |

> Cron ที่มีความเสี่ยงสูงควร disable fallback หรือใช้ fallback ที่มีคุณภาพใกล้เคียงเท่านั้น เพื่อหลีกเลี่ยง silent downgrade. [OC-MODELS]

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

These checks rely on OpenClaw Models CLI and gateway status commands. [OC-MODELS]

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

Prompt output budgets and tool-scope constraints reduce recurring cost and agent-risk amplification. [NEWS-REUTERS] [SEC-PRISM]

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

Use a verified economy/balanced model route and document the owner, source scope, output budget, and fallback rule before enabling production recurrence. [OC-MODELS] [NEWS-REUTERS]

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

- [ ] Prompt จำกัด scope แล้ว [SEC-PRISM]
- [ ] Output จำกัดความยาวแล้ว [NEWS-REUTERS]
- [ ] ใช้ model ที่เหมาะกับต้นทุนและความเสี่ยง [OA-GPT56]
- [ ] Model route ตรวจจาก catalog แล้ว [OC-MODELS]
- [ ] Pricing/quota ตรวจแล้วก่อน production [NEWS-REUTERS]
- [ ] ไม่อ่านไฟล์ใหญ่ทั้งไฟล์ [SEC-PRISM]
- [ ] ไม่ใช้ข้อมูลลับจริง [SEC-PRISM]
- [ ] จำกัด tool calls แล้ว [SEC-PRISM]
- [ ] ใช้ isolated session [SEC-PRISM]
- [ ] ตรวจ logs หลัง run แล้ว [SEC-PRISM]
- [ ] มี owner รับผิดชอบ cron job

---

## Stop / Pause Conditions

หยุดหรือ pause cron ทันทีเมื่อพบ:

- ค่าใช้จ่ายสูงผิดปกติ [NEWS-REUTERS]
- provider error ซ้ำ
- model route หายจาก catalog [OC-MODELS]
- output ยาวกว่าที่กำหนด
- tool calls เกิน scope [SEC-PRISM]
- logs มี secrets หรือข้อมูลส่วนบุคคล [SEC-PRISM]
- agent สรุปข้อมูลโดยไม่มี source ทั้งที่งานต้องการ source

---

## Reference Links

[OC-MODELS]: https://docs.openclaw.ai/cli/models
[OA-GPT56]: https://openai.com/index/gpt-5-6/
[NEWS-REUTERS]: https://www.reuters.com/business/retail-consumer/openai-cuts-prices-smaller-models-businesses-scrutinize-ai-spend-2026-07-30/
[SEC-PRISM]: https://arxiv.org/abs/2603.11853
