# Chapter 1: Java 8, 9, 10, and 11 – 무슨 일이 일어나고 있는가?

## 1.1 역사의 흐름은 무엇인가?

Java는 1996년 JDK 1.0이 출시된 이후 꾸준히 발전해왔다. 특히 Java 8은 Java의 역사에서 가장 중요한 변화 중 하나로 평가되며, 람다 표현식, 스트림 API, 메서드 참조, 기본 메서드(default methods) 등의 기능이 추가되었다. 이러한 기능들은 코드의 가독성과 간결성을 높이고, 멀티코어 프로세서를 보다 쉽게 활용할 수 있도록 한다.

Java 9에서는 모듈 시스템을 도입하여 대규모 애플리케이션의 관리가 용이해졌으며, Java 10과 Java 11에서는 개발자의 편의를 위한 다양한 기능들이 추가되었다.

### Java 8의 주요 기능

**람다 표현식 (Lambda Expressions)**  
> 익명 클래스를 대체하여 보다 간결하게 함수형 프로그래밍 스타일을 적용할 수 있도록 한다.

#### 기존 Java 코드 (익명 클래스 사용)
```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("마!");
    }
};
r.run();
```

#### Java 8 코드 (람다 표현식 사용)
```java
Runnable r = () -> System.out.println("마!");
r.run();
```

**스트림 API (Stream API)**  
- 컬렉션 데이터를 함수형 스타일로 처리할 수 있으며, 내부적으로 병렬 처리 지원이 가능하다.

#### 기존 코드 (반복문 사용)
```java
List<String> names = Arrays.asList("동준", "은영", "예림", "아름");
List<String> result = new ArrayList<>();
for (String name : names) {
    if (name.startsWith("A")) {
        result.add(name);
    }
}
```

#### Java 8 코드 (스트림 API 사용)
```java
List<String> result = names.stream()
                          .filter(name -> name.startsWith("A"))
                          .collect(Collectors.toList());
```

**메서드 참조 (Method Reference)**  
- 기존 메서드를 보다 간결하게 사용할 수 있도록 한다.

#### 기존 코드 (람다 표현식 사용)
```java
List<String> names = Arrays.asList("동준", "허각", "Charlie Puth");
names.sort((a, b) -> a.compareTo(b));
```

#### Java 8 코드 (메서드 참조 사용)
```java
names.sort(String::compareTo);
```

**기본 메서드 (Default Methods)**  
- 인터페이스에 기본 구현을 제공하여 하위 호환성을 유지할 수 있도록 한다.

#### 기본 메서드 예제
```java
interface Vehicle {
    default void start() {
        System.out.println("부릉!");
    }
}

class Car implements Vehicle {
    // 별도로 start()를 구현하지 않아도 기본 메서드가 사용됨
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.start(); // "부릉!"
    }
}
```

---

## 1.2 왜 아직도 자바는 변화하는가? : 자바 8에서 도입된 새로운 개념을 중심으로.

### 1.2.1 멀티코어 프로세서 환경에서의 병렬 처리 요구

- 기존 Java의 단일 스레드 방식 → 멀티코어 프로세서를 효율적으로 활용하지 못하는 경우가 많았음.
- Java 8에서는 스트림 API와 `CompletableFuture`를 통해 병렬 처리를 더욱 쉽게 구현할 수 있도록 개선했다.

> `CompletableFuture` v.s `Future` 
> - 후자는 결과 회수 요청(`get()`) 호출 시 이에 대한 처리가 Blocking 형태로 이루어짐.
> - 전자의 경우 이를 비동기적으로 처리할 수 있도록 지원.

### 1.2.2 코드의 간결성과 유지보수성

#### 기존 Java 코드 (익명 클래스 사용)
```java
List<String> names = Arrays.asList("동준", "예림", "은영");
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
});
```

#### Java 8 코드 (람다 표현식 사용)
```java
List<String> names = Arrays.asList("동준", "예림", "은영");
names.sort((a, b) -> a.compareTo(b));
```
더 나아가 메서드 참조를 활용하면 더욱 간결해진다.
```java
names.sort(String::compareTo);
```

## 1.3 함수형 프로그래밍의 도입

### 1.3.1 메서드 참조

Java 8에서는 메서드를 값처럼 다룰 수 있도록 **메서드 참조** 기능을 추가했다.

#### 기존 코드
```java
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
    @Override
    public boolean accept(File file) {
        return file.isHidden();
    }
});
```

#### Java 8 코드 (메서드 참조 사용)
```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```
메서드 참조를 사용하면 기존 메서드를 람다 표현식 없이 그대로 활용할 수 있어 더욱 간결한 코드 작성이 가능하다.

## 1.4 스트림 API (Stream API)

스트림 API는 컬렉션 데이터를 함수형 스타일로 처리할 수 있도록 설계되었다. 스트림은 내부 반복을 사용하여 데이터를 처리하고, 병렬 처리(Parallel Processing)를 쉽게 구현할 수 있도록 돕는다.

### 1.4.1 스트림 API의 주요 특징
- **선언적 코드 작성 가능**: 기존 반복문보다 간결하고 가독성이 높은 코드 작성이 가능함.
- **내부 반복 지원**: 컬렉션의 요소를 개발자가 직접 반복하지 않고 스트림 API가 내부적으로 처리함.
- **병렬 처리 용이**: `parallelStream()`을 사용하면 멀티코어 환경에서 손쉽게 병렬 처리 가능.
- **중간 연산(Intermediate Operations)과 최종 연산(Terminal Operations) 분리**: 스트림 연산은 데이터 흐름을 명확히 분리하여 가독성을 높인다.

### 1.4.2 기존 방식과 스트림 API 비교

#### 기존 코드 (반복문 사용)
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
List<String> result = new ArrayList<>();
for (String name : names) {
    if (name.startsWith("A")) {
        result.add(name);
    }
}
```

#### Java 8 코드 (스트림 API 사용)
```java
List<String> result = names.stream()
                          .filter(name -> name.startsWith("A"))
                          .collect(Collectors.toList());
```

### 1.4.3 병렬 처리 예제
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
List<String> result = names.parallelStream()
                          .filter(name -> name.startsWith("A"))
                          .collect(Collectors.toList());
```
위 코드에서 `parallelStream()`을 사용하면 병렬 처리되어 성능이 향상될 수 있다. 단, 데이터 크기와 연산 비용에 따라 성능이 오히려 저하될 수도 있으므로 주의해야 한다.


## 1.5 결론

Java 8 이후의 변화는 코드의 간결성과 유지보수성을 높이는 방향으로 발전해왔다.
- **람다 표현식과 스트림 API**를 통해 더욱 선언적이고 함수형 스타일의 코드 작성이 가능해졌다.
- **기본 메서드와 모듈 시스템**은 대규모 애플리케이션의 유지보수를 용이하게 만든다.
- **Java 10, 11의 추가 기능**들은 개발자의 생산성을 높이고 최신 요구 사항에 대응할 수 있도록 지원한다.

Java는 계속해서 변화하고 있으며, 최신 기능을 익히고 활용하는 것이 더욱 중요해지고 있다.
