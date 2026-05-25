# Syllabus: Flutter AI Work Log & Summary App

**เป้าหมาย:** portfolio แอป productivity สำหรับสมัคร Flutter Dev สาย startup/tech (ย่านลาดพร้าว/จตุจักร)
**Stack:** Flutter + Riverpod + Clean Architecture + Isar DB + Claude/Gemini API + Excel Export
**ผู้เรียน:** มีพื้นฐาน Java, เคยทำ Flutter (ระบบตั๋วรถเมล์) 2 ปีก่อน

---

## ภาพรวม App

```
Features:
✅ บันทึกงาน free-text ไม่ lock format
✅ AI สรุปงาน weekly/monthly (Claude API + Gemini API สลับได้)
✅ Dynamic Excel export (เลือก column เองได้)
✅ Offline cache ด้วย Isar DB
✅ Dark/Light mode
✅ Auth: Local mock → Firebase Auth (Bonus บทสุดท้าย)

Tech ที่โชว์ใน Portfolio:
- REST API integration (Dio + interceptors)
- Riverpod state management (AsyncNotifier pattern)
- File I/O (Excel export + path_provider)
- Error handling + Retry logic
- Clean Architecture 3 ชั้น
```

---

## Module 1 — Clean Architecture Scaffold
> **B4A parallel:** ใน B4A โค้ดทุกอย่างมักกองอยู่ใน Activity เดียว (Sub ยาวๆ ทำทุกอย่าง) — Clean Architecture คือการแยกโค้ดออกเป็น 3 ชั้น ชัดเจน ไม่ปนกัน

### Core Concepts
```
lib/
├── data/
│   ├── datasources/
│   │   ├── work_log_local_datasource.dart   ← Isar DB
│   │   └── work_log_remote_datasource.dart  ← REST API
│   ├── models/
│   │   └── work_log_model.dart              ← JSON parsing
│   └── repositories/
│       └── work_log_repository_impl.dart    ← implements Domain interface
├── domain/
│   ├── entities/
│   │   └── work_log.dart                   ← pure Dart class (ไม่มี JSON/DB)
│   ├── repositories/
│   │   └── work_log_repository.dart        ← abstract class (≈ Java interface)
│   └── usecases/
│       └── get_work_logs.dart              ← single responsibility
└── presentation/
    ├── providers/
    │   └── work_log_notifier.dart          ← Riverpod AsyncNotifier
    ├── screens/
    │   └── home_screen.dart
    └── widgets/
        └── work_log_card.dart
```

**กฎทอง (ห้ามละเมิด):**
- Domain layer ต้องไม่ import Flutter หรือ package ภายนอกใดๆ
- Presentation ไม่รู้จัก Isar หรือ Dio
- Data ไม่รู้จัก Widget

### Common Pitfalls
- ❌ เขียน business logic ใน Widget (นิสัยมาจาก Java Activity God-class)
- ❌ Entity class มี `toJson()` — entity ควรบริสุทธิ์, model หน้าที่นี้
- ❌ Import `package:isar` ใน domain layer

### Homework
1. สร้าง folder structure ตามโครง
2. สร้าง `WorkLog` entity (id, date, content, tags)
3. สร้าง `WorkLogRepository` abstract class พร้อม 4 methods: getAll, getById, save, delete

---

## Module 2 — Riverpod AsyncNotifier
> **B4A parallel:** ใน B4A ใช้ Global Variables เก็บ state + Events แจ้งเตือนเมื่อค่าเปลี่ยน — Riverpod ทำทั้งสองอย่างนี้อัตโนมัติ พร้อม handle loading/error ให้ด้วย

### Core Concepts

```dart
// Java Singleton + Observer ≈ Riverpod @riverpod + AsyncNotifier
@riverpod
class WorkLogNotifier extends _$WorkLogNotifier {
  @override
  Future<List<WorkLog>> build() async {
    // เรียกตอน provider ถูกสร้าง (≈ Java init())
    return ref.read(workLogRepositoryProvider).getAll();
  }

  Future<void> addLog(WorkLog log) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(
      () => ref.read(workLogRepositoryProvider).save(log),
    );
    ref.invalidateSelf(); // reload list
  }
}
```

**ref.watch vs ref.read:**
| | `ref.watch` | `ref.read` |
|---|---|---|
| ใช้ใน | `build()` method เท่านั้น | action methods, callbacks |
| Rebuild เมื่อ | provider เปลี่ยน ✅ | ไม่ rebuild ✅ |
| นอก build() | 💥 crash / infinite loop | ✅ ปลอดภัย |

**autoDispose:** ใส่เมื่อ data ใช้ชั่วคราว (detail screen, search result)
ไม่ใส่เมื่อ: global state (auth, theme, work log list)

### Common Pitfalls
- ❌ `ref.watch(someProvider)` ใน action method → runtime crash
- ❌ ไม่ handle `AsyncValue.error` state → UI หายไปเงียบๆ
- ❌ ใช้ `StateNotifier` ตอนนี้ (deprecated ใน Riverpod 3.x ใช้ AsyncNotifier แทน)

### Homework
สร้าง `WorkLogNotifier` ที่:
- `build()` return mock list ของ WorkLog 3 รายการ
- `addLog()` เพิ่ม log ใหม่เข้า list
- `deleteLog(id)` ลบ log ออก
- UI แสดง loading / error / data state ครบ

---

## Module 3 — Isar DB Offline Cache
> **B4A parallel:** ใน B4A ใช้ SQLite ต้องเขียน SQL string เอง เช่น `"SELECT * FROM logs WHERE date=?"` — Isar ทำแบบเดียวกันแต่เขียนเป็น Dart code ล้วนๆ ไม่มี SQL string เลย พลาดยากกว่ามาก

### Core Concepts

```dart
// Schema (≈ Java @Entity ใน Room)
@collection
class WorkLogModel {
  Id id = Isar.autoIncrement;

  @Index()
  late DateTime date;
  late String content;
  late List<String> tags;
}

// Datasource
class WorkLogLocalDatasource {
  final Isar _isar;
  WorkLogLocalDatasource(this._isar);

  Future<List<WorkLogModel>> getAll() =>
      _isar.workLogModels.where().sortByDateDesc().findAll();

  Future<void> save(WorkLogModel model) async {
    await _isar.writeTxn(() => _isar.workLogModels.put(model));
  }

  Future<void> delete(int id) async {
    await _isar.writeTxn(() => _isar.workLogModels.delete(id));
  }
}
```

**เปิด Isar (ทำครั้งเดียว):**
```dart
// ใน main.dart หรือ Riverpod Provider
@riverpod
Future<Isar> isar(IsarRef ref) async {
  return Isar.open([WorkLogModelSchema]);
}
```

### Common Pitfalls
- ❌ `Isar.open()` หลายครั้ง → `IsarError: Instance already opened`
- ❌ write นอก `writeTxn` → exception
- ❌ ลืม run `dart run build_runner build` หลังแก้ schema → generated file เก่า

### Homework
สร้าง `WorkLogLocalDatasource` ครบ 4 operations แล้ว integrate เข้า Repository impl
ทดสอบโดย: save 3 logs → restart app → ข้อมูลยังอยู่ครบ

---

## Module 4 — REST API + Error Handling + Retry
> **B4A parallel:** ใน B4A ใช้ OkHttpUtils2 หรือ HttpUtils2 ยิง HTTP request — Dio คือตัวเทียบเท่าใน Flutter แถมมี interceptors (เหมือน middleware) ที่ใส่ logic ได้โดยไม่ต้องแก้ทุก request

### Core Concepts

```dart
// Dio setup ใน Provider
@riverpod
Dio dio(DioRef ref) {
  final dio = Dio(BaseOptions(
    baseUrl: 'https://api.yourapp.com',
    connectTimeout: const Duration(seconds: 10),
    receiveTimeout: const Duration(seconds: 10),
  ));
  dio.interceptors.add(LogInterceptor(responseBody: true));
  dio.interceptors.add(_AuthInterceptor(ref));
  return dio;
}

// Repository impl: API first, fallback to local
class WorkLogRepositoryImpl implements WorkLogRepository {
  final WorkLogRemoteDatasource _remote;
  final WorkLogLocalDatasource _local;

  @override
  Future<List<WorkLog>> getAll() async {
    try {
      final models = await _remote.getAll();
      await _local.saveAll(models); // cache ลง Isar
      return models.map((m) => m.toDomain()).toList();
    } on DioException {
      // offline fallback
      return (await _local.getAll()).map((m) => m.toDomain()).toList();
    }
  }
}
```

**Retry Logic:**
```dart
Future<T> withRetry<T>(Future<T> Function() fn, {int maxAttempts = 3}) async {
  for (var attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (e) {
      if (attempt == maxAttempts) rethrow;
      await Future.delayed(Duration(seconds: attempt)); // exponential backoff
    }
  }
  throw Exception('unreachable');
}
```

### Common Pitfalls
- ❌ race condition: user กด refresh ซ้ำๆ ขณะ fetch → ใช้ `CancelToken` หรือ guard
- ❌ API key ใน `baseOptions.headers` แบบ hardcode → ใช้ `flutter_dotenv`
- ❌ ไม่แยก `DioException` types (timeout vs 401 vs 500 ต้องจัดการต่างกัน)

### Homework
1. ใส่ retry logic ใน `WorkLogRemoteDatasource`
2. สร้าง `Failure` sealed class (`NetworkFailure`, `CacheFailure`, `ServerFailure`)
3. UI แสดง error message ที่ user-friendly (ไม่ expose stack trace)

---

## Module 5 — AI Summary Integration (Claude + Gemini)
> **B4A parallel:** ใน B4A ถ้าอยากสลับ AI provider ต้องเขียน If/Else กระจายทั่วโค้ด — ใน Flutter ใช้ abstract class เดียว สลับ implementation ได้โดยแก้แค่จุดเดียว UI ไม่รู้เลยว่าใช้ Claude หรือ Gemini

### Core Concepts

```dart
// Strategy interface (≈ Java interface)
abstract class AiSummaryService {
  Future<String> summarize({
    required List<WorkLog> logs,
    required SummaryPeriod period, // weekly / monthly
  });
}

// Claude implementation
class ClaudeAiService implements AiSummaryService {
  final Dio _dio;
  static const _model = 'claude-sonnet-4-6';

  @override
  Future<String> summarize({required logs, required period}) async {
    final response = await _dio.post(
      'https://api.anthropic.com/v1/messages',
      options: Options(headers: {
        'x-api-key': Env.claudeApiKey,
        'anthropic-version': '2023-06-01',
      }),
      data: {
        'model': _model,
        'max_tokens': 1024,
        'messages': [
          {'role': 'user', 'content': _buildPrompt(logs, period)},
        ],
      },
    );
    return response.data['content'][0]['text'] as String;
  }
}

// Gemini implementation
class GeminiAiService implements AiSummaryService {
  @override
  Future<String> summarize({required logs, required period}) async {
    // POST to generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent
    ...
  }
}

// Riverpod: สลับ provider จาก UI
enum AiProviderType { claude, gemini }

@riverpod
AiProviderType selectedAiProvider(SelectedAiProviderRef ref) =>
    AiProviderType.claude;

@riverpod
AiSummaryService aiSummaryService(AiSummaryServiceRef ref) {
  final type = ref.watch(selectedAiProviderProvider);
  return switch (type) {
    AiProviderType.claude => ref.read(claudeAiServiceProvider),
    AiProviderType.gemini => ref.read(geminiAiServiceProvider),
  };
}
```

**Prompt Template (Thai output):**
```
สรุปงานประจำ{period}จาก work log ต่อไปนี้เป็นภาษาไทย:
- เน้น accomplishments และ impact
- ระบุ blockers หรือปัญหาที่พบ
- ความยาว 3-5 ย่อหน้า

Work Logs:
{logs}
```

### Common Pitfalls
- ❌ API key ใน source code → ใช้ `flutter_dotenv` + `.env` ใน `.gitignore`
- ❌ ไม่ handle rate limit (429) → ใส่ retry + แจ้ง user
- ❌ response ใหญ่เกิน `max_tokens` → truncate logs ก่อนส่ง

### Homework
สร้าง SummaryScreen:
- Dropdown เลือก Claude / Gemini
- Dropdown เลือก weekly / monthly
- กด "สรุป" → แสดง loading → แสดงผล
- กด "Copy" หรือ "Share" ผลลัพธ์

---

## Module 6 — Dynamic Excel Export
> **B4A parallel:** ใน B4A ใช้ ExcelLibrary หรือ jExcelApi สร้างไฟล์ .xls — Flutter ใช้ `excel` package ทำแบบเดียวกัน แถมแชร์ไฟล์ผ่าน `share_plus` ได้ทันที ไม่ต้องเปิด file manager

### Core Concepts

```dart
// Dynamic column export
class ExcelExportService {
  Future<File> export({
    required List<WorkLog> logs,
    required List<ExportColumn> selectedColumns,
  }) async {
    final workbook = Excel.createExcel();
    final sheet = workbook['Work Logs'];

    // Header row
    for (var i = 0; i < selectedColumns.length; i++) {
      sheet.cell(CellIndex.indexByColumnRow(columnIndex: i, rowIndex: 0))
        ..value = TextCellValue(selectedColumns[i].label)
        ..cellStyle = CellStyle(bold: true);
    }

    // Data rows
    for (var r = 0; r < logs.length; r++) {
      for (var c = 0; c < selectedColumns.length; c++) {
        final value = selectedColumns[c].getValue(logs[r]);
        sheet.cell(CellIndex.indexByColumnRow(columnIndex: c, rowIndex: r + 1))
          .value = TextCellValue(value);
      }
    }

    // Save to temp file
    final dir = await getTemporaryDirectory();
    final file = File('${dir.path}/work_logs_${DateTime.now().millisecondsSinceEpoch}.xlsx');
    await file.writeAsBytes(workbook.encode()!);
    return file;
  }
}

// Column definition
class ExportColumn {
  final String label;
  final String Function(WorkLog) getValue;
  const ExportColumn({required this.label, required this.getValue});
}

// Available columns
final kAvailableColumns = [
  ExportColumn(label: 'Date', getValue: (l) => l.date.toString()),
  ExportColumn(label: 'Content', getValue: (l) => l.content),
  ExportColumn(label: 'Tags', getValue: (l) => l.tags.join(', ')),
];
```

### Common Pitfalls
- ❌ `getApplicationDocumentsDirectory()` ใน iOS ไม่ shareable → ใช้ `getTemporaryDirectory()` แล้ว `share_plus`
- ❌ ลืม `share_plus` permission ใน iOS Info.plist
- ❌ encode() return null เมื่อ sheet ว่าง → guard ก่อน write

### Homework
สร้าง ExportScreen:
- Checkbox list ให้ user เลือก columns
- Date range picker
- กด Export → ได้ .xlsx ที่ share ได้

---

## Module 7 — Auth: Local Mock → Firebase
> **B4A parallel:** ใน B4A มักเริ่มด้วย hardcode username/password ก่อน พอจะ deploy จริงค่อยต่อ server — pattern นี้เหมือนกันแต่ทำอย่างเป็นระบบ: Mock → Firebase โดยไม่ต้องแตะ UI เลย

### Core Concepts

```dart
// Auth entity
class AppUser {
  final String id;
  final String name;
  final String email;
  const AppUser({required this.id, required this.name, required this.email});
}

// Repository interface
abstract class AuthRepository {
  Future<AppUser?> getCurrentUser();
  Future<AppUser> signIn({required String email, required String password});
  Future<void> signOut();
  Stream<AppUser?> get authStateChanges;
}

// Mock implementation (บทที่ 7)
class MockAuthRepository implements AuthRepository {
  final _controller = StreamController<AppUser?>.broadcast();
  AppUser? _currentUser;

  @override
  Future<AppUser> signIn({required email, required password}) async {
    await Future.delayed(const Duration(seconds: 1)); // simulate network
    if (email == 'demo@test.com' && password == '1234') {
      _currentUser = const AppUser(id: '1', name: 'Demo User', email: 'demo@test.com');
      _controller.add(_currentUser);
      return _currentUser!;
    }
    throw Exception('Invalid credentials');
  }

  @override
  Stream<AppUser?> get authStateChanges => _controller.stream;
  // ...
}

// AuthNotifier
@riverpod
class AuthNotifier extends _$AuthNotifier {
  @override
  Future<AppUser?> build() async {
    ref.onDispose(() => /* cancel stream sub */);
    return ref.read(authRepositoryProvider).getCurrentUser();
  }
}

// GoRouter redirect
redirect: (context, state) {
  final authState = ref.read(authNotifierProvider);
  final isLoggedIn = authState.valueOrNull != null;
  if (!isLoggedIn && !state.matchedLocation.startsWith('/login')) {
    return '/login';
  }
  return null;
},
```

**Firebase Migration (Bonus):**
เพียงสร้าง `FirebaseAuthRepository implements AuthRepository` แล้ว swap ใน provider
ไม่ต้องแตะ Presentation layer เลย — นี่คือ proof ว่า Clean Architecture ใช้งานได้จริง

### Common Pitfalls
- ❌ `StreamSubscription` ไม่ cancel ใน `ref.onDispose` → memory leak
- ❌ GoRouter redirect loop (ลืม check current route ก่อน redirect)
- ❌ `ref.read` authState ใน redirect (ควรใช้ synchronous valueOrNull)

### Homework
1. LoginScreen พร้อม form validation
2. Protected routes ด้วย GoRouter
3. Logout ใน AppBar
4. (Bonus) swap MockAuthRepository → FirebaseAuthRepository

---

## Module 8 — Dark/Light Mode + Portfolio Polish
> **B4A parallel:** ใน B4A เปลี่ยนสีต้องวน loop แก้ทุก View ทีละตัว — Flutter ใช้ ThemeData เปลี่ยนครั้งเดียวทั้งแอปเปลี่ยนหมด ไม่ต้องแตะ Widget เลย

### Core Concepts

```dart
// Theme provider (persist ด้วย SharedPreferences)
@riverpod
class ThemeModeNotifier extends _$ThemeModeNotifier {
  @override
  ThemeMode build() {
    final prefs = ref.read(sharedPreferencesProvider);
    final saved = prefs.getString('theme_mode') ?? 'system';
    return ThemeMode.values.firstWhere((e) => e.name == saved);
  }

  void toggle() {
    state = state == ThemeMode.dark ? ThemeMode.light : ThemeMode.dark;
    ref.read(sharedPreferencesProvider)
       .setString('theme_mode', state.name);
  }
}

// MaterialApp
MaterialApp.router(
  theme: ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo)),
  darkTheme: ThemeData.dark(useMaterial3: true).copyWith(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.indigo,
      brightness: Brightness.dark,
    ),
  ),
  themeMode: ref.watch(themeModerNotifierProvider),
)
```

**Portfolio Polish Checklist (ทุก Screen ต้องมี):**
- [ ] Loading state: `CircularProgressIndicator` ขณะโหลด
- [ ] Empty state: แสดง illustration + CTA เมื่อไม่มีข้อมูล
- [ ] Error state: ข้อความ friendly + ปุ่ม retry
- [ ] Offline indicator: banner เมื่อ connectivity หาย

### Common Pitfalls
- ❌ `Consumer` ครอบทั้งหน้า → rebuild ทุก widget เมื่อ theme เปลี่ยน
- ❌ hardcode สีใน Widget (`Colors.blue`) แทน `Theme.of(context).colorScheme.primary`
- ❌ ลืม test dark mode → text กลายเป็น white-on-white

### Homework
1. ใส่ theme toggle icon ใน AppBar
2. Review ทุก Screen → ใส่ empty/error state
3. ใส่ `ConnectivityWrapper` widget แสดง offline banner
4. Screenshot ทั้ง light และ dark mode สำหรับ README portfolio

---

## Summary: Tech Stack ที่จะ Showcase

| Tech | Module | HR จะเห็น |
|------|--------|-----------|
| Clean Architecture | 1 | โค้ดสะอาด, maintainable |
| Riverpod AsyncNotifier | 2 | state management ระดับ production |
| Isar DB | 3 | offline-first thinking |
| Dio + Retry | 4 | error handling ครบ |
| Claude/Gemini API | 5 | AI integration, Strategy pattern |
| Dynamic Excel | 6 | File I/O, real-world feature |
| Auth (Mock→Firebase) | 7 | ออกแบบ extensible ตั้งแต่ต้น |
| Dark/Light + Polish | 8 | UI/UX awareness, attention to detail |

---

## การติดตาม Progress
`/teacher <module>` — เริ่มบทเรียน module นั้น
`/senior` — review โค้ดที่เขียน
`/dashboard` — ดู progress ทั้งหมด
