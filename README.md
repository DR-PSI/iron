# IronEye — ระบบเฝ้าระวังการใช้ไฟฟ้าผิดปกติ

ตรวจจับไฟรั่ว/ไฟหายในระบบจำหน่าย โดยเทียบหน่วยที่มิเตอร์หลักจ่ายออก กับผลรวมหน่วยที่มิเตอร์บ้านลูกค้าวัดได้ ส่วนต่างที่เกินเกณฑ์คือสัญญาณว่ามีการใช้ไฟที่ไม่ผ่านมิเตอร์

รองรับสองช่องทางพร้อมกัน: **RS485 ต่อสายตรง** และ **LoRaWAN ไร้สาย** เขียนลง Firebase โครงเดียวกัน เว็บอ่านรวมกันได้โดยไม่ต้องแยก

---

## ไฟล์ที่ใช้งานจริง

### หน้าเว็บ

| ไฟล์ | คืออะไร |
|---|---|
| `IronEye Dashboard.dc.html` | แดชบอร์ดหลัก 5 เมนู (ภาพรวม · บ้านลูกค้า · มิเตอร์ · มอนิเตอร์ · แจ้งเตือน) มีโหมด CONNECT อ่าน Firebase จริง และ SIMULATION สำหรับสาธิต · 3 ธีม |
| `System Communication.dc.html` | บล็อกไดอะแกรมอธิบายการสื่อสารทั้งเส้น 8 ชั้น พร้อมตาราง byte ไทม์ไลน์ และข้อจำกัด |
| `Node-RED RS485 Flow.dc.html` | ภาพอธิบาย flow ฝั่ง Node-RED |
| `github/index.html` | เวอร์ชันไฟล์เดียวพร้อม deploy ขึ้น GitHub Pages |

### แบบ LoRaWAN (ไร้สาย)

| ไฟล์ | คืออะไร |
|---|---|
| `simulator/rs485ln-at-commands.txt` | โปรแกรมที่ยิงใส่ตัวแปลง Dragino RS485-LN — คำสั่ง AT ครบชุด คัดลอกวางใน Tera Term ได้เลย |
| `simulator/rs485ln-decoder.js` | payload formatter สำหรับ TTN / ChirpStack แปลง 36 byte เป็นตัวเลข |
| `simulator/node-red-PAC3100-lora.json` | Node-RED flow รับ MQTT จาก TTN → ถอด payload → คำนวณหน่วย → ส่งเข้า n8n **มีปุ่มทดสอบด้วย payload ปลอม ใช้ได้โดยไม่ต้องมีอุปกรณ์** |
| `simulator/README-RS485-LN-LoRaWAN.md` | ขั้นตอนติดตั้งทั้งหมด ตั้งแต่ต่อสายจนขึ้นเว็บ |

### แบบ RS485 ต่อสายตรง

| ไฟล์ | คืออะไร |
|---|---|
| `simulator/node-red-PAC3100-com4.json` | Node-RED flow อ่าน PAC3100 ผ่าน USB-RS485 (COM4) ทุก 10 วินาที — flow ที่ทดสอบแล้วใช้งานได้ |
| `simulator/node-red-PAC3100-scan.json` | flow สแกนหา address / unit id / quantity ที่มิเตอร์ยอมตอบ ใช้ตอนต่อมิเตอร์ตัวใหม่ |
| `simulator/README-PAC3100-register.md` | ตาราง register จริงของ PAC3100 (ต่างจาก PAC3200) |

### ส่วนกลาง (ใช้ร่วมทั้งสองแบบ)

| ไฟล์ | คืออะไร |
|---|---|
| `simulator/n8n-workflow.json` | workflow วิเคราะห์ความผิดปกติ แล้วเขียน Firebase — ต้องใส่ secret ใน 3 โหนด |
| `simulator/firebase-rules.json` | กฎความปลอดภัยของ Realtime Database |
| `simulator/firebase-rtdb-sample.json` | ตัวอย่างโครงข้อมูล ใช้ import เพื่อทดสอบเว็บก่อนต่อของจริง |
| `simulator/README-interval-15min.md` | ที่มาของการเก็บรอบ 15 นาที และวิธีคิดดีมานด์ |

### ไฟล์เก่า เก็บไว้อ้างอิง

`README-PAC3200-test.md` · `README-RS485.md` · `README-modbusread.md` · `README-registers.md` · `node-red-pac3200-*.json` · `node-red-flow.json` — งานตอนที่ยังเข้าใจว่าเป็น PAC3200 และรุ่นทดลองก่อนหน้า ไม่ต้องใช้แล้ว

---

## เส้นทางข้อมูล

**แบบ LoRaWAN**
```
PAC3100 ──RS485 Modbus RTU──▶ RS485-LN ──LoRa AS923──▶ LPS8N
   ──UDP──▶ TTN ──MQTT──▶ Node-RED ──HTTP──▶ n8n ──REST──▶ Firebase ──▶ เว็บ
```

**แบบต่อสาย**
```
PAC3100 ──RS485 Modbus RTU──▶ USB-RS485 ──▶ Node-RED ──HTTP──▶ n8n ──REST──▶ Firebase ──▶ เว็บ
```

ทั้งสองเส้นบรรจบที่ Firebase โครงเดียวกัน แยกกันด้วย field `transport`

---

## ค่าที่อ่านได้

RS485-LN ส่งมา 36 byte ต่อรอบ (float32 big-endian)

| byte | ค่า | register |
|---|---|---|
| 0–11 | แรงดัน L1 L2 L3 | reg 0–5 |
| 12–23 | กระแส L1 L2 L3 | reg 12–17 |
| 24–35 | กำลังจริง L1 L2 L3 | reg 24–29 |

จาก 36 byte นี้คำนวณต่อได้: PF รายเฟส · PF รวม · กำลังปรากฏ · ความไม่สมดุลแรงดัน/กระแส · กระแสไหลย้อน · หน่วยที่ใช้ในรอบ · ดีมานด์และพีค

ยังอยู่ใต้เพดาน 51 byte ของ AS923 ที่ SF10 จึงส่งได้ใน uplink เดียว

---

## โครง Firebase

```
/sites/site01/devices/{meterId}
    ts          เวลาที่รับค่า (ISO string)
    transport   'lorawan' หรือ 'rs485'
    rssi, snr   ความแรงสัญญาณ (เฉพาะ LoRaWAN)
    phases/L1   { v, i, kw, pf }
    phases/L2   { v, i, kw, pf }
    phases/L3   { v, i, kw, pf }
    registers   ค่ารวมทั้งระบบ
    energy      { importKwh, intervalKwh, demandKw, peakDemandKw, slot }
```

---

## เริ่มใช้งาน

**ยังไม่มีอุปกรณ์** — import `node-red-PAC3100-lora.json` เข้า Node-RED แล้วกดปุ่ม inject "ทดสอบ (payload ปลอม)" flow ทั้งเส้นจะทำงานเหมือนของจริง

**มีอุปกรณ์แล้ว** — อ่าน `README-RS485-LN-LoRaWAN.md` ทำตามข้อ 1–6

**ต่อสายตรง** — import `node-red-PAC3100-com4.json` แก้พอร์ต COM ให้ตรง

---

## สถานะปัจจุบัน

ต่อ PAC-MAIN-01 (unit id 3) ผ่าน RS485/USB สำเร็จแล้ว ค่าไหลถึงเว็บครบทั้งเส้น
ฝั่ง LoRaWAN เขียนโปรแกรมและ flow ครบแล้ว รอ USB-TTL มาถึงเพื่อตั้งค่าตัวแปลง

**ยังไม่ได้ทำ** — เพิ่ม MTR-0142 และ MTR-0117 การตรวจจับไฟรั่วต้องมีมิเตอร์อย่างน้อย 2 ตัวจึงเริ่มคิด loss% ได้

---

## ข้อจำกัดที่ต้องรู้

- LoRaWAN ส่งทุก 15 นาที ค่าบนหน้าจอจะนิ่งระหว่างรอบ แถบสถานะลิงก์บอกเวลาที่รับล่าสุดไว้แทน
- Class A สั่งย้อนกลับได้เฉพาะช่วงสั้นหลัง uplink การเปลี่ยนค่าตั้งมีผลในรอบถัดไป
- กำลังรีแอกทีฟรายเฟส กระแสนิวทรัล แรงดัน line-to-line ไม่อยู่ใน 36 byte
- เกตเวย์หนึ่งตัวรับทุกมิเตอร์ เน็ตหลุดคือข้อมูลขาดทั้งไซต์
- อย่าตั้ง `meterId` ซ้ำกันระหว่างสอง flow ถ้ารันพร้อมกัน
