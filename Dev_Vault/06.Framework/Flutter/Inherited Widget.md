# **개요**

위젯트리 최상단 위젯의 어떤 데이터를 하위 위젯에서 사용하고 싶을 때 가장 기본적인 방법은  
생성자에 파라미터로 받을 수 있도록 뚫어주는 방법이 있지만... 귀찮다! 유지보수도 힘들고 중간과정의 위젯도 모두 뚫어줘야하는 문제가 있다!  
이럴 때 사용할 수 있는 방법이 바로 **InheritedWidget**이다!

> [!info] 요약
> 위젯트리 최상단을 InheritedWidget으로 시작하여 언제든지 하위위젯에서 접근가능하도록 할 수 있다!!

# **사용법**

InheritedWidget을 상속받는 Class를 위젯트리 최상단이 되도록 구현하고  
하위 위젯에서 of 메서드를 활용해서 접근할 수 있다.

## InheritedWidget 구현

``` dart
class MyInheritedWidget extends InheritedWidget {
  final int counter;

  const MyInheritedWidget({
    Key? key,
    required this.counter,
    required Widget child,
  }) : super(key: key, child: child);

  // 하위 위젯이 이 위젯을 구독하기 위한 정적 메서드
  // dependOnInheritedWidgetOfExactType를 통해서 context상에서 가장 가까운 inheritedWidget 찾아줌
  static MyInheritedWidget of(BuildContext context) {
    final MyInheritedWidget? result =
        context.dependOnInheritedWidgetOfExactType<MyInheritedWidget>();
    assert(result != null, 'No MyInheritedWidget found in context');
    return result!;
  }

  // counter 값이 바뀌었을 때만 하위 위젯을 리빌드할지 결정
  @override
  bool updateShouldNotify(MyInheritedWidget oldWidget) {
    return oldWidget.counter != counter;
  }
}
```

> [!note] of 함수
> - 현재 위젯의 위쪽 방향으로 가장 가까운 위젯을 찾는 함수
> - Scaffold.of(context) 현재 context기준에서 위쪽방향으로 가장 가까운 Scaffold 찾기
> - cost가 많이 드는 함수 → 대신 global key 이용하는 편
> 


## 하위 위젯에서 접근하기

```
void main() {
  runApp(
    MyInheritedWidget(
      counter: 5,
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    final inherited = MyInheritedWidget.of(context)!;

    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Text('Counter: ${inherited.counter}'),
        ),
      ),
    );
  }
}
```

# **사실 이미 쓰고 있었던 inherited Widget**

나도 모르게 사용하고 있던 경우가 있는데 바로 다음과 같은 경우이다.

-   Theme.of(context) → 내부적으로 InheritedWidget(Theme) 사용
-   MediaQuery.of(context) → 화면 크기 정보를 제공하는 InheritedWidget
-   Localizations.of(context) → InheritedWidget을 활용하여 현재 앱의 언어 설정 데이터를 제공

# **Provider도 inherited Widget?**

상태(state)를 관리하고 트리 내 여러 위젯이 동일한 데이터를 쉽게 공유할 수 있도록 해주는 라이브러리인  
Provider는 InheritedWidget을 더 편리하게 사용하기 위해 만들어진 추상화 라이브러리입니다.  
즉, Provider는 InheritedWidget의 래퍼(wrapper) 라고 보면 정확합니다.