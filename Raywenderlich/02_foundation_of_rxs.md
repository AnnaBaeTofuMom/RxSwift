# Raywenderlich Chapter 1 - Foundation of RxSwift

## RxSwift의 기초

반응형 프로그래밍은 새로운 개념은 아닙니다. 꽤나 오랜기간 존재해 왔지만 그 핵심 개념은 지난 10년동안 눈에 띄게 귀환했습니다. 

그 기간동안, 웹 앱이 좀 더 대중화 되었고, 복잡한 비동기 UI 처리가 이슈로 떠올랐죠. 서버 쪽에서는 반응형 시스템이 필수가 되었습니다. 

마이크로소프트의 한 팀은 비동기, 확장성, 실시간 앱 개발이라는 문제를 풀기뒤한 도전을 하기 시작했습니다. 2009년 즈음 그들은 새로운 클라이언트와 서버사이드 프레임워크를 만들었고 그것이 바로 .NET을 위한 Reactive Extensions, 즉 Rx입니다. 

.NET을 위한 Rx는 2012년 소스를 오픈해서 다른 언어나 플랫폼에 같은 기능을 재적용 하는 것을 허용하기 시작했습니다. 이러한 과정을 통해 Rx는 플랫폼을 넘나드는 하나의 기준이 되어갔죠. 

오늘날에 여러분은 RxJS, RxKotrlin, Rx.NET, RxScala, RxSwift와 그 밖의 것들을 제공받고 있죠. 모두 같은 동작을 적용하고, Reactive Extensions 규칙에 따른 표현형 API를 적용하려 힘쓰고 있습니다. 궁극적으로, RxSwift를 사용해 iOS 앱을 개발한 개발자와 RxJS를 활용해 웹을 개발한 개발자와 로직에 관해 자유롭게 토론할 수 있게 만들어 주었죠. 

태초의 Rx처럼, RxSwift 또한 지금까지 언급했던 개념들과 같이 동작합니다. 이벤트의 순서를 구성할 수 있도록 해주고, 아키텍쳐적인 개념인 코드 독립성, 재사용성과 비의존성 등을 향상시켜줄 수 있죠. 

Rx 코드들은 크게 <b>Observables, Operators, Schedulers</b>로 구성되어 있습니다. 아래의 섹션들은 이 구성요소들에 대한 상세 설명입니다. 

## Observables

`Observable<Element>`는 Rx code의 근간이 됩니다. 비동기적으로 이벤트의 연속을 만들고, 이때 불변 상태의 generic data를 옮길(carry) 수 있는 능력이 있죠. 단순하게 말하면, '소비자'가 다른 객체가 발신하는 이벤트나 값을 <b>구독</b>할 수 있게 해주는 것이죠. 

`Observable` 클래스는 하나 또는 복수개의 observer가 어떤 이벤트든 실시간으로 반응할 수 있도록 해주고, 앱의 UI를 업데이트 해주거나 새로운 데이터나 들어오는 데이터를 활용할 수 있게 만들어 줍니다. 

`ObservableType` 프로토콜은 정말 간단합니다. Observable은 딱 3개의 타입의 이벤트만 방출이 가능합니다.

- next 이벤트 : 이벤트가 최신의 데이터 값을 전달합니다. 이를 통해 observer가 값을 전달받게 되죠. Observable은 terminating event가 방출될 때 까지 무한히 값들을 전달하게 됩니다. 

- complete 이벤트 : 이 이벤트는 기존의 연속되고 있는 이벤트를 성공으로 간주하고 종료시킵니다. Observable이 성공적으로 스스로가 가진 lifecycle을 다하게 되면 더이상 이벤트를 방출하지 않는다는 뜻이죠. 

- error 이벤트 : Observable이 에러와 함께 종료되며 더이상 이벤트를 방출하지 않습니다. 

비동기 이벤트가 시간에 따라 방출될 때, observable의 흐름을 타임라인위의 정수들로 표현해 보면 다음과 같습니다.

![](https://assets.alexandria.raywenderlich.com/books/rxs/images/f8d3cff7dafeb96562b1d9031cf41b30959aea0c036be76b0bb03070e392fed9/original.png)

Observable이 방출할 수 있는 이 세가지 이벤트야말로 Rx의 시작이자 끝이라고 볼 수 있습니다. 굉장히 보편적이고, 이것을 통해 정말 복잡한 로직도 다 구현해 낼 수 있기 때문이죠. 

Observable contract는 Observable 또는 observer의 본질에 어떠한 추측도 하지 않기 때문에, 이벤트 시퀀스를 사용하는 것이야 말로 궁극의 decoupling 사례입니다. 

클래스 간에 서로 대화하도록 하기 위해서 델리게이트 프로토콜을 사용하거나, 클로저를 삽입할 일이 없어지죠.

![](https://assets.alexandria.raywenderlich.com/books/rxs/images/5e255ce9e0cb680c862ff81cddb3f721957ced5c1fa019660974b265367f0fd2/original.png)

실제 상황으로 예를 들어서 감을 잡아보기 위해, 여기 두개의 서로 다른 observable sequence들을 준비했습니다. finite(한정)와 infinite(무한정), 두 가지로요. 

### Finite observable sequences 

어떤 observable sequence들은 0개, 1개 또는 여러개의 값들을 방출하고, 나중에는 성공적으로 종료(terminate)되거나 에러를 반환하며 종료되기도 하죠. 

iOS앱에서 인터넷 상에서 파일을 다운로드 받는 코드를 쓴다고 가정해보세요. 

- 먼저, 다운로드를 시작하고, 들어오는 데이터를 observe하기 시작할 것 입니다.

- 그리고 나서 파일의 일부인 데이터 덩어리들을 반복적으로 받게 되겠지요.

- 중간에 갑자기 네트워크 연결이 끊어진다면, 다운로드는 중단되고 시간 초과와 함께 에러를 반환할 것 입니다. 

- 반대로 파일의 데이터를 다 다운로드 받으면, 성공과 함께 완료를 하겠죠. 

이러한 workflow가 전형적인 observable의 lifecycle을 표현한 것 입니다. 아래의 코드를 한 번 보시겠습니다.

```Swift
API.download(file: "http://www...")
   .subscribe(
     onNext: { data in
      // Append data to temporary file
     },
     onError: { error in
       // Display error to user
     },
     onCompleted: {
       // Use downloaded file
     }
   )

```

`API.download(file:)`은 Observable<Data> 인스턴스를 반환하고 있습니다. 이 인스턴스는 네트워크에서 fetch해 온 데이터 덩어리를 `Data`의 갑으로 방출하죠. 

여러분은 `onNext` 클로저가 제공하는 next 이벤트를 구독하고 있습니다. 다운로드 예제에서 여러분은 데이터를 디스크에 저장된 임시 파일에 넣어주고 있죠.

여러분은 `onError` 클로저가 제공하는 error도 구독하고 있습니다. 이 클로저 내에서, 여러분은 `error.localizedDescription`을 얼럿 상자에 띄워 보여줄 수도 있고, 여러분이 직접 error를 핸들링 하셔도 됩니다. 

마지막으로 `completed` 이벤트를 핸들링 하기 위해 `onCompleted` 클로저를 활용해 새로운 뷰 컨트롤러를 push 해 주거나, 다운로드 받은 파일을 보여주거나 하는 등 앱 로직히 원하는 무엇이든 할 수 있죠. 

### Infinite observable sequences

자연스럽게든, 강제로든 종료를 시켜줘야 하는 파일을 다운로드와는 다르게 무한히 sequence가 이어지는 경우들이 있습니다. UI 이벤트들이 대부분 infinite observable sequence가 됩니다. 

예를 들어서, 기기의 오리엔테이션이 바뀔 때 반응해 주는 코드를 상상해보세요. 

- `UIDeviceOrientationDidChange` 의 알림이 `NotificationCenter`를 통해 전달될 때 그것을 관찰하고 있는 클래스를 만들어 줍니다. 

- 메서드 콜백을 만들어 orientation의 변화를 핸들링할 것입니다. 현재 orientation을 `UIDevice`로 부터 받아서 가지고 있다가 최신 값에 따라 반응해줘야 겠죠. 

이런 orientation의 변화는 사실 자연적인 끝이라는건 없습니다. 디바이스가 존재하고, orientation이 바뀔 확률이 있다면 계속 존재하는거죠. 게다가, 이러한 sequence가 무한하고, stateful하기 때문에, 그것을 관찰하기 시작하는 그 순간부터 항상 초기값을 가지고 있어야 합니다. 

![](https://assets.alexandria.raywenderlich.com/books/rxs/images/37f39e4b50e8f7ba826a8fd6c8293695a8bb55e51a4c9a15b8a9e1590fd0c20a/original.png)

유저가 화면을 아예 안돌린다고 해서 이 이벤트의 sequence가 종료되는 것은 아닙니다. 그냥 지금까지 아무런 이벤트가 방출되지 않았다는 것만을 의미하지요. 

RxSwift로 기기의 orientation을 핸들링하는 코드를 쓰면 아래와 같을 것 입니다. 

```Swfit
UIDevice.rx.orientation
  .subscribe(onNext: { current in
    switch current {
    case .landscape:
      // Re-arrange UI for landscape
    case .portrait:
      // Re-arrange UI for portrait
    }
  })

```

`UIDevice.rx.orientation`은 가상의 제어 프로퍼티로 `Observable<Orientation>`을 생산해냅니다. 여러분은 그것을 구독해서, 여러분의 앱 UI를 현재 orientation에 맞게 업데이트를 해 줄것 입니다. 이 때, `onError`와 `onCompleted` 표현은 생략합니다. 왜냐하면 이 이벤트는 observable로부터 방출되는 것이 아니니까요. 


