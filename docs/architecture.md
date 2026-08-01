# Architecture Guide

> เอกสารนี้อธิบายโครงสร้างการทำงานของ OpenClaw เมื่อเชื่อมกับ OpenAI GPT-5.x เพื่อให้ผู้ใช้เข้าใจ flow ก่อนเริ่มตั้งค่าจริง

---

## System Flow

> Workflow tone: **Dark** เพื่อให้สอดคล้องกับภาพรวมของ AI-agent / developer documentation และช่วยให้ node, line และ layer แยกจากกันชัดเจนขึ้นบน GitHub

```mermaid
%%{init: {"theme": "dark", "themeVariables": {"background": "#0d1117", "primaryColor": "#161b22", "primaryTextColor": "#f0f6fc", "primaryBorderColor": "#58a6ff", "lineColor": "#8b949e", "secondaryColor": "#1f6feb", "tertiaryColor": "#21262d", "fontFamily": "Inter, Arial, sans-serif"}}}%%
flowchart TD
    U[User / Trainer / Operator] --> C[Channel Layer<br/>Dashboard / Telegram / CLI]
    C --> G[OpenClaw Gateway]
    G --> S[Agent Session]
    S --> M[OpenAI GPT-5.x Provider]
    S --> T[Tools Layer]
    T --> W[Web Search]
    T --> F[Files]
    T --> CR[Cron Automation]
    T --> L[Logs]
    M --> O[Output<br/>Summary / Report / Action]
    W --> O
    F --> O
    CR --> O
    L --> O

    classDef userLayer fill:#161b22,stroke:#58a6ff,color:#f0f6fc,stroke-width:2px;
    classDef gatewayLayer fill:#1f2937,stroke:#7c3aed,color:#f9fafb,stroke-width:2px;
    classDef modelLayer fill:#172554,stroke:#38bdf8,color:#f8fafc,stroke-width:2px;
    classDef toolsLayer fill:#14532d,stroke:#34d399,color:#f0fdf4,stroke-width:2px;
    classDef outputLayer fill:#3b0764,stroke:#c084fc,color:#faf5ff,stroke-width:2px;

    class U,C userLayer;
    class G,S gatewayLayer;
    class M modelLayer;
    class T,W,F,CR,L toolsLayer;
    class O outputLayer;
```

---

## Component Responsibilities

| Component | Responsibility |
|---|---|
| Channel | รับคำสั่งจาก Dashboard, Telegram หรือ CLI |
| Gateway | จัดการ routing, session, provider และ tool access |
| Agent Session | เก็บ context ของงานและควบคุม workflow |
| Model Provider | ประมวลผล reasoning, generation และ classification |
| Tools Layer | เชื่อม web search, file workflow, cron และ logs |
| Output | ส่งผลลัพธ์เป็น summary, report, action item หรือข้อความตอบกลับ |

---

## Teaching Notes

- OpenClaw ไม่ใช่ model โดยตรง แต่เป็น gateway/orchestration layer
- OpenAI เป็น provider ที่ให้บริการ model
- Tool access ต้องถูกควบคุม เพราะมีผลต่อข้อมูล ต้นทุน และความปลอดภัย
- Cron automation ควรใช้ session แยกและจำกัด output เสมอ
