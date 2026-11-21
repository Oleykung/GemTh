# GE:M BOSS TIMER TH

ระบบเว็บไซต์ติดตามเวลาเกิดบอส Granado Espada M เซิร์ฟเวอร์ไทย พร้อม Discord Bot

## ฟีเจอร์
- เว็บไซต์สาธารณะเลือกเซิร์ฟเวอร์และดูเวลานับถอยหลัง
- แยกเวลาบอสตามเซิร์ฟเวอร์
- Admin API ตั้งเวลาเกิดบอส
- ระบบรายงานชุมชนต้องมีสมาชิก 2 คนขึ้นไปยืนยันก่อนเผยแพร่เวลา
- ป้องกันผู้รายงานยืนยันรายงานของตัวเอง และเก็บสถานะ/ประวัติการยืนยัน
- Discord แจ้งเตือนก่อนเกิด 10 นาที และ `@everyone` เมื่อถึงเวลา
- Slash commands: `/boss list`, `/boss check`, `/kill`, `/set`
- ป้องกันการแจ้งเตือนซ้ำด้วย unique notification event ในฐานข้อมูล

## 1) เตรียม Supabase
1. สร้างโปรเจกต์ที่ https://supabase.com
2. เปิด SQL Editor แล้วรันไฟล์ `sql/schema.sql`
3. คัดลอก Project URL และ `service_role` key จาก Project Settings > API
4. ห้ามนำ `service_role` key ไปใส่ในไฟล์หน้าเว็บหรือเปิดเผยต่อสาธารณะ

## 2) เตรียม Discord Bot
1. สร้าง Application/Bot ใน https://discord.com/developers/applications
2. เชิญบอทเข้าเซิร์ฟเวอร์ โดยให้สิทธิ์ View Channels, Send Messages และ Mention Everyone
3. เปิด Developer Mode แล้วคัดลอก Channel ID ของห้องแจ้งเตือน
4. ตั้ง `DISCORD_TOKEN`, `DISCORD_CLIENT_ID`, `DISCORD_CHANNEL_ID`
5. เพิ่ม Discord User ID ผู้ดูแลใน `DISCORD_ADMIN_IDS` คั่นด้วย comma และกำหนด `DISCORD_CHANNEL_ID`
6. รัน `npm run register` เพื่อลงทะเบียน Slash Commands

## 3) รันในเครื่อง
ต้องใช้ Node.js 20 ขึ้นไป

```bash
npm install
cp .env.example .env
# แก้ค่าใน .env ให้ครบ
npm run register
npm start
```

เปิด http://localhost:3000

Windows PowerShell ใช้ `Copy-Item .env.example .env` แทน `cp` ได้

## 4) Deploy ออนไลน์
ตัวอย่างนี้เตรียม `Dockerfile` และ `render.yaml` สำหรับ Render แล้ว
1. อัปโหลดโฟลเดอร์นี้ขึ้น GitHub repository ส่วนตัว
2. สร้าง Web Service บน Render จาก repository
3. เพิ่ม Environment Variables ตาม `.env.example`
4. เพิ่ม `DISCORD_CHANNEL_ID` และ `DISCORD_ADMIN_IDS` ใน Environment ของ Render ด้วย
5. Deploy แล้วตรวจ `/health`
6. ตั้งค่าโดเมนและ HTTPS ตามผู้ให้บริการ

บริการต้องทำงานต่อเนื่องเพื่อให้ Discord scheduler ส่งแจ้งเตือนได้ แนะนำ Web Service แบบ always-on ไม่ใช้ static hosting อย่างเดียว

## 5) ระบบยืนยันรายงานจากสมาชิก 2 คน
- รายงานใหม่จะอยู่ในสถานะ “รอยืนยัน” และยังไม่เปลี่ยนเวลาบอสสาธารณะ
- ผู้รายงานนับเป็นสมาชิกคนที่ 1; ต้องมีเบราว์เซอร์/รหัสสมาชิกอีกชุดมายืนยันเป็นคนที่ 2
- ผู้รายงานไม่สามารถกดยืนยันรายงานของตัวเองได้ และการยืนยันซ้ำจากรหัสเดิมจะไม่นับเพิ่ม
- เมื่อครบ 2 รหัสสมาชิก ระบบจึงเปลี่ยนสถานะเป็น “ยืนยันแล้ว” และอัปเดตตัวนับเวลาบอส
- เว็บไซต์สร้างรหัสสมาชิกแบบสุ่มเก็บไว้ใน localStorage ของเบราว์เซอร์ ไม่ได้ใช้บัญชีล็อกอิน ดังนั้นนี่เป็นการลดการปั่นรายงานระดับเบื้องต้น ไม่ใช่การยืนยันตัวตนที่กันคนใช้หลายอุปกรณ์/โหมดไม่ระบุตัวตนได้ทั้งหมด
- สำหรับระบบสาธารณะที่ต้องการความน่าเชื่อถือสูง ควรเพิ่ม Discord/OAuth login, จำกัดสิทธิ์ตามบัญชี และเพิ่มปุ่มแจ้งรายงานผิด/การตรวจสอบโดยผู้ดูแล

### อัปเกรดฐานข้อมูลเดิม
หากติดตั้งเวอร์ชันก่อนหน้านี้แล้ว ให้เปิด Supabase SQL Editor และรัน `sql/verification-migration.sql` หนึ่งครั้งก่อน deploy โค้ดเวอร์ชันนี้ รายงานเก่าจะถูกเก็บไว้เป็น legacy-confirmed เพื่อไม่ทำให้เวลาที่เคยเผยแพร่หายไป

## 6) ตั้งค่าเซิร์ฟเวอร์และบอส
- ตาราง `ge_servers` เว้นว่างไว้ให้สมาชิกเพิ่มชื่อเซิร์ฟเวอร์จริงจากหน้าเว็บ
- ตาราง `ge_bosses` มีชื่อบอส 12 ตัวจากรายการที่เคยรวบรวมไว้
- เพิ่มชื่อภาษาไทย, แผนที่ และ `respawn_minutes` หลังตรวจสอบข้อมูลจริง
- สร้างเวลาให้แต่ละเซิร์ฟเวอร์ผ่านหน้า Admin ในเว็บไซต์
- `/kill` บันทึกเวลารอบถัดไปจาก `respawn_minutes`; `/set` ใช้กำหนดนาทีเอง

## ข้อควรทราบก่อนเปิดใช้งานจริง
- ตารางเวลาเกิดและชื่อแผนที่ยังไม่ถูกยืนยันโดยเกม จึงไม่ใส่ค่าคาดเดาไว้ให้
- `/boss list` และ `/boss check` แสดงเวลาที่ตั้งไว้ในฐานข้อมูล
- ตั้งค่า `DISCORD_CHANNEL_ID` เพื่อเลือกห้องแจ้งเตือน
- ควรเพิ่มระบบล็อกอินผู้ดูแล, OAuth/Discord login สำหรับสมาชิก, rate limiting แบบกระจาย, audit log และ backup ก่อนเปิดให้ผู้เล่นจำนวนมากใช้งาน
- `@everyone` จะทำงานได้ต่อเมื่อบอทมี permission `Mention Everyone` และสมาชิกไม่ได้ปิดการแจ้งเตือนของเซิร์ฟเวอร์

## Environment variables
ดู `.env.example` สำหรับค่าที่ต้องกำหนดทั้งหมด
