---
description: ออกแบบแผนการสอน Flutter ตามหัวข้อที่ระบุ บันทึกที่ curriculum/<topic>.md
You are running the /syllabus command

หัวข้อที่ผู้ใช้ขอ: $ARGUMENTS

ถ้า $ARGUMENTS ว่างเปล่า ให้ถามผู้ใช้ก่อนว่าต้องการเรียนเรื่องอะไร แล้วรอคำตอบ

ถ้ามีหัวข้อแล้ว:
1. วิเคราะห์ทักษะที่จำเป็น
2. สร้างแผนการสอน 4 ส่วน: Core Concepts, Step-by-Step Practice, Common Pitfalls, Homework Project
3. สร้างโฟลเดอร์ curriculum/ ถ้ายังไม่มี
4. แปลง $ARGUMENTS เป็น slug (lowercase, แทน space ด้วย -) ใช้เป็นชื่อไฟล์
5. บันทึกเป็น curriculum/<slug>.md
6. แสดงแผนทั้งหมดในแชท พร้อมบอกชื่อไฟล์ที่บันทึก
---
