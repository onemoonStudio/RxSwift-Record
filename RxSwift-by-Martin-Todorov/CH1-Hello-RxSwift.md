# Hello RxSwift

### RxSwift 란 뭔가?

- **RxSwift is a library for composing asynchronous and event-based code by using observable sequences and functional style operators, allowing for parameterized execution via schedulers.**
- RxSwift 는 옵저버블 시퀀스와 함수 스타일 연산자를 사용하여 비동기 및 이벤트 기반 코드를 구성하도록 도와주는 라이브러리 이며, 스케쥴러를 통해서 매개변수화 된 실행을 가능하게 한다.
- 본질적으로 새로운 데이터에 반응하고 순차적으로 격리된 방식으로 처리할 수 있도록 하여 비동기 프로그램 개발을 단순화 한다.

### 비동기 프로그래밍

- 단순하게 아래 리스트중 무엇이든 혹은 더 많은 것을 하는 것이다.
  - 버튼 탭에 반응하는것
  - 텍스트 필드가 focus 를 잃었을 때 애니메이션 처리
  - 인터넷에서 큰 사진을 다운로드 하는 것
  - 오디오 재생 
  - 등등...
- 자신의 프로그램의 모든 서로 다른 부분들이 실행에 대해서 서로에게 block시키지 않는 것이다.

### Cocoa & UIKit 비동기 API

- Apple에서 제공하는 비동기를 도와주는 대표적인 API는 다음과 같다.

  - NotificationCenter : 
    - 어떤 관심 있는 이벤트가 발생 했을 때 언제든 코드를 실행 할 수 있게끔 하는 것 
    - 예를 들어 화면의 방향 전환 및 키보드 보여주는 이벤트를 캐치할 수 있다.
  - Delegate Pattern
    - 어떤 객체에 대한 행동을 대표해주는 객체를 정의하는 것

  - GCD ( Grand Centeral Dispatch )
    - 작업의 실행을 추상화 할 수 있도록 도와주는 것
    - 코드의 실행 계획을 세울 수 있다. ( 예를 들어 serial Queue 혹은 concurrent on diffrent Queue & priorities 등을 활용한다.)
  - Closures
    - 코드를 전달하기 위해서 코드의 일부분을 떼어내는 것

- 애플에서 제공하는 API들은 굉장히 강력하다 하지만 애플에서 다양한 API를 제공해 줌으로써 쓰기 복잡할 수도 있다.

  <img src="./imgs/img11.png" alt="img11" style="zoom:30%;" />

### 동기 코드

- for문을 생각해보자 
  - 동기적인 코드란 collection이 변하지 않고 반복한다는 것이다.
  - collection이 있는지 확인할 필요가 없으며, 다른 쓰레드가 element를 추가하는 것을 생각하지 않아도 된다.

```swift
var myArray = [1,2,3]
for element in myArray {
    print(element) 
    myArray = [4,5,6]
}
print(myArray)
// 1
// 2
// 3
// [4, 5, 6]

```



### 비동기코드

- 위의 코드는 array를 모두 확인하는데 문제가 없다 그렇다면 아래의 코드를 보자

```swift
var currentIndex = 0 
@IBAction func didTapButton(_ sender: UIButton) {
  print(myArray[currentIndex])
  
  currentIndex += 1
}
```

- 해당 코드의 경우 array를 모두 확인하는데 문제가 있을뿐 더러, 예외 처리도 필요하다.
- 이처럼 비동기 코드는 두가지 이슈가 있다.
  1. 작업들의 실행 순서
  2. 변경할 수 있는 공유 데이터
- RxSwift는 이를 적절하게 고려했다.

### 비동기 프로그래밍 용어

- 비동기, 반응형, 함수형 프로그래밍은 몇가지 기본 용어들을 공유하고 있다.

1. State, and specificlally, shared mutable state

이해하기 쉽지 않지만 다음 예제를 확인해보자. 컴퓨터를 처음 키고 실행한다면 아무 문제가 없다 하지만 몇일 혹은 몇주가 지나고 비정상적인 동작을 하게 되는데, 이때 컴퓨터를 리부팅 한다면 처음과 동일하다. 이 상황에서 하드웨어 (코드)는 동일하지만 컴퓨터의 메모리등 여러가지 리소스가 문제를 야기 시키는 것이다. 이때의 리소스들을 State라고 한다. 특히 여러 가지의 비동기 컴포넌트를 가지고 있는 앱의 State를 관리하는 방법을 배울 것이다.

2. Imperative programming ( 명령형 프로그래밍 )

프로그램의 상태를 변경하기 위해서 상태를 사용하는 프로그래밍 패러다임이다. 명령형 코드를 사용하여 앱이 정확하게 언제 그리고 어떻게 동작할지를 알려주는 방식이다. CPU가 이해하는 방식이지만, 이때의 이슈는 사람이 비동기적인 앱의 복잡한 코드를 따라가지 못한다는 점이다. 특히 공유된 상태가 포함되어 있을때 더더욱 그렇다.

3. Side effects

Side effects란 현재 범위를 벗어난 상태에서의 변화를 말한다. 예를 들어서 connectUI() 가 있는데, 이 함수가 만약 UI 에 대해서 eventhandling 을 한다고 했을 때, 실행 전과 실행 후가 다를 수 있기 때문에 이런 상황을 부작용 이라고 한다. 데이터를 수정하거나 화면에 있는 텍스트를 업데이트 할 때마다 부작용이 발생한다. 부작용은 그 자체로 나쁜 것이 아니다. 사실 프로그램 자체의 목적이 부작용을 발생시키는 것이고 이를 제어한 방식대로 진행되도록 하는 것이 중요한 것이다. 

4. Declarative code ( 선언적 코드 )

이는 동작을 정의할 수 있다. Rx 는 관련된 이벤트가 발생할 때마다 동작을 정의할 수 있고, 이로 인해서 불변의 격리된 데이터를 제공받을 수 있다. 비동기로 이를 작업할 수 있지만 조금 더 편하게 사용할 수 있다.

5. Reactive Systems 

- Responsive
  - UI를 항상 업데이트 하고, 앱의 최신 상태를 보여주는 것
- Resilient ( 회복력있는 )
  - 각각의 행동은 격리되어 정의되며, 오류 회복에 대한 유연성을 제공한다.
- Elastic ( 탄력적인 )
  - 코드는 다양한 작업량을 처리하며, 이벤트 스로틀링 , 자원 공유등 다양한 기능을 구현한다.
- Message driven
  - 요소들은 향상된 재사용성과 격리성, 라이프사이클과 클래스 수행의 분리를 위해서 메세지 기반의 대화를 한다.

### Foundation of RxSwift

- 변동 가능한 상태를 해결하고 이벤트 시퀀스를 구성할 수 있으며 코드 분리, 재사용성 그리고 디커플링과 같은 아키텍쳐 개념들을 개선할 수 있다.
- RxSwift 는 명령형 cocoa 코드와 순수한 함수형 코드 사이의 sweet spot을 찾았다. 비동기 코드들을 작성하기 위한 불변하는 코드 정의들을 사용함으로서 이벤트에 결정적이고 구성적으로 반응할 수 있다.

- 해당 책을 공부한다면 RxSwift의 기본 틀을 잡을 수 있을 것이다.
- Rx code를 작성하는데 필요한 세가지 요소는 **Observable, Operators, schedulers** 이다. 

#### Observable

Observable<T> 클래스는 Rxcode 의 기본을 제공한다. 이 기본이란 제네릭 데이터의 불변적인 스냅샷을 가지고 있는 이벤트의 시퀀스를 비동기적으로 제공하는 것이다. 단순하게 말해서, 다른 객체가 이벤트, 값을 계속해서 구독한다는 것이다.

또한 클래스는 하나 혹은 여러개의 observers에 반응할 수 있도록 허용할 수 있다. 

ObserbableType 이라는 Observerble 클래스가 채택한 프로토콜은 상당히 간단하다. Observerble은 세가지 타입의 이벤트만 발생시킬 수 있다.

1. A next event
   - 최신의 데이터 값을 전달하는 이벤트이다. 이를 통해서 Observer는 값을 받을 수 있다. 종료 이벤트가 발생할 때 까지 무한정으로 해당 이벤트를 발생시킬 수 있다.
2. A completed event
   - 성공적으로 이벤트 시퀀스를 종료하는 이벤트이다. 이는 Observable이 라이프 사이클을 성공적으로 완료하고 더이상 추가 이벤트를 발생시키지 않는다는 것을 의미한다.
3. An error event
   - Observable이 에러와 함께 종료되고 더이상 추가적인 이벤트를 발생시키지 않는다.

비동기 이벤트의 방출에 대해서 얘기를 할때 이를 시각화 할 수 있다. (RxMarbles app을 살펴보자)

Observable 이 3가지 이벤트를 발생시킨다는 단순한 계약이 Rx의 모든 것이다. 왜냐하면 이는 너무 보편적이며, 가장 복잡한 로직의 앱 또한 개발할 수 있기 때문이다. 이런 Observable 계약은 Observable 혹은 observer에 대해서 어떤 가정도 하지 않기 때문에, 이벤트 시퀀스를 이용하는 것은 궁극적인 분리 방법이다. 클래스들 끼리 대화를 주고 받기 위해서 델리게이트 패턴이나 클로저 삽입등을 할 필요가 전혀 없다. 현실 상황에 적용하기 위해서 두가지 타입의 observable이 있다.

#### Finite observable sequence

한 마디로 끝이 있는 observable 이다. 에러로 끝이 나던, 성공으로 끝이 나던 끝이 있는 observable를 말한다. 다운로드를 예로 들어보자 subscribe를 통해서 onnext 에는 데이터를 붙이는 코드, 그리고 중간에 에러가 발생할 여지가 있으며 결국에는 completed 로 언젠간 끝나게 될 것이다.

#### Infinite observable sequence

보통 UI 이벤트와 같이 끝이 없는 경우를 말한다. device 의 방향을 돌린다고 생각해보자 (landscape or portrait ) 이는 언제든지 일어날 수 있으며 어떤 기간이 있는 것이 아니다. 유저가 기계를 돌리지 않을 수 있지만 이는 이벤트가 종료 되었다는 것이 아니다. 단지 이벤트가 발생하지 않았을 뿐이다. 이 경우 enError 혹은 onCompleted 가 발생하지 않기 때문에 스킵하자.

### Operators

위와 같은 여러가지 조각들 (여러가지 비동기를 위한 메서드들 포함한 Observable 의 인스턴스)은 복잡한 로직을 구현하기 위해서 하나로 합쳐질 수 있다. 왜냐하면 이들은 강하게 분리되고 구성되어 있기 때문인데, 이런 메서드들을 **operators** 라고 부른다.

이러한 Operator들은 비동기 입력을 받고 사이드 이펙트가 없는 결과물만 한다. 이러한 operators는 굉장히 구성적이며, 이는 항상 그들만의 입력과 출력을 갖고 있고, 원하는 결과물을 위해서 적절하게 체이닝 할 수 있다는 것을 의미한다.

### Schedulers

Schedulers 는 Rx의 디스패치큐에 해당한다. RxSwift에는 사전에 정의된 다수의 스케쥴러가 있고, 사용하려는 대부분의 상황에 대응할 수 있어서 따로 만들지 않아도 괜찮을 것이다. RxSwift 덕분에 다양한 스케줄러에서 동일한 구독의 다른 작업을 스케줄링 할 수 있다. RxSwift 는 구독들과 스케쥴러 사이에 dispatcher 처럼 행동한다, 정확한 컨텍스트에 일련의 작업들을 보내고 끊임없이 서로의 결과물을 보면서 일을 할 수 있도록 한다.



### App architecture

- RxSwift 가 앱의 아키텍쳐를 바꾸지 않는다. 단순히 이벤트를 다루거나, 비동기 데이터들을 다루는 것이다.
- 어떤 아키텍쳐이던지 RxSwift는 단방향의 데이터 플로우 아키텍쳐를 구현하는데 굉장히 유용할 것이다.
- 반응형 앱을 만들기 위해서 프로젝트를 처음부터 시작할 필요는 없다. 새로운 피쳐를 만들거나 반복적으로 리펙토링을 해도 가능하다.
- MS사의 MVVM은 데이터를 바인딩하는 플랫폼에서 event-driven 형식의 소프트웨어를 만들기 위해서 만들어졌다. 따라서 RxSwift 와 MVVM은 아주 나이스하게 결합이 될 것이다. 
  - 그 이유는 ViewModel이 Observable 속성을 드러내도록 허용하고, 이를 ViewController의 연결 코드에서 UIKit과 직접적으로 바인딩을 할 수 있기 때문이다. 
  - 한마디로 UI와 데이터 바인딩이 코드에서 표현하기 쉽게 해놓았다. 아래 그림 참고
![img19](/Users/KimHyeontae/Documents/dev/GITHUB/RxSwift-Record/RxSwift-by-Martin-Todorov/imgs/img19.png)

### RxCocoa

- RxSwift 는 공통적이고 플팻폼에 종속적이지 않으며, Rx의 구현체이다. 따라서 Cocoa나 UI에 전반적인 내용을 모른다. 
  - 이를 위해서 나온것이 RxCocoa이다. RxSwift의 동료 라이브러리이며, UIKit과 Cocoa를 활용한 개발을 위해 모든 클래스를 담고 있다.
  - 고급 클래스들과 더불어서 많은 UI component에 반응형 확장을 추가했으며 많은 UI Component에서 즉시 구독할 수 있다.







