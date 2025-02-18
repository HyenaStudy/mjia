# CHAPTER 3 람다 표현식

### 람다
- 메서드로 전달할 수 있는 익명 함수를 단순화한 것
- 특징
  - 함수 : 메서드처럼 특정 클래스에 종속되지 않지만 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함함
  - 전달 : 람다 표현식을 메서드 인수로 전달하거나 변수로 저장
  - 간결성, 익명성
<br>

- 람다식 표현 예제
```java
Compatator<Apple> byWeight = new Comparator<Apple>() {
  public int compare(Apple a1, Apple a2) {
    return a1.getWeight().compareTo(a2.getWeight());
  }
};

// 람다식으로 변경
Compatator<Apple> byWeight =
  (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
<br>

- 자바8에서 지원하는 람다 표현식
```java
1. 파라미터 하나, int 반환 (return 생략 가능)
(String s) -> s.length()

2. 파라미터 하나, boolean 반환
(Apple a) -> a.getWeight() > 150

3. 파라미터 둘, 여러 줄 가능
(int x, int y) -> {
  System.out.println("Result");
  System.out.println(x + y);
}

4. 파라미터 X, int 반환
() -> 42

5. 두 개의 Apple 객체 비교
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())
```
<br><br>

### 함수형 인터페이스
- 하나의 추상 메서드만 가지는 인터페이스
- @FunctionalInterface 어노테이션 사용

#### Predicate
```java
public interface Predicate<T> {
    boolean test(T t);
}

Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4)); // true
```
- 입력을 받아 boolean을 반환하는 조건 검사용
<br><br>

#### Consumer
```java
public interface Consumer<T> {
    void accept(T t);
}

Consumer<String> toUpper = s -> System.out.println(s.toUpperCase());
toUpper.accept("hello"); // HELLO
```
- 입력값을 받아 소비(사용) -> 반환 X
<br><br>

#### Function
```java
public interface Function<T, R> {
    R apply(T t);
}

Function<String, Integer> strToLength = s -> s.length();
System.out.println(strToLength.apply("hello")); // 5
```
- T 타입을 받아 R 타입으로 변환해서 반환
- 타입변환이 목적일 때 사용
<br><br>

#### Supplier
```java
public interface Supplier<T> {
    T get();
}

Supplier<Double> randomNum = () -> Math.random();
System.out.println(randomNum.get()); // 0.8573
```
- 입력없이 반환하며 데이터를 생성하거나 가져올 때 사용
<br><br>

#### UnaryOperator (단항)
```java
public interface UnaryOperator<T> {
    T apply(T t);
}

UnaryOperator<String> toUpper2 = str -> str.toUpperCase();
System.out.println(toUpper2.apply("hello")); // HELLO
```
- 입력을 받아 **같은 타입**으로 변환
<br><br>

#### BinaryOperator (이항)
```java
public interface BinaryOperator<T> {
    T apply(T t1, T t2);
}

BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
System.out.println(max.apply(5, 10)); // 10
```
- 입력값 두 개를 받아 하나로 합치거나 비교
<br><br>

### 함수 디스크립터
- 람다 표현식의 입력/반환 타입을 서술하는 개념
- 함수형 인터페이스를 람다식으로 표현할 때 간략하게 보여주는 것

| 함수형 인터페이스    | 함수 디스크립터 |
|-------------------|--------------|
| Predicate<T>      | T -> boolean |
| Consumer<T>       | T -> void    |
| Function<T, R>    | T -> R       |
| Supplier<T>       | () -> T      |
| UnaryOperator<T>  | T -> T       |
| BinaryOperator<T> | (T, T) -> T  |
<br><br>