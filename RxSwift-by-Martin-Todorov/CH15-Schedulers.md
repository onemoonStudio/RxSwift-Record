# Schedulaers

- Scheduler 는 thread 가 아니라는 사실
  - single scheduler 에 multi thread 를 사용할 수도 있고
  - 여러개의 scheduler가 하나의 thread 를 사용할 수 도 있다.
  - 즉 one-to-one relationship이 아니라는 사실



### Switching schedulers

- rx에서 중요한 기능중 하나는 언제든지 scheduler 를 변경할 수 있다는 점이다.
- rx에서는 스케줄러를 자유롭게 전환할 수 있지만, 기본 코드의 논리를 위반할 수 있다는 사실을 알아둬야 한다. ( 한마디로 thread safe 하지 않다는 점 )

### subscribeOn

- scheduler 를 바꾸는 코드이며 이름을 헷갈리지 말자
  - subscribeOn(SomeScheduler) 의 뜻은 SomeScheduler에 있는 operator를 구독하겠다는 뜻이다. ( 즉 작업은 SomeScheduler에서 이루어진다. )
- 계산하는 코드가 실행되어야 할 때, 즉 계산된 값을 통해서 event 를 emitting 하고자 할때 사용될 수 있다.
- 만약 subscribeOn 코드가 없다면 현재 실행되는 thread 에서 작업을 진행 할 것이다.

```swift
let globalScheduler = ConcurrentDispatchQueueScheduler(queue: DispatchQueue.global())

fruit	
	.subscribeOm(globalScheduler)	
	. // do something ...
```



### observeOn

- observer에서 너의 코드를 동작시키고 싶은 스케쥴러를 변경할 때 사용하는 것이 observeOn이다.
- 즉 어디서 관찰을 할 지 선택한다. 따라서 UI update 가 필요한 경우에는 MainScheduler.instance 를 선택하면 된다.



### Pitfalls



- hot vs cold observable

