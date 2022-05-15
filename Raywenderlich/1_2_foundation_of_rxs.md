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


## Operators

`ObservableType`과  `Observable` 클래스 내부의 잘게 쪼개놓은 비동기 작업과 이벤트 변형을 추상화 해주는 수 많은 메서드들은 복잡한 로직을 구현하는데 사용이 됩니다. 이런 메서드들을 우리는 operator라고 부릅니다. 

operator들은 대부분 비동기 input을 받아서 side effect를 일으키지 않을 output만 생산해 내기 때문에, 퍼즐 조각들처럼 서로 잘 맞물리며 사용할 수 있습니다. 

예를 들어서, 이런 수학 식을 사용한다고 해 볼게요. : `(5 + 6) * 10 - 2`

`*`, `()`, `+`, `-` 같은 이 수식 내의 연산자들은 이미 순서가 정해져 있고, 어떤 데이터를 받아 어떤 과정을 통해 어떤 결과값을 만들어 낼 때 까지 계속 연산을 하게됩니다.

Rx operator들도 비슷한 느낌으로 `Observable`로부터 방출된 이벤트들을 코드가 끝나서 마지막 값을 도출할 때 까지 input과 output을 처리해 냅니다. 마지막으로 도출된 값을 side effect, 즉 다른 객체의 상태나 내용을 바꿔줄 때 활용하게 해 줍니다. 

위에서 언급된 기기의 orientation의 변화를 감지하는 코드를 Rx operator를 사용해 조정해 준 예시입니다. 

```Swift
UIDevice.rx.orientation
  .filter { $0 != .landscape }
  .map { _ in "Portrait is the best!" }
  .subscribe(onNext: { string in
    showAlert(text: string)
  })

```

`UIDevice.rx.orientation`이 `.landscape` 또는 `.portrait` 생산해 낼 때 마다, RxSwift는 `filter`나 `map`을 적용해 줄 것입니다. 
 
 ![](https://assets.alexandria.raywenderlich.com/books/rxs/images/4733ac0cfa80353413c2f1e0a5058322dfc7daca1d6cd13323b5b3fe85378083/original.png)

 먼저, `filter`는 `.landscape`가 아닌 값들만을 통과시켜줄 것 입니다. 만약에 기기가 landscape 모드라면 위 코드는 실행이 되지 않을거예요. 왜냐면 `filter`가 이 이벤트를 넘겨주지 않을 테니까요. 

 `.protrait` 값의 경우에는 `map` operator가 `Orientation`타입을 input 받아서 String output으로 변환을 해 줄 것입니다. 그 텍스트가 바로 `"Portrait is the best`입니다. 

 마지막으로, `subscribe`가 `next` 이벤트를 구독하면서, String 값을 전달해줍니다. 여기에서는 얼럿창과 텍스트를 화면에 띄워주는 메서드를 호출해 주네요. 

 Operator들은 조합을 하기가 매우 좋습니다. 언제나 데이터를 input으로 받고, 연산 결과를 output으로 내놓죠. 여러개를 다양한 방식으로 연결하면 하나의 operator가 할 수 있는 것 이상으로 다양한 역할을 할 수 있습니다. 

 ## Schedulers 
 Scheduler는 Rx와 관련있는 dispatch queue나 operation queue 입니다. 더 쉽고, 좀 더 약 빨아버린 버전이라는 것만 다르죠. Scheduler를 사용하면 태스크의 조각들에 어떤 실행 방식을 적용할 지 정의해 둘 수 있습니다. 

RxSwift는 여러개의 미리 정의해 놓은 스케줄러를 가지고 있는데요. 99%의 경우 이들을 사용하면 알맞고, 여러분이 직접 만들어 사용할 일이 거의 없다는 것을 의미하죠. 

사실, 이 교육과정의 전반전에 나오는 예시들은 거의 데이터를 observe하고 UI를 업데이트 하는 내용이라 기초적인 내용을 다 학습하기 전에는 스케줄러를 공부하시진 않을겁니다. 

그 말인 즉슨, 스케줄러가 굉장히 강력하다는 말이죠.

예를 들면, 여러분이 `SerialDispatchQueueScheduler`가 방출하는 `next`이벤트를 observe하기로 했다고 해 봅시다. 이 스케줄러는 GCD를 활용해 주어진 queue에서 여러분의 코드를 순차적으로 실행할 것 입니다. 

`ConcurrentDispatchQueueScheduler`는 코드를 concurrent하게 실행하고, `OperationQueueScheduler`는 여러분의 구독을 주어진 `OperationQueue`에 스케줄링 하도록 해 줄 거예요. 

RxSwift의 장점은 같은 구독 내에서도 각각의 수행할 업무를 다른 스케줄러에서 처리해서 여러분의 사용처에 맞게 더 좋은 효율을 내도록 스케줄링 할 수 있다는 것입니다.

RxSwift는 마치 여러분의 구독과 스케줄러 사이의 우체부처럼, 업무의 일부를 전달하고 아주 매끄럽게 각자의 output을 활용할 수 있게 해주죠. 

![](https://assets.alexandria.raywenderlich.com/books/rxs/images/28bdd14bbb8cebcb00fcdc724a10d4f34c19a2b14bcdda5c7ed1f59af513b6f4/original.png)

위의 그림을 보면, 색깔로 칠해진 각각의 업무들이 서로다른 스케줄러에 스케줄링 되어있어요. 예를 들어서 :

- 파란색 네트워크 구독은 맨 아래 `CustomNSOperationScheduler`에서 (1)번 작업을 수행할 코드 일부분을 실행시킵니다. 
- 여기에서 나온 데이터 output을 다음 블럭인 (2)번 작업의 input으로 넣어줍니다. 이 때는 또 다른 스케줄러인 `BackgroundConcurrentScheduler`에서 처리를 해주네요.

- 마지막 코드 조각인 (3)번 작업은 메인 스테드 스케줄러에서 실행시켜줘서 새 데이터를 담은 UI로 업데이트 해줍니다. 

되게 재미있고 편해보이긴 하겠지만, 지금 너무 신경쓰지는 마세요. 나중에 더 학습을 하게 될겁니다. 

## App 아키텍처

RxSwift가 여러분의 앱 아키텍처를 변화시키지는 않는다는 걸 꼭 언급을 해 두어야 겠습니다. RxSwift는 이벤트, 비동기 데이터의 sequence, 그리고 보편적인 통신 contract만을 다루고 있어요. 

그리고 꼭 언급해야 할 것은 여러분이 반응형 앱을 만들고 싶다고 해서 꼭 완전 처음부터 앱을 새로 만들어야 하는 것은 아니예요. 현재 가지고 있는 프로젝트를 차차 리팩토링 해 나가거나, 앱에 새 기능을 개발할 때 RxSwift를 써보시면 됩니다. 

MVC, MVP, MVVM 또는 여러분이 좋아하는 어떤 패턴을 쓰든 여러분은 Rx를 앱에 적용하실 수 있어요. 

RxSwift와 MVVM이 궁합이 잘 맞기는 합니다. View Controller의  UIKit와 직접적으로 bind해줄 수 있도록 Observable 프로퍼티를 드러내게 해 주기 때문이예요. MVVM을 쓰면 데이터 모델과 UI를 bind하기 쉽게 해주고 작성하기도 쉬워지죠. 

![](https://assets.alexandria.raywenderlich.com/books/rxs/images/0625dc8cc2e93bdc9324fafea84fadaaf4729dfd39d114996486bb185bdb53e0/original.png)




