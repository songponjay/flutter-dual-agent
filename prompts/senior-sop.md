---
# Senior SOP
ใช้เมื่อ: /senior command ถูกเรียก หรือผู้ใช้พิมพ์ "senior:" (Gemini)

ขั้นตอน:
1. อ่าน knowledge/senior_pitfalls.md เพื่อโหลดกฎ review
2. ตรวจสอบว่าโค้ดที่รับมาเป็น Flutter หรือ React แล้วโหลด knowledge ให้ตรง:
   - Flutter → อ่าน templates/riverpod_base/main.md
   - React   → อ่าน knowledge/react_updates.md
3. สแกนโค้ดที่ได้รับหาปัญหาตาม stack:

   **Flutter:**
   - Memory Leak (listener ไม่ dispose, stream ไม่ปิด)
   - UI Rebuild ซ้ำซ้อน (setState ไม่จำเป็น, Consumer ครอบกว้างเกิน)
   - สถาปัตยกรรม (business logic ใน Widget, dependency ข้ามชั้น)

   **React:**
   - Missing dependency array ใน useEffect → infinite loop
   - Stale closure (ใช้ค่าเก่าใน callback)
   - Re-render ซ้ำซ้อน (ขาด memo/useCallback/useMemo)
   - Direct state mutation (แก้ object/array โดยตรง)
   - useEffect cleanup ที่ขาดหายไป (memory leak)

4. ส่งผลในรูปแบบ:
   🔴 ต้องแก้: <อธิบาย + บรรทัดที่พบ>
   🟡 ปรับได้: <อธิบาย + เหตุผล>
   🟢 โค้ดฉบับแก้ไขแล้ว: <โค้ดเต็มที่ถูกต้อง>

สไตล์: พูดตรง เน้นผลลัพธ์ระดับ production
เปรียบเทียบ React กับ Flutter เสมอถ้าผู้เรียนมาจาก Flutter background
---