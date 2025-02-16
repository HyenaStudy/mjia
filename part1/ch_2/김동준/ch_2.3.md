# 람다 이모저모

## 1. 기본 문법

### (1) 함수형 인터페이스

람다식은 함수형 인터페이스(`@FunctionalInterface`)를 구현하는 것으로부터 시작됨. 이 함수형 인터페이스의 특징은 **무조건 추상 메소드를 하나만 가질 것**이다. 자바 API가 기본적으로 제공하는 함수형 인터페이스들(`Predicate`, `Comparator`, `Runnable`, `Callable`) 외에도 개발자가 커스텀하게 함수형 인터페이스 선언이 가능

```java
@FunctionalInterface
public interface Calculator {
    int calculate(int x, int y);
}
```
```java
public class Main {
    public static void main(String[] args) {
        Calculator plus = (x, y) -> x + y;
        Calculator minus = (x, y) -> x - y;
        Calculator multiple = (x, y) -> x * y;
        Calculator divide = (x, y) -> x / y;

        System.out.println(plus.calculate(6, 2));
        System.out.println(minus.calculate(6, 2));
        System.out.println(multiple.calculate(6, 2));
        System.out.println(divide.calculate(6, 2));
    }
}
```

아래처럼 람다식의 형태가 동일해도 상위 함수형 인터페이스가 다른 경우가 있을 수도 있다.

```java
import java.util.function.Function;
import java.util.function.Predicate;

public class Main {
    public static void main(String[] args) {
        Predicate<String> predicate = str -> str.length() > 5;
        Function<String, Boolean> function = str -> str.length() > 5;

        System.out.println(predicate.test("abcdef")); // true
        System.out.println(function.apply("abcdef")); // true
    }
}
```

람다식 그 자체로는 구현의 기반이 된 함수형 인터페이스가 어떤 것인지 타입을 확인하기 어렵다. 자바 컴파일러는 아래와 같은 정보들을 바탕으로 타입을 추론한다. 위처럼 람다식이 할당된 변수의 타입을 통해 컴파일러가 타입을 추론하게 된다.
> 1. 람다식이 할당된 변수의 타입
> 2. 람다식을 인자로 받는 메서드의 매개변수 타입
> 3. 람다식이 사용된 컨텍스트(타입을 요구하는 상황)

즉 위와 같은 정보들이 없으면 컴파일러는 람다식의 타입을 추론할 수 없게 되므로 컴파일 에러를 일으키게 된다.

<img width="80%" alt="스크린샷 2025-02-16 오후 7 14 15" src="https://github.com/user-attachments/assets/6a76e88d-4e09-459e-a8fe-083c7bdefea9" />

---

만약 메소드 오버로딩을 활용해 다른 함수형 인터페이스 타입을 인자로 받게 해서, 동일한 형식의 람다식을 메소드에게 넘겨주면 어떻게 될까?

```java
import java.util.function.Function;
import java.util.function.Predicate;

public class Main {
    public static void main(String[] args) {
        Lambda lambda = new Lambda();
        lambda.method(x -> x.equals("String"), "String");  // ???
    }
}

class Lambda {
    public boolean method(Predicate<String> predicate, String data) {
        return predicate.test(data);
    }

    public boolean method(Function<String, Boolean> function, String data) {
        return function.apply(data);
    }
}
```

<img width="80%" alt="스크린샷 2025-02-16 오후 9 46 37" src="https://github.com/user-attachments/assets/e3cf6f81-1309-4a5f-9f0a-b925a95dbae3" />

두 개의 컴파일 경고 이슈가 확인되는데, 각각 하나씩 살펴보자면

#### Ambiguous method call issue

메소드 호출이 애매하다고 경고한다. psvm 내부에서 호출한 `method()`는 `Lambda` 클래스 인스턴스의 오버로딩 메소드 시그니처 2개와 동시에 일치하기 때문에 컴파일러가 어느 메소드를 호출한 것인지를 결정하지 못해서 컴파일 에러를 일으키는 것이다.

#### Cannot resolve method

이것은 람다식에 있는 `x` 변수의 타입을 추론하지 못해서 생기는 이슈다. 근데 의아한 게, 오버로딩된 메소드들을 살펴보면 어떤 것을 선택해도 결국 `String` 타입임에는 분명함에도 컴파일러가 이것조차 추론을 못 한다고 한다. 왜냐하면 앞전의 애매한 메소드 호출 이슈로 인해 애시당초 컴파일러가 어떤 함수형 인터페이스 타입인지조차 결정하지 못하기 때문에 함수형 인터페이스의 제네릭 타입이 문자열인 것도 추론하지 못하면서 컴파일 경고를 내뱉는 것이다. 실제로 실행시켜도 어차피 **Ambigious method call issue**를 내뱉으며 에러를 일으킬 것이므로 굳이 여기까지 가지 않아도 된다.

위의 이슈를 해결하려면 결국 람다식 앞에 타입을 명시적으로 추가하면서 형변환을 해줘야 해결할 수 있다.


### (2) 클로저(Closure)

**변수 캡처**는 람다식이나 익명 클래스가 외부 변수의 값을 사용할 때 발생하는 현상으로, 람다식 내부에서 외부 변수에 접근하는 방식이다.

<img width="80%" alt="스크린샷 2025-02-16 오후 7 53 43" src="https://github.com/user-attachments/assets/a4bdab85-2b66-4ee6-bc09-9e0e5554bb7e" />
<img width="80%" alt="스크린샷 2025-02-16 오후 8 01 06" src="https://github.com/user-attachments/assets/d83ca0fa-53f6-4707-91d1-f55bd08f98f2" />

원칙적으로 전역변수가 아니면 메소드 내부에서 다루는 변수들은 전부 지역변수로써 취급되면서 그 생명주기를 공유해야 하지만, 변수 캡쳐는 그 예외로 볼 수 있다. 위의 예제에서는 `int` 타입의 `x` 변수가 람다식에 의해 캡쳐되면서 **참조**되고 있다. 이 참조가 중요한데, 값을 그대로 끌고오는 것이 아니다. 이를 통해 함수형 프로그래밍에서 외부 변수의 상태를 참조하고 변경하지 않으면서 활용하는 방법으로 활용할 수 있다.

단, 변수 캡쳐의 전제 조건은 **변수가 불변**이어야 한다. 즉 변수의 상태가 `final`이거나 `effectively final`이어야 한다. 불변이 지켜지지 않으면 컴파일 에러가 발생한다.

<img width="80%" alt="스크린샷 2025-02-16 오후 8 16 16" src="https://github.com/user-attachments/assets/51ac54e5-8cf1-4360-a4e3-3f31d08fe65c" />

위의 경우는 변수 캡쳐된 `x` 변수가 처음에는 `effectively final` 상태(즉, 할당된 이후에 값의 변화가 발생하지 않으리라 기대하는 상태)여서 캡쳐가 허용됐으나 다음 라인에서 변수의 불변성이 깨지면서 컴파일 에러를 유발하는 사례다. `x` 변수는 원시 타입(`int`)이기 때문에 직접 참조되기 때문에 값의 변화가 곧 참조의 변화로 이어지기 때문에 변수 불변성이 지켜지지 않는 것이다.

---

**클로저**는 람다식이나 익명 클래스가 외부 변수의 값을 캡처하고 그 변수를 계속 유지하는 성질을 뜻한다.


```java
public class Closure {
    public static void main(String[] args) {
        Example example = new Example(10);
        Runnable runnable = () -> System.out.println("변수 값: " + example.getVariable());
        runnable.run();

        example.setVariable(20);
        runnable.run();
    }
}

class Example {
    int variable;

    public Example(int variable) {
        this.variable = variable;
    }

    public int getVariable() {
        return variable;
    }

    public void setVariable(int variable) {
        this.variable = variable;
    }
}
```
