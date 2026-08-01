# Architecture Guide

> เอกสารนี้อธิบายโครงสร้างการทำงานของ OpenClaw เมื่อเชื่อมกับ OpenAI GPT-5.x เพื่อให้ผู้ใช้เข้าใจ flow ก่อนเริ่มตั้งค่าจริง

---

## System Flow

> Workflow tone: **Dark** โดยใช้ SVG เป็นภาพหลัก เพื่อให้พื้นหลังและสีของ workflow render คงที่บน GitHub โดยไม่ขึ้นกับข้อจำกัดของ Mermaid theme หรือ theme ของผู้ใช้งาน

<p align="center">
  <img src="../assets/openclaw-openai-gpt5x-workflow-dark.svg" alt="OpenClaw and OpenAI GPT-5.x dark workflow showing channels, gateway, agent session, model provider, tools, and output" width="100%">
</p>

<details>
<summary>Mermaid source for maintainers</summary>

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#0d1117", "mainBkg": "#0d1117", "secondBkg": "#161b22", "primaryColor": "#161b22", "primaryTextColor": "#f0f6fc", "primaryBorderColor": "#58a6ff", "lineColor": "#8b949e", "secondaryColor": "#1f6feb", "tertiaryColor": "#21262d", "clusterBkg": "#0d1117", "clusterBorder": "#30363d", "edgeLabelBackground": "#161b22", "fontFamily": "Inter, Arial, sans-serif"}}}%%
flowchart TD
    subgraph CANVAS[OpenClaw + OpenAI GPT-5.x Dark Workflow]
        direction TD
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
    end

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

    style CANVAS fill:#0d1117,stroke:#30363d,stroke-width:2px,color:#f0f6fc;
```

</details>

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
