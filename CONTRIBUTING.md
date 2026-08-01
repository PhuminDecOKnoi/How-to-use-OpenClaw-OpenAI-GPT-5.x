# Contributing Guide

ขอบคุณที่ช่วยปรับปรุง repository นี้ครับ เอกสารนี้กำหนดมาตรฐานสำหรับการแก้ไขเนื้อหา ตัวอย่างคำสั่ง และโครงสร้างไฟล์

---

## Branch Workflow

```bash
# สร้าง branch ใหม่จาก main ก่อนแก้ไขเสมอ
# ตัวอย่าง: b.1.1-standard-readme-structure
git checkout main
git pull
git checkout -b b.<version>-<short-description>
```

---

## Documentation Standards

- ใช้ Markdown ที่อ่านง่าย มี heading ชัดเจน
- แยกเอกสารยาวออกไปไว้ใน `docs/`
- ใช้ `examples/` สำหรับตัวอย่าง prompt/config เท่านั้น
- ใช้ `bash` สำหรับ shell commands
- ใช้ `console` สำหรับ terminal output
- ใช้ `markdown` หรือ `text` สำหรับ prompt/template
- ใช้ `mermaid` สำหรับ diagram
- ใส่ comment ภาษาไทยใน code block เมื่อใช้เพื่อการสอน

---

## Security Rules

ห้าม commit:

- API keys
- Tokens
- Passwords
- Real `.env` files
- Logs containing secrets
- Customer/student/private data

ใช้เฉพาะ `.env.example` ที่มี placeholder เท่านั้น

---

## Pull Request Checklist

ก่อนเปิด PR ให้ตรวจว่า:

- [ ] ไม่มี secrets หรือข้อมูลส่วนบุคคล
- [ ] README link ใช้งานได้
- [ ] code fence มี language tag ถูกต้อง
- [ ] command block มีคำอธิบายที่จำเป็น
- [ ] docs/examples ถูกจัดไว้ใน folder ที่เหมาะสม
- [ ] CHANGELOG.md ถูกอัปเดตเมื่อมีการเปลี่ยนแปลงสำคัญ
