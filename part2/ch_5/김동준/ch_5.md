# 병렬 스트림

## 1. 개념

### 1) 정의

얘도 스트림이다. 지연 연산이니, 소모성이니 하는 개념은 전부 공유한다. 다만 일반적인 스트림과는 다르게 **병렬**적으로 수행한다는 게 핵심\
원래 스트림이라는 것은 컬렉션에서의 연산인데, 병렬 스트림은 그 연산이 순차적으로 이뤄지는 게 아닌 병렬적으로 이뤄지고 그말인 즉슨 멀티 스레딩 개념이 적용된다.

<img width="80%" alt="스크린샷 2025-03-03 오후 9 46 58" src="https://github.com/user-attachments/assets/9444f2d4-4494-40a1-a99c-45670a3d9ed7" />

위 사진처럼 병렬 스레드의 경우에는 실행하는 스레드가 1개가 아니고 순서가 보장되지 않는 것을 볼 수 있다.

멀티 스레딩으로 동작하는 이상 그 로직을 디버깅하기란 상당히 까다롭다. 동작 로직을 확인하려고 해도, 다른 스레드에서 동작 중이라 자동으로 스킵 처리가 된다. 그래서 이론적으로 어떻게 동작하는 지에 대해 확인해 볼 예정

<img width="740" alt="스크린샷 2025-03-03 오후 9 50 51" src="https://github.com/user-attachments/assets/71715f2d-f4c8-46e3-9137-eb18c1ee627b" />

### 2) Fork / Join

선언형의 핵심은 **무엇을**에 그친다고 했다. **어떻게**를 개발자가 모르게 하는 것(내부 구현에 그치게 하는 것)이 명령형에서 선언형으로 나아가는 과정이고 자바 8은 이를 충실하게 준수하고 있다. 그리고 이 점은 병렬 스트림에서도 빛을 발한다. 다만 그렇다고 멀티 스레딩을 알아서 처리한다고, 병렬 스트림이 멀티 스레딩을 완전 자동 제어하는 것은 아니다. 병렬 스트림과 일반적인 멀티 스레딩을 비교하자면...

| 비교 항목          | 병렬 스트림 (Parallel Stream)       | 일반 멀티 스레딩 (Thread, Executor) |
|-------------------|--------------------------------|-----------------------------|
| **스레드 관리**   | 포크조인 프레임워크 기반 | 개발자가 직접 생성 및 관리 |
| **작업 분할**     | 자동으로 데이터 청크 분할       | 수동으로 분할해야 함        |
| **순서 보장**     | 기본적으로 순서 보장 안 됨   | 직접 제어 가능          |
| **공유 자원 접근** | 동기화 문제 발생 가능       | 동기화 필요              |
| **성능 최적화**   | 데이터 크기에 따라 자동 최적화 | 개발자가 최적화 고민 필요  |
| **예외 처리**     | 개별 스레드에서 발생하면 감지 어려움 | 직접 try-catch 가능 |
| **적용 대상**     | 데이터 처리(배치 작업, 스트림 연산) | 병렬 I/O, 네트워크 작업 등 |

여기서 병렬 스트림의 핵심은 **포크조인 프레임워크(ForkJoin Framework)** 를 사용한다는 것인데, 자바에서는 `ForkㅓoinPool`과 `ForkJoinTask`를 바탕으로 기능이 동작한다.

1) **ForkJoinPool** (작업을 수행하는 공간 풀)
- 병렬 처리를 담당하는 스레드 풀 역할
- `parallelStream()`을 호출하면 내부적으로 **공유된 ForkJoinPool**을 사용
- 기본적으로 CPU 코어 개수에 맞춰 병렬 실행됨 (`Runtime.getRuntime().availableProcessors()`)
- 직접 크기를 지정하려면 `System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "4")` 와 같이 설정 가능
2) **ForkJoinTask** (작업을 수행하는 단위)
- 병렬 실행할 작업을 나타내는 객체
- `RecursiveTask<V>` : 값을 반환하는 작업
- `RecursiveAction` : 값을 반환하지 않는 작업

병렬 처리의 과정은 흡사 **분할 정복 알고리즘**과 유사하게 이뤄진다. 애초에 fork = 쪼개다, join = 모으다 라는 뜻을 가지고 있다. 병합 정렬 혹은 고속 거듭제곱 알고리즘과 매커니즘이 유사하다. 아래 두 개의 메소드는 `ForkJoinTask`에 존재하는 `fork()` 메소드와 `join()` 메소드다.

<img width="80%" alt="스크린샷 2025-03-03 오후 10 19 54" src="https://github.com/user-attachments/assets/3e6e3622-60fe-4272-9e72-138845163293" />

`fork()` 메소드는 현재 실행 중인 작업을 비동기적으로 실행할 수 있도록 포크(분기)하는 역할을 한다. 즉, `ForkJoinPool` 내부에서 현재 작업(Task)을 큐에 넣고 실행을 대기하도록 한다. 이 메서드는 현재 스레드가 풀 내부에서 실행하고 있는 워커 스레드인 `ForkJoinWorkerThread`인지 확인한 후 적절한 큐에 작업을 푸시한다.

```java
if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) { // 이미 ForkJoinPool 안에서 작업 중인 스레드를 가져왔다면?
    p = (wt = (ForkJoinWorkerThread)t).pool; // 현재 작업이 속한 풀 갖고오기
    q = wt.workQueue; // 현재 작업 중인 큐 갖고오기
}
else
    q = (p = ForkJoinPool.common).submissionQueue(false); // 아니라면 공용 풀을 가져와서 사용(추후, 공용 스레드가 워크 스틸링이 가능해짐)
```

현재 실행 중인 스레드가 풀 외의 스레드(일반 스레드)라면 공용 풀에 제출되어 추후 공용 스레드들의 **워크 스틸링**이 일어나는데, 이 워크 스틸링은 작업이 존재하지 않는 공용 스레드가 작업의 균등 분배 및 자원의 낭비를 막고 로드 밸런싱을 기대할 수 있게 된다.

![image](https://github.com/user-attachments/assets/9be5c124-5184-4b46-9e34-a711fb9a73b1)

위와 같이 작업중인 스레드 A는 자신의 데크에서 작업을 가져오는데, 작업이 비어있는 스레드 B는 작업이 쌓여있는 스레드 A의 데크의 반대 쪽에서 작업을 훔쳐오게 된다. 이를 통해 스레드 A와 스레드 B 둘 다 대기시간 없이 작업을 병렬적으로 수행할 수 있게 된다.

<img width="80%" alt="스크린샷 2025-03-03 오후 10 20 27" src="https://github.com/user-attachments/assets/d604e3e2-0278-4804-809a-9691050f4121" />

`join()` 메소드는 `ForkJoinTask` 클래스에서 작업 결과를 가져오게 된다.

```java
if ((s = status) >= 0) // 현재 작업의 상태를 가져옴
    s = awaitDone(s & POOLSUBMIT, 0L); // 작업이 완료되지 않았다면 완료될 때까지 기다림

if ((s & ABNORMAL) != 0) // 작업이 비정상적으로 종료되었는지 확인
    reportException(s); // 비정상 종료 시 예외를 던짐

return getRawResult(); // 작업이 정상적으로 종료되었다면 계산된 결과를 반환
```

최종적으로 작업들의 분기들이 다시 모아지면서 결과가 합산되며 반환되는데 그 과정들을 전부 포함한 내용은 아래 그림과 같다.

![image](https://github.com/user-attachments/assets/b822b3e0-adb2-43bb-bff5-b6fec6f30a44)

작업을 청크 단위로 적당히 자동으로 쪼개서 연산을 수행하고 그 조각난 연산 결과들을 모으면서 합산을 하여 최종 결과 데이터를 반환하게 하는 것이 기본적인 병렬 스트림의 원리인 셈이다. 병렬 스트림은 대규모 데이터셋에서 성능을 크게 향상시킬 수 있지만 그 이점을 얻기 위해서는 데이터의 크기, 멀티스레드 환경에서 발생할 수 있는 오버헤드와 동기화 문제를 고려해야 된다.

## 2. 시나리오 테스트

진짜로 그냥 스트림과 병렬 스트림이 성능 차이가 나는지, 시나리오를 가정하고 테스트를 해보자. 교재에서 알려준 **JMH(Java Microbenchmark Harness)** 를 활용하여 각 기능에 대해 벤치마크(소프트웨어나 하드웨어의 성능을 객관적으로 평가하는 과정)를 수행해서 비교할 예정

### 1) JMH 세팅

>**참조소스**
>- https://velog.io/@anak_2/Java-JMH-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%84%B1%EB%8A%A5%ED%85%8C%EC%8A%A4%ED%8A%B8
>- https://github.com/melix/jmh-gradle-plugin

나는 주로 Gradle을 활용해 자바 프로젝트를 빌드한다. 위의 내용들을 참조해서 세팅을 적용했다.

```gradle
plugins {
    id 'java'
    id "me.champeau.jmh" version "0.7.1"
}

dependencies {
    // ...

    jmh 'org.openjdk.jmh:jmh-core:0.9'
    jmh 'org.openjdk.jmh:jmh-generator-annprocess:0.9'
    jmh 'org.openjdk.jmh:jmh-generator-bytecode:0.9'
}

jmh{
    fork = 1 // JVM 내부 최적화 영향 배제를 위한 여러 프로세스 독립 실
    warmupIterations = 1 // JVM 초기 최적화
    iterations = 1 // 반복 테스트를 통한 신뢰성 확보
}
```

`jmh` 명의의 별개 소스 모듈을 만들고 그 내부의 `java` 패키지에 속한 커스텀 패키지를 하나 세팅해서 테스트 대상 클래스를 작성 후, 벤치마크 관련 어노테이션을 부여해준 후 전체 프로젝트 모듈 루트에서 `./gradlew jmh`을 호출하면 테스트가 시작된다.

### 2) 시나리오 1: 대용량 연산 비교

병렬 처리라는 특성을 생각했을 때, 멀티 스레드를 활용하는 이유는 연산의 동시 처리를 통해 시간적인 이점을 앞당기는 것이다. 즉 연산의 길이가 아주 복잡하고 길 때, 해당 연산을 순차적으로 처리하는 것이 아닌 병렬적으로 처리하는 것이 시간적으로 더 빠르게 결과를 반환받을 수 있을 것이다. 이 점에 착안해서 벤치마크를 세팅하여서 테스트를 수행했다.

연산은 다음과 같이 정의하고 용량 기준별로 추가 분류하여 총 4개의 메소드를 정의했다.

>1. 짝수 요소 필터링
>2. 요소 2배 매핑
>3. 제수가 3인 연산의 나머지가 1인 요소 필터링
>4. 요소 제곱 매핑
>5. 최종 합산

```java
@State(Scope.Benchmark) // 같은 벤치마크끼리 객체 공유(멀티스레드 측정용)
@OutputTimeUnit(TimeUnit.MICROSECONDS) // 벤치마킹 결과 단위 설정
@BenchmarkMode(Mode.All) // JMH의 벤치마크 실행 범위 지정
public class LargeScaleComputation {

    private static final long SMALL_SIZE = 10_000;
    private static final int LARGE_SIZE = 10_000_000;

    @Benchmark
    public long sequentialSmall() {
        return LongStream.rangeClosed(1, SMALL_SIZE)
                .filter(n -> n % 2 == 0)
                .map(n -> n * 2)
                .filter(n -> n % 3 == 1)
                .map(n -> n * n)
                .sum();
    }

    @Benchmark
    public long parallelSmall() {
        return LongStream.rangeClosed(1, SMALL_SIZE)
                .parallel()
                .filter(n -> n % 2 == 0)
                .map(n -> n * 2)
                .filter(n -> n % 3 == 1)
                .map(n -> n * n)
                .sum();
    }

    @Benchmark
    public long sequentialLarge() {
        return LongStream.rangeClosed(1, LARGE_SIZE)
                .filter(n -> n % 2 == 0)
                .map(n -> n * 2)
                .filter(n -> n % 3 == 1)
                .map(n -> n * n)
                .sum();
    }

    @Benchmark
    public long parallelLarge() {
        return LongStream.rangeClosed(1, LARGE_SIZE)
                .parallel()
                .filter(n -> n % 2 == 0)
                .map(n -> n * 2)
                .filter(n -> n % 3 == 1)
                .map(n -> n * n)
                .sum();
    }
}
```

<img width="80%" alt="스크린샷 2025-03-04 오후 2 57 01" src="https://github.com/user-attachments/assets/e120535b-3eb4-41c9-a5c4-d9b2b8ad4795" />


| 벤치마크                               | 모드   | 초당 연산 횟수 (ops/us)    | 평균 (us/op)     | 중앙값 (us/op)   | 최악 케이스 (us/op)   |
|------------------------------------------|--------|-------------------|------------------|-----------------|-----------------|
| **sequentialSmall**                      | thrpt  | 0.026             | 38.649           | 37.568          | 1359.872        |
| **parallelSmall**                        | thrpt  | 0.029             | 34.093           | 33.024          | 612.352         |
| **sequentialLarge**                      | thrpt  | ≈ 10⁻⁵            | 40121.422        | 38469.632       | 42139.648       |
| **parallelLarge**                        | thrpt  | ≈ 10⁻⁴            | 10541.860        | 10723.328       | 13549.568       |

꽤 유의미한 결과가 나왔다. 각 기준에 맞춰서 테스트 결과를 분류해보자면

#### (1) SMALL_SIZE(일만) 기준
일반 스트림의 평균 초당 처리횟수(약 `38.6 us/op`)나 병렬 스트림의 평균 초당 처리횟수(약 `34.1 us/op`)는 크게 차이가 나지 않는다.

#### (2) LARGE_SIZE(대용량) 기준
일반 스트림의 평균 초당 처리횟수(약 `40121 us/op`)는 병렬 스트림의 평균 초당 처리횟수(약 `10541 us/op`)의 4배 차이로, 병렬 스트림이 연산 속도가 4배 더 빠르다. 이를 통해 연산량이 많아질 수록 일반 스트림보다 병렬 스트림이 더 빠른 속도를 낼 수 있음을 확인할 수 있다.


### 2) 시나리오 2: 파일 입출력
