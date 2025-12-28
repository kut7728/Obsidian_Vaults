Flutter/Riverpod에서 **지금 보여준 class WorklogManager { ... } 같은 “순수 static 메서드만 가진 클래스”**는
다른 객체처럼 메모리에 올려지는 방식이 아니라 특수한 방식으로 동작함.

정확히 설명해줄게.

⸻

✅ 결론: static-only 클래스는 인스턴스가 생성되지 않으며,

Dart VM이 앱 시작 시 클래스 메타정보를 읽을 때 메모리에 로드된다.

즉:
µ
✔ static 메서드는
	•	앱 실행 → Dart VM이 코드 로드 → 클래스 정의를 테이블에 등록
	•	필요할 때 바로 호출
	•	인스턴스는 생성되지 않음.

⸻

📌 1. static 메서드만 있는 클래스의 메모리 로딩 방식

Dart는 다음 순서로 동작함:

① Dart VM이 앱을 실행시키면

→ 코드 전체를 읽고 모든 타입(class) 정의를 메타 영역(Metadata)에 등록함.

이것은 C++, Java 등과 유사한 방식임.

② 그런데 static-only 클래스는 인스턴스가 없음

예:

class WorklogManager {
  static void startWork(...) { ... }
}

이 경우:
	•	WorklogManager라는 클래스 메타정보만 로드
	•	생성자 호출이 없어 **객체(heap)**에 올라가지 않음
	•	static 메서드는 **클래스 레벨 테이블(VTable)**에 직접 존재
	•	호출되면 그냥 즉시 실행됨

✔ 즉, “static 클래스” 자체는 앱 실행 시점에 메모리에 로드되는 코드 정보일 뿐

별도의 객체가 heap에 생성되지 않음.

⸻

📌 2. static 클래스가 메모리를 점유하는지?

✔ 차지하는 메모리
	•	클래스 메타데이터(클래스 이름, static 메서드 참조) → 매우 작음
	•	static 변수(전역 변수처럼 동작) → 있다면 load됨

✔ 안 차지하는 메모리
	•	인스턴스 필드
	•	인스턴스 메서드 테이블(Vtable)

즉, static-only 클래스는 사실상 “네임스페이스” 같은 역할만 한다.

⸻

📌 3. static 메서드는 호출할 때만 실행된다

예:

WorklogManager.startWork(ref);

이 한 줄이 실행될 때:
	•	객체 생성 없음
	•	ref를 붙여 호출하는 “그 함수만” 실행

그래서 static 메서드는 메모리 비용 거의 없음 + 호출 비용도 매우 낮음

⸻

📌 4. Riverpod과 static 클래스 관계

여기서 매번 static 메서드에서:

ref.read(...);

처럼 provider를 직접 읽고 있어서,
이 static 클래스는 사실상 전역 비즈니스 로직 모듈처럼 쓰고 있음.

이때도 인스턴스는 절대 만들어지지 않는다.

⸻

📌 5. static 클래스는 진짜 객체가 아니기 때문에

❌ 아래처럼 사용해서 “메모리 누수” 걱정할 필요 없음

class WorklogManager {
  static void startWork(...) { ... }
}

❌ dispose도 필요없음

(static 메서드는 lifecycle과 무관)

❗단, static 변수를 가지고 있다면 메모리에 상주함

하지만 지금 코드를 보면
static 변수 없음 → 메모리 상주 문제 없음

⸻

✔ 요약

항목	static-only 클래스
언제 메모리에 로드?	앱 실행 시, 코드 로딩 단계에서 클래스 메타정보만 올라감
인스턴스 생성?	❌ 없다
메모리 점유?	메타정보 정도로 매우 적음
언제 실행?	메서드 호출 순간
Riverpod 영향?	ref를 전달하면 해당 provider들이 메모리에 살아있을 뿐, static 클래스는 부담 없음


⸻

원하면
WorklogManager를 static 클래스 대신 Riverpod 로직으로 구조 개선할 수도 있음.
(예: Service Provider 패턴으로 변경)

프로퍼티나 일반적인 메서드가 포함되면 
인스턴스 생성됨 → 힙 메모리에 자리 차지