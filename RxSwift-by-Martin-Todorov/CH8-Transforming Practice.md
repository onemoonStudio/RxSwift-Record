# Transforming Operators in Practice

#### Github API 를 사용하여 특정 repo 의 issue 들을 확인해보자

1. github json api 를 활용하여 json을 받고 이를 변환하는 것
2. 정보들을 디스크에 저장하여 fresh 하기 전에도 데이터를 표현하는 것

### Using Map to build a request

- string 으로 Observable을 만들어서 작업 한다. 
  - 유연한 타입을 통해서 여러 곳에서 사용할 수 있도록 할 수 있기 때문이다.

### Using flatMap to wait for web response

- flatmap을 활용함으로서 다음과 같은 이점을 얻을 수 있다.
  1. elements를 바로 방출하고 종료할 수 있다.
  2. 비동기 작업을 실행하며, 효과적으로 observable이 종료 하기를 기다릴 수 있다. 이후에 나머지 chain이 실행된다.



#### URLSession.rx.response

- .next 를 단 한번만 보내고 completes 되기 때문에 동일한 url로 다시 한번 구독을 하게 된다면 두번 요청을 보내게 된다.
- 이때 사용할 수 있는 것이 buffer에 저장할 수 있도록 하는 share(replay:scopre:) 메소드 이다.
  - scope에는 .forever과 .whileConnected 가 있다.
  - .forever 는 buffer를 계속 유지한다.
  - .whileConnected 는 구독자가 있는 경우 계속 살아있다. 구독이 끊기는 경우에 버퍼를 지운다.