# Node.js + Express Knowledge Base
> สำหรับโจทย์ Haupcar — CRUD Car Management REST API
> เปรียบเทียบกับสิ่งที่รู้ใน Flutter เสมอ

---

## Node.js เทียบกับ Flutter

| Node.js / Express | Flutter | หมายเหตุ |
|---|---|---|
| `server.js` | `main.dart` | entry point |
| `express()` | `MaterialApp` | app instance |
| `app.listen(3001)` | `runApp()` | เริ่ม app |
| `router.get('/cars')` | route handler | ตอบ request |
| `req.body` | form data | รับข้อมูลจาก client |
| `res.json({})` | `return` JSON | ส่งผลกลับ |
| `SQLite / better-sqlite3` | sqflite | database |
| `cors()` middleware | — | อนุญาต React เรียก API ได้ |

---

## Setup โปรเจค Backend

```bash
mkdir backend
cd backend
npm init -y
npm install express better-sqlite3 cors
```

**package.json scripts:**
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js"
  }
}
```

---

## โครงสร้าง Backend

```
backend/
├── db/
│   └── database.js      ← setup SQLite + create table
├── routes/
│   └── cars.js          ← CRUD endpoints
└── server.js            ← entry point
```

---

## db/database.js — ตั้งค่า SQLite

```javascript
const Database = require('better-sqlite3');
const path = require('path');

// สร้างไฟล์ cars.db ใน folder db/
const db = new Database(path.join(__dirname, 'cars.db'));

// สร้าง table ถ้ายังไม่มี (เหมือน migration ใน Flutter sqflite)
db.exec(`
  CREATE TABLE IF NOT EXISTS cars (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    registration TEXT NOT NULL,
    brand       TEXT NOT NULL,
    model       TEXT NOT NULL,
    notes       TEXT,
    created_at  TEXT DEFAULT (datetime('now'))
  )
`);

module.exports = db;
```

---

## routes/cars.js — CRUD Endpoints ครบ 4 ตัว

```javascript
const express = require('express');
const router = express.Router();
const db = require('../db/database');

// GET /api/cars — ดูรถทั้งหมด
// เหมือน SELECT * FROM cars ใน sqflite
router.get('/', (req, res) => {
  const cars = db.prepare('SELECT * FROM cars ORDER BY id DESC').all();
  res.json(cars);
});

// GET /api/cars/:id — ดูรถ 1 คัน
router.get('/:id', (req, res) => {
  const car = db.prepare('SELECT * FROM cars WHERE id = ?').get(req.params.id);
  if (!car) return res.status(404).json({ error: 'Car not found' });
  res.json(car);
});

// POST /api/cars — เพิ่มรถใหม่
// req.body มาจาก JSON ที่ React ส่งมา
router.post('/', (req, res) => {
  const { registration, brand, model, notes } = req.body;

  // validate
  if (!registration || !brand || !model) {
    return res.status(400).json({ error: 'registration, brand, model are required' });
  }

  const result = db.prepare(
    'INSERT INTO cars (registration, brand, model, notes) VALUES (?, ?, ?, ?)'
  ).run(registration, brand, model, notes || '');

  // ส่งคืนรถที่เพิ่งเพิ่ม
  const newCar = db.prepare('SELECT * FROM cars WHERE id = ?').get(result.lastInsertRowid);
  res.status(201).json(newCar);
});

// PUT /api/cars/:id — แก้ไขรถ
router.put('/:id', (req, res) => {
  const { registration, brand, model, notes } = req.body;
  const { id } = req.params;

  const car = db.prepare('SELECT * FROM cars WHERE id = ?').get(id);
  if (!car) return res.status(404).json({ error: 'Car not found' });

  db.prepare(
    'UPDATE cars SET registration=?, brand=?, model=?, notes=? WHERE id=?'
  ).run(registration, brand, model, notes || '', id);

  const updated = db.prepare('SELECT * FROM cars WHERE id = ?').get(id);
  res.json(updated);
});

// DELETE /api/cars/:id — ลบรถ
router.delete('/:id', (req, res) => {
  const { id } = req.params;
  const car = db.prepare('SELECT * FROM cars WHERE id = ?').get(id);
  if (!car) return res.status(404).json({ error: 'Car not found' });

  db.prepare('DELETE FROM cars WHERE id = ?').run(id);
  res.json({ message: 'Deleted successfully', id: Number(id) });
});

module.exports = router;
```

---

## server.js — Entry Point

```javascript
const express = require('express');
const cors = require('cors');
const carsRouter = require('./routes/cars');

const app = express();
const PORT = 3001;

// Middleware
app.use(cors());              // อนุญาต React (port 5173) เรียก API ได้
app.use(express.json());      // parse req.body เป็น JSON

// Routes
app.use('/api/cars', carsRouter);

// Health check
app.get('/', (req, res) => {
  res.json({ message: 'Haupcar API running' });
});

app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

---

## ทดสอบ API ด้วย curl

```bash
# ดูรถทั้งหมด
curl http://localhost:3001/api/cars

# เพิ่มรถ
curl -X POST http://localhost:3001/api/cars \
  -H "Content-Type: application/json" \
  -d '{"registration":"กข 1234","brand":"Toyota","model":"Camry","notes":"รถผู้จัดการ"}'

# แก้ไขรถ id=1
curl -X PUT http://localhost:3001/api/cars/1 \
  -H "Content-Type: application/json" \
  -d '{"registration":"กข 1234","brand":"Toyota","model":"Camry 2024","notes":"อัปเดต"}'

# ลบรถ id=1
curl -X DELETE http://localhost:3001/api/cars/1
```

---

## React เรียก API (fetch)

```jsx
// GET — ดึงรถทั้งหมด
useEffect(() => {
  fetch('http://localhost:3001/api/cars')
    .then(res => res.json())
    .then(data => setCars(data));
}, []);

// POST — เพิ่มรถ
const addCar = async (carData) => {
  const res = await fetch('http://localhost:3001/api/cars', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(carData),
  });
  const newCar = await res.json();
  setCars(prev => [newCar, ...prev]); // เพิ่มหัว list
};

// PUT — แก้ไขรถ
const updateCar = async (id, carData) => {
  const res = await fetch(`http://localhost:3001/api/cars/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(carData),
  });
  const updated = await res.json();
  setCars(prev => prev.map(car => car.id === id ? updated : car));
};

// DELETE — ลบรถ
const deleteCar = async (id) => {
  await fetch(`http://localhost:3001/api/cars/${id}`, { method: 'DELETE' });
  setCars(prev => prev.filter(car => car.id !== id));
};
```

---

## Common Pitfalls Backend

### 1. ลืม cors() → React เรียก API ไม่ได้
```javascript
// ❌ ผิด — ไม่มี cors
app.use(express.json());

// ✅ ถูก — cors ต้องมาก่อน routes
app.use(cors());
app.use(express.json());
```

### 2. ลืม express.json() → req.body เป็น undefined
```javascript
// ❌ ผิด
router.post('/', (req, res) => {
  console.log(req.body); // undefined!
});

// ✅ ถูก — ใน server.js ต้องมี
app.use(express.json());
```

### 3. req.params.id เป็น string ไม่ใช่ number
```javascript
// ❌ ผิด
const id = req.params.id; // "1" (string)
if (id === 1) ...         // false เสมอ

// ✅ ถูก
const id = Number(req.params.id); // 1 (number)
// หรือใช้ better-sqlite3 ก็ตาม type ของ column เอง
```
