# Beginning RxCocoa



#### ControlProperty

- data 를 bind 하기 위해서 사용한다. 

#### ControlEvent

- UI component의 특정 이벤트를 받기 위해서 사용된다.
- 따라서 만약 component가 UIControl.Events를 사용하면 활용이 가능하다.

#### Driver

- 에러를 발생시킬 수 없으며, 모든 프로세스는 메인 스레드에서 실행된다는 것을 보장한다. 

#### Signal

- Driver와 굉장히 유사하다. 다른 점이라면, 자원을 공유하지 않는다는 것이다. 따라서 subscriber에게 element를 replay 하지 않는다.

##### Driver vs Signl

- Signal은 event를 모델링할 때 유용하고, Driver는 상태에 따른 모델링을 할 때 유용하다.
  - 예제를 조금 더 찾아보자



### Binding observables

