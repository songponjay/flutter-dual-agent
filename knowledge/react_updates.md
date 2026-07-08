# React Knowledge Base (2025-2026)
> สำหรับผู้มีพื้นฐาน Flutter + Dart — เปรียบเทียบทุกแนวคิดกับ Flutter

---

## Core Concepts เทียบกับ Flutter

| React | Flutter | หมายเหตุ |
|---|---|---|
| Functional Component | StatelessWidget | ฟังก์ชันคืนค่า JSX แทน Widget |
| useState | setState / ValueNotifier | เก็บ state ภายใน component |
| useEffect | initState + dispose | side effects + cleanup |
| useContext / Context API | Provider / Riverpod | ส่งค่าข้าม tree |
| useRef | GlobalKey / ref | เข้าถึง DOM element |
| useMemo | computed value | cache ผลลัพธ์ที่ซับซ้อน |
| useCallback | ฟังก์ชัน stable ref | กัน re-render จาก callback |
| React.memo | const Widget | กัน rebuild ถ้า props ไม่เปลี่ยน |
| props | constructor parameters | ส่งข้อมูลจาก parent → child |
| key prop | key: Key() | บอก React/Flutter ว่า element ไหนคืออันเดิม |

---

## Hooks ที่ต้องรู้สำหรับสัมภาษณ์

### useState
```jsx
const [count, setCount] = useState(0);

// ❌ ผิด — mutate โดยตรง
count = 5;

// ✅ ถูก
setCount(5);
setCount(prev => prev + 1); // ใช้ updater function เมื่อค่าใหม่ขึ้นกับค่าเดิม
```

### useEffect
```jsx
useEffect(() => {
  // ทำงานหลัง render
  const sub = api.subscribe();

  return () => {
    // cleanup — เหมือน dispose() ใน Flutter
    sub.unsubscribe();
  };
}, [dependency]); // [] = ทำครั้งเดียวเหมือน initState
```

**Dependency Array Rules:**
- `[]` = run once (เหมือน initState)
- `[value]` = run ทุกครั้งที่ value เปลี่ยน
- ไม่ใส่ = run ทุก render (อันตราย)

### useCallback
```jsx
// ❌ ผิด — สร้าง function ใหม่ทุก render → child re-render
const handleClick = () => doSomething(id);

// ✅ ถูก — function เดิมถ้า id ไม่เปลี่ยน
const handleClick = useCallback(() => doSomething(id), [id]);
```

### useMemo
```jsx
// ❌ ผิด — คำนวณใหม่ทุก render
const result = expensiveCalc(data);

// ✅ ถูก — คำนวณใหม่เฉพาะเมื่อ data เปลี่ยน
const result = useMemo(() => expensiveCalc(data), [data]);
```

---

## Common Pitfalls (ที่ถามบ่อยในสัมภาษณ์)

### 1. Stale Closure
```jsx
// ❌ ผิด — count เป็น 0 ตลอด (stale)
useEffect(() => {
  const timer = setInterval(() => {
    console.log(count); // จะเป็น 0 เสมอ
  }, 1000);
  return () => clearInterval(timer);
}, []); // ลืมใส่ count ใน dependency

// ✅ ถูก
useEffect(() => {
  const timer = setInterval(() => {
    console.log(count);
  }, 1000);
  return () => clearInterval(timer);
}, [count]);
```

### 2. Direct State Mutation
```jsx
// ❌ ผิด — React ไม่รู้ว่า state เปลี่ยน
const [items, setItems] = useState([]);
items.push('new'); // mutate โดยตรง

// ✅ ถูก — สร้าง array ใหม่
setItems(prev => [...prev, 'new']);
```

### 3. useEffect Infinite Loop
```jsx
// ❌ ผิด — object ใหม่ทุก render → loop ไม่หยุด
const options = { page: 1 };
useEffect(() => {
  fetchData(options);
}, [options]); // options เปลี่ยนทุก render

// ✅ ถูก
useEffect(() => {
  fetchData({ page: 1 });
}, []); // หรือ useMemo สำหรับ options
```

---

## State Management

| วิธี | ใช้เมื่อ | เทียบ Flutter |
|---|---|---|
| useState | local state ใน component เดียว | setState |
| useContext | share state ไม่กี่ level | Provider |
| Zustand | global state เบา | Riverpod |
| Redux Toolkit | large-scale, complex state | BLoC |

---

## Interview Questions ที่ถามบ่อย

1. **Virtual DOM คืออะไร?**
   React เก็บ copy ของ DOM ไว้ใน memory เมื่อ state เปลี่ยนจะ diff กับของเดิมก่อน แล้ว update เฉพาะส่วนที่เปลี่ยน (เหมือน Flutter ที่ repaint เฉพาะ dirty widget)

2. **useEffect dependency array ทำงานยังไง?**
   React เปรียบ shallow equality ของทุกค่าใน array ถ้าค่าไหนเปลี่ยน → run effect ใหม่

3. **ความแตกต่างระหว่าง useMemo และ useCallback?**
   - useMemo → cache ผลลัพธ์ (value)
   - useCallback → cache ฟังก์ชัน (reference)

4. **React.memo ต่างจาก useMemo ยังไง?**
   React.memo = HOC ครอบ component ทั้งตัว ป้องกัน re-render เมื่อ props ไม่เปลี่ยน
   useMemo = cache ค่าภายใน component

5. **Key prop สำคัญยังไงใน list?**
   ช่วย React ระบุว่า element ไหนคืออันเดิมเมื่อ list เปลี่ยน ถ้าไม่ใส่อาจ re-render ผิดตัว
