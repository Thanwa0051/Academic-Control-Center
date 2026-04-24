🧠 ReAct Log (Reasoning and Acting History)

Phase 1: Data Initialization Errors

Observation: ผู้ใช้รันสคริปต์ครั้งแรกแล้วเจอ Error TypeError: Cannot set properties of null (setting 'innerText') ทันที

Thought: Error นี้เกิดจาก Javascript พยายามเอาข้อมูลจาก localStorage ไปยัดใส่ DOM element (document.getElementById(...)) ที่ยังเรนเดอร์ไม่เสร็จ หรือ Element นั้นถูกคอมเมนต์ทิ้งไป

Action: เพิ่มฟังก์ชันเช็ค Null (if (element) element.innerText = ...) และปรับลำดับการเรียก render() ให้อยู่ถูกที่หลัง DOM โหลดเสร็จ

Result: หาย Error ระบบสามารถเรนเดอร์ UI ได้ปกติ

Phase 2: Persona Pivot (Teacher to Student Only)

Observation: ระบบแรกเริ่มมีการสร้างโหมด 'Teacher' และ 'Student' ปนกัน ผู้ใช้สั่งให้ตัดโหมด Teacher ทิ้งและเน้นทำเป็นแอปสำหรับนักเรียนอย่างเดียว พร้อมขอข้อมูลตัวอย่าง

Thought: ต้องลบ Logic สลับโหมดทิ้งทั้งหมด ลบการแยกระดับ Object ใน localStorage ให้เหลือแค่ระดับ User ชั้นเดียว จากนั้นเขียนฟังก์ชัน getMockData() (ต่อมาเปลี่ยนเป็น DEFAULT_DATA) ที่อัดแน่นด้วยตารางเรียนจริงของนักเรียน ม.ปลาย สายวิทย์-คณิต

Action: รื้อโครงสร้าง localStorage เล็กน้อย ลบปุ่ม Toggle สลับโหมด และเพิ่มข้อมูลวิชาฟิสิกส์, เคมี, ชีวะ, แคลคูลัส ลงไปใน Mock Data

Result: UI สะอาดขึ้น โฟกัสชัดเจนขึ้น ตอบโจทย์ผู้ใช้ได้ดีมาก

Phase 3: UX Improvement - Tooltips and Details

Observation: ผู้ใช้ต้องการให้เห็นรายละเอียดทั้งแบบชี้เมาส์ (Hover) และแบบคลิกดูเต็มๆ

Thought: ต้องออกแบบโครงสร้าง HTML แบบผสมผสาน สร้าง .subject-card-wrapper ครอบไว้

จังหวะ Hover -> CSS .tooltip จะ visibility: visible และ opacity: 1

จังหวะ Click -> ยิงฟังก์ชัน Javascript showDetails(type, id) เพื่อเปิด Full Overlay

Action: วางโค้ด Tooltip ซ้อนไว้ในการ์ดทุกใบ และปรับ Z-Index เป็น 50 เพื่อให้ลอยเหนือองค์ประกอบอื่น

Result: ได้ UX ที่ลื่นไหลมาก ผู้ใช้เห็นภาพรวมผ่าน Tooltip และดูโน้ตลึกๆ ผ่านการคลิก

Phase 4: The Complex Drag & Drop Issue (CRITICAL FIX)

Observation: ฟังก์ชันลากตารางเรียนมีบั๊ก เมื่อผู้ใช้ลากวิชาจาก "รายวิชาที่มี (Sidebar)" ลงไปในวันจันทร์-ศุกร์ วิชาในตาราง Sidebar หายไปเลย (โดน Move ไม่ใช่ Clone) และไม่สามารถตั้งเวลา/ห้องเรียนที่แตกต่างกันในแต่ละวันได้

Thought: 1. วิชารายสัปดาห์บางวิชาเรียนหลายวัน แต่วันละห้อง วันละเวลา
2. แปลว่า "รายวิชาที่มี (Sidebar)" ต้องทำตัวเป็นแค่ "แม่แบบ (Template)"
3. ถ้าลากจาก Template ลง Grid ต้อง Clone ข้อมูล, เปิด Modal ให้กรอกเวลา/ห้อง, แล้วสร้าง ID ใหม่
4. แต่ถ้าลากวิชาที่อยู่ใน Grid อยู่แล้ว (เช่น ย้ายจากจันทร์ไปอังคาร) อันนี้คือ Move ปกติ ไม่ต้องเด้ง Modal

Action: เขียน Logic ดักจับใน handleDrop(e, day)

ตรวจสอบ item.day === '0' (0 หมายถึงอยู่ใน Sidebar)

ถ้าเป็น 0: เซ็ต edit-id เป็น "NEW_DROP_" + day เปิด Modal

ถ้ากรอก Modal เสร็จกด Save -> เช็คค่า "NEW_DROP_" แล้วทำการ data.schedules.push({ ...ใหม่ })

ถ้าเป็น 1-5 (Move ภายใน Grid): ให้อัปเดต item.day = day และ Save เลย

Result: ระบบทำตารางสัปดาห์ทำงานได้ยืดหยุ่นและสมบูรณ์แบบเทียบเท่าโปรแกรมจัดตารางสอนระดับสูง

Phase 5: Event Bubbling & Modal Visibility Bugs (CRITICAL FIX)

Observation: ผู้ใช้แจ้งว่า 1) ระบบค้างกดอะไรไม่ได้หลังใช้งานไปสักพัก 2) กดย้าย Task ด้วยปุ่มลูกศรไม่ได้

Thought: - บั๊ก 1 (ระบบค้าง): เกิดจากฟังก์ชัน closeModals() ดันไปเขียนสั่งปิดทุกอย่างที่มีคลาส .modal-animate ซึ่งโครงสร้างหลักบางตัวดันมีคลาสนี้อยู่ด้วย ทำให้โดนซ่อนจนเหลือแต่ Layer พื้นหลังสีดำใส (Invisible Overlay) บังจออยู่ จึงกดคลิกอะไรทะลุลงไปไม่ได้

บั๊ก 2 (ปุ่มลูกศรย้ายงาน): โค้ดเดิมเขียนปุ่มซ้อนอยู่ใน Card ที่มี onclick="showDetails()" อยู่ ทำให้พอกดปุ่ม มันเกิด Event Bubbling เด้งไปกระตุ้น Card ด้วย และการสั่ง onclick="event.stopPropagation()" ดิบๆ บน HTML ในบาง Browser ทำให้เกิด Syntax Error หรือหา Event ไม่เจอจนสคริปต์หยุดทำงาน (Halt)

Action: - แก้บั๊ก 1: เปลี่ยน closeModals() ให้ระบุ ID ไปเลยว่าให้ปิดแค่ 'modal-overlay', 'task-modal', 'schedule-modal' อย่าไปใช้การ Loop หาคลาสปิดมั่วซั่ว

แก้บั๊ก 2: เปลี่ยนการเรียกฟังก์ชันบนปุ่มลูกศรให้เป็น onclick="moveTask(event, 'id', dir)" โดยการส่ง event ตรงเข้าไปที่ฟังก์ชัน และใน Javascript ค่อยสั่ง if (e) e.stopPropagation(); อย่างปลอดภัย

Result: อาการค้างหายขาด 100% กดย้าย Task ได้ลื่นไหล ไม่พัง