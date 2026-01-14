![[CleanShot 2026-01-13 at 16.04.26.png]]
## `rootNavigator: true`가 필요한 이유

### 문제 상황

Flutter에서는 **여러 개의 Navigator가 중첩**될 수 있습니다:

```
Root Navigator (최상위)
  └─ MaterialApp의 Navigator
      └─ Modal/BottomSheet의 Navigator (현재 team_settings_modal.dart)
          └─ Dialog의 Navigator
```

`team_settings_modal.dart`는 **Modal**이기 때문에 자체 Navigator 컨텍스트를 가지고 있습니다.

### 문제 발생 원리

```dart
// ❌ 문제가 있는 코드
Navigator.of(context).pop();
```

이렇게 하면:
1. `context`는 Modal의 Navigator를 가리킴
2. `pop()`이 Modal의 Navigator에서 실행됨
3. **하지만 로딩 다이얼로그는 Root Navigator에 떠있음!**
4. 결과: 엉뚱한 Navigator에서 pop을 시도 → 다이얼로그가 안 닫힘

### 해결 방법

```dart
// ✅ 올바른 코드
Navigator.of(context, rootNavigator: true).pop();
```

`rootNavigator: true`를 추가하면:
1. 가장 상위의 Root Navigator를 직접 찾아감
2. 로딩 다이얼로그가 떠있는 바로 그 Navigator에서 pop 실행
3. 결과: 다이얼로그가 제대로 닫힘

---

## 실제 동작 비교

### Before (작동 안 함)
```
showDialog() → Root Navigator에 다이얼로그 추가
getCurrentWifiSSID() → 비동기 작업
Navigator.of(context).pop() → Modal Navigator에서 pop 시도 ❌
```

### After (작동함)
```
showDialog() → Root Navigator에 다이얼로그 추가
getCurrentWifiSSID() → 비동기 작업
Navigator.of(context, rootNavigator: true).pop() → Root Navigator에서 pop ✅
```

---

## 언제 `rootNavigator: true`를 쓰나?

| 상황 | rootNavigator 필요 여부 |
|------|------------------------|
| Modal/BottomSheet 내부에서 Dialog 닫기 | ✅ 필요 |
| 일반 화면에서 Dialog 닫기 | ❌ 불필요 |
| Nested Navigator가 있는 경우 | ✅ 필요 |
| 단순한 화면 전환 | ❌ 불필요 |

현재 `TeamSettingsModal`은 Modal이므로 rootNavigator가 필수입니다!