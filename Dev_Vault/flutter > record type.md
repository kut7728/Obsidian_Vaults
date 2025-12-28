({int month, int? year}) ← 네가 선택한 **이 타입이 바로 Dart의 Record 타입**이야.

---

# **✅ Record 타입이란?**

  

Dart 3부터 추가된 **튜플(Tuple) 같은 값 묶음 타입**

→ 여러 개의 값을 하나의 객체처럼 묶어서 전달할 수 있어.

  

예:

```
(int, String)
```

또는 이름 있는 형태:

```
({int month, int year})
```

---

# **🔥 네 코드에서 쓰는 건 “Named Record” (이름 있는 레코드)**

```
({int month, int? year})
```

이 의미는:

- 하나의 값인데
    
- 그 안에 key-value 형태로 2개의 값을 담고 있고
    
- 객체(Class) 생성 없이도 args.month, args.year로 접근 가능
    

  

즉, 아래 클래스를 따로 정의하지 않아도 됨:

```
class WorklogArgs {
  final int month;
  final int? year;
}
```

이걸 한 줄로 대체한 게 Record 타입.

---

# **📌 Record 타입의 특징**

  

### **✔ 클래스를 만들지 않고 여러 값을 한 번에 전달 가능**

  

### **✔ immutable(불변)**

  

### **✔ 이름 있는 필드 사용 가능**

  

### **✔ 생산성↑, boilerplate↓**

  

### **✔ Riverpod family와 매우 잘 맞음**

---

# **📌 사용 예시**

  

### **1) 선언**

```
({int month, int year}) args = (month: 12, year: 2025);
```

### **2) 접근**

```
print(args.month); // 12
print(args.year);  // 2025
```

### **3) Provider 호출**

```
ref.watch(
  workTimesheetsProvider((month: 12, year: 2025)),
);
```

---

# **📌 네 파일에서 record 타입은 이렇게 동작**

```
FutureProvider.family<List<WorkLog>, ({int month, int? year})>(
  (ref, args) async {
     args.month; // 접근 가능
     args.year;
  }
)
```

args는 아래 형태의 **레코드**:

```
(month: int, year: int?)
```

---

# **🎯 왜 Riverpod에서 Record를 많이 쓰는가?**

  

기존에는 파라미터 여러 개 넘길 때 “값 하나만 허용해서” 무조건 클래스를 만들어야 했음.

  

### **Before (Dart 2.x)**

```
final provider = FutureProvider.family((ref, MyArgs args) {...});

class MyArgs {
  final int month;
  final int year;
}
```

### **After (Dart 3.x)**

```
final provider = FutureProvider.family((ref, ({int month, int year}) args) {...});
```

→ 간결하고 가독성 좋고 유지보수 편함.

---

# **✨ 한 줄 요약**

  

**Record(레코드)는 여러 값을 하나의 묶음으로 표현하는 Dart 3의 튜플 타입이며,**

**클래스를 만들지 않고도 args.month처럼 사용 가능한 값 컨테이너다.**

---

원하면:

- Positional Record (int, String) vs Named Record ({int month}) 차이
    
- Record를 사용하는 Riverpod 고급 패턴
    
- Record vs class 성능 비교
    

  

까지 상세하게 정리해줄게!