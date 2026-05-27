# Quiz Log

## Module 8 — Dark/Light Mode + Portfolio Polish

**ข้อ 1:** ทำไมต้องลบ `const` ออกจาก `ProviderScope` เมื่อใส่ `overrides`?
- คำตอบที่ตอบ: C (ProviderScope ไม่รองรับ const เลย)
- เฉลย: B ❌
- อธิบาย: `const` ต้องการค่า compile-time แต่ `SharedPreferences.getInstance()` โหลดตอน runtime จึงใส่ใน const expression ไม่ได้

**ข้อ 2:** ถ้า hardcode `Colors.white` ใน Widget จะเกิดอะไรใน dark mode?
- คำตอบที่ตอบ: C (Flutter auto-convert ให้)
- เฉลย: B ❌
- อธิบาย: Flutter ไม่ auto-convert สี — ถ้า hardcode ไว้ Widget นั้นยังขาวอยู่ อาจกลายเป็น white-on-white อ่านไม่ออก ควรใช้ `Theme.of(context).colorScheme` แทนเสมอ

## Module 7 — Auth: Local Mock → Firebase

**ข้อ 1:** ทำไม GoRouter ต้องใช้ refreshListenable?
- คำตอบที่ตอบ: C (ให้ LoginScreen rebuild อัตโนมัติ)
- เฉลย: B ❌
- อธิบาย: refreshListenable บอก GoRouter ให้ re-evaluate redirect() ใหม่เมื่อ auth state เปลี่ยน ไม่ใช่ rebuild Widget

**ข้อ 2:** ถ้าอยากเปลี่ยนจาก MockAuthRepository เป็น FirebaseAuthRepository ต้องแก้ไฟล์ไหน?
- คำตอบที่ตอบ: B
- เฉลย: B ✅
- อธิบาย: แก้แค่ authRepositoryProvider บรรทัดเดียว เพราะ AuthRepository เป็น abstract class ทุกส่วนอื่นไม่รู้ว่าใช้ implementation ไหน

## Module 6 — Dynamic Excel Export

**ข้อ 1:** ควรใช้ directory ไหนก่อน share_plus?
- คำตอบที่ตอบ: C
- เฉลย: B (getTemporaryDirectory()) ❌
- อธิบาย: iOS sandbox ไม่อนุญาตให้ share ไฟล์จาก Documents โดยตรง ต้องใช้ tmp folder

**ข้อ 2:** workbook.encode() return null แล้วไม่เช็ค จะเกิดอะไร?
- คำตอบที่ตอบ: C
- เฉลย: C ✅
- อธิบาย: encode() คืนค่า List<int>? — ถ้า sheet ว่างจะ return null แล้ว ! operator crash ทันที

## Module 5 — AI Summary Integration

**ข้อ 1:** ทำไม `AiSummaryService` ถึงต้องอยู่ใน domain layer แทนที่จะอยู่ใน data layer?
- คำตอบที่ตอบ: B
- เฉลย: B ✅
- อธิบาย: domain layer ไม่ขึ้นกับ package ภายนอก — ถ้าวันหลังเปลี่ยนจาก Gemini เป็น OpenAI ไม่ต้องแก้ domain เลย

**ข้อ 2:** ถ้า Gemini API ตอบกลับ HTTP 429 (rate limit) app ควรทำอะไร?
- คำตอบที่ตอบ: A (แสดง error ทันที)
- เฉลย: B — retry อัตโนมัติหลัง delay แล้วค่อยแสดง error ถ้าหมด max attempts
- หมายเหตุ: ไม่นับผิด เพราะยังไม่ได้สอนเรื่อง 429 ก่อนถาม
- อธิบาย: 429 = "ส่ง request เร็วเกินไปชั่วคราว" ไม่ใช่ quota หมดถาวร — wait 1-2 วิแล้ว retry มักผ่านเลย
