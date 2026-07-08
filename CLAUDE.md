---
Project: Flutter & React Dual-Agent System (Teacher & Senior)
What this is: ระบบคู่หูพัฒนาทักษะ Flutter และ React แบ่งบทบาทผ่าน Slash Commands

กฎก่อนทำงาน:
- ก่อนแก้โครงสร้างโปรเจกต์หรือเขียนโค้ดตัวอย่าง ให้ขออนุญาตก่อน
- Flutter: โค้ดทุกชิ้นต้องรองรับ Null Safety และเขียนตาม Clean Architecture 2026
- React: ใช้ Functional Components + Hooks เท่านั้น ห้ามใช้ Class Components

โครงสร้าง:
- curriculum/       แผนการเรียนและประวัติ progress
- knowledge/        ข้อมูลอัปเดต (flutter_updates.md, react_updates.md, senior_pitfalls.md)
- templates/        โค้ด Boilerplate อ้างอิง
- prompts/          SOP กลางที่ทั้ง Teacher และ Senior ใช้ร่วมกัน
- .claude/commands/ slash command files

ห้าม:
- Flutter: เขียนโค้ด Deprecated หรือไม่รองรับ Null Safety
- React: ใช้ Class Components, componentDidMount, this.setState

Agent Voices:
- /teacher → อ่าน prompts/teacher-sop.md แล้วทำตามทุกขั้นตอน (สอนได้ทั้ง Flutter และ React)
- /senior  → อ่าน prompts/senior-sop.md แล้วทำตามทุกขั้นตอน (review ได้ทั้ง Flutter และ React)
Logic ทั้งหมดอยู่ใน prompts/ ห้าม hardcode พฤติกรรมซ้ำในไฟล์อื่น

ผู้เรียน:
- พื้นฐาน Java + Flutter (DayDone AI ครบ 8 modules)
- เรียน React เพื่อเตรียมสัมภาษณ์งาน
---
