# Observables

지금까지 RxSwift에 대한 기초 개념을 배웠으니, 이제 Observable을 가지고 놀아볼 시간입니다.

이번 챕터에서는, 옵저버블을 만들고, 구독하는 여러 예시에 대해 배우게 될 것입니다. 현업에 나가서 보는 옵저버블은 다소 이해하기 어려워 보일 수 있지만, 여러분은 이제 중요한 스킬들을 획득하고, RxSwift로 만들 수 있는 여러 종류의 옵저버블에 대해 배우게 될겁니다. 여러분이 배운 스킬은 이 책을 끝낼때까지, 그리고 그 이상으로 계속 쓰일것입니다. 

## 준비하기
이번 챕터에서, 여러분은 Xcode 프로젝트를 사용하게 될거고, 그 안에는 이미 플레이 그라운드와 RxSwift 프레임워크가 들어있을 겁니다. 시작하기에 앞서서 터미널을 여시고, 이번 장의 starter 폴더로 들어가셔서 RxPlayground 프로젝트 폴더로 이동하세요. 마지막으로 `bootstrap.sh` 커맨드를 입력하시면 됩니다.

```
./bootstrap.sh
```

부트스트랩 과정은 몇 초 정도 걸릴겁니다. 유념하실 부분은 여러분이 playground 프로젝트를 여실 때, 위의 과정을 꼭 반복하셔야 한다는 것 입니다. 그냥 바로 playground 파일을 열거나 workspace를 바로 여시면 안됩니다. 

**프로젝트 네비게이터**에서 **RxSwiftPlayground**를 선택하시면 아래와 같은 화면이 보이실 겁니다.
![](https://assets.alexandria.raywenderlich.com/books/rxs/images/43bec9ed4df895e2b0cf433715118e967aadfb48cc97bbe3aec8ff81d587222e/original.png)

**프로젝트 네비게이터**내의 **Source**폴더에서 playground 페이지를 열어서 **SupportCode.swift**을 선택해주세요. 아래와 같은 함수 `example(of:)`가 있을겁니다. 

```Swift
public func example(of description: String, action: () -> Void) {
  print("\n--- Example of:", description, "---")
  action()
}
```

여러분은 이제 이 함수를 서로다른 예시들을 캡슐화 하는데 사용하시게 될 겁니다. 이 함수를 어떻게 사용할지는 곧 알게 되실거예요. 그보다, 지금이야말로 이 질문을 던지기 좋은 타이밍인것 같군요. 도대체 *옵저버블*이란 무엇일까요?

