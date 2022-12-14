### 노드

Node.js는 자바스크립트를 이용해서 서버를 만들 수 있는 개발 도구이다.

### 비동기 입출력 방식

하나의 요청 처리가 끝날 때까지 기다리지 않고 다른 요청을 동시에 처리할 수 있는 **비동기 입출력(논블로킹 입출력, Non-Blocking IO) 방식**을
적용했다.

비동기 입출력 방식이 노드의 대표적인 특징이다.

### 콜백 함수

자바스크립트에서는 변수에 함수를 할당할 수 있다. 따라서 변수에 할당된 함수를 다른 함수의 파라미터로 전달할 수 있다.
이렇게 파라미터로 전달된 함수를 다른 함수의 내부에서 호출하는 것이 콜백 함수이다.

### 이벤트 기반 입출력 (Event driven I/O) 모델 

파일 시스템에서 콜백 함수를 호출하는데, 파일 시스템이 이벤트와 함께 호출하는 방식이면 이벤트 기반 입출력 모델이라고 부른다.

### 바인딩(Binding)

자바스크립트에서는 on() 메소드를 사용해 이벤트를 콜백 함수와 바인딩 할 수 있다. 따라서 응답 객체인 res 객체의 on() 메소드를 사용해 data 이벤트와 
콜백 함수를 바인딩하면 data라는 이름의 이벤트를 받았을 때 등록한 콜백 함수가 실행된다.

