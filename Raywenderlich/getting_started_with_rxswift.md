# Getting Started With RxSwift and RxCocoa

: chocolate-buying 앱을 짜증나는 “명령형”에서 “반응형”로 데려가기 위해 RxSwift와 RxCocoa 프레임워크를 사용해보자.

---

다음 [원문](https://www.raywenderlich.com/1228891-getting-started-with-rxswift-and-rxcocoa)를 해석한 내용입니다.

---

(나의 고양이와는 달리) 댕신이 원하는 대로 정확히 코드가 작동할 때는 행복하다.  
프로그램을 변경하고, 코드에게 업데이트할 것을 말해주고, 그것이 잘 작동한다면 좋은 코드이다!

</br>

객체지향 시대의 대부분의 프로그래밍은 `명령형` 이었다.  
코드는 당신의 프로그램이 무엇을 할지 말해주고 변경하기 위해 들을(listen) 방법은 많이 가지고 있다.  
하지만, 무엇가 변화할 때 당신이 시스템에게 알려 주어야만 한다.

</br>

코드가 자동으로 변화를 반영할 수 있게 설정해둘 수 있다면 훨씬 좋지 않을까?  
이게 바로 `반응형 프로그래밍`의 아이디어 이다.
```
당신의 앱이 당신이 무엇을 하라고 지시하는 것 없이 데이터의 변화에 따라 반응할 수 있다.
```
이것은 특정한 상태를 유지하는 것보다 로직에 집중하기 더 쉽게 만들어 준다.

</br>

스위프트에서는 `KVO(Key-Value Observartion)`과 `didSet`을 사용하여 구현할 수 있지만,  
설정하기 까다로울 수 있다.  
대신, 스위프트에는 반응형 프로그래밍을 가능하게 해줄 몇몇 프레임워크가 있다.


> 추가적인 배경지식으로,  
> Colin Eberhandt가 주요 프레임워크들의 차이를 비교한 [좋은 글](https://www.raywenderlich.com/1190-reactivecocoa-vs-rxswift)이 있다.  
> 추가적인 토론을 위해 comments 섹션을 확인해보자.

</br>

이 튜토리얼에서는, `RxSwift` 프레임워크를 사용할 것이고 그것과 짝을 이루는 `RxCocoa`를 사용하여 chocolate-buying 앱을 명령형에서 반응형으로 데려갈 것이다.

## 💬 RxSwift와 RxCocoa는 무엇인가요?

`RxSwift`와 `RxCocoa`는 다수의 프로그래밍 언어와 플랫폼에 걸쳐 [ReacticeX](http://reactivex.io/)(Rx) 에 적합한 언어 툴의 일부이다.  

</br>

`ReactiveX`는 `.NET/C#` 생태계의 일부로써 시작되었고 그동안, `Ruby, JavaScript를 사용하는 사람들` 그리고 특히 `Java와 안드로이드 개발자들` 사이에서 극도로 유명해졌다.  

</br>

`RxSwift`는 스위프트 언어와 상호작용하는 프레임워크인 반면,  
`RxCocoa`는 iOS와 OS X에서 사용된 Cocoa API를 반응형 기술과 함께 더 사용하기 쉽게 만들어주는 프레임워크이다.

</br>

`ReactiveX` 프레임워크는 작업(tasks)을 위한 다른 프로그래밍 언어를 통틀어 반복적으로 사용되는 공통의 단어/어휘들을 제공한다.
이것은 각각의 새로운 언어의 공통된 작업을 어떻게  구현할지 알아내는 것 보다 언어 자신의 구조(syntax) 자체에 집중하기 쉽게 해준다.

## 💬 Observables와 Observers

두 주요 개념은 `Observable`과 `Observer` 이다.
```
- Observable : 변화의 알림들(notifications)을 내뿜는다.
- Observer : 하나의 'Observable' 구독하고, 'Observable'이 변화했을 때 알림을 받는다(알아차려진다).
```

</br>

하나의 `Observable`을 주시하고 있는(listening) 다수의 `Observer`들을 가질 수 있다.  
`Observable`이 변화할 때, 그것의 모든 `Observer`들을 알아차릴(notify) 것이다.

## 💬 DisposeBag
```
'DisposeBag'는 'RxSwift'가 ARC와 메모리 관리를 다루는데 도움을 제공하는 추가적인 툴이다.  
```
`DisposeBag` 안에서 `Observer` 오브젝트의 페기물(disposal) 안의 부모 오브젝트 결과들을 메모리에서 해제(Deallocating)한다.

</br>

`DisposeBag`을 들고있는 오브젝트에 `deinit()` 이 호출될 때, 각 `disposable Observer`는 자동으로 그것들이 감시(observing)하고 있던 것의 구독을 해제(unsubscribe)한다.

</br>

`DisposeBag`이 없다면, 당신은 두 결과 중 하나를 얻게 될 것이다.  
`Observer`가 순환 참조(retain cycle)을 만들어내어 무기한 감시하고 있거나, 메모리에서 할당 해제될 순 있지만 충돌(crash)를 일으킬 것이다.

</br>

좋은 ARC 시민이 되기 위해선, 어느 `Observable` 오브젝트를 설정할 때 그것을 `DisposeBag`에 추가하는 것을 기억해라.  
`DisposeBag`이 당신을 위해 훌륭하게 치워줄 것이다.

</br>
</br>
</br>

.  
.  
.  
[이어보기](https://github.com/keenkim1202/RxSwift_Practice/blob/main/raywenderlichEx/getting_started_with_rxswift_project.md)
