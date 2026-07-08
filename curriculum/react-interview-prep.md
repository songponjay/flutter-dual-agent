# Haupcar Software Developer Test
> บริษัท ฮ้อปคาร์ จำกัด
> กำหนดส่ง: ศุกร์ที่ 10 ก.ค. 2569 ก่อนเที่ยง
> Stack: React.js + Node.js + Database + Git

---

## โจทย์สรุป
สร้างเว็บไซต์จัดการข้อมูลรถยนต์ของบริษัท ทดแทนการจดในสมุด

---

## Functional Requirements (ต้องทำให้ครบ)

| # | ฟีเจอร์ | รายละเอียด |
|---|---|---|
| 1 | Add Car | เพิ่มรถใหม่: ทะเบียน, ยี่ห้อ, รุ่น, หมายเหตุ |
| 2 | View All Cars | ดูรถทั้งหมดในระบบ |
| 3 | Edit Car | แก้ไขข้อมูลรถที่มีอยู่ |
| 4 | Delete Car | ลบรถออกจากระบบ |

---

## Technical Requirements

| # | Tech | รายละเอียด |
|---|---|---|
| 1 | Frontend | React.js (Functional + Hooks) |
| 2 | Backend | Node.js — แนะนำ Express.js |
| 3 | Database | SQLite / MySQL / MongoDB (เลือกตามถนัด) |
| 4 | UI Framework | Ant Design หรือ Bootstrap |
| 5 | Version Control | Git ตั้งแต่ต้นโปรเจค + push GitHub public |
| 6 | Docs | README วิธี setup + run |

---

## แผนเรียนและทำ (พุธ-ศุกร์)

### วันพุธ (วันนี้) — ตั้ง project + สอน React พื้นฐาน
- [ ] `/teacher react 1` — useState + useEffect + fetch API
- [ ] สร้าง React project (`npm create vite@latest`)
- [ ] สร้าง component แสดง list รถ (mock data ก่อน)

### วันพฤหัส — Backend + เชื่อม Frontend
- [ ] `/teacher react 2` — Node.js Express + CRUD API
- [ ] สร้าง Express server + endpoints ครบ 4 ตัว
- [ ] เชื่อม React กับ API (fetch/axios)
- [ ] ทำ Add / Edit / Delete ให้ทำงานได้จริง

### วันศุกร์ (ก่อน 12.00) — Polish + Submit
- [ ] UI ให้สวยด้วย Ant Design / Bootstrap
- [ ] เขียน README (setup + run instructions)
- [ ] git commit ทุก feature ตั้งแต่ต้น
- [ ] push GitHub public
- [ ] ส่งลิงก์ repo

---

## โครงสร้างโปรเจคที่แนะนำ

```
haupcar-car-management/
├── frontend/          ← React app
│   ├── src/
│   │   ├── components/
│   │   │   ├── CarList.jsx
│   │   │   ├── CarForm.jsx
│   │   │   └── CarTable.jsx
│   │   ├── App.jsx
│   │   └── main.jsx
│   └── package.json
├── backend/           ← Node.js API
│   ├── routes/
│   │   └── cars.js
│   ├── db/
│   │   └── database.js
│   ├── server.js
│   └── package.json
└── README.md
```

---

## API Endpoints ที่ต้องสร้าง

| Method | URL | หน้าที่ |
|---|---|---|
| GET | /api/cars | ดูรถทั้งหมด |
| POST | /api/cars | เพิ่มรถใหม่ |
| PUT | /api/cars/:id | แก้ไขรถ |
| DELETE | /api/cars/:id | ลบรถ |

---

## Car Data Model

```json
{
  "id": 1,
  "registration": "กข 1234",
  "brand": "Toyota",
  "model": "Camry",
  "notes": "รถผู้จัดการ",
  "created_at": "2026-07-10"
}
```

---

## Bonus Features (ถ้ามีเวลา — สร้างความประทับใจ)
- Search/Filter รถตามยี่ห้อหรือทะเบียน
- Pagination
- Confirmation dialog ก่อนลบ
- Toast notification เมื่อ add/edit/delete สำเร็จ

---

## README Template (ต้องส่งด้วย)

```markdown
# Haupcar Car Management System

ระบบจัดการข้อมูลรถยนต์ สร้างด้วย React.js + Node.js + SQLite

## Tech Stack
- Frontend: React.js + Vite + Ant Design
- Backend: Node.js + Express.js
- Database: SQLite

## Setup

### Backend
cd backend
npm install
node server.js
# รันที่ port 3001

### Frontend  
cd frontend
npm install
npm run dev
# รันที่ port 5173

## Features
- เพิ่ม/ดู/แก้ไข/ลบข้อมูลรถยนต์
- ...
```

---

*เริ่มด้วย `/teacher react 1` เพื่อเรียน React พื้นฐานก่อนลงมือทำ*
