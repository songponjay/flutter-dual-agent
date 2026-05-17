---
# Senior Pitfalls
<!-- เพิ่มรายการใหม่ด้านบนสุด ตาม template ใน README.md -->

## การใช้ ref.watch() ผิดบริบทนอกเมธอด build()
- อาการ: แอปพลิเคชันเกิด Runtime Crash, เกิด Infinite Loop หรือข้อมูลอัปเดตผิดเพี้ยนคาดเดาไม่ได้เมื่อมีการสลับหน้าจอ
- สาเหตุ: เรียกใช้ `ref.watch()` ภายใน Event Callback (เช่น `onPressed`) หรือใน Lifecycle (เช่น `initState`) ซึ่งทำให้บิดเบือนกลไกการ Rebuild ของระบบ
- วิธีแก้: เปลี่ยนไปใช้ `ref.read()` เพื่อดึงค่า snapshot ณ เวลานั้น หรือเรียกผ่าน Notifier แทน

```dart
// ❌ โค้ดตัวอย่างส่วนที่มีปัญหา
onPressed: () {
  final cart = ref.watch(cartNotifierProvider); // ผิดบริบท
  cart.addToCart('PROD_001');
}

// ✅ โค้ดตัวอย่างที่แก้ไขแล้วอย่างถูกต้อง
onPressed: () {
  ref.read(cartNotifierProvider.notifier).addToCart('PROD_001'); // ถูกต้อง
}
ปัญหาการเกิดสภาวะแข่งขัน (Race Conditions) ในการดึงข้อมูล Async
อาการ: หน้าจอแสดงข้อมูลสลับกันไปมา ผิดหมวด ผิดแท็บ เมื่อผู้ใช้กดเปลี่ยนฟิลเตอร์หรือแท็บอย่างรวดเร็ว

สาเหตุ: ยิง Request แบบ Async ชิ้นใหม่ไปโดยไม่ได้ยกเลิก (Cancel) Request ตัวเก่าที่กำลังโหลดค้างอยู่ ทำให้ผลลัพธ์จากเครือข่ายที่มาถึงช้ากว่าวิ่งกลับมาเขียนสเตตทับค่าล่าสุด

วิธีแก้: ใช้ CancelToken ของ Dio ร่วมกับ ref.onDispose() เพื่อยกเลิก Request เก่าทันทีเมื่อมีการเปลี่ยนค่า หรือใช้ Parameterized Provider (Family)

Dart
// ❌ โค้ดตัวอย่างส่วนที่มีปัญหา
Future<void> fetchFilteredData(String category) async {
  state = const AsyncValue.loading();
  final data = await ref.read(apiServiceProvider).getData(category); // ไม่มีการ Cancel ตัวเก่า
  state = AsyncValue.data(data);
}

// ✅ โค้ดตัวอย่างที่แก้ไขแล้วอย่างถูกต้อง
@riverpod
Future<List<Item>> fetchItemsByCategory(FetchItemsByCategoryRef ref, String category) async {
  final cancelToken = CancelToken();
  ref.onDispose(() => cancelToken.cancel()); // ยกเลิกอัตโนมัติเมื่อพารามิเตอร์เปลี่ยน
  return ref.watch(apiServiceProvider).getData(category, cancelToken);
}
StreamSubscription ที่ไม่ dispose
อาการ: memory ค่อยๆ เพิ่มขึ้นเมื่อเปลี่ยนหน้าไปมา (Memory Leak) จนแอปค้างหรือดับไปเอง

สาเหตุ: subscribe ฟังข้อมูลจาก Stream หรือใช้งาน Controller แล้วไม่ cancel หรือเคลียร์ทิ้งในลูปวงจรชีวิต

วิธีแก้: เก็บ subscription ไว้ใน variable แล้วเรียก _sub.cancel() และ _controller.dispose() ใน dispose()

Dart
// ❌ โค้ดตัวอย่างส่วนที่มีปัญหา
// เปิดใช้งานใน initState แต่ไม่มีการล้างพลังงานในลูปทำลาย Widget
_subscription = Stream.periodic(const Duration(seconds: 1)).listen((val) => print(val));

// ✅ โค้ดตัวอย่างที่แก้ไขแล้วอย่างถูกต้อง
@override
void dispose() {
  _subscription?.cancel(); // ยกเลิกการฟังข้อมูล
  _searchController.dispose(); // ล้างหน่วยความจำ Controller
  super.dispose();
}
Consumer ครอบ Widget ทั้งหน้า
อาการ: UI rebuild ทั้งหน้าทุกครั้งที่ state เปลี่ยน เกิดอาการกระตุก (UI Jank) และเฟรมเรตร่วง

สาเหตุ: Consumer หรือ watch() อยู่สูงเกินไปใน widget tree หรือเฝ้าดู Object ขนาดใหญ่โดยไม่เจาะจงฟิลด์

วิธีแก้: ย้าย Consumer ให้ครอบเฉพาะ Widget ที่ต้องการข้อมูลนั้นจริงๆ หรือใช้ .select() เพื่อเฝ้าดูเฉพาะฟิลด์ที่ใช้งาน

Dart
// ❌ โค้ดตัวอย่างส่วนที่มีปัญหา
Widget build(BuildContext context, WidgetRef ref) {
  final userProfile = ref.watch(profileProvider); // ฟังทั้งก้อน เปลี่ยนฟิลด์เดียว Rebuild ทั้งหน้า
  return Scaffold(body: Column(children: [Text(userProfile.name), const DeepComplexWidget()]));
}

// ✅ โค้ดตัวอย่างที่แก้ไขแล้วอย่างถูกต้อง
Widget build(BuildContext context, WidgetRef ref) {
  // ใช้ .select เพื่อเจาะจงเฝ้าดูเฉพาะฟิลด์ name เท่านั้น ฟิลด์อื่นเปลี่ยน หน้าจอไม่ Rebuild ซ้ำซ้อน
  final userName = ref.watch(profileProvider.select((p) => p.name));
  return Scaffold(body: Column(children: [Text(userName), const DeepComplexWidget()]));
}
## การละเลยการล้างสเตต (Missing autoDispose) บนข้อมูลชั่วคราว
- อาการ: เมื่อผู้ใช้เปิดหน้าเดิมซ้ำ ข้อมูลเก่าที่เคยโหลดไว้จะแสดงค้างขึ้นมาก่อน 1-2 วินาที ก่อนที่ข้อมูลใหม่จะโหลดเสร็จ หรือเกิดการกินทรัพยากรเบื้องหลังตลอดเวลา
- สาเหตุ: ใช้ Provider แบบปกติ (ไม่มี `autoDispose` หรือเปิด `keepAlive: true` บน Notifier) กับข้อมูลชั่วคราวระดับหน้าจอ (เช่น หน้าดีเทลสินค้า) ทำให้เมื่อผู้ใช้ปิดหน้านั้นไปแล้ว สเตตก็ยังคงถูกเก็บไว้ในหน่วยความจำ ไม่ถูกทำลายทิ้ง
- วิธีแก้: ใช้ `@riverpod` เพื่อให้ระบบใส่ `autoDispose` เป็นค่าเริ่มต้น หรือระบุเจาะจงเพื่อให้ล้างสเตตทับทันทีเมื่อไม่มี Widget ไหนคอยเฝ้าดูแล้ว

```dart
// ❌ โค้ดตัวอย่างส่วนที่มีปัญหา
// ข้อมูลถูกขังไว้ในหน่วยความจำตลอดไป แม้จะปิดหน้าจอไปแล้ว
@Riverpod(keepAlive: true) 
class ProductDetailNotifier extends _$ProductDetailNotifier { ... }

// ✅ โค้ดตัวอย่างที่แก้ไขแล้วอย่างถูกต้อง
// สเตตจะถูกเคลียร์ทิ้งทันทีเมื่อผู้ใช้กดย้อนกลับ (Unmount Widget)
@riverpod
class ProductDetailNotifier extends _$ProductDetailNotifier { ... }
ปัญหาสเตตตีกันจากการแก้ไขข้ามโมดูล (Cross-Provider Side Effects) ในเมธอด build
อาการ: แอปพลิเคชันแจ้งเตือนข้อผิดพลาดเกี่ยวกับการเรียกใช้สเตตซ้อนกัน หรือเกิดบั๊กสเตตเพี้ยนเมื่อโมดูลที่เกี่ยวข้องกันสองตัวอัปเดตพร้อมกัน

สาเหตุ: มีการไปสั่งแก้ไขค่า (Modify/Write) สเตตของ Provider ตัวอื่น ภายในเมธอด build() หรือภายในกระบวนการคิดคำนวณสเตตของตัวเอง (ซึ่งเป็นช่วงเวลาของ Read-Only เท่านั้น)

วิธีแก้: ห้ามสร้างผลกระทบข้างเคียง (Side Effects) ในเมธอดสร้างสเตตเด็ดขาด หากต้องการแก้ไขสเตตอื่น ให้เรียกผ่าน Event Callback หรือรอให้ Frame เรนเดอร์เสร็จก่อนโดยใช้ Future.microtask

Dart
// ❌ โค้ดตัวอย่างส่วนที่มีปัญหา
@riverpod
class AuthNotifier extends _$AuthNotifier {
  @override
  AuthState build() {
    // ผิดมหันต์: ห้ามแอบไปแก้สเตตอื่นขณะที่ตัวเองกำลังสร้างสเตต
    ref.read(logProvider.notifier).addLog("Auth initialized"); 
    return AuthState.unauthenticated();
  }
}

// ✅ โค้ดตัวอย่างที่แก้ไขแล้วอย่างถูกต้อง
@riverpod
class AuthNotifier extends _$AuthNotifier {
  @override
  AuthState build() {
    // ถูกต้อง: ใช้ Future.microtask เพื่อสั่งทำงานในคิวถัดไปหลังจากกระบวนการสร้างสเตตปัจจุบันเสร็จสิ้น
    Future.microtask(() {
      ref.read(logProvider.notifier).addLog("Auth initialized");
    });
    return AuthState.unauthenticated();
  }
}
ปัญหา Rebuild พร่ำเพื่อจากครอบสเตตประเภท List หรือ Map (Object Identity Mismatch)
อาการ: หน้าจอลิสต์รายการกระตุกหนักมาก ทั้งๆ ที่ระบุเฝ้าดูสเตตด้วย .select() หรือแยกคลาสย่อยแล้ว

สาเหตุ: มีการส่งคืน Collection (List/Map) ตัวใหม่ที่เกิดจากการจัดเรียงข้อมูล (Sorting/Filtering) ในเมธอด build ตลอดเวลา แม้ว่าไส้ในของข้อมูลจะเหมือนเดิมเป๊ะ แต่ Dart ถือว่าเป็น Object คนละตัว (Identity Mismatch) ทำให้เข้าใจว่ามีข้อมูลใหม่และสั่ง Rebuild ซ้ำซ้อน

วิธีแก้: ใช้ตัวเทียบค่าแบบลึก (Deep Equality) ผ่านแพ็กเกจ collection (ฟังก์ชัน ListEquality().equals) เพื่อเช็กเนื้อข้อมูลภายในจริงๆ แทนการเช็ก Reference ของ Object

Dart
// ❌ โค้ดตัวอย่างส่วนที่มีปัญหา
// ทุกครั้งที่ยอดเงินเปลี่ยน list ตัวนี้จะถูกกรองใหม่ ได้ Reference ใหม่ -> บังคับ Rebuild ปุ่มนี้ทุกรอบ!
final activeTodoIds = ref.watch(todoListProvider.select((list) => list.where((t) => !t.isDone).map((t) => t.id).toList()));

// ✅ โค้ดตัวอย่างที่แก้ไขแล้วอย่างถูกต้อง
// ใช้ตัวกรองในระดับ Provider แยกต่างหาก หรือใช้ตัวเทียบค่าโครงสร้างเพื่อป้องกัน Reference เปลี่ยน
@riverpod
List<String> activeTodoIds(ActiveTodoIdsRef ref) {
  return ref.watch(todoListProvider.select((list) => list.where((t) => !t.isDone).map((t) => t.id).toList()));
}
---