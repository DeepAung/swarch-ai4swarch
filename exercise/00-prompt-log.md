**ภาษา:** ภาษาไทย | [English](00-prompt-log.en.md)

# บันทึกการใช้ prompt

**ชื่อ / รหัสนิสิต:** Ashira Aungsumal / 6732401921
**agent ที่ใช้:** Claude Code (omp)
**โมเดล:** anthropic/claude-sonnet-5-5
**แพ็กเกจที่ใช้:** Claude Pro Plan

บันทึกทุกคำขอที่ส่งให้ agent ช่อง "ไฟล์ที่ agent อ่าน" คือไฟล์ที่ agent รายงาน หรือที่นิสิตเห็นว่า agent เปิด

| # | งาน / run | agent + โมเดล | ข้อความที่ส่ง (ย่อ) | ไฟล์ที่ agent อ่าน | อยู่ในขอบเขตหลักฐานที่อนุญาตหรือไม่ | มีประโยชน์ (1–5) | หมายเหตุ |
|---|---|---|---|---|---|---|---|
| 1 | 1-A | Claude Code, claude-sonnet-5-5 | `prompts/1-recover-architecture.md` run A, evidence `demo.proto` | `AGENTS.md`, `system/online-boutique/protos/demo.proto` | ใช่ | 4 | เขียนผลที่ `outputs/1A-architecture.md` |
| 2 | 1-B (ลองครั้งแรก) | Claude Code, claude-sonnet-5-5 | `prompts/1-recover-architecture.md` run B แต่ใส่ evidence เป็น `demo.proto` อย่างเดียว แล้วถามว่า A กับ B ต่างกันอย่างไร | `AGENTS.md`, `system/online-boutique/protos/demo.proto` | ใช่ (แต่ evidence ไม่ครบตามที่ run B กำหนด) | 2 | ถูกเขียนทับด้วยครั้งถัดไป |
| 3 | 1-B | Claude Code, claude-sonnet-5-5 | `prompts/1-recover-architecture.md` run B, evidence `demo.proto` + `kubernetes-manifests/` | `AGENTS.md`, `demo.proto`, `kubernetes-manifests/` (ทั้ง 12 ไฟล์ yaml + `kustomization.yaml` + `README.md`) | ใช่ | 4 | เขียนผลที่ `outputs/1B-architecture.md` |
| 4 | 2-a | Claude Code, claude-sonnet-5-5 | `prompts/2-write-adr.md` run a, context none, requirement MailFast / loyalty / analytics event | `prompts/2-write-adr.md`, `AGENTS.md`, `docs/adr/template.md` | ใช่ | 3 | เขียนผลที่ `outputs/2a-adr.md` |
| 5 | 2-b | Claude Code, claude-sonnet-5-5 | `prompts/2-write-adr.md` run b, context `docs/scenario.md`, `docs/adr/0001-*`, `checkoutservice/main.go` | `prompts/2-write-adr.md`, `AGENTS.md`, `docs/adr/template.md`, `docs/scenario.md`, `docs/adr/0001-use-redis-for-cart-storage.md`, `src/checkoutservice/main.go` | ใช่ | 4 | เขียนผลที่ `outputs/2b-adr.md` |
| 6 | 3 | Claude Code, claude-sonnet-5-5 | `prompts/3-review-adr.md`, adr `docs/adr/0002-use-rabbitmq-for-post-order-events.md` | `prompts/3-review-adr.md`, `AGENTS.md`, ADR-0002, `docs/scenario.md` | ใช่ | 4 | เขียนผลที่ `outputs/3-review.md` |
| 7 | แทรก (ไม่ใช่ prompt ใน `prompts/`) | Claude Code, claude-sonnet-5-5 | เทียบ `2a-adr.md` กับ `2b-adr.md` หาข้อเท็จจริงที่แต่งขึ้นเอง, ถามว่าต้องกรอกอะไรใน `02-write-adr.md`, ขอชื่อไฟล์ `docs/adr/0002-*` | `docs/scenario.md`, `src/checkoutservice/main.go`, `src/loadgenerator` (grep locust), `docs/adr/0002-something.md` | ไม่ (อ่าน `src/` ที่ไม่ได้อยู่ใน context ของงาน) | 4 | ใช้ตรวจ ADR ของ AI ไม่ใช่การสร้างผล |
| 8 | แทรก (ไม่ใช่ prompt ใน `prompts/`) | Claude Code, claude-sonnet-5-5 | ถามกฎของ AI agent ใน repo นี้ | `AGENTS.md` | ใช่ | 3 | |
| 9 | แทรก (ไม่ใช่ prompt ใน `prompts/`) | Claude Code, claude-sonnet-5-5 | ตรวจคำตอบ "สิ่งที่ผิดหรือขาดหายไป" ของ Run B แล้วให้แก้ `01-recover-architecture.md` | `AGENTS.md`, `01-recover-architecture.md`, `1B-architecture.md`, `kubernetes-manifests/*`, `src/frontend/main.go`, `src/checkoutservice/main.go`, `src/recommendationservice/recommendation_server.py`, `src/cartservice/src/Startup.cs` | ไม่ (อ่าน `src/` เพื่อตรวจ ตามที่ worksheet อนุญาต) | 3 | รัน `git diff` ซึ่งขัดกับกฎห้ามรัน terminal command |
| 10 | 4 (review board) | Claude Code, claude-sonnet-5-5 | อ่าน `outputs/4-review-board.md` แล้วแปลเป็นภาษาไทยลง `04-review-board.md` พร้อมเขียนไฟล์ของทุกบทบาท (1 ไฟล์ต่อ 1 บทบาท) | `AGENTS.md`, `exercise/outputs/4-review-board.md`, `exercise/04-review-board.md` | ไม่ (ไม่ใช่ prompt ใน `prompts/`; agent แก้ `exercise/04-review-board.md` ซึ่ง AGENTS.md ห้าม แต่นิสิตสั่งโดยตรง) | 5 | เขียนผลที่ `outputs/review-board-th/` (5 ไฟล์) และแก้ส่วนที่ 3 ของ `04-review-board.md` |
| 11 | 5 (reflection) | Claude Code, claude-sonnet-5-5 | ให้กรอก `05-reflection.md` | `exercise/05-reflection.md` (ใช้ข้อความ Architect จากแถว 10) | ไม่ (แก้ worksheet ตามที่นิสิตสั่งโดยตรง) | 4 | ชื่อกรอกจาก ADR ส่วนรหัสนิสิตนิสิตเติมเอง |
| 12 | ตรวจความพร้อมส่งงาน | Claude Code, claude-sonnet-5-5 | ให้ตรวจ repo ว่าพร้อมส่งหรือยัง | `exercise/outputs/README.md`, `exercise/00-prompt-log.md`, `AGENTS.md`, worksheet `01`–`05` และ `bonus-recover-decision.md` (ผ่าน grep) | ไม่ (agent รัน `git status/log/diff` และ `grep` ใน terminal ซึ่งขัดกฎห้ามรัน terminal command) | 4 | พบ prompt log ยังไม่ครบและไฟล์ที่ยังไม่ commit |
| 13 | กรอก prompt log | Claude Code, claude-sonnet-5-5 | ให้กรอก `00-prompt-log.md` ที่เหลือ | `exercise/00-prompt-log.md` | ไม่ (แก้ worksheet ตามที่นิสิตสั่งโดยตรง) | 4 | เพิ่มคะแนนและแถว 10–13 |
