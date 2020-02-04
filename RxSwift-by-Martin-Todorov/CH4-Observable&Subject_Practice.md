# ch4 Observable & Subjects in practice

- observable과 subject의 차이점에 대해서 제대로 알아보자

## Using a subject in a view controller 

- disposebag을 사용함으로 인해서 viewcontroller가 해제 되었을 때, all observable subscription 이 dispose 될 것이다. 굉장히 심플하다. 하지만 rootviewcontroller와 같은 VC는 원하는 대로 작동하지 않을 수도 있다. 이와 관련해서는 추후에 다룰 것이다.
- accept(_)
  - 구독자에게 업데이트된 값을 전달하기 위해서 사용한다. 이를 통해서 .next 이벤트가 발생할 것이다.

#### Adding photos to the collage

- subject이지만 Observable 의 subClass이기 때문에 직접적으로 구독이 가능하다.
- 항상 dispoebag에 제대로 담아 놓았는지 확인하자.
- subject.subscribe 를 통해서 업데이트 로직을 클로저에 담아놓자.
  - 이 때 subscribe(onNext : {}) 를 사용하여 값이 들어오는 것을 쉽게 확인할 수 있도록 하자
  - subject는 단순히 accept 를 통해서 값을 받아들이고, 방출하는 역할만 한다. 에러나 종료는 처리하지 않지만 subscribe에서는 다른 이벤트도 받을 수 있다고 판단하기 때문에 `subject.subscribe(onNext:  {})` 를 활용하자

#### Talking to other view controllers via subjects

- 만약 cocoa pattern 으로 앱을 만든다면 delegate를 만들고 이를 연결 해줘야 할 것이다. 하지만 이 방법은 non-reactive way이다.
- 하지만 RxSwift에서는 Observable 을 통해서 두개의 클래스 사이에 대화를 할 수 있다.
  - Observable은 어떠한 종류의 메세지도 담을 수 있고 어느 구독자에게나 전달할 수 있기 때문이다.

#### Creatng an observable out of the selected photos

- push 된 Photos View Controller 에서 subject를 추가하자.
  - 예제에서는 Publish Subject 그리고 Observable 을 선언했다. ( Observable 은 Subject.asObservable() 사용 ) 왜 그랬을까?
    - 아마도 캡슐화. 즉 일부의 접근을 제한하려는 의도라고 생각한다. ( 선택 된 사진들을 조작하지 못하도록 )
- 이 Observable을 통해서 접근할 수 있고, 이를 통해서 delegate를 없앨 수 있다. 관계가 굉장히 단순해지므로 기억해두자

#### Observing the sequence of selected photos

- 관찰하고자 하는 부분에서 Observable을 참조하여, subscribe를 한다. 그리고 disposed 또한 제대로 실행이 되는 지 확인해보자

#### Disposing subscriptions

- 예제대로 진행한다면 dispose 가 제대로 되지 않을 것이다. 그 이유는 사라지는 viewcontroller 에서 disposebag을 연결하지 않았기 때문이다.
  - Observable이 있는 ViewController의 disposeBag을 사용해야 한다는 사실을 잊지 말자
  - 혹은 VC가 사라질 때 completed event 를 내보내자 자동으로 dispose될 것이다.

## Creating a custom observable

- 자신만의 Observable 을 만들 수 있다. 이를 통해서 apple api를 reactive 한 class 로 변경해 볼 것이다.

#### Wrapping an existing API





