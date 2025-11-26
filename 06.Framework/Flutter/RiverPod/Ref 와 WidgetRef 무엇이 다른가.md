
<br>

# 개요
- provider를 사용하기 위해 필요한 ref
- 하지만 가끔 어디서는 Ref ref, 어디서는 WidgetRef ref 인 경우가 있다
- 둘의 차이는 뭐고, 왜 같이 사용할 수 없는 걸까

> [!info] 요약
> Ref : Provider 내부에서 다른 provider와 소통할 때 사용
> WidgetRef : Widget 내부에서 provider와 소통할 때 사용

## Ref
- Provider가 생성되거나 동작할 때 전달되는 객체
- 사용위치:
	- Provider 정의부
	- Notifier 클래스 내부
- 주요역할 : 다른 provider 상태 읽기, 구독하기, provider의 생명주기 관리

``` dart
// Provider 정의부
final cityProvider = Provider((ref) => 'Seoul');

// weatherProvider가 cityProvider를 참조(watch)하고 있습니다.
// 이때 전달받는 ref는 'Ref' 타입입니다.
final weatherProvider = Provider((ref) {
  // 1. 다른 Provider의 값을 읽어옴
  final city = ref.watch(cityProvider);
  
  // 2. 생명주기 관리 (이 Provider가 파괴될 때 실행)
  ref.onDispose(() {
    print('Weather Provider Disposed');
  });

  return '$city is Sunny';
});
```


## WidgetRef
- Flutter의 위젯트리 속에서 provider에 접근할 수 있도록 해주는 객체
- 사용 위치 : 
	- consumerWidget의 build 메서드 속
	- consumerStatefulWidget의 State 클래스 속
- 주요 역할 : Provider의 상태를 구독하여 위젯 다시 렌더링, 상태 수정

``` dart
// ConsumerWidget을 상속받음
class WeatherScreen extends ConsumerWidget {
  @override
  // build 메서드의 두 번째 인자로 'WidgetRef'가 들어옵니다.
  // 보통 변수명을 'ref'라고 짓기 때문에 헷갈리지만, 타입은 WidgetRef입니다.
  Widget build(BuildContext context, WidgetRef ref) {
    
    // 1. 상태 구독 (값이 변하면 위젯 리빌드)
    final weather = ref.watch(weatherProvider);
    
    return Scaffold(
      body: Center(
        child: Text(weather),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // 2. 이벤트 내에서는 리빌드가 필요 없으므로 read 사용
          print(ref.read(cityProvider)); 
        },
      ),
    );
  }
}
```

<br>

# 두군데에서 다 쓰고 싶으면요?
- "이 로직은 위젯의 버튼을 눌렀을 때도 실행돼야 하고, 다른 프로바이더가 상태를 갱신할 때도 실행돼야 하는데?" 싶은 경우면?
- 해당 메서드를 포함해서 클래스를 만들고, 프로바이더로 제공한다!
- "Ref를 넘기지 말고, Provider를 넘기세요"

