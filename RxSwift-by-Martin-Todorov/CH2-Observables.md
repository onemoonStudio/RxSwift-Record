# Observables

### What is an observable?

- Observables 는 Rx 의 중심이다. Observable sequence , Observable , sequeunce 등의 말들이 많이 쓰이는데 이는 모두 동일한 말이다. 혹은 stream 이라는 말을 들을 수 있지만 모두 sequence 라는 것을 기억하자.

![img23](/Users/KimHyeontae/Documents/dev/GITHUB/RxSwift-Record/RxSwift-by-Martin-Todorov/imgs/img23.png)

- Observable은 단지 sequence 이며 특별한 힘을 가졌다. 사실 가장 강력한 힘중 하나는 비동기적이라는 것이다. Observable은 이벤트들을 생산하고, 이 과정을 emitting 이라고 한다. 이벤트는 값을 가질 수도 있고 탭과 같은 제스쳐를 가질 수도 있다. 
- 타임라인을 통해서 이 개념들을 설명할 수 있다. 타임라인 위에 시퀀스의 값들이 표현되어 있으며 왼쪽에서 오른쪽으로 가는 것은 시간을 의미한다. 원안에 값이 있는것이 표현된 것은 방출 (emitted)되었다는 것을 의미한다. 

### Lifecycle of an observable

- observable은 next event일때 원소를 방출한다. 
- marble diagram에서 vertical bar가 있다면 이는 observable이 더 이상 쓰이지 않는 다는 것을 의미하며completed event이다. 혹은 termianated 라고 표현한다.
- 만약 에러가 발생한 경우에는 빨간색으로 X가 표시되며 error event 라고 할 수 있다. 에러가 발생하면 더이상 이벤트를 발생시키지 않고 종료된다.
- Event는 열거형으로 표현되며, Element를 담고 있는 .next, Error를 담고 있는 .error, 그리고 종료를 나타내는 completed 로 표현된다.

### Creating observables

- Observable<T>.just
  - observable의 type method 이다.
  - 하나의 요소만 가지고 있는 observable sequence 를 만든다.
- Observable.of
  - 타입을 명시하지 않아도 되며 여러개의 요소들을 받을 수 있다. 
  - of는 variadic (임의 개수의 인자를 지원하는 ) prameter 를 취한다.
    - 따라서 observable array 를 만들고 싶다면 Observable.of([one,two]) 를 사용하면 되고 이는 Observable.of( one, two ) 와 다르다는 것을 기억하자
    - 이때 observable array는 하나의 element를 가지고 있다고 한다.
- Observable.from
  - array를 individual element 로 풀어서 Observable 을 만든다. 따라서 from element 는 array 만 받는다.

### Subscribing to observables

- RxSwift 의 Observers는 우리에게 익숙한 observer 에게 broadcast 형식으로 알려주는 notification center와 다르다. 

- 사실 동작 자체는 비슷하다. 하지만 보통 .default instance 만 쓰는 notification center와 다르게 observable 각각이 다른 인스턴스이다.

- 또한 중요한점은 observable은 subscriber 가 없다면 메세지를 보내거나 어떤 액션도 취하지 않는다는 점이다.

- subscribe operator를 확인해보면 escaping closure 가 Element 타입의 Event를 인자로 취하는 것을 볼 수 있고, 아무것도 리턴하지 않는다는 것을 알 수 있다. 그리고 subscribe 는 Disposable 을 리턴한다.

- 이때 단순히 subscribe 를 한다면 모든 이벤트를 받을 가능성이 있다. 이때 next event만 element 를 가지고 있기 때문에 event.element에 접근하려 했을 때 element는 optioanl이다. 

  - 이러한 불편함 때문에 subscribe에서는 각 이벤트에 대한 단축어가 있다.

  - ```swift
    observable.subscribe { event in
                          // unwarpping is required because only next event has a element
                          if let element = event.element { 
                            print(element)
                          }
    }
    // short cut of next event
    observable.subscribe(onNext: {element in 
                                  print(element) // not optioanl
                                 })
    ```

- 만약 어떠한 원소도 없는 Observable 을 만들고 싶다면 Observable<Void>.empty()를 사용하자 .completed event만 방출할 것이다.
  - 추론이 불가능 하다면 observable 은 특정한 인자로 정의가 되어야 한다.
  - 즉시 종료되는 observable 이 리턴되어야 하거나 의도적으로 아무런 값이 없는 observable이 필요할 때 유용하게 쓰일 것이다.
- empty 와 반대로 never 라는 operator 가 있다. Observable<Any>.never() 로 사용하며,  이는 어떠한 것도 방출하지 않으며, 종료되지도 않는 observable이다. 
  
  - 이는 무한 기간을 나타내는데 사용할 수 있다.

### Disposing and terminating

- observable 은 구독(subscription)을 받기 전 까지는 아무것도 하지 않는다. 구독은 Observable이 끝날 때 까지 일하게 한다. 또한 직접 구독을 취소하여 Observable을 종료시킬 수 있다.

- 직접적으로 구독을 취소시키기 위해서 dispose()를 호출하자. 이때 observable 은 이벤트를 더이상 발생시키지 않을 것이다.

  - 일일이 구독을 관리하는 것이 귀찮을 수 있다. 그래서 RxSwift에서는 DisposeBag이라는 타입을 포함한다. 
  - disposebag은 disposable을 들고 있다가 disposebag이 deallocated 될때 dispose()를 각각 호출한다.
  - subscribe를 통해서 disposable이 리턴되면 이를 disposebag에 구독시키는 것이 자주 활용되는 패턴일 것이다.

- create operator를 이용하면 Observable이 subscriber에게 방출할 모든 이벤트를 설정할 수 있다.

  - ```swift
    Observable<String>.create { observer in
      observer.onNext("1")
      
      observer.onCompleted()
      
      observer.onNext("?")
      
      return Disposables.create()
    }
    ```

  - 이때 마지막 "?"는 호출이 될 것인가? 
    - 아니다 completed 가 있기 때문에!
    - 또한 .next 와 .onCompleted 사이에 onError 가 있는 경우에는 completed 는 호출되지 않는다. ( 종료되었기 때문에 )

-  만약 dispose도 없고, 끝나지도 않는다면 memory leak을 만드는 것이다. observable은 영원히 끝나지 않을것이며 폐기되지도 않을 것이다.

###### 궁금한점 

1. disposeBag을 통해서 어차피 dispose 가 호출되기 때문에 complete 혹은 error 이벤트는 신경 안써도 되지 않을까? 
   - 맞다. 어차피 disposebag이 dispose 시켜준다.
2. dispose 시키지 않고 complete event를 발생시키는 경우는 메모리 릭이 발생할까? complete 시키면 자동으로 해제를 시킬까?
   - 핵심을 놓쳤다. error, complete event가 발생하면 dispose가 호출된다. (해제가 된다는 말)

### Creating observable factories

- subscriber에 생성되는 observable을 자동으로 구독시킬 수 있는 공장을 만들 수 있다.
  - deferred operator를 사용하자.
  - 이후 외부에서 subscribe를 할 때마다 해당 factory 가 observable을 만들어서 subscriber에게 전달한다.

### Using Traits

- Traits 는 Observable보다 활동이 적은 Observable이다. 사용하는 것은 선택사항이며 observable 대신에 사용할 수도 있다. Traits의 목적은 독자들에게 코드의 의도를 더욱 명확하게 전달하는 것이다.
- RxSwift에서는 3가지의 Traits 가 있다. Single, Maybe, Completable 이다.
  1. Single은 .success(value) 혹은 .error event를 발생시킨다.
     - .success 는 .next(value) 그리고 .completed 이벤트가 합쳐진 결과물이다. 따라서 한번만 사용되는 프로세스에 사용하기에 좋다. (예를 들어 다운로드 혹은 로딩)
  2. Completable은 .completed 혹은 .error event를 발생시킨다.
     - 따라서 단순히 실행이 제대로 되었는지 혹은 실패하였는지를 알고 싶을 때 유용하게 사용될 수 있다.
  3. Maybe의 경우는 Single과 Completable의 조합이다.
     - .success(value), .completed, .error 를 모두 발생시킬 수 있다.
     - 따라서 성공 및 실패의 가능성이 있으며, 성공 또한 값이 있을지 없을 지 불확실한 경우 사용된다.

### Challange

1. do operator

   - observable의 method이며 observable을 그대로 리턴한다. subscribe 전 do operator안의 클로저들이 먼저 실행되며 subscribe에 아무런 영향을 미치지 않는다. 

   - 특히 onSubscribe 등 subscribe에 없는 클로저도 있으니 참고하자

   - ```swift
     someObservable.do(onNext: {
             print($0)
         }).subscribe { event in
             print(event)
         }
     ```

2. debug operator

   - side effects를 발생시키는 것도 rx를 디버깅 하는데 좋지만, debug operator가 더욱 유용할 수 있다는 것을 알아두자 observable 의 모든 이벤트를 알려줄 것이다.

   - observable의 메서드이며 다음과 같이 프린트 된다.

   - ```swift
     020-01-27 00:24:02.196: RxSwiftPlayground.playground:64 (__lldb_expr_3) -> subscribed
     2020-01-27 00:24:02.197: RxSwiftPlayground.playground:64 (__lldb_expr_3) -> Event completed
     Completed
     2020-01-27 00:24:02.197: RxSwiftPlayground.playground:64 (__lldb_expr_3) -> isDisposed
     ```



































