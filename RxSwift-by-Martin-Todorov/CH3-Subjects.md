# Subjects

- Subject 는 observable 그리고 observer의 역할을 같이할 수 있다.

- 다양한 타입의 subject를 알아봄과 동시에 subject 를 감싸는 래퍼인 relay 또한 알아볼 것이다.

- subject는 정보를 받은 다음 그 정보를 subscribers 에게 전달한다.

  - 초기화되는 순간부터 문자열을 받을 준비가 되어있다.

  - 초기화 후에 이벤트를 정의하고 단순히 subscribe만 한다고 해서 이벤트를 받고 실행되지 않는다. 그 이유는 PublishSubject는 현재 subscribers 에게만 이벤트를 전달하기 때문이다. 따라서 subscriber 가 존재한 다음에 이벤트를 전달해야 제대로 동작을 할 것이다.

  - ```swift
    let subject = PublishSubject<String>()
    subject.onNext("Is anyone listening?") // nop
    let subscriptionOne = subject
    .subscribe(onNext: {
      print($0)
    })
    subject.on(.next("1")) // yes 1 is printed 
    ```

  - on(.next) 는 subject에 이벤트를 전달하는 방식이며 observable.onNext(_:)와 동일하다. 단축어라고 생각하면 된다. 

### What are subjects?

그렇다면 실제로 왜 쓰는지 알아보자

- 위와 동일하게 시작하자면 Subject는 observable 과 subscriber의 역할을 동시에 할 수 있다.

- 4가지 타입의 subject와 2가지 타입의 relay 가 있다.

  - **Subject**

  1. PublishSubject
     - empty 에서 시작을 해서 새로 들어온 원소들만 전달한다.
  2. BehaviorSubject
     - 초기 값들과 함께 시작을 해서 새로운 구독자에게 재생을 시킨다.
  3. RelaySubject
     - buffer size 와 함께 시작을 해서 해당 사이즈를 유지시킨다. 그리고 이를 새로운 구독자에게 전달한다.
  4. AsyncSubject
     - 구독자가 completed 를 받을 때 sequence에서 마지막 .next 이벤트만 전달한다. 거의 사용되지 않지만 알아두자

  - **Relay**는 PublishRelay 그리고 BehaviorRelay 가 있다. 이것들은 그들의 subjects를 감싸고 .next 이벤트만 받는다. .completed 혹은 .error 는 relay에서 받을 수 없다. 따라서 끝나지 않는 sequence 에서 사용하기 좋다.

### Working with publish subjects

- publish subject는 구독이 취소되거나 subject가 종료되기 전까지, 단순히 구독이 시작된 뒤에 새로운 이벤트를 전달하는데 유용하게 쓰인다. 
- marble diagram에서 upward-pointing은 구독을 나타내며, downward-pointing 은 이벤트의 전달을 나타낸다. 
- 만약 중간에 .completed 혹은 .error로 인해서 subject 가 종료 되었다면 이후에 새로운 subscriber에게 해당 종료 이벤트만 재 방출하게 된다. 따라서 stop 에 관련된 핸들러가 있는 것이 좋은 생각 일 것이다.
- 시간관련해서 민감한 데이터를 다룰 때 publish subject를 다루게 될 수도 있다. 예를 들어서 경기의 시작을 알리는 서비스가 있을 때 32분에 시작된 경기를 35분에 초대할 필요가 없는 것 처럼 말이다.

### Working with behavior subjects

- publish subject와 비슷하다. 다른 점 하나는 마지막 .next event를 새로운 구독자에게 replay한다는 것이다.
  - 이때 만약 .error 혹은 .completed 로 종료가 되었다면 .next 는 아니지만 새로운 subscriber에게 전달이 된다.
- behavior subject는 언제나 최신 값을 구독 당시에 제공한다. 따라서 초기화 할 당시에 초기값이 있어야 하고 만약에 초기값을 설정할 수 없는 상황이라면 그것은 publish subject를 써야 한다는 말이 된다.
- 이전의 값들을 활용하고 새로운 값들을 미리 채우기 위해서 사용할 때 유용할 것이다. 

### Working with replay subjects

- replay subjects는 일시적으로 정해진 사이즈까지 마지막 element들을 캐싱한다. 그리고 나서 새로운 구독자에게 이를 전달한다. 
- 메모리에서 유지시키고 있기 때문에 많은 양의 사이즈를 갖는 것은 메모리에 부담이 갈 수 있다는 것을 알아두자
- 또한 array를 담고 있는 replay subject를 만드는 것에도 조심해야 한다. 위와 마찬가지로 메모리에 부담을 줄 수 있기 때문이다.
- 어떤 subject가 1, 2 element를 버퍼에 담고 있을 때 onError 가 발생했다고 치자, 새로운 구독자는 .next(1), .next(2), .error()를 받게 될 것이다.
  - 이유는 버퍼에서 재방출을 항상 기다리고 있기 때문이다. 1, 2는 버퍼에 담겨져 있기 때문에 error혹은 completed 로 종료가 되더라도 버퍼에 있는 값들과 종료 이벤트가 같이 전달이 된다. 
  - 그렇다면 아예 dispose가 된 경우를 생각해보자. 이 경우에는 약간 다르게 버퍼에 있는 값들이 아닌 dispose 되었다는 에러를 받게 될 것이다. 실제로는 disposebag에 담겨 사용할 일이 없겠지만 dispose가 된다면 버퍼에 있는 element들이 사라진다는 것을 알 수 있다.

### Working with relays

- relay는 replay behavior를 유지하면서 subject를 감싸고 있다.
- 다른 subjects( also observable ) 와는 다르게 accept(_:)를 사용해서 값을 추가한다. 왜냐하면 단순히 값만 받아들이고 .error나 .completed와 같은 종료 이벤트는 받을 수 없기 때문이다.
- PublishRelay 는 PublishSubject를 담고있고, BehaviorRelay는 BehaviorSubjects를 담고 있다. 
  - 단순히 wrapping된 subjects와 다른 점은 종료되지 않는 것이 보장되기 때문이다.
  - 따라서 다시 말하자면 그 점을 제외하고는 subject와 동일하다는 것이다.
- behaviorRelay의 경우에는 값을 항상 가지고 있다는 특징이 있다 (behavior subject 처럼). 따라서 relay.value를 호출하면 최신 값을 얻을 수 있다.















