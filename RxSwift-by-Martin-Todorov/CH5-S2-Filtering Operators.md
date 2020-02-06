#  Section 2: Operators & Best Practice

- Operators는 변형할수 있고 이벤트에 반응할 수 있도록 하는 Rx의 구현체이다. 
- 덧셈, 뺄셈들을 합치는 것 처럼 Rx에서의 simple operators 또한 체인 형식으로 연결할 수 있다.
- 해당 챕터에서는 다음과 같은 것을 배울 예정이다
  1. 특정 이벤트만 받고 나머지를 무시하는 filtering 을 먼저 배울 것이다.
  2. 복합한 데이터 변형을 만들거나 표현하는 transforming operators에 대해서도 배울 것이다.
  3. 대부분의 다른 연산자를 강력하게 구성할 수 있는 combining operators 에 대해서 배울 것이다.
  4. Explore operators는 시간을 기본으로 하는 작업이다.

##### 각 Opertators의 in practice는 하나씩 해보자

# CH5 Filtering Operators

- .next events 에서 원하는 elements 만 받을 수 있도록 filter 기능을 사용할 수 있다. swift에서의 filter 기능과 굉장히 유사하다.

### Ignoring operators

- .next event elements를 무시한다.
- 그리고 .stop , .completed 가 들어오면 동일하게 종료된다.
- 따라서 해당 operator는 해당 observable이 .completed 혹은 .error 로 인해서 언제 종료 되는지 관심이 있는 경우에 유용하다.
  - 만약 subscribe closure에서 print를 한다고 하자, onNext("value")를 했을 때는 이벤트를 무시할 뿐 더러 scribe 또한 하지 않는다. 
  - 이런 상황에서 subscribe closure를 실행시키고 싶다면 .completed 혹은 .error 를 넘겨야 한다.

#### elementAt

- n 번째에 방출된 event의 element를 얻고 싶은 경우 사용할 수 있다. 예시를 보자면 다음과 같다.

``` swift
Observable
    .elementAt(2)
    .subscribe(onNext: { _ in
      print("You're out!")
    })
    .disposed(by: disposeBag)
```

- 만약 위에서 Observable 에 3개의 .next 이벤트가 발생한 경우 2번째 이벤트만 받고 나머지는 무시하게 될 것이다.
- 흥미로운 점은 필요한 인덱스의 이벤트를 받는 순간 subscription이 종료 된다는 점이다.

#### 상단의 두개의 filting elements로 부족하다면 filter 를 사용하자

``` swift
Observable
    .filter { $0 % 2 == 0 } 
// value 가 짝수인 것만 .next 를 넘긴다.
```

### Skipping Operators

- skip operator 를 통해서 특정 인덱스까지 무시할 수 있다.
- 예를 들어서 **skip(4)** 라고 한다면 들어오는 4개의 이벤트를 무시한 뒤 그 이후 이벤트들을 넘긴다.
- 비슷하게 skipWhile operator 가 있는데, 해당 operator는 조건이 true가 되기 전까지 무시하고 **true가 되는 순간 부터 필터링 없이 이벤트를 넘긴다.**
- Dynamic 한 filter 를 생각해보자. 2개의 Observable이 있고, skipUntil 을 사용한다면 trigger Observable이 방출되고 나서 Base Observable에 이벤트를 전달할 수 있다.
  - 

```swift
  let subject = PublishSubject<String>()
  let trigger = PublishSubject<String>()
  subject
    .skipUntil(trigger) // trigger 에서 이벤트를 받을 때 까지 기다린다.
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
// trigger가 .next 를 받기 전 까지는 subject 의 .next event 는 무시된다.
```

#### Taking operators

- 위에서 설명한 skipping과 반대라고 생각하자.
- take(2) 라고 한다면 2번째 event까지 넘긴다.
- 마찬가지로 skipwhile의 반대 takeWhile 이 있다. 

####  enumerated

- 반복문과 동일하게 해당 인덱스가 필요한 경우가 있을수도 있다. 이때 enumarated() operator를 사용하면 2개의 인자를 받는 closure를 사용할 수 있다.

#### Distiunct operators

- distinctUntilChanged operator를 사용하면 연속하게 반복된 값이 들어온 경우 이후에 들어온 이벤트를 무시할 수 있다.
  - ex ) 1 , 2 , 3, 3 , 4 가 들어오고 subscribe에 값을 프린트 한다고 하면 1, 2, 3, 4 가 프린트 될 것이다.
- 이때 closure를 직접 설정해서 원하는 필터링 값을 설정할 수 있다. ( 굳이 값이 같지 않아도 첫번째 글자만 똑같으면 거른다던지 등 .... )

















