## 시작하기

이제 초콜릿 맛 좀 볼까요?   
페이지 상단 또는 하단에서 튜토리얼 프로젝트를 다운받아주세요.  
받으신 뒤에는 `Chocotastic.xcworkspace`파일을 열어주세요.  
빌드하고 실행해 보세요.  
아래와 같이 각각의 유럽 국가에서 살 수 있는 초콜릿의 가격을 리스트로 보여주는 화면을 보시게 될 겁니다. 

초콜릿을 탭해서 장바구니에 담아보세요!  
우측 상단의 장바구니를 탭해보세요! 

다음 페이지에서 결제로 진행하거나 카트를 리셋할 수 있습니다.  
결제버튼을 클릭하면 신용카드 입력폼이 나오도록 되어있어요.  

튜토리얼을 진행하다 보면 이 화면을 완전히 반응형 프로그래밍으로만 만들게 될 거예요. 지금은 카트 버튼을 눌러서 카트 요약 화면으로 돌아갑시다.  
그리고 리셋 버튼을 탭해서 장바구니를 비우고 메인 화면으로 돌아가세요. 

</br>
</br>

## 시작 지점: Nonreactivity 

앱이 어떻게 동작하는지 보셨으니 이제 앱이 어떻게 동작하는지를 알아볼 차례군요.  
`ChocolatesOfTheWorldViewController.swift` 파일을 열어보세요. 기본적인 `UITableViewDelegate`와 `UITableViewDataSource` 익스텐션이 보일겁니다. 

그럼 이제 `updateCartButton()`을 보세요.  
이 메서드는 카트 버튼을 현재 카트에 들어있는 초콜릿의 개수와 함께 업데이트 해주고 있습니다. 이 메서드는 두 인스턴스를 통해 카트를 업데이트 해주고 있어요. 
```
1. viewWillAppear(_:) : 뷰 컨트롤러가 보이기 전에
2. tableView(_:didSelectRowAt:) : 유저가 새로운 초콜릿을 카트에 담았을 때
```

위 두 경우는 명령형으로 장바구니 개수를 변경해주고 있어요. 꼭 명시적으로 메서드를 호출해서 개수를 업데이트 해줘야 하는거죠.  

이제 이 코드를 반응형 테크닉을 사용해 다시 써볼겁니다. 그러면 버튼이 알아서 업데이트를 하게 될거예요. 

</br>
</br>

## RxSwift: 장바구니 개수를 반응형으로 세도록 만들기

장바구니에 담긴 아이템의 개수를 참조하는 메서드들은 `ShoppingCart.sharedCart`라는 싱글턴을 참조하고 있어요.  
`ShoppingCart.Swift` 파일을 열어보시면 싱클턴 인스턴스 내에 기본적으로 변수가 선언되어 있는 것을 보실 수 있을거예요. 

```swift
var chocolates: [Chocolate] = []
```

지금 당장은 이 chocolates라는 변수의 변화를 감지하지는 못하실거예요.  
`didSet` 클로저를 사용하실 수도 있지만, `didSet`은 전체 배열이 업데이트 되었을 때만 불러와집니다.  
배열 원소의 일부가 변경되었을 때는 `didSet`이 호출되지 않지요. 

다행히도 RxSwift가 이 문제를 해결할 수 있습니다. `chocolates` 변수를 아래와 같은 코드로 변경해보세요.
```swift
let chocolates: BehaviorRelay<[Chocolate]> = BehaviorRelay(value: [])
```
> 참고 : 이렇게 변경하시면 아마 사이드바에 오류가 뜨게 될겁니다. 곧 고칠거니까 걱정 마세요.

이런 문법을 보고 머리가 어질어질 하실 수도 있어요. 뭐가 어떻게 돌아가는지 설명을 해드릴게요. 
가장 중요한 것은 `chocolates`이라는 `Chocolate` 객체들을 원소로 하는 Swift 배열로 만들어 준 바로 그것을 
RxSwift의 `BehaviorRelay`의 타입으로 정의해 준 것이라고 생각하시면 됩니다. 

`BehaviorRelay`는 클래스이기 때문에, reference semantics(참조 특성)를 사용합니다.  
즉, `chocolates`가 `BehaviorRelay` 인스턴스를 참조하고 있다는 것을 의미합니다.

`BehaviorRelay`는 `value`라는 프로퍼티를 가집니다. 이 프로퍼티에 `Chocolate` 객체의 배열을 저장하는 것이죠. 
`BehaviorRelay`의 장점은 `asObservable()` 메서드를 호출할 때 드러납니다.  
수동으로 매번 `value`를 체크할 필요 없이, `Observer`를 추가해서 변하는 값을 감시할 수 있는것이죠. 
값이 바뀔 때, `Observer`가 알려주니까 어떠한 업데이트에도 반응할 수 있게 됩니다. 

단점이라고 하면 만약 여러분이 `chocolates`의 배열 안에 뭔가를 바꿔주거나 접근하려고 할 때, 무조건 `accept(_:)`를 통해서 접근하셔야 한다는 거예요. 
`BehaviorRelay`내의 이 메서드가 `value` 프로퍼티를 업데이트 해 주는 녀석입니다. 
그래서 컴파일러가 그렇게 빼액 울어대고, 에러 뭉텅이를 보여주는거죠. 그럼 이제 한번 고쳐볼까요?

</br>
</br>

### ShoppingCart.swift 파일에서...
`totalCost()` 메서드를 찾고  
다음의 라인을 :
```swift
return chocolates.reduce(0) {
```
아래와 같이 바꾸세요 :
```swift
return chocolates.value.reduce(0) {
```

</br>

`itemCountString()` 메서드를 찾고  
다음의 라인을 :
```swift
guard chocolates.count > 0 else {
```
아래와 같이 바꾸세요 :
```swift
guard chocolates.value.count > 0 else {
```

</br>


다음의 라인을 :
```swift
let setOfChocolates = Set<Chocolate>(chocolates)
```
아래와 같이 바꾸세요 :
```swift
let setOfChocolates = Set<Chocolate>(chocolates.value)
```

</br>

마지막으로
다음의 라인을 :
```swift
let count: Int = chocolates.reduce(0) {
```
아래와 같이 바꾸세요 :
```swift
let count: Int = chocolates.value.reduce(0) {
```

</br>
</br>

### CartViewController.swift 파일에서...
`reset()` 메서드를 찾고  
다음의 라인을 :
```swift
ShoppingCart.sharedCart.chocolates = []
```
아래와 같이 바꾸세요 :
```swift
ShoppingCart.sharedCart.chocolates.accept([])
```

</br>
</br>

### ChocolatesOfTheWorldViewController.swift 파일로 돌아와...
`updateCartButton()` 메서드가 적용된 부분을
아래와 같이 바꾸세요 :
```swift
cartButton.title = "\(ShoppingCart.sharedCart.chocolates.value.count) \u{1f36b}"
```

</br>

`tableView(_:didSelectRowAt:)` 메서드를 찾고
다음의 라인을 :
```swift
ShoppingCart.sharedCart.chocolates.append(chocolate)
```
아래와 같이 바꾸세요 :
```swift
let newValue =  ShoppingCart.sharedCart.chocolates.value + [chocolate]
ShoppingCart.sharedCart.chocolates.accept(newValue)
```

</br>

웨우!! 위의 작업을 통해 애러들을 처리하고 Xcode를 행복하게 만들어줬네요.  
이제 당신은 반응형 프로그래밍의 이점을 취하고 `chocolates`를 감시할 수 있습니다!

이제 `ChocolatesOfTheWorldViewController.swift`로 돌아가서 아래의 코드를 프로퍼티 목록에 추가해주세요.

```swift
private let disposeBag = DisposeBag()
```

이 코드는 `DisposeBag`을 만들어 주는 코드입니다. 이걸 사용해서 우리가 만든 Observer들을 청소해(clean up) 줄 것입니다. 

이제 `//MARK: Rx Setup`주석 아래의 extension에 추가해주세요.

```swift
func setupCartObserver() {
  //1
  ShoppingCart.sharedCart.chocolates.asObservable()
    .subscribe(onNext: { //2
      [unowned self] chocolates in
      self.cartButton.title = "\(chocolates.count) \u{1f36b}"
    })
    .disposed(by: disposeBag) //3
}
```
이 함수는 장바구니를 자동으로 업데이트 하도록 반응형 옵저버를 만들어 줍니다. 보시다시피 RxSwift는 연속된 함수들을 
굉장히 많이 사용하는데요, 그 말인 즉슨 각각의 함수들이 이전 함수의 결과값을 사용하고 있다는 뜻이죠.

이 예시에 어떻게 그런 현상이 나타나는지 봅시다:

1. 장바구니의 `chocolates` 변수를 `Observable`로 만들어 줍니다.

2. 만들어준 `Observable`에서 `subscribe(onNext: )`를 호출해서 `Observable`의 값이 변화하는 것을 발견하도록 합시다. `subscribe(onNext: )`은 값이 변할때 마다 실행되는 클로저를 차용하고 있습니다. 이 클로저에 들어오는 파라미터는 여러분이 만든 `Observable`의 새로운 값입니다. 이런 구독관계를 구독취소(unsubscribe)나 버리기(Dispose)할 때 까지는 계속 이런 알림(notification)을 받게 되실거예요. 이 방법을 통해 여러분이 돌려받는 것은 `Disposable`에 부합하는 `Observer`입니다. 

3. 이전 단계에서 받은 `Observer`를 `disposeBag`에 넣어주세요. 이를 통해 구독중인 객체를 없애줌으로서 구독을 없앨 수 있어요. 

마지막으로 명령형 함수인 `updateCartButton()`을 지워주세요. 이렇게 하시면 아마 `viewWillAppear(_:)`과 `tableView(_:didSelectRowAt)`에서 호출이 되었다보니 에러를 일으킬거예요. 

이걸 고치려면, 일단 `viewWillAppear(_:)` 전체를 지워주세요. `super` 이외에는 `updateCartButton()`밖에는 호출하지 않고 있으니까요. 그러고 나서 `tableView(_:didSelectRowAt)`내의 `updateCartButton()`를 지워주세요.

빌드하고 실행시켜보세요. 아래와 같은 초콜릿의 리스트가 보이실거예요. 


어이쿠야. 장바구니 버튼에 Item이라고만 뜨네요. 그리고 초콜릿 목록을 탭해도 아무일도 일어나지 않아요. 뭐가 문제일까요?

여러분은 Rx Observer를 만드는 함수를 만드셨지만, 그 함수를 전혀 호출해주고 있지 않기때문에 Observer가 응답을 하지 않는거예요. `ChocolatesOfTheWorldViewController.swift` 파일을 여시고 `viewDidLoad()` 끝에 아래의 코드를 더해주세요. 

```swift
setupCartObserver()
```

빌드하고 실행시켜보시면 아까처럼 초콜릿의 리스트가 보이실겁니다.

아무 초콜릿이나 탭해보세요 -- 이제 장바구니의 아이템 갯수가 자동으로 업데아트가 되는군요!

대성공! 이제 어떤 초콜릿이든 장바구니에 담을 수 있게 되었습니다. 
