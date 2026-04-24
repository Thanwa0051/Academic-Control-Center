🛠️ Academic Control Center - Technical Specifications

1. Technology Stack

HTML Structure: Single-File Architecture (ทุกอย่างอยู่ใน index.html)

Styling: Tailwind CSS (ผ่าน Script Tag CDN)

Icons: Lucide Icons (ผ่าน Script Tag CDN lucide.createIcons())

JavaScript: Vanilla JS (ES6+), Event-driven

Storage: window.localStorage

2. Data Models (JSON Schema)

ระบบใช้ localStorage โดยมี Key หลัก 2 ประเภท:

acad_auth_users: เก็บข้อมูลรหัสผ่าน (Key-Value: {"username": "password"})

user_data_{username}: เก็บตารางเรียนและงานของผู้ใช้คนนั้นๆ

Schema of user_data_{username}

{
  "schedules": [
    {
      "id": "s1690000000000",       // (String) Unique ID สร้างจาก Date.now() หรือรหัสม็อค (เช่น 'm1', 't1')
      "subject": "ฟิสิกส์เพิ่มเติม",   // (String) ชื่อรายวิชา
      "day": "1",                   // (String) "0" = Unassigned/Pool, "1"-"5" = จันทร์-ศุกร์
      "time": "08:30",              // (String) เวลาเริ่มเรียน Format HH:MM
      "room": "Lab 1",              // (String) ห้องเรียน
      "difficulty": "hard",         // (String) "hard" | "medium" | "easy"
      "note": "ทบทวนเรื่องกฎนิวตัน"     // (String) บันทึกช่วยจำ
    }
  ],
  "tasks": [
    {
      "id": "t1690000000000",       // (String) Unique ID สร้างจาก Date.now() หรือรหัสม็อค (เช่น 'tk1')
      "title": "ทำแล็บเคมี",           // (String) ชื่องาน
      "subject": "เคมีเพิ่มเติม",        // (String) วิชาที่เกี่ยวข้อง (ดึงมาจาก schedules หรือพิมพ์เอง)
      "date": "2024-05-25",         // (String) วันที่กำหนดส่ง Format YYYY-MM-DD
      "priority": "hard",           // (String) "hard" | "medium" | "easy"
      "note": "ส่งวันอังคารหน้า",         // (String) บันทึกรายละเอียดงาน
      "status": "todo"              // (String) "todo" | "doing" | "done"
    }
  ]
}


3. UI & CSS Modifiers (รหัสสีและ CSS)

ระบบมีการใช้สีเพื่อบ่งบอก Difficulty (ตารางเรียน) และ Priority (งาน):

Hard / สำคัญมาก:

Tailwind: bg-red-50, text-red-600, border-l-red-500

Custom CSS: .diff-hard { border-left: 5px solid #ef4444 !important; }

Medium / ปานกลาง:

Tailwind: bg-amber-50, text-amber-600, border-l-amber-500

Custom CSS: .diff-medium { border-left: 5px solid #f59e0b !important; }

Easy / ทั่วไป:

Tailwind: bg-emerald-50, text-emerald-600, border-l-emerald-500

Custom CSS: .diff-easy { border-left: 5px solid #10b981 !important; }

Z-Index Hierarchy

เพื่อให้ Modal และ Tooltip ทำงานซ้อนทับกันได้โดยไม่มีปัญหา:

z-40: Floating Action Button (FAB)

z-50: Tooltips (.tooltip)

z-[100]: Auth Screen (หน้า Login/Register)

z-[250]: Weekly Overlay (หน้าจัดการตารางสัปดาห์)

z-[400]: Action Modals (เพิ่มงาน, เพิ่มวิชา)

z-[500]: Detail / Delete UI Overlay (หน้าต่างแสดงรายละเอียด)

4. Drag and Drop (DnD) Logic

นี่คือกลไกที่ซับซ้อนที่สุดของระบบ กรุณาทำความเข้าใจก่อนปรับแก้:

Drag Start (handleDragStart):

เมื่อผู้ใช้ลากการ์ดวิชา (Draggable) ฟังก์ชันจะเก็บ id ของวิชานั้นไว้ในตัวแปร Global draggedItemId

เพิ่มคลาส .dragging เพื่อลด Opacity

Drag Over / Leave: - พื้นที่เป้าหมาย (Drop Zones) คือ div#drop-day-1 ถึง div#drop-day-5

จะมีการเพิ่มคลาส .drag-over ให้เห็นพื้นที่วาง (Border dashed)

Drop (handleDrop):

ดึง item จาก data.schedules ด้วย draggedItemId

CONDITION A (ลากจาก Sidebar day === '0'):

จะเปิด Modal กึ่งกลางจอ (schedule-modal)

ตั้งค่า edit-id เป็น NEW_DROP_{day} เพื่อสื่อสารกับฟังก์ชัน Save ว่านี่คือการ Copy Item ใหม่

CONDITION B (ลากจากวันอื่น day > 0):

ทำการเปลี่ยนค่า item.day = day และเรียก saveData(data) ทันที

Save (schedule-form submit):

หาก id เริ่มด้วย NEW_DROP_ -> ใช้ data.schedules.push() โดยให้ id ใหม่ แต่ใช้เนื้อหาที่เพิ่งกรอกในฟอร์ม

5. Event Handling & Strict Mode Fallbacks

ฟังก์ชัน moveTask(e, id, dir): ต้องรับ parameter e (event) เข้ามาเพื่อทำการ e.stopPropagation() ป้องกัน Event Bubbling ไปกระตุ้น onclick ของกล่องคลุมข้างนอก (Detail Overlay)

document.querySelectorAll('.modal-animate').forEach(m => m.classList.add('hidden')): ใช้สำหรับปิดหน้าต่างเฉพาะส่วนที่ Pop-up ขึ้นมา โดยไม่ไปยุ่งกับ Container หลักเพื่อป้องกันบั๊กหน้าจอโปร่งใสบัง UI