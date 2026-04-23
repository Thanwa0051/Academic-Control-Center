# Academic Control Center - Complete Specification

**Document**: Comprehensive project specification combining TaskGraph, TechSpec, and ReAct Log  
**Date**: April 23, 2026  
**Status**: Complete

---

## Table of Contents
1. [TaskGraph - Work Breakdown](#taskgraph---work-breakdown)
2. [Technical Specification](#technical-specification)
3. [ReAct Log - Decision Trace](#react-log---decision-trace)

---

# TaskGraph - Work Breakdown

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

---

# Technical Specification

## 1. Product Overview
Academic Control Center is a local-first web app for students to manage class schedules and homework in one frictionless interface. The app runs entirely in the browser and stores all data in `localStorage`.

## 2. Scope
### In Scope
- Task management for homework and exams
- Weekly fixed schedule display
- Today view and weekly toggle for schedules
- Kanban board with 3 statuses
- Add/Edit/Delete/Move task actions
- Validation, toast feedback, and confirmation dialogs

### Out of Scope
- Backend or server API
- Authentication
- Drag and drop interaction
- Cross-device sync
- Push notifications
- Complex calendar or recurring schedule engine

## 3. Data Model

### Task
```json
{
  "id": 1,
  "title": "อ่านบทที่ 3",
  "subject": "Math",
  "dueDate": "2026-04-24",
  "priority": "high",
  "status": "todo",
  "createdAt": 1713840000000
}
```

### Schedule
```json
{
  "id": 1,
  "subject": "Physics",
  "dayOfWeek": 1,
  "startTime": "09:00",
  "room": "B204"
}
```

## 4. Storage Design
- Use one `localStorage` key for app state, for example `academic-control-center-state`
- Store tasks and schedules as JSON arrays inside a single state object
- Read on app start, write after every mutation
- If the stored payload is missing or invalid, initialize from defaults

## 5. UI Requirements

### Schedule Module
- Show today's classes based on current weekday
- Provide a toggle to switch between Today and Weekly views
- Weekly view shows all fixed classes grouped by day

### Kanban Board
- Three columns: To Do, Doing, Done
- Sort tasks by `dueDate` ascending inside each column
- Show status controls on each card

### Task Card
- Display subject, task title, due date, priority, and status action buttons
- Highlight tasks due today or tomorrow with a soft red background and warning icon

### FAB
- A floating circular `+` button fixed to the bottom-right
- Opens the input modal for new task entry

### Input Modal
- Fields: subject, title, due date, priority
- Use a simple modal overlay
- Support submit and cancel

## 6. Business Logic

### Auto-Sort
- Re-render the Kanban board after every state change
- Sort by `dueDate` before rendering cards

### Validation
- `dueDate` must be today or in the future
- Required fields must not be empty
- Invalid data must not be saved
- On validation failure, call `add_error_case` in console and show a toast message

### Delete Safety
- Single delete requires one confirmation modal
- Batch delete requires a second confirmation step before execution

### Persistence
- Every Add/Edit/Delete/Move operation must immediately update `localStorage`

## 7. Suggested Module Breakdown
- `state.js` for load/save and state transitions
- `dateUtils.js` for due-date checks and formatting
- `renderSchedule.js` for today and weekly schedule views
- `renderKanban.js` for board and card rendering
- `modal.js` for create/edit dialog behavior
- `notifications.js` for toast and error logging

## 8. Non-Functional Constraints
- Must work offline in a single browser session
- Must remain simple and stable, with minimal moving parts
- Must not depend on frameworks unless later explicitly added

## 9. Acceptance Rules
- The app starts without a backend
- Data survives refresh through `localStorage`
- Task ordering is deterministic and auto-sorted
- Dangerous actions are gated by confirmation dialogs
- Invalid inputs are blocked and reported

---

# ReAct Log - Decision Trace

## Step 1: Read the request
Need to create three documentation files from the provided project specification: `TaskGraph.md`, `TechSpec.md`, and `ReAct-log.md`.

## Step 2: Inspect workspace
Checked the workspace root and confirmed it is empty. No existing versions of the requested files were found, so there was no risk of overwriting user content.

## Step 3: Form a local implementation plan
The safest output is documentation that converts the spec into three distinct artifacts:
- `TaskGraph.md` for work breakdown and build order
- `TechSpec.md` for system design and requirements
- `ReAct-log.md` for reasoning trace and decision history

## Step 4: Create the files
Created all three markdown files with content derived directly from the specification, including:
- localStorage-only persistence
- no backend or authentication
- no drag and drop
- schedule today/weekly toggle
- 3-column Kanban board
- validation, confirmation flows, and error logging

## Step 5: Verify outcome
The workspace now contains the requested documentation files and no other project files were modified.

---

**End of Combined Specification**
