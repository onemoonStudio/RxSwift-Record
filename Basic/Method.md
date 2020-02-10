### orEmpty

- ControlProperty<String?> 을 ControlProperty<String>으로 unwrapping 해주는 역할을 갖고 있다.



### debounce vs throwttle

https://eunjin3786.tistory.com/80

- 예를 들어서 시간을 3초로 설정한다고 하자
- 버튼을 3번 탭한다. 
  - 0초(a) 1초(b) 2초(c)
  - throwttle은 3초에 2초(c) 이벤트가 들어와서 실행이 된다.
  - debounce는 2초부터 다시 3초를 카운트해서 5초에 2초(c) 이벤트가 실행이 된다.

### driver



