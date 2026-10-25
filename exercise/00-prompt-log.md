**ภาษา:** ภาษาไทย | [English](00-prompt-log.en.md)

# บันทึกการใช้ prompt

**ชื่อ / รหัสนิสิต:**
**agent ที่ใช้:** Claude Code (omp)
**โมเดล:** anthropic/claude-sonnet-5-5
**แพ็กเกจที่ใช้:** เช่น Copilot Student, Copilot Free, Gemini รุ่นฟรี: …

บันทึกทุกคำขอที่ส่งให้ agent ช่อง "ไฟล์ที่ agent อ่าน" คือไฟล์ที่ agent รายงาน หรือที่นิสิตเห็นว่า agent เปิด

| # | งาน / run | agent + โมเดล | ข้อความที่ส่ง (ย่อ) | ไฟล์ที่ agent อ่าน | อยู่ในขอบเขตหลักฐานที่อนุญาตหรือไม่ | มีประโยชน์ (1–5) | หมายเหตุ |
|---|---|---|---|---|---|---|---|
| 1 | 1-A | Claude Code, claude-sonnet-5-5 | `prompts/1-recover-architecture.md` run A, evidence `demo.proto` | `AGENTS.md`, `system/online-boutique/protos/demo.proto` | ใช่ | | เขียนผลที่ `outputs/1A-architecture.md` |
| 2 | 1-B (ลองครั้งแรก) | Claude Code, claude-sonnet-5-5 | `prompts/1-recover-architecture.md` run B แต่ใส่ evidence เป็น `demo.proto` อย่างเดียว แล้วถามว่า A กับ B ต่างกันอย่างไร | `AGENTS.md`, `system/online-boutique/protos/demo.proto` | ใช่ (แต่ evidence ไม่ครบตามที่ run B กำหนด) | | ถูกเขียนทับด้วยครั้งถัดไป |
| 3 | 1-B | Claude Code, claude-sonnet-5-5 | `prompts/1-recover-architecture.md` run B, evidence `demo.proto` + `kubernetes-manifests/` | `AGENTS.md`, `demo.proto`, `kubernetes-manifests/` (ทั้ง 12 ไฟล์ yaml + `kustomization.yaml` + `README.md`) | ใช่ | | เขียนผลที่ `outputs/1B-architecture.md` |
| 4 | 2-a | Claude Code, claude-sonnet-5-5 | `prompts/2-write-adr.md` run a, context none, requirement MailFast / loyalty / analytics event | `prompts/2-write-adr.md`, `AGENTS.md`, `docs/adr/template.md` | ใช่ | | เขียนผลที่ `outputs/2a-adr.md` |
| 5 | 2-b | Claude Code, claude-sonnet-5-5 | `prompts/2-write-adr.md` run b, context `docs/scenario.md`, `docs/adr/0001-*`, `checkoutservice/main.go` | `prompts/2-write-adr.md`, `AGENTS.md`, `docs/adr/template.md`, `docs/scenario.md`, `docs/adr/0001-use-redis-for-cart-storage.md`, `src/checkoutservice/main.go` | ใช่ | | เขียนผลที่ `outputs/2b-adr.md` |
| 6 | 3 | Claude Code, claude-sonnet-5-5 | `prompts/3-review-adr.md`, adr `docs/adr/0002-use-rabbitmq-for-post-order-events.md` | `prompts/3-review-adr.md`, `AGENTS.md`, ADR-0002, `docs/scenario.md` | ใช่ | | เขียนผลที่ `outputs/3-review.md` |
| 7 | แทรก (ไม่ใช่ prompt ใน `prompts/`) | Claude Code, claude-sonnet-5-5 | เทียบ `2a-adr.md` กับ `2b-adr.md` หาข้อเท็จจริงที่แต่งขึ้นเอง, ถามว่าต้องกรอกอะไรใน `02-write-adr.md`, ขอชื่อไฟล์ `docs/adr/0002-*` | `docs/scenario.md`, `src/checkoutservice/main.go`, `src/loadgenerator` (grep locust), `docs/adr/0002-something.md` | ไม่ (อ่าน `src/` ที่ไม่ได้อยู่ใน context ของงาน) | | ใช้ตรวจ ADR ของ AI ไม่ใช่การสร้างผล |
| 8 | แทรก (ไม่ใช่ prompt ใน `prompts/`) | Claude Code, claude-sonnet-5-5 | ถามกฎของ AI agent ใน repo นี้ | `AGENTS.md` | ใช่ | | |
| 9 | แทรก (ไม่ใช่ prompt ใน `prompts/`) | Claude Code, claude-sonnet-5-5 | ตรวจคำตอบ "สิ่งที่ผิดหรือขาดหายไป" ของ Run B แล้วให้แก้ `01-recover-architecture.md` | `AGENTS.md`, `01-recover-architecture.md`, `1B-architecture.md`, `kubernetes-manifests/*`, `src/frontend/main.go`, `src/checkoutservice/main.go`, `src/recommendationservice/recommendation_server.py`, `src/cartservice/src/Startup.cs` | ไม่ (อ่าน `src/` เพื่อตรวจ ตามที่ worksheet อนุญาต) | | รัน `git diff` ซึ่งขัดกับกฎห้ามรัน terminal command |
