
> 현재는 모달 형태의 페이지를 푸시하면서 네비게이션 중
> 모달창에서 항목 눌러서 이동시에 모달 형태 풀리는게 이것 때문으로 추측 중

좋아. **Nested Navigator**는 처음 보면 헷갈리는데,

개념만 잡히면 “아, 그래서 필요했구나”가 바로 와.

  

아래는 **정의 → 왜 필요 → 언제 쓰는지 → 구조 → 예제 → 주의점** 순서로 설명할게.

---

# **1️⃣ Nested Navigator란?**

  

> **Navigator 안에 또 하나의 Navigator를 두는 구조**

  

즉,

- 앱 전체 라우트 스택과
    
- 특정 영역(모달/탭/서브플로우) 전용 스택을
    
    **분리해서 관리**하는 것
    

---

## **그림으로 생각하면**

```
App Navigator (전체 앱)
 ├─ HomePage
 ├─ ProfilePage
 └─ TeamModal  ← (여기까지는 앱 라우트)
      └─ Nested Navigator (모달 전용)
           ├─ TeamMenu
           ├─ CreateTeam
           └─ InvitationCode
```

👉 **모달 안에서 push/pop 해도**

- 앱의 메인 네비게이션 스택은 **전혀 안 건드림**
    
- “모달 내부 흐름”만 바뀜
    

---

# **2️⃣ 왜 Nested Navigator가 필요하냐?**

  

### **❌ Nested Navigator 없을 때 문제**

  

모달 안에서 그냥 Navigator.push(context, ...)를 쓰면:

- 이 push가 **앱 전체 Navigator**에 쌓임
    
- 뒤로 가기 누르면:
    
    - 모달이 먼저 닫히지 않고
        
    - 예상 못 한 화면으로 튐
        
    
- 모달을 “한 덩어리 UI”로 다루기 어려움
    

---

### **⭕ Nested Navigator가 있으면**

- 모달 안의 이동은 **모달 내부에서만 해결**
    
- 모달을 닫으면 → 내부 스택도 같이 사라짐
    
- UX가 직관적
    

---

# **3️⃣ 언제 Nested Navigator를 쓰는 게 맞나?**

  

### **꼭 쓰는 경우**

- 모달/바텀시트 안에 **여러 화면 흐름**이 있음
    
- Wizard / Stepper / 설정 플로우
    
- “모달 안에서 뒤로가기”가 의미 있어야 할 때
    
- 탭마다 독립적인 히스토리가 필요할 때
    

  

### **굳이 안 써도 되는 경우**

- 모달 안에 버튼 몇 개
    
- 한 화면에서 상태만 바뀌는 경우
    
- push 한 번만 하고 끝
    

---

# **4️⃣ 기본 구조 (핵심 코드)**

  

### **🔹 모달 최상위에 Navigator를 하나 둔다**

```
class TeamModal extends StatelessWidget {
  const TeamModal({super.key});

  @override
  Widget build(BuildContext context) {
    return Navigator(
      initialRoute: '/menu',
      onGenerateRoute: (settings) {
        switch (settings.name) {
          case '/menu':
            return MaterialPageRoute(
              builder: (_) => const TeamMenuPage(),
            );
          case '/create':
            return MaterialPageRoute(
              builder: (_) => const CreateTeamPage(),
            );
          case '/invite':
            return MaterialPageRoute(
              builder: (_) => const InvitationCodePage(),
            );
        }
        return null;
      },
    );
  }
}
```

👉 이 Navigator는 **모달 전용**

---

# **5️⃣ 모달 내부에서 이동하는 법**

```
Navigator.of(context).pushNamed('/create');
```

⚠️ 중요한 포인트

이 context는 **Nested Navigator 아래의 context**여야 해.

---

# **6️⃣ 모달 닫기는 어떻게?**

  

모달을 닫고 싶을 땐 **rootNavigator**를 사용:

```
Navigator.of(context, rootNavigator: true).pop();
```

👉 이건 **모달 자체를 닫는 동작**

(내부 스택 pop 아님)

---

# **7️⃣ 뒤로가기 버튼 처리 (중요)**

  

Nested Navigator가 있으면:

- Android back 버튼
    
- iOS 스와이프
    

  

이게 **내부 Navigator부터 pop**함.

  

원하면 이렇게 제어도 가능:

```
WillPopScope(
  onWillPop: () async {
    final canPop = navigatorKey.currentState!.canPop();
    if (canPop) {
      navigatorKey.currentState!.pop();
      return false;
    }
    return true; // 모달 닫기 허용
  },
  child: Navigator(...),
);
```

---

# **8️⃣ GoRouter랑 같이 쓸 수 있나?**

  

👉 **쓴다. 그리고 이게 실무 정석이다.**

- 앱 전체 페이지 → **GoRouter**
    
- 모달/탭 내부 흐름 → **Nested Navigator**
    

  

GoRouter도 내부적으로 Navigator를 여러 개 쓴다

(ShellRoute가 대표적인 예)

---

# **9️⃣ 지금 네 코드 기준으로 판단하면**

  

네 TeamModal은:

- 메뉴 → 생성 → 초대 → 설정
    
- 전형적인 **모달 내부 플로우**
    

  

👉 다음 중 하나로 정리하면 좋아:

  

### **선택 A (간단)**

- 메뉴 → 새 화면은 **모달 닫고 이동**
    
- pop + push(rootNavigator)
    

  

### **선택 B (구조적으로 깔끔) ⭐**

- TeamModal 안에 **Nested Navigator**
    
- 모달 내부에서만 push/pop
    

---

## **한 줄 요약**

  

> **Nested Navigator는**

> **“모달/탭 같은 UI 덩어리 안에서만 쓰는 전용 네비게이션 스택”이다.**

---

원하면 다음 단계로:

- **지금 TeamModal을 Nested Navigator 구조로 리팩토링**
    
- **Riverpod 상태랑 네비게이션 분리하는 법**
    
- **GoRouter + Nested Navigator 실전 패턴**
    

  

중에서 하나 골라줘.