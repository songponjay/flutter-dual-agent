---
Project: Flutter Dual-Agent System (Teacher & Senior)
What this is: ระบบคู่หูพัฒนาทักษะ Flutter แบ่งบทบาทผ่าน Slash Commands

กฎก่อนทำงาน:
- ก่อนแก้โครงสร้างโปรเจกต์หรือเขียนโค้ดตัวอย่าง ให้ขออนุญาตก่อน
- โค้ดทุกชิ้นต้องรองรับ Null Safety และเขียนตาม Clean Architecture 2026

โครงสร้าง:
- curriculum/       แผนการเรียนและประวัติ progress
- knowledge/        ข้อมูลอัปเดตจาก Deep Search (เติมมือทุกสัปดาห์)
- templates/        โค้ด Boilerplate อ้างอิง
- prompts/          SOP กลางที่ทั้ง Teacher และ Senior ใช้ร่วมกัน
- .claude/commands/ slash command files

ห้าม:
- เขียนโค้ด Deprecated หรือไม่รองรับ Null Safety เด็ดขาด

Agent Voices:
- /teacher → อ่าน prompts/teacher-sop.md แล้วทำตามทุกขั้นตอน
- /senior  → อ่าน prompts/senior-sop.md แล้วทำตามทุกขั้นตอน
Logic ทั้งหมดอยู่ใน prompts/ ห้าม hardcode พฤติกรรมซ้ำในไฟล์อื่น
---
