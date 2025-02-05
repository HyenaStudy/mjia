# 1. Java 버전 변화 개요

사실 교재가 다루는 범위는 자바 8부터 11까지지만, 가장 큰 변화는 8에서 일어났고 LTS는 8과 11, 이렇게 2개로 요약할 수 있어서 웬만한 내용은 8에 집중되고 11을 곁들이는 식으로 설명할 예정.

## 1) 병렬성의 극대화

```java
// Before Java 8

int numTasks = 5; // 작업 개수(1 ~ 5)
ExecutorService executor = Executors.newFixedThreadPool(5); // 스레드풀 생성
Future<Integer>[] futures = new Future[numTasks]; // 결과 담기 위한 Future 배열

// 작업 분배 및 실행
for (int i = 1; i <= numTasks; i++) {
    final int taskId = i;
    futures[i - 1] = executor.submit(() -> {
        System.out.println(
                taskId + " 번 작업이 " + Thread.currentThread().getName() + " 스레드에 의해 시작됨");

        return taskId * taskId; // 연산
    });
}

// 결과 출력
for (int i = 0; i < numTasks; i++) {
    System.out.println(futures[i].get()); // 결과 출력
}

executor.shutdown(); // Executor 종료
```
```java
// After Java 8

List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5); // 처리할 데이터

List<Integer> results = numbers.parallelStream() // 병렬 스트림 사용
        .map(n -> {
            System.out.println(
                    n + " 번 작업이 " + Thread.currentThread().getName() + " 스레드에 의해 시작됨");

            return n * n; // 연산
        })
        .toList(); // 수집

// 결과 출력(메소드 참조)
results.forEach(System.out::println);
```

- 기존의 자바(자바 8 이전)은 기능, 성능의 병렬 처리를 멀티스레딩 수작업을 통해 수행
- 데이터 분할, 스레드 생성 및 작업 분배의 책임이 개발자에게만 위임
- 코드 레벨에서의 멀티스레드를 활용한 병렬 처리의 극대화가 필요
- 기존의 코드 레벨 병렬 처리의 책임을 개발자로부터 포크조인 프레임워크(parellelstream)와 스트림(stream)에게 넘김

<img width="80%" alt="스크린샷 2025-01-30 오후 11 57 40" src="https://github.com/user-attachments/assets/bc5f486a-c758-46a9-857c-b7e7a69d995b" />


## 2) 함수형 프로그래밍의 가능성

스트림 API의 도입으로 선언형 코드 작성이 가능해지면서 자바에서도 함수형 프로그래밍 작성이 가능해졌다.

비 함수형 프로그래밍에서는 '무엇을' + '어떻게' 전부 개발자가 책임을 지게 되지만, 함수형 프로그래밍에서는 '무엇을' 까지만 개발자가 책임을, '어떻게'는 라이브러리(스트림 api)에게 책임을 위임함으로써 정말 말 그대로 무엇을 처리할 지에 대해서만 **선언**을 하고 그 내부 동작은 라이브러리에게 위임하는 것이다.

### (1) 불변성, 고차함수

```js
// React JS

function App() {

  // 불변성 : count 변수는 변하지 않음
  const [count, setCount] = useState(0)

  // 함수 정의
  const increase = (value) => value + 1

  // 함수를 인자로 전달받는 고차함수
  const onClick = () => setCount(increase(count))
  
  // 결과
  return <button onClick={onClick}>{count}</button>
}
```
```java
// After Java 8

List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

int sum = numbers
        .stream() // 불변성 : numbers 변수는 변하지 않고 스트림으로 변환
        .filter(number -> number % 2 == 0) // '짝수 검증 함수'를 고차함수의 인자로 전달
        .mapToInt(number -> number * 2) // '두 배 증가 함수'를 고차함수의 인자로 전달
        .sum();

System.out.println(sum); // 결과
```

함수 인자 전달은 함수형 프로그래밍의 수많은 특징 중 하나일 뿐이지만 비중을 가장 많이 차지하기 떄문에 대표적인 예시로 가져왔음

### (2) 람다

```java
// 기존 방식 (익명 클래스 사용)
Runnable oldRunnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("익명 구현체 출력");
    }
};

// 람다식 사용
Runnable lambdaRunnable = () -> System.out.println("람다식 출력");

oldRunnable.run();
lambdaRunnable.run();
```

고차함수의 핵심이자 다형성 실현 수단

>1. 데이터와 데이터 처리 공간이 있다.
>2. 데이터 처리 수단을 마련해야 한다.
>3. 외부에서 함수를 제공한다.
>4. 데이터를 처리한다.
>5. 데이터를 다르게 처리하고 싶어졌다.
>6. 함수만 바꾸면 된다.

### (3) 모듈
- 단일 책임 원칙 강화
- 단일 코드 레벨의 함수형 프로그래밍 실현 : 스트림 API
- 범 코드적으로 필터 역할 함수형 인터페이스와 매핑, 리듀싱 함수형 인터페이스들 조립해서 함수형 프로그래밍 실현 : 모듈
- 다만 기업에서 잘 쓰이지 않는다고...

### (4) 디폴트 메소드

```java
interface Example {
    void abstractMethod();

    default void defaultMethod() {
        System.out.println("디폴트라 별도 구현이 불필요");
    }
}
```

- 인터페이스의 재사용성 강화, 메소드 체이닝 활용성 향상

# 2. 어떻게 활용할 것인가?

자바 8에서 도입된 **병렬 처리 및 함수형 프로그래밍 개념**은 이후 비동기 처리 및 리액티브 프로그래밍으로 확장 가능\  
**스프링 네티(Netty) 엔진 기반의 비동기 처리 서버**를 공부하면서 **리액티브 프로그래밍을 활용하는 방식**을 익힐 예정

## 1) Netty 기반의 비동기 서버 개발
자바 8 이후, 비동기 프로그래밍을 지원하는 대표적인 기술 중 하나가 **Netty 기반의 비동기 이벤트 드리븐 서버**이다.\
이러한 서버는 **CompletableFuture**나 **Reactive Streams** 같은 개념과 결합하여 효율적인 병렬 처리를 수행할 수 있다.  


## 2) 리액티브 프로그래밍과의 연계
자바 8 이후, **Reactive Streams**의 개념이 발전하면서 **Spring WebFlux** 같은 리액티브 웹 프레임워크가 등장했다.\  
이러한 환경에서는 **Stream API, CompletableFuture, 함수형 인터페이스**가 중요한 역할을 하며, 선언형 프로그래밍 방식으로 데이터를 처리할 수 있다.  

```java
// Spring Netty + Spring WebFlux

@RestController
public class ReactiveController {

    @GetMapping("/test")
    public Mono<String> test() {
        return Mono.fromSupplier(() -> fetch())
                   .map(data -> "처리 결과 : " + data);
    }

    private String fetch() {
        return "데이터 처리";
    }
}
```

- **Netty 기반의 비동기 이벤트 처리 모델**  
- **리액티브 프로그래밍(WebFlux, Reactor)과 함수형 프로그래밍**  
- **병렬 처리(CompletableFuture, parallelStream) 활용**  
