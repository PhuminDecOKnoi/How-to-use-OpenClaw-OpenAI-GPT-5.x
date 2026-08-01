# Cron Automation Guide

> แนวทางออกแบบ scheduled AI-agent workflow แบบควบคุมต้นทุนและความเสี่ยง

---

## Cost-Safe Cron Principles

- ใช้ prompt ที่สั้นและชัดเจน
- จำกัดจำนวนผลลัพธ์
- จำกัดความยาวคำตอบ
- ใช้ session แยกสำหรับ scheduled job
- หลีกเลี่ยงการอ่านไฟล์ใหญ่ทั้งไฟล์
- ตรวจ log หลังเริ่มใช้งานครั้งแรก
- หลีกเลี่ยง schedule ถี่เกินจำเป็น

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
  --model "openai/<model-id>" \
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
- [ ] ใช้ model ที่เหมาะกับต้นทุน
- [ ] ไม่อ่านไฟล์ใหญ่ทั้งไฟล์
- [ ] ไม่ใช้ข้อมูลลับจริง
- [ ] ตรวจ logs หลัง run แล้ว
