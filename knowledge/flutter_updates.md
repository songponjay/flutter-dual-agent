---
# Flutter Updates Log
<!-- เพิ่มรายการใหม่ด้านบนสุด ตาม template ใน README.md -->

2026-02-11 | Dart 3.11: Dot Shorthand Analyzer Improvementsสิ่งที่เปลี่ยน: ปรับปรุงเครื่องมือวิเคราะห์ (Analyzer) และระบบช่วยเหลือใน IDE ให้รองรับ Dot Shorthands อย่างสมบูรณ์ แก้ไขข้อผิดพลาดในการนำทางโค้ด (Navigation) และการเติมคำอัตโนมัติ (Code Completion) ที่เคยพบในเวอร์ชันก่อนหน้า version ที่ affect: Dart 3.11วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว
// การใช้งาน Shorthand กับ Generic อาจทำให้ IDE แจ้งเตือนผิดพลาดในเวอร์ชัน 3.10
List<int> numbers = List.filled(5, 0); 
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
// ระบบรองรับการเติมคำและแก้ไขโค้ดผ่าน IDE ได้เสถียรแล้ว
List<int> numbers =.filled(5, 0); 
2026-02-11 | Flutter 3.41: findItemIndexCallbackสิ่งที่เปลี่ยน: เปลี่ยนชื่อพารามิเตอร์ findChildIndexCallback เป็น findItemIndexCallback ในคอนสตรัคเตอร์ .separated ของ ListView และ SliverList เพื่อความชัดเจนและสอดคล้องกับมาตรฐาน API ใหม่ version ที่ affect: Flutter 3.41วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว
ListView.separated(
  itemCount: items.length,
  itemBuilder: (context, index) => ListTile(title: Text(items[index])),
  separatorBuilder: (context, index) => const Divider(),
  findChildIndexCallback: (Key key) {
    return items.indexWhere((m) => ValueKey(m.id) == key);
  },
);
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
ListView.separated(
  itemCount: items.length,
  itemBuilder: (context, index) => ListTile(title: Text(items[index])),
  separatorBuilder: (context, index) => const Divider(),
  findItemIndexCallback: (Key key) { // เปลี่ยนชื่อพารามิเตอร์
    return items.indexWhere((m) => ValueKey(m.id) == key);
  },
);
2026-02-11 | Flutter 3.41: isSemantics Matcherสิ่งที่เปลี่ยน: ยกเลิกการใช้ containsSemantics และแทนที่ด้วย isSemantics ในระบบการทดสอบ (Testing) เพื่อระบุว่าเป็น Partial Matcher (ตรวจสอบเฉพาะคุณสมบัติที่ระบุ) ให้ชัดเจนตามมาตรฐาน version ที่ affect: Flutter 3.41วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว
expect(
  tester.getSemantics(find.byType(MyWidget)),
  containsSemantics(label: 'Submit', isButton: true),
);
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
expect(
  tester.getSemantics(find.byType(MyWidget)),
  isSemantics(label: 'Submit', isButton: true),
);
2026-01-17 | Riverpod 3.2: family.overrideWith2สิ่งที่เปลี่ยน: แนะนำ overrideWith2 สำหรับ Family Providers โดย callback ใหม่จะรับอาร์กิวเมนต์ของ family เป็นพารามิเตอร์ตัวที่สอง ช่วยให้การ Mock ข้อมูลในระดับการทดสอบทำได้ง่ายขึ้นโดยไม่ต้องเข้าถึงผ่าน ref version ที่ affect: Riverpod 3.2วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว (Deprecated ใน 3.2)
final userProvider = Provider.family<User, int>((ref, id) => User(id));
final container = ProviderContainer(
  overrides:,
);
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
final userProvider = Provider.family<User, int>((ref, id) => User(id));
final container = ProviderContainer(
  overrides:,
);
2025-12-26 | Riverpod 3.1: AsyncValue.requireValue for Combinationสิ่งที่เปลี่ยน: อนุญาตให้ใช้ AsyncValue.requireValue ภายในฟังก์ชันเริ่มต้นของ Provider เพื่อรวมสถานะจาก Asynchronous Providers หลายตัวเข้าด้วยกันในรูปแบบ Synchronous version ที่ affect: Riverpod 3.1วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว (ซับซ้อน)
final combined = Provider((ref) {
  final a = ref.watch(aProvider);
  final b = ref.watch(bProvider);
  return a.whenData((aVal) => b.whenData((bVal) => aVal + bVal));
});
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
final combined = Provider((ref) {
  // ใช้ requireValue ได้ปลอดภัยในบริบทการ init ของ provider
  return ref.watch(aProvider).requireValue + ref.watch(bProvider).requireValue;
});
2025-11-12 | Dart 3.10: Dot Shorthandsสิ่งที่เปลี่ยน: เพิ่มไวยากรณ์ Dot Shorthands ช่วยให้ละเว้นชื่อคลาสเมื่อเรียกใช้งาน Enum, Static Members หรือ Constructors ได้หากคอมไพเลอร์ทราบประเภทข้อมูลจากบริบท version ที่ affect: Dart 3.10วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว
Text(
  'Hello',
  style: TextStyle(fontWeight: FontWeight.bold),
);
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
Text(
  'Hello',
  style: TextStyle(fontWeight:.bold), // ละเว้นชื่อ FontWeight ได้
);
2025-11-12 | Flutter 3.38: iOS UIScene Lifecycleสิ่งที่เปลี่ยน: บังคับใช้การจัดการ Lifecycle ผ่าน UISceneDelegate แทน AppDelegate บน iOS เพื่อรองรับมาตรฐานใหม่ของ Apple และฟีเจอร์ Multi-window version ที่ affect: Flutter 3.38วิธีปรับโค้ด:Swift// ios/Runner/AppDelegate.swift แบบเดิม
@objc class AppDelegate: FlutterAppDelegate {
  override func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions:...) -> Bool {
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
Swift// ios/Runner/AppDelegate.swift แบบใหม่ปี 2026
@objc class AppDelegate: FlutterAppDelegate, FlutterImplicitEngineDelegate {
  override func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions:...) -> Bool {
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  // แยกการลงทะเบียนปลั๊กอินมาไว้ที่นี่ตามมาตรฐาน UIScene
  func didInitializeImplicitFlutterEngine(_ engineBridge: FlutterImplicitEngineBridge) {
    GeneratedPluginRegistrant.register(with: engineBridge.pluginRegistry)
  }
}
2025-09-10 | Riverpod 3.0: Notifier API Unificationสิ่งที่เปลี่ยน: ยุบรวมคลาส AutoDisposeNotifier และคลาสย่อยอื่นๆ เข้ากับคลาสหลักอย่าง Notifier หรือ AsyncNotifier โดยใช้พารามิเตอร์ isAutoDispose หรือการตั้งค่าใน @Riverpod แทน version ที่ affect: Riverpod 3.0วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว
class MyNotifier extends AutoDisposeNotifier<int> {
  @override
  int build() => 0;
}
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
// ใช้ Notifier ตัวเดียวและกำหนดค่า autoDispose ผ่าน provider
class MyNotifier extends Notifier<int> { 
  @override
  int build() => 0;
}
2025-09-10 | Riverpod 3.0: Legacy Import Pathสิ่งที่เปลี่ยน: ย้าย StateProvider, StateNotifierProvider และ ChangeNotifierProvider ไปยัง Path ใหม่ที่ legacy.dart เพื่อเตรียมยกเลิกใช้งานในอนาคตและผลักดันให้ใช้ Notifier API version ที่ affect: Riverpod 3.0วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว
import 'package:flutter_riverpod/flutter_riverpod.dart';
final countProvider = StateProvider((ref) => 0);
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
import 'package:flutter_riverpod/legacy.dart'; // ต้อง import ผ่าน path legacy
final countProvider = StateProvider((ref) => 0);
2025-08-28 | Flutter 3.35: RadioGroup Migrationสิ่งที่เปลี่ยน: ยกเลิกการจัดการสถานะ groupValue และ onChanged ในวิดเจ็ต Radio รายตัว และเปลี่ยนไปใช้ RadioGroup ครอบเพื่อประสิทธิภาพและความถูกต้องของ Accessibility version ที่ affect: Flutter 3.35วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว
Column(
  children:,
)
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
RadioGroup<int>(
  groupValue: _val,
  onChanged: (v) => setState(() => _val = v!),
  child: Column(
    children:,
  ),
)
2025-05-20 | Dart 3.8: Null-aware Elementsสิ่งที่เปลี่ยน: เพิ่มสัญลักษณ์ ? หน้าสมาชิกใน Collection (List, Set, Map) เพื่อข้ามการเพิ่มสมาชิกนั้นหากมีค่าเป็น null แทนการใช้ if-statement version ที่ affect: Dart 3.8วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว
var tags =;
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
var tags =;
2025-02-12 | Dart 3.7: Wildcard Variablesสิ่งที่เปลี่ยน: อนุญาตให้ใช้เครื่องหมาย _ เป็นตัวแปรโลคอลหรือพารามิเตอร์ที่ไม่มีการผูกมัดข้อมูล (Non-binding) ได้หลายครั้งใน Scope เดียวกัน version ที่ affect: Dart 3.7วิธีปรับโค้ด:Dart// โค้ดแบบเดิมที่ห้ามใช้แล้ว
var (id, _temp1, _temp2) = getData(); // ต้องตั้งชื่อเลี่ยง
Dart// โค้ดแบบใหม่ที่ถูกต้องในปี 2026
var (id, _, _) = getData(); // ใช้ _ ซ้ำได้ทันที
---