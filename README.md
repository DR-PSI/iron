# IronEye — 3 Phase Meter Web Dashboard

Dashboard ตรวจจับการขโมยไฟจากมิเตอร์ 3 เฟส (Sentron PAC3200) ผ่านเส้นทาง
**มิเตอร์ → Node-RED → n8n → Firebase RTDB → IronEye Web**

## ไฟล์
- `index.html` — เว็บทั้งหมดในไฟล์เดียว ไม่ต้องติดตั้งอะไร เปิดในเบราว์เซอร์ได้เลย
- `../simulator/` — flow จำลอง Node-RED, workflow n8n, โครงสร้างและ rules ของ Firebase

## เปิดใช้งานบน GitHub Pages
1. push โฟลเดอร์นี้ขึ้น repo
2. **Settings → Pages → Source: Deploy from a branch**
3. เลือก branch `main` และ folder `/github` (หรือย้าย `index.html` ไปไว้ root แล้วเลือก `/`)
4. เว็บจะขึ้นที่ `https://<user>.github.io/<repo>/`

## หน้าในระบบ
| เมนู | เนื้อหา |
|---|---|
| DB ภาพรวม | pipeline, KPI, กราฟสมดุลพลังงาน 24 ชม., ค่ามิเตอร์หลัก 3 เฟส |
| HM รายการบ้านลูกค้า | สถานะรายหลัง เรียงบ้านที่สงสัยขโมยไฟขึ้นก่อน |
| MT มิเตอร์ทั้งหมด | อุปกรณ์วัดทุกตัว พร้อมบ้านลูกค้าที่ผูกไว้ |
| AL แจ้งเตือน | เหตุการณ์ที่ตรวจพบ ส่งต่อ n8n และ LINE Notify |
| SG ตั้งค่ามิเตอร์ | เพิ่ม / แก้ไข / ลบมิเตอร์และบ้านลูกค้า |

หมายเหตุ: 1 มิเตอร์ = 1 บ้านลูกค้า · ข้อมูลในไฟล์นี้เป็นค่าจำลอง ยังไม่ผูกกับ Firebase จริง
