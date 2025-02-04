## Part 1 . Java 8, 9, 10, 11 의 주요 변경점

> part 1 에서는 자바 주요 변경점,이의 배경, 이후 서술될 내용들에대한 기본적인 개념을 설명한다.
>
> 그동안의 자바 패러다임과는 다른 전환점을 만들어낸 버전이라 Java 8의 설명이 대부분이다.
>
> LTS인 8,11,17,21 위주로 보면 좋을것 같다 그사이에는 어떤시도를 했는지 정도..

<br>

### 🔥Java 패러다임 변화 : Stream API의 등장
> 명령형 프로그래밍(기존) :  무엇을 어떻게 처리 할지를 하나하나 명령
>
> 🏷️선언형 프로그래밍 : Stream API + 람다표현식 -> 무엇을 할지를 선언하는 방식
>
> 🏷️함수형프로그래밍으로의 확장 :  함수형 인터페이스와 고차함수 개념이 활용되면서 함수형프로그래밍 패러다임까지 확장

<br>

### 🔥Stream API 정리 
- `선언형`으로 `간결한` 코드
- Stream API의 동작 흐름
    - Stream<T> 생성
    - 중간 연산 (함수형 인터페이스인자 받음 -> `새로운` Stream<T> 반환)
      - 중간 연산 메서드 체이닝 가능 
    - 최종 연산 (함수형 인터페이스인자 받음 -> `새로운` Stream<T> 반환)
      - 최종연산에선 스트림을 소모하여 결과 반환
    - 결과 반환 (List, int, Optional )
- `지연연산`으로 `성능 최적화` (최종연산 전까지 지연, 불필요한 연산제외) 
- Stream 라이브러리의 내부연산으로 스레드없이 멀티코어 병렬처리가 가능하다
- Stream은 `소모`되도록 설계됨
  - 안전한 데이터 처리를 위한 메커니즘
  - Thread safe 병렬처리
  - `메모리 누수방지` 
- `불변성`유지 (새로운 컬렉션(또는 스트림)을 반환)
  - `원본 데이터 보호`
  - `병렬 처리 안정성`
  - 함수형 프로그래밍의 원칙

###### 멀티스레드 환경에서의 안정성
> 가변 데이터(ArrayList등)에 직접 접근하면 데이터 유실이나 동기화 문제가 발생
>
> 스트림 내에서 불변 데이터를 사용하거나 새로운 스트림을 만들어 작업해야 한다.

```JAVA
/*
List<Integer> numbers = IntStream.range(0, 1000)
                                 .boxed() // IntStream → Stream<Integer> 변환
                                 .collect(Collectors.toList());

List<Integer> parallelProcessed = numbers.parallelStream() // 컬렉션에서 병렬 스트림 생성
                                         .map(n -> n * 2)
                                         .collect(Collectors.toList());
*/

List<Integer> list = new ArrayList<>();
IntStream.range(0, 1000)
  .parallel() // 병렬 스트림 변환 (멀티스레드 사용)
  .forEach(list::add); // 여러 스레드 동시에 list.add() 실행

List<Integer> safeList = IntStream.range(0, 1000)
                                  .parallel() // 기존 스트림 병렬로 변환
                                  .boxed()
                                  .collect(Collectors.toList()); // 새로운 리스트 생성

```

<br>

### 동작 파라미터, 함수형 인터페이스,람다
> [동작 파라미터]
> 
> 메서드나 함수가 수행할 `동작`을 `외부에서 전달`하는 개념
> 
> 메서드가 수행할 동작을 `호출 시점`에서 `외부에서 정의`할 수 있다
> 
> 이를 위해 함수형 인터페이스를 사용하여 동작을 정의, 이를 매개변수로 메서드에 전달
>
> 함수형 인터페이스를 구현하는 간결한 방법 : `람다`

<br>

### Stream API와 람다표현식 
> Stream API는 메서드는` 함수형 인터페이스`를 `매개변수`로 받도록 설계되어있다.
>
> 어떻게 할지를 선언하는것을 표현하는 함수가 필요하다
>
> 람다는 함수형 인터페이스를 간결하게 구현한것을 표현하는 문법이다.
>
> 아래의 Stream API + 람다의 예시가 데이터 처리로직을 간결하게 작성하고 있음을 확인할수 있다.

<br>

###### 1. Stream API (람다X,익명 클래스O)
> 명시적으로 Predicate<String> 구현체를 제공해야 하므로 코드가 장황
```JAVA
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

List<String> filtered = names.stream()
    .filter(new Predicate<String>() {  // 람다 없이 익명 클래스 사용 (Predicate의 단일 추상메서드 test)
        @Override
        public boolean test(String s) {
            return s.startsWith("A");
        }
    })
    .collect(Collectors.toList());
```
###### 2. 람다 (함수형 인터페이스, 람다)
> 함수형 인터페이스의 단일 메서드를 람다로 구현
>
> 구현한걸 사용한 예시일 뿐이다.

```JAVA
Predicate<String> startsWithA = s -> s.startsWith("A");
boolean result = startsWithA.test("Alice"); // true
```
###### 3. Stream API + 람다
> 스트림(Stream API) 은 컬렉션(리스트 등)의 요소들을 처리하는 흐름이고
>
> 람다는 그 과정에서 개별 요소를 평가하거나 변환하는 역할을 한다.
```JAVA
List<String> filtered = names.stream()
    .filter(s -> s.startsWith("A")) 
    .collect(Collectors.toList());
```
###### 4. Stream API + 함수형 인터페이스
```JAVA
// 1. 람다
Predicate<String> startsWithA = s -> s.startsWith("A");

List<String> filtered = names.stream()
    .filter(startsWithA) // 함수형 인터페이스를 매개변수로 전달
    .collect(Collectors.toList());

// 2. 인터페이스 직접선언
@FunctionalInterface
interface StringFilter {
    boolean check(String str);
}

StringFilter startsWithA = s -> s.startsWith("A");

List<String> filtered = names.stream()
    .filter(startsWithA::check) // 함수형 인터페이스를 메서드 참조로 전달
    .collect(Collectors.toList());
```

<br>

##### 참고
###### 정리 표
|Stream 메서드|요구하는 함수형 인터페이스|예제 (람다 사용)  | 
|------------------------------------------------|-----------------------------------------------|-----------------------------------------------|
| `filter` | `Predicate<T>`| `filter(n -> n % 2 == 0)` |
| `map` | `Function<T, R>`| `map(n -> n * 2)` |
| `forEach` | `Consumer<T>`| `forEach(n -> System.out.println(n))` |
| `reduce` | `BinaryOperator<T>`| `reduce((a, b) -> a + b)`|


###### ❓Stream 메서드내에서 어떻게 람다가 함수형인터페이스를 만족시키는지

```JAVA

// Function<T, R> 인터페이스 정의
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t); // T를 받,R을 반환 
}

// map 사용 예제
List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
List<Integer> doubled = numbers.stream()
    .map(n -> n * 2)  // Function<Integer, Integer> 적용
    .collect(Collectors.toList());

// BinaryOperator<T> 인터페이스 정의
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    T apply(T t1, T t2);  //동일타입 T 두 개 받아서, 같은 타입 T의 값을 반환
}

// reduce 사용 예제 : 입력 a, b를 받아 a + b를 반환
List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
int sum = numbers.stream()
    .reduce((a, b) -> a + b)  // BinaryOperator<Integer> 적용
    .orElse(0); // 10
```

<br>
 

##### 함수형 인터페이스
> 단일 추상 메서드를 가지는 인터페이스임
>
> 람다 표현식이나 메서드 참조로 객체처럼 생성 가능

| 함수형 인터페이스 | 추상 메서드|역할 | 
|------------------------------------------------|-----------------------------------------------|-----------------------------------------------|
| `Comparator<T>` | `int compare(T o1, T o2) `              | `두 객체 비교` |
| `Function<T, R>` | `R apply(T t)`              | `입력을 받아 결과 반환` |
| `Predicate<T>` | `boolean test(T t)`              | `조건을 검사 true/false 반환` |


```JAVA
// 익명클래스
Comparator<Integer> comparator = new Comparator<Integer>() {
    @Override
    public int compare(Integer a, Integer b) {
        return a.compareTo(b);
    }
};
// 람다
Comparator<Integer> comparator = (a, b) -> a.compareTo(b);

// 메서드참조
Comparator<Integer> comparator = Integer::compareTo;
```

<br>

##### 메서드 참조와 일급 시민
> 메서드 참조 : 람다를 더 간결하게 표현
```JAVA
// 이미 정의된 메서드를 간단히 참조
Comparator<Integer> comparator = Integer::compareTo;
```
> 일급 시민(일급 객체) : 특정 개념이나 요소가 "일급 객체"로 취급
>
> 변수에 할당되거나, 함수의 인자로 전달되거나, 반환값으로 사용될 수 있음

##### 메서드 참조예시
```
/*
정적 메서드 참조
인스턴스 메서드 참조 (특정 객체의 메서드)
인스턴스 메서드 참조 (임의의 객체)
생성자 참조
*/
```


<br><br><br>


참고)

#### 버전별 주요변경 사항 한눈에 보기 

| **📌Java 8**                                    | **Java 9**                                      | **Java 10**                                    | **📌Java 11**                                    |
|-----------------------------------------------|------------------------------------------------|-----------------------------------------------|-----------------------------------------------|
| `Lambda_Expressions` | `모듈시스템(JPMS)` | `var 키워드(지역 변수) `              | `HttpClient 표준화` |
| `Stream_API` | `JShell(REPL)` | `G1 GC 개선`        | `jlink`   |
| `Default_Methods` | `HTTP 2 지원` | `JVM 메모리 개선 (컨테이너환경 메모리리포팅 지원)` | `ZGC저지연 지원`  |
| `Optional` | `private method- interface 내에 정의 가능`   |                             | `deprecated API 제거`   |
| `java.time_패키지`  |                      |                                              |                     |
| `Functional_Interfaces` |                |                                              |                 |

###### java8 : PermGem 영역 삭제, G1 GC가 기본 GC
###### var 키워드 : 10에서 도입, 이후 lts에도 사용이 가능했다..

##### java8 : 대규모 데이터 멀티코어 병렬 처리에대한 개선의 시작 및 함수형 프로그래밍 도입
> Stream API로 데이터 선언적으로 처리 (병렬 : parallelStream)
>
> Fork/Join Framework 개선으로 병렬 연산 최적화

##### java11 : 장기 지원(LTS) 및 성능 최적화
> GC 정지 시간 최소화로 안정성 극대화

<br>

#### java8 이후 버전 reference ( 8-> 17 , 21 )
[Java, Spring 버전 선택 가이드 2024 ( java21 )](https://kghworks.tistory.com/197) 📌📌📌

[8,11,17 간단 정리 ](https://cheerup313.tistory.com/86) 

[우리팀이 JDK 17을 도입한 이유](https://techblog.gccompany.co.kr/%EC%9A%B0%EB%A6%AC%ED%8C%80%EC%9D%B4-jdk-17%EC%9D%84-%EB%8F%84%EC%9E%85%ED%95%9C-%EC%9D%B4%EC%9C%A0-ced2b754cd7)
