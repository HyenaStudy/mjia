### 람다 표현식이란?

람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 표현 방식입니다.

- 이름이 없는 함수
- 파라미터 리스트, 바디, 반환 타입, 발생할 수 있는 예외 리스트를 가짐
- 코드를 간결하게 유지하고 유지보수에 좋음

```java
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
};

Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

```
---

### 람다 표현식 문법

- (parameters) -> expression
- (parameters) -> { statements; }

```java
(String s) -> s.length()  // 입력된 문자열의 길이를 반환
(Apple a) -> a.getWeight() > 150  // 무게가 150보다 큰지 여부 확인
(int x, int y) -> { System.out.println(x + y); }  // 두 개의 숫자를 더한 후 출력
() -> 42  // 파라미터 없이 항상 42를 반환

```

---

### 함수형 인터페이스

- 람다 표현식은 함수형 인터페이스를 기반으로 동작합니다.
- 함수형 인터페이스 : 단하나의 추상 메서드만 가진 인터페이스
- 람다 표현식은 이러한 인터페이스의 단일 메서드 구현을 대신할 수 있다.

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

```

### 람다 표현식 활용

**필터링**

```java
List<Apple> greenApples = inventory.stream()
    .filter((Apple a) -> "GREEN".equals(a.getColor()))
    .collect(Collectors.toList());

```

**Comparator**

```java
Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

```

**Runnable**

```java
Runnable r1 = () -> System.out.println("Hello World!");

```

---

## 5. 실행 어라운드 패턴

![image.png](attachment:da981886-39df-453f-9c8f-75354c8cab58:image.png)

람다 표현식을 활용하면 리소스 정리 코드의 중복을 줄일 수 있다.

```java
// 기존
public String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine();
    }
}

```

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br);
    }
}

// 람다 표현식 사용
String result = processFile((BufferedReader br) -> br.readLine());

```

### function 패키지

Predicate : True/False 반환

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

```

**consumer : 입력값을 받아서 어떤 동작을 수행하지만 결과를 반환하지 않음**

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

```

**Function<T, R> : 입력값을 받아서 출력값을 반환**

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

---

### 형식 검사, 형식 추론, 제약

람다 표현식을 사용할 때 함수형 인터페이스의 인스턴스를 생성할 수 있습니다. 하지만 람다 표현식 자체에는 구체적인 형식 정보가 포함되지 않으므로, 이를 이해하기 위해 형식 검사와 형식 추론 개념을 살펴봐야 합니다.

**형식 검사**

- 람다가 사용되는 컨텍스트를 이용해 람다의 형식(type) 을 추론할 수 있다.

```java
List<Apple> heavierThan150g =
    filter(inventory, (Apple apple) -> apple.getWeight() > 150);

```


### 형식 검사 과정

1. filter메서드가 사용된 부분을 확인한다.
2. filter메서드는 두 번째 파라미터로 Predicate<Apple>을 기대한다.
3. 람다 표현식 (Apple apple) -> apple.getWeight() > 150의 형식이 Predicate<Apple>에 맞는지 검사한다.
4. Predicate<Apple>의 test 메서드에 해당하는 부분인지 확인한다.
5. filter 메서드로 전달된 람다가 올바르게 사용될 수 있는지 최종 검사한다.

---

같은 람다 표현식이라도 컨텍스트 에 따라 다른 함수형 인터페이스의 구현이 될 수 있다.

```java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;

```

- Callable<Integer>: call() 메서드를 구현하는 함수형 인터페이스
- PrivilegedAction<Integer>: run() 메서드를 구현하는 함수형 인터페이스


**형식 추론**

자바 컴파일러는 **형식 추론(type inference)** 을 사용하여 람다 표현식의 파라미터 형식을 유추할 수 있다.

```java
List<Apple> greenApples =
    filter(inventory, apple -> GREEN.equals(apple.getColor()));

```

apple의 형식을 명시적으로 적지 않아도, `filter` 메서드의 정의를 통해 `Apple` 타입임을 추론할 수 있다.

### 명시적 형식과 생략된 형식 비교

```java
// 명시적 형식 지정
Comparator<Apple> c1 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

// 형식 추론 사용
Comparator<Apple> c2 = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());

```

---
###지역 변수 사용

람다 표현식 내부에서 지역 변수를 참조할 수 있습니다.

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);

```
```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 31337;

```

###메서드 참조

람다 표현식과 유사하지만 더 간결한 방식으로 메서드를 참조할 수 있는 방법입니다.


```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
inventory.sort(Comparator.comparing(Apple::getWeight));

```

유형 1 : 정적 메서드 참조 Integer::parseInt
유형 2 : 특정 객체의 인스턴스 메서드 참조 expensiveTransaction::getValue
유형 3: 임의 객체의 인스턴스 메서드 참조 string:length
유형 4 : 생성자 참조 Apple::new
