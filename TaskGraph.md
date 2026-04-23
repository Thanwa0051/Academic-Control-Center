# TaskGraph

## เป้าหมายงาน
สร้าง Web App "Academic Control Center" แบบ frictionless สำหรับรวมตารางเรียนและการบ้านไว้ในที่เดียว โดยใช้ HTML, CSS, และ JavaScript เท่านั้น พร้อมบันทึกข้อมูลลง `localStorage`.

## ลำดับงานหลัก
1. กำหนดโครงสร้างข้อมูล
   - Tasks: `id`, `title`, `subject`, `dueDate`, `priority`, `status`, `createdAt`
   - Schedules: `id`, `subject`, `dayOfWeek`, `startTime`, `room`
2. ออกแบบ layout หลัก
   - ส่วนสรุปวันนี้
   - Schedule Module
   - Kanban Board 3 คอลัมน์
   - FAB เพิ่มข้อมูล
   - Modal เพิ่ม/แก้ไขข้อมูล
3. สร้าง persistence layer
   - โหลดข้อมูลจาก `localStorage`
   - บันทึกกลับทุกครั้งหลัง Add/Edit/Delete/Move
4. สร้าง logic การจัดเรียงและการแสดงผล
   - Kanban auto-sort ตาม `dueDate`
   - Highlight งานที่ครบกำหนดวันนี้/พรุ่งนี้
   - Toggle ระหว่าง Today Schedule และ Weekly Schedule
5. เพิ่ม HITL safety flow
   - Confirmation ก่อนลบงานเดี่ยว
   - Confirmation 2 ชั้นสำหรับ batch delete
   - Validate ข้อมูลก่อนบันทึก
   - แสดง toast และ log error เมื่อข้อมูลไม่ผ่าน

## งานย่อยตามโมดูล

### 1) Data & State
- กำหนด schema สำหรับ task และ schedule
- สร้างฟังก์ชัน `loadState`, `saveState`, `seedState` ถ้าข้อมูลยังไม่มี
- กำหนด helper สำหรับ sort, format date, และตรวจวัน

### 2) Schedule Module
- แสดงวิชาเรียนของวันปัจจุบัน
- มีปุ่ม toggle ไปยัง weekly view
- แสดงเวลา ห้องเรียน และลำดับคาบ

### 3) Kanban Board
- สร้าง 3 คอลัมน์: To Do, Doing, Done
- เรียงการ์ดตาม `dueDate` จากใกล้สุดไปไกลสุด
- การ์ดแสดง subject, title, due date, priority
- มีปุ่มเปลี่ยนสถานะการ์ด

### 4) Input Modal
- ฟอร์มเพิ่ม task ใหม่
- ฟิลด์ขั้นต่ำ: subject, title, dueDate, priority
- ตรวจสอบวันที่ต้องเป็นวันนี้หรืออนาคต

### 5) Delete & Safety
- แสดง confirmation modal ก่อนลบแต่ละครั้ง
- ล้างงาน Done ทั้งหมดต้องยืนยัน 2 ชั้น
- ถ้าลบไม่สำเร็จหรือข้อมูลผิด ให้แจ้งผู้ใช้ด้วย toast

## Acceptance Criteria
- ข้อมูลทั้งหมด persist ผ่าน `localStorage`
- Kanban sort อัตโนมัติทุกครั้งที่ render
- งานที่ due วันนี้หรือพรุ่งนี้ถูก highlight ชัดเจน
- ไม่มี backend, login, หรือ drag-and-drop
- ตารางเรียนเป็นแบบ fixed weekly only

## Suggested Build Order
1. State + localStorage
2. Render Schedule Module
3. Render Kanban Board
4. Add/Edit Modal
5. Status change actions
6. Delete confirmation flows
7. Validation + toast + error logging
