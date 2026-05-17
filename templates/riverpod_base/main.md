---
# Riverpod Boilerplate 2026 (AsyncNotifier + Clean Architecture)

## โครงสร้างโฟลเดอร์
lib/
├── data/
│   ├── repositories/        impl ของ repository
│   └── datasources/         API calls, local DB
├── domain/
│   ├── entities/            pure Dart models
│   └── repositories/        abstract interfaces
└── presentation/
    ├── providers/            Riverpod providers
    ├── screens/
    └── widgets/

## Provider ต้นแบบ (AsyncNotifier)
```dart
// presentation/providers/user_provider.dart
@riverpod
class UserNotifier extends _$UserNotifier {
  @override
  Future<User> build() async {
    return ref.watch(userRepositoryProvider).getUser();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(
      () => ref.watch(userRepositoryProvider).getUser(),
    );
  }
}
```