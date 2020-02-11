# CH7 Transforming Operators

- 가장 중요한 파트 중 하나인 Transforming Operators 에 대해서 알아본다.
- 구독자를 위해서 데이터를 준비해가는 과정이다.
- 해당 챕터가 끝나면 모든 데이터 들을 변환할 수 있을 것이다.

### Transforming elements

- Observable 에서는 보통 element들이 독립적으로 오게 되어 있고, 우리는 array 등으로 작업하는 것이 편하다. 나중에 ui와 관련된 class 에서 작업하는 것은 따로 배우겠지만. 만약 array 를 쓰고 싶다면 단순히 toArray 를 사용하자
- .completed 가 호출이 되는 순간 받았던 element를 Array 로 감싸서 .next로 다시 보낸다.

```swift
// 1
  Observable.of("A", "B", "C")
    // 2
    .toArray()
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)

// [ A, B, C ]
```

- map Operator 는 Swift 에서의 고차함수 map 과 동일하다. Subscribe 할때 적용시켜주면된다.
  - 또한 enumarated 를 통해서 Index를 가져올 수도 있다.

``` swift
Observable.of(1, 2, 3, 4, 5, 6)
.enumerated()
// with (index, value)
.map { index, integer in // mapping
      index > 2 ? integer * 2 : integer
     }
.subscribe(onNext: {
  print($0)
})
```



### Transforming inner observables

- observable 과 해당 값에 접근할 수 있는 flatmap famiy 에 대해서 알아보자
- flatmap : 여러 개의 sequence 를 하나로 합쳐서 array 로 만든다. 
  - 이때 하나의 sequence (Observable)에서 값이 변경 된다면 해당 값 또한 array 에 포함 된다.
- flatmap은 observable이 갖고 있는 특정 value를 다른 observable로 변환하기 위해서 사용된다.
- flatMapLatest : 1, 2, 3 sequences 가 있다고 하자 이때 flatMapLastest 를 하게 되면 마지막 3 번째 sequence만 추적을 한다. 즉 새로운 sequence가 들어오게 되면 그 전에 추적하던 sequence 는 무시한다.

### Map vs FlatMap

- Map 은 Observable<R> 을 Observable<T>로 변환시키는 것이다.
  - value 자체에 연산을 한다.
- FlatMap은 Observable에 있는 value를 통해서 다른 Observable로 만드는 것이다.
  - value 를 통해서 값을 가져온 뒤 연산을 한다.

### Observing events

- Observable을 Observable event로 변경 시키고 싶은 순간이 있을 수 있다.
- meterialize operator를 사용하면 값들을 event의 흐름으로 감쌀수 있다.
- 그리고 이렇게 감싸 놓은 events를 demeterialize 를 통해서 해제할 수 있다.

