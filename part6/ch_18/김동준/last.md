# 리액티브 프로그래밍

사실 이번 스터디를 하면서 가장 바랐던 것은 **리액티브 프로그래밍** 구현을 위한 배경 지식을 쌓는 것이었다. 다만 자바란 언어 자체가 함수형과 리액티브 프로그래밍 등에 어울리지는 않다. 철저하게 객체지향적으로 코드가 설계돼고 그 과정에서 엄격한 타입 검증이 이뤄지기 때문에 좋게 말하자면 개발자의 의도를 듬뿍 담는 거고 나쁘게 말하면 일일이 개발자가 전부 챙겨야 하는 점에서, 선언형이 중시되는 함수형 프로그래밍과는 거리가 먼 언어라고 할 수 있다.

그렇기 때문에 자바를 메인으로 삼은 나로서는 단순히 자바의 함수형 패러다임 관련 API를 이해하는 것에 그치지 않고, 스프링 웹플럭스와 같은 추가적인 프레임워크 활용 공부나 코틀린, 파이썬 등까지 나아가야 리액티브 프로그래밍이 무엇인지, 실전적으로 어떻게 활용할 수 있을 지에 대해 논할 수 있을 것이다. 그런 점에서 이번 교재는 아쉬움이 많이 남는다고 할 수 있겠다. 할 게 많네...

## 1. 개념 정리

사실 리액티브 프로그래밍의 용어조차 혼동이 잦을 때가 많다. 비동기 프로그래밍, 함수형 프로그래밍... 비슷하면서도 뭔가 다르지만 뭐가 구체적인 것인지 확실하게 안 잡혀 있어서 일단은 개념 교통정리부터 하고 넘어갈 예정

![image](https://github.com/user-attachments/assets/6b19f546-96ba-48c8-ba52-bca1bff26185)

### 1) 비동기 프로그래밍

![image](https://github.com/user-attachments/assets/1cbd1a34-fdf0-4ebd-9809-060e449783b1)

기존의 동기식 프로그래밍의 가장 큰 문제는 선형적인 시간 흐름 속에서 자원이 낭비되는 케이스가 잦은 것이었다. 즉, 다른 작업이 완료돼서 해당 작업의 응답이 도착할 때까지 자신의 작업은 중단된 상태로 잠드는 것이다. 간단하게 멀티 스레드를 생각했을 떄, 스레드 B가 작업 중이고 해당 작업이 완료된 응답을 대기하기 위해 스레드 A가 잠든 상태를 생각하면 된다(**블로킹**). 다만, 스레드는 잠들어도 여전히 자원을 점유하고 있게 된다. 이 과정에서 잠들지 않고 스레드 B의 작업과 별개로 스레드 A의 작업을 여전히 계속 이어나갈 수 있게 한다면(**논 블로킹**)? 이게 곧 비동기 프로그래밍 기법의 기초적인 시작점이라 볼 수 있다.

#### (1) 간단한 예저

```java
public record Shop(String name) {

    public static void delay() {
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    public double getPrice() {
        delay();
        Random random = new Random();
        return (name.charAt(0) + name.charAt(1)) * random.nextDouble();
    }
}
```

간단한 불변 객체를 만들고 해당 로직에서 일부러 딜레이를 걸어준다. 즉, 기능 수행 과정에서 1초(조금 넘는?) 시간이 소요되도록 처리해뒀다. 
그리고 아래와 같이, 데이터들의 일반적인 순차 스트림 동기 처리, 병렬 스트림 처리, 스트림 비동기 처리 로직을 짜고 벤치마킹해본다.

```java
@State(Scope.Benchmark) // 같은 벤치마크끼리 객체 공유(멀티스레드 측정용)
@OutputTimeUnit(TimeUnit.MICROSECONDS) // 벤치마킹 결과 단위 설정
@BenchmarkMode(Mode.All) // JMH의 벤치마크 실행 범위 지정
public class AsyncTest {

    private static final List<Shop> shops = Arrays.asList(
            // 100개가 넘는 Shop 인스턴스들
    );


    @Benchmark
    public List<String> findPrices() {
        return shops.stream()
                .map(s -> String.format("%s price is %.2f",
                        s.name(), s.getPrice()))
                .toList();
    }

    @Benchmark
    public List<String> findPricesByParallelStream() {
        return shops.parallelStream()
                .map(s -> String.format("%s price is %.2f",
                        s.name(), s.getPrice()))
                .toList();
    }

    @Benchmark
    public List<String> findPricesByAsync() {
        List<CompletableFuture<String>> priceFutures =
                shops.stream()
                        .map(s -> CompletableFuture.supplyAsync(
                                () -> s.name() + " price is " + s.getPrice()
                        )).toList();

        return priceFutures.stream().map(CompletableFuture::join).toList();
    }
}
```
<img width="80%" alt="스크린샷 2025-03-18 오후 2 31 35" src="https://github.com/user-attachments/assets/e4070b8e-6e09-4495-bd91-3bfec5162595" />

| 테스트 명칭                        | 측정 모드 | 실행 시간 (us/op) | 비고 |
|---------------------------------|---------|------------------|------|
| **AsyncTest.findPrices**         | `thrpt` | ≈ 10⁻⁸ ops/us   | 처리량 매우 낮음 (순차 실행) |
| **AsyncTest.findPricesByAsync**  | `thrpt` | ≈ 10⁻⁷ ops/us   | 비동기 방식, 10배 향상 |
| **AsyncTest.findPricesByParallelStream** | `thrpt` | ≈ 10⁻⁷ ops/us   | 병렬 스트림, 10배 향상 |
| **AsyncTest.findPrices**         | `avgt`  | 104,459,682.67  | 약 104초, 가장 느림 |
| **AsyncTest.findPricesByAsync**  | `avgt`  | 15,056,668.96   | 약 15초, 7배 이상 향상 |
| **AsyncTest.findPricesByParallelStream** | `avgt`  | 14,056,501.83   | 약 14초, 가장 빠름 |

많은 결과들이 있지만 **처리량(thrpt)** 과 **평균 소요 시간(avgt)** 기준으로 비교하자면, 순차 스트림 방식의 처리량이 매우 낮고 소요 시간이 매우 오래 걸린 것을 파악할 수 있다. 이것을 동일 데이터 기준에서 비동기 처리(병렬 스트림 역시 비동기 처리이므로)로 개선했을 때, 처리량이 약 10배 이상 향상됐고 소요 시간은 거의 7배 가량 단축된 것을 확인할 수 있다.

#### (2) 콜백 지옥

다만 비동기 프로그래밍을 위해 코드를 작성할 때, 흔하게 마주할 수 있는 가독성 나쁜 문제가 있는데 바로 **콜백 지옥**이다.

```java
// Java
public static void main(String[] args) {
    task1(result1 -> {
        System.out.println("Task 1 완료: " + result1);
        task2(result2 -> {
            System.out.println("Task 2 완료: " + result2);
            task3(result3 -> {
                System.out.println("Task 3 완료: " + result3);
                task4(result4 -> {
                    System.out.println("Task 4 완료: " + result4);
                });
            });
        });
    });
}
```
```js
// JS
task1(() => {
  task2(() => {
    task3(() => {
      console.log('All tasks completed');
    });
  });
});
```
```py
# Python
def start():
    task1(lambda: task2(lambda: task3(lambda: task4(lambda: print("모든 작업 완료")))))

```

이 콜백 지옥이 발생하는 이유는 콜백 함수를 인자로 전달하는 과정이 중첩되면서 발생하게 된다. 콜백 함수는 파라미터로 전달되어 특정 시점에 호출되는 함수를 뜻하는데, 보통 콜백 함수의 기능 작성을 파라미터 위치에서 수행(자바의 경우에는 익명 클래스 혹은 람다식, 자바스크립트는 화살표 익명 함수, 파이썬은 람다 익명 함수)하기 때문에, 작성 과정에서 가독성이 나빠지게 되는 구조가 된다.

이 콜백 지옥을 보완할 수 있는 방법 중 하나가 **함수형 프로그래밍**이다. 

```java
public static void main(String[] args) throws InterruptedException {
    CompletableFuture.supplyAsync(() -> task1())
            .thenApply(result1 -> {
                System.out.println("Task 1 완료: " + result1);
                return task2();
            })
            .thenApply(result2 -> {
                System.out.println("Task 2 완료: " + result2);
                return task3();
            })
            .thenApply(result3 -> {
                System.out.println("Task 3 완료: " + result3);
                return task4();
            })
            .thenAccept(result4 -> System.out.println("Task 4 완료: " + result4));
}
```

위의 예제는 아까 본 자바의 콜백 지옥 예제 코드와 동일한 기능을 수행하지만, 가독성이 어느 정도 보완된 것을 확인할 수 있다. `CompletableFuture`의 메소드 체이닝을 통해 각 작업들을 연결하듯이 작성하면서 각 작업 명시를 명확하게 표현하고 있다. 이것 외에도 여러 리액티브 라이브러리를 통해 간결하면서 가독성 좋게 표현할 수 있다.

후술하겠지만, 리액티브 프로그래밍의 핵심이 바로 이 비동기 프로그래밍을 기반으로 함수형 프로그래밍을 통해 보완하는 방향을 통해 데이터를 반응형으로 처리하는 기법이라고 할 수 있다.

## 2. 간단한 리액티브 애플리케이션 예제

비동기 프로그래밍을 기반으로 함수형 프로그래밍을 통해 보완하는 코드 설계 방향을 파악했으니, 이제 데이터를 **반응형**으로 처리한다는 것에 대해 알아보자.
반응형으로 처리한다는 것은 기존의 명령형 프로그래밍의 순차(위에서부터 한 줄씩)적인 처리를 한다는 것에서 큰 차이가 있는데 아래 그림을 보자

![image](https://github.com/user-attachments/assets/49124801-2617-4fc7-8354-a979f0dbb0ae)

기존 명령형은 위의 라인에서 코드를 그대로 읽어내려오면서 순차적으로 처리되므로 A의 값이 10으로 업데이트되는 것과 기존의 B와 C의 연산은 별개의 독립된 내용이다. 반대로 리액티브에서는 A의 값이 업데이트 되는 것이 **이벤트**처럼 인지되면서 기존의 B, C 연산들이 영향을 받아 반응(**Reactive**)하는 형태가 되기 때문에 연계된 연산들 역시 추가로 업데이트된다. 이벤트가 발생하는 것에 맞춰 추가 연산이 이뤄져야 하기 떄문에 **병렬 처리, 비동기 처리**가 리액티브 프로그래밍에서 핵심을 차지하게 된다. 그와 동시에, B와 C의 입장에서는 A가 1일 때와 A가 10일 때, 연산이 2번 이뤄진 셈이므로 캐싱이나 구독 취소, 조건 등을 통해 자원 관리에 주의를 기울여야 한다.

### 1) 자바 Flow 클래스

넷플릭스 등에서 제시한 RxJava 모듈 등도 있지만, 발행, 구독 관련 기능을 제공하는 자바 기본 **Flow** 클래스를 통해 간단한 리액티브 앱을 만들 수 있다.
해당 API를 구성하는 핵심 인터페이스(`Publisher<T>`, `Subscription`, `Subscriber<T>`)들을 정리하자면 아래와 같다.

![image](https://github.com/user-attachments/assets/66272598-533b-48df-be82-176bd55ecac6)

- `Publisher`는 반드시 `Subscription`의 `request` 메서드에 정의된 개수 이하의 요소만 `Subscriber`에 전달해야 한다.
- `Publisher`는 지정된 개수보다 적은 수의 요소를 `onNext`로 전달할 수 있으며 동작이 성공적으로 끝났으면 `onComplete`를 호출하고 문제가 발생하면 `onError`를 호출해 `Subscription`을 종료할 수 있다.
- `Subscriber`는 요소를 받아 처리할 수 있음을 `Publisher`에 알려야 한다. 이런 방식으로 `Subscriber`는 `Publisher`에 역압력을 행사항 수 있고 `Subscriber`가 관리할 수 없이 너무 많은 요소를 받는 일을 피할 수 있다.
- `onComplete`나 `onError` 신호를 처리하는 상황에서 `Subscriber`는 `Publisher`나 `Subscription`의 어떤 메서드도 호출할 수 없으며, `Subscription`이 최소 되었다고 가정해야 한다.
- `Subscriber`는 `Subscription.request()` 메서드 호출이 없이도 언제든 종료 시그널을 받을 준비가 되어있어야 하며 `Subscription.cancel()`이 호출된 이후에라도 한 개 이상의 `onNext`를 받을 준비가 되어있어야 한다.
- `Publisher`와 `Subscriber`는 정확하게 `Subscription`을 공유해야 하며 각각이 고유한 역할을 수행해야 한다. 그러려면 `onSubscribe`와 `onNext` 메서드에서 `Subscriber`는 `request` 메서드를 동기적으로 호출할 수 있어야 한다.
- 표준에서는` Subscription.cancel()` 메서드는 몇 번을 호출해도 한 번 호출한 것과 같은 효과를 가져야 하며, 여러 번 이 메서드를 호출해도 다른 추가 호출에 영향이 없도록 ThreadSafe 해야 한다고 명시한다. 같은` Subscriber` 객체에 다시 가입하는 것은 권장하지 않지만 이런 상황에서 예외가 발생해야 한다고 명세서가 강제하진 않는다. 이전에 취소된 가입이 영구적으로 적용되었다면 이후의 기능에 영향을 주지 않을 가능성도 있기 때문이다.

>*참조 : https://devbksheen.tistory.com/entry/%EB%AA%A8%EB%8D%98-%EC%9E%90%EB%B0%94-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C-%EC%8A%A4%ED%8A%B8%EB%A6%BC%EA%B3%BC-Flow-API%EB%9E%80*

참고로, 웬만한 자바 인터페이스 모듈들은 기본적인 구현체들을 제공하는데(`List<T>` 인터페이스의 기본 구현체인 `ArrayList<T>`) Flow API는 그런 게 없다. 왜냐면 API를 만들 당시에 이미 RxJava, Akka 등의 다양한 리액티브 스트림의 자바 코드 라이브러리가 존재했기 때문이다. 라이브러리들이 독립적으로 개발돼서 존재하였고, 자바 9의 표준화 과정에서 Flow API의 인터페이스를 기반으로 구현하도록 진화됐기 때문에 기본 구현 클래스가 존재하지 않는다.

### 2) Pub/Sub 모델 구현

리액티브 애플리케이션을 구현할 수 있는 대표적인 패턴이자 모델로 **발행/구독 모델(Publisher/Subscriber 패턴)** 이 있다. 위에서 소개한 자바 Flow 클래스 역시 발행/구독 모델을 구현하기에 최적화된 인터페이스들을 제공한다. 간단하게 채팅 메세지를 발행해서 구독자들에게 전달하는 로직을 짜보자.

비동기 스트림의 대상 데이터인 `Message`, 데이터의 흐름을 제어하는 `Subscription` 구현체, 데이터를 수신해서 처리하는 `Subscriber<Message>` 구현체를 작성한다.

```java
// 메세지 객체(전달자 명칭 파라미터 기반 임의의 랜덤한 메세지 내용 생성)
public record Message(String name, String message) {

    public static final Random random = new Random();

    public static Message fetch(String name) {
        if (random.nextInt(10) == 0) {
            // 10% 확률로 메세지 전송 실패
            throw new RuntimeException("메세지 전송 오류!");
        }

        char randomUpperCase = (char) (random.nextInt(26) + 'A');
        return new Message(name, String.valueOf(randomUpperCase));
    }

    @Override
    public String toString() {
        return name + " : " + message;
    }
}
```
```java
// 데이터 흐름 제어 목적의 중재자 객체
public class ChatSubscription implements Subscription {

    private final Subscriber<? super Message> subscriber;
    private final String name;

    public ChatSubscription(Subscriber<? super Message> subscriber, String name) {
        this.subscriber = subscriber;
        this.name = name;
    }

    @Override
    public void request(long n) {
        for (long i = 0L; i < n; i++) {
            try {
                subscriber.onNext(Message.fetch(name)); // 현재 메세지 Subscriber 전달
            } catch (Exception e) {
                subscriber.onError(e); // 에러 발생 시, Subscriber로 에러 전달
                break;
            }
        }
    }

    @Override
    public void cancel() {
        subscriber.onComplete(); // 구독이 취소되면 완료 신호 Subscriber에게 전달
    }
}
```
```java
// 발행 메세지 수신 및 처리 구독자 객체
public class ChatSubscriber implements Subscriber<Message> {

    private Subscription subscription;

    @Override
    public void onSubscribe(Subscription subscription) {
        this.subscription = subscription;
        subscription.request(1); // 구독 저장 후, 첫 번째 요청 전달
    }

    @Override
    public void onNext(Message item) {
        System.out.println(item); // 메세지 출력
        this.subscription.request(1); // 다음 정보 요청
    }

    @Override
    public void onError(Throwable throwable) {
        System.out.println(throwable.getMessage()); // 에러 발생 시, 에러 메세지 출력
    }

    @Override
    public void onComplete() {
        System.out.println("종료");
    }
}
```

이제 Main 클래스의 psvm에서 발행자 `Publisher<Message>` 구현체를 통해 메세지를 발행해본다.

```java
public class Main {
    public static void main(String[] args) {
        // 사용자 '김동준'을 만들고 Subscriber 구독시킴
        getChatMessage("김동준").subscribe(new ChatSubscriber());
    }

    private static Publisher<Message> getChatMessage(String name) {
        return subscriber -> subscriber.onSubscribe(
                new ChatSubscription(subscriber, name)
        ); // 구독한 Subscriber에게 Chat 구독(ChatSubscription)을 전송하는 Publisher 반환
    }
}
```
<img width="80%" alt="스크린샷 2025-03-18 오후 4 35 11" src="https://github.com/user-attachments/assets/d97517f7-3da3-498c-84f3-ce01c0d411f9" />

전체 흐름은 아래와 같다.

1. `getChatMessage("김동준")` 호출:
    - `Publisher<Message>` 반환. 이 `Publisher`는 `ChatSubscription`을 사용하여 구독자를 처리
2. `Main` 클래스에서 구독:
    - `getChatMessage("김동준").subscribe(new ChatSubscriber())`를 통해 `ChatSubscriber`가 구독자로 등록
    - 구독자가 `onSubscribe()`를 호출하여 첫 번째 메시지를 요청
3. `ChatSubscription`에서 메시지 발행:
    - `ChatSubscription`은 메시지를 발행하고 `subscriber.onNext()`를 통해 구독자에게 메시지를 전달
    - 구독자는 `onNext()`에서 메시지를 처리하고 다음 메시지를 요청
4. `ChatSubscriber`에서 메시지 처리:
    - 구독자는 `onNext()`를 통해 메시지를 처리. `request(1)`을 호출하여 다음 메시지를 요청
    - 메시지가 모두 처리되면 `onComplete()`가 호출되어 종료 메시지가 출력
  
기존의 명령형에서는 메세지를 생산한 객체가 수신자에게 전달하는 책임까지 부담하는 구조로 코드가 짜여지는 것을, 발행자 - 중재자- 구독자의 세 단계로 나눠서 데이터의 발행과 처리, 제어에 대한 책임을 나눠서 컴포넌트의 독립성을 강화하고 시스템의 유연성을 확보하게 된다.

## 3. 자바에서의 리액티브 프로그래밍

위에서 언급했듯이 자바 자체가 리액티브 프로그래밍에 엄청 어울리는 언어는 아니라고 생각한다. 물론 다양한 함수형 API들이 제공되고 비동기 처리를 위한 멀티 스레드 개념을 적극적으로 활용할 수 있는 수단들이 제공되는 것은 분명 함수형 패러다임에 대한 훌륭한 대비책이겠지만, 태생 자체가 객체지향성을 바라보는 목적성은 코드 작성 과정에서도 드문드문 드러나는 것이 느껴졌다.

그렇기 때문에 여기서 그치지 않고 **Spring WebFlux** 같은 모듈이나 코틀린, 파이썬 같은 다른 언어들을 학습하여 리액티브 앱을 구현하기 위한 최적의 환경에 대한 이해도를 갖춰야 할 것이다. 어찌됐든 자바가 다른 언어 대비 가지는 최대의 장점이 엔터프라이즈 규모의 앱에 대하여 강력한 안정성 및 고성능 제공므로 이런 기존의 장점들을 바탕으로 어떻게 리액티브 앱 구현에서 자바를 활용할 수 있을 지를 고민하는 과정을 거쳐봐야겠다.
