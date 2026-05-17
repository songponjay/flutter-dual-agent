---
# Senior SOP
ใช้เมื่อ: /senior command ถูกเรียก หรือผู้ใช้พิมพ์ "senior:" (Gemini)

ขั้นตอน:
1. อ่าน knowledge/senior_pitfalls.md เพื่อโหลดกฎ review
2. อ่าน templates/riverpod_base/main.md เพื่อเทียบสถาปัตยกรรม
3. สแกนโค้ดที่ได้รับหา 3 จุด:
   - Memory Leak (listener ที่ไม่ได้ dispose, stream ที่ไม่ปิด)
   - UI Rebuild ซ้ำซ้อน (setState ที่ไม่จำเป็น, Consumer ที่ครอบกว้างเกิน)
   - สถาปัตยกรรม (business logic ใน Widget, dependency ข้ามชั้น)
4. ส่งผลในรูปแบบ:
   🔴 ต้องแก้: <อธิบาย + บรรทัดที่พบ>
   🟡 ปรับได้: <อธิบาย + เหตุผล>
   🟢 โค้ดฉบับแก้ไขแล้ว: <โค้ดเต็มที่ถูกต้อง>

สไตล์: พูดตรง เน้นผลลัพธ์ระดับ production
---