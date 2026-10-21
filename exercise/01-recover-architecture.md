**ภาษา:** ภาษาไทย | [English](01-recover-architecture.en.md)

# งานที่ 1: กู้คืนสถาปัตยกรรม (ประมาณ 15 นาที)

> **เชื่อมโยงกับการบรรยาย:** T7 การกู้คืนสถาปัตยกรรม, AICH2 (การตามรอย), AICH3 (บริบท) และข้อค้นพบจาก JSS ว่า 93% ของงานวิจัยไม่เคยตรวจสอบผลลัพธ์จาก AI
> **คำถาม:** AI กู้คืนสถาปัตยกรรมได้มากแค่ไหน และขึ้นอยู่กับหลักฐานที่ AI ได้อ่านมากเพียงใด

**ระหว่างทำงานนี้ ห้ามเปิด repository ต้นฉบับของ Online Boutique** เพราะ README ของต้นฉบับมีแผนภาพสถาปัตยกรรมที่นิสิตกำลังกู้คืนอยู่

ในแต่ละ run ให้เริ่ม **แชตหรือ session ใหม่** แล้วส่งข้อความด้านล่างให้ agent โดย agent จะทำตาม [`prompts/1-recover-architecture.md`](../prompts/1-recover-architecture.md) และเขียนผลลัพธ์ไว้ที่ `exercise/outputs/1<run>-architecture.md`

**Run A: เฉพาะ interface**
```
Follow prompts/1-recover-architecture.md
run: A
evidence: system/online-boutique/protos/demo.proto
```

**Run B: interface + ไฟล์ตั้งค่าการ deploy**
```
Follow prompts/1-recover-architecture.md
run: B
evidence: system/online-boutique/protos/demo.proto, system/online-boutique/kubernetes-manifests/
```

ระหว่างที่ agent ทำงาน ให้สังเกตว่า agent เปิดไฟล์ใดบ้าง หาก agent อ่านไฟล์นอกเหนือจากหลักฐานที่อนุญาต ให้บันทึกไว้ใน prompt log

## ผลลัพธ์

- Run A: [`outputs/1A-architecture.md`](outputs/1A-architecture.md)
- Run B: [`outputs/1B-architecture.md`](outputs/1B-architecture.md)

เปิดแต่ละไฟล์ด้วย **Markdown: Open Preview** (`Ctrl+Shift+V` / `⇧⌘V`) เพื่อดูแผนภาพ

## ตรวจสอบ Run B ด้วยตัวเอง (ส่วนที่สำคัญที่สุด)

ตรวจสอบ dependency ในตารางของ Run B **อย่างน้อยห้ารายการ** เทียบกับไฟล์จริง โดยเปิดไฟล์ที่ AI อ้างถึงและหาหลักฐาน นิสิตสามารถตรวจซอร์สโค้ดของ service ได้ด้วย เช่น `src/checkoutservice/main.go`

| ผู้เรียก → ผู้ถูกเรียก | หลักฐานที่ AI อ้าง | ยืนยันแล้วหรือไม่ ✅ / ❌ / ❓ | สิ่งที่นิสิตพบ |
|---|---|---|---|
| End user → frontend | `frontend.yaml` (`frontend-external` LoadBalancer, port 80 → 8080; `httpGet /_healthz`) | ✅ | AI เช็คว่า Service kind ใช้ Type เป็น LoadBalancer ซึ่งถูกต้อง |
| loadgenerator → frontend | `loadgenerator.yaml` (`FRONTEND_ADDR=frontend:80`, `wget http://${FRONTEND_ADDR}`) | ✅ | AI เช็คว่าใน script ของ loadgenerator pod มีการ wget frontend address ซึ่งก็ตรงกับ port ที่ `frontend.yaml` expose |
| frontend → productcatalogservice | `frontend.yaml` (`PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550`) | ✅ | ใน `system/online-boutique/src/frontend/main.go` มีการอ่าน `PRODUCT_CATALOG_SERVICE_ADDR` และนำมาใช้ในการสร้าง gRPC client conection |
| checkoutservice → productcatalogservice | `checkoutservice.yaml` (`PRODUCT_CATALOG_SERVICE_ADDR`) | ✅ | ใน `system/online-boutique/src/checkoutservice/main.go` มีการอ่าน `PRODUCT_CATALOG_SERVICE_ADDR` และนำมาใช้สร้าง gRPC client connection |
| recommendationservice → productcatalogservice | `recommendationservice.yaml` (`PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550`) | ✅ | ใน `system/online-boutique/src/recommendationservice/recommendation_server.py` มีการอ่าน `PRODUCT_CATALOG_SERVICE_ADDR` และนำมาใช้สร้าง gRPC client connection |

- **สิ่งที่ผิดหรือขาดหายไป** ใน Run B (dependency ที่ไม่มีจริง, dependency ที่ขาดไป, container ที่น่าสงสัย):
  - `shoppingassistantservice`: `frontend.yaml` ตั้ง `SHOPPING_ASSISTANT_SERVICE_ADDR` และ `src/frontend/main.go` บังคับให้ต้องมีค่านี้ แต่ไม่มี manifest ของ service นี้ และไม่อยู่ใน diagram (AI ใส่ไว้แค่ใน Gaps)
  - `loadgenerator`: AI วาดเป็น container ของระบบ แต่ถูก comment ออกจาก `kustomization.yaml`
  - `COLLECTOR_SERVICE_ADDR`: `frontend` และ `checkoutservice` ใช้ส่ง tracing แต่ Run B ไม่มี (ไม่ได้เช็คว่าเปิดใช้จริงหรือไม่)

## สะท้อนคิด (ข้อละ 2–3 ประโยค)

1. เปรียบเทียบ A กับ B: agent **ไม่สามารถ**รู้อะไรได้จากไฟล์ `.proto` เพียงไฟล์เดียว และเพราะเหตุใด

- ไม่สามารถรู้ได้ว่า service ใด call service ใด เพราะในไฟล์ `.proto` บอกแค่ interface กลาง

2. ข้อผิดพลาด**ข้อใดข้อหนึ่ง**ที่จะทำให้สมาชิกใหม่ของทีมเข้าใจผิดมากที่สุด หากเผยแพร่แผนภาพนี้โดยไม่ตรวจสอบ

- ในแผนภาพ A ไม่ได้บอกว่า service มีการ call กันอย่างไร อาจทำให้เข้าใจผิดว่าแต่ละ service isolate ต่อกัน
