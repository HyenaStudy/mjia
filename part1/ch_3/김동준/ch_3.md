# 람다 이모저모

## 1. 함수형 인터페이스

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


## 2. 변수 캡처(Variable Capture)와 클로저(Closure)

**변수 캡처**는 람다식이나 익명 클래스가 외부 변수의 값을 사용할 때 발생하는 현상으로, 람다식 내부에서 외부 변수에 접근하는 방식이다.

### 1) 원시 타입의 변수 캡처

<img width="80%" alt="스크린샷 2025-02-16 오후 7 53 43" src="https://github.com/user-attachments/assets/a4bdab85-2b66-4ee6-bc09-9e0e5554bb7e" />
<img width="80%" alt="스크린샷 2025-02-16 오후 8 01 06" src="https://github.com/user-attachments/assets/d83ca0fa-53f6-4707-91d1-f55bd08f98f2" />

원칙적으로 전역변수가 아니면 메소드 내부에서 다루는 변수들은 전부 지역변수로써 취급되면서 그 생명주기를 공유해야 하지만, 변수 캡쳐는 그 예외로 볼 수 있다. 위의 예제에서는 `int` 타입의 `x` 변수가 람다식에 의해 캡쳐되면서 **변수의 값이 복사**되고 있다. 분명히 `x`라는 변수는 람다식 입장에서는 외부(렉시컬 스코프)에 위치해 있고 람다식 내부만 봤을 때는 뜬금없이 `x`라는 타입조차 추론이 안 될 변수가 등장한 거라 원칙적으로는 컴파일 에러가 나야되지만 정상적으로 동작하는 것을 볼 수 있다.

단, 변수 캡쳐의 전제 조건은 **변수가 불변**이어야 한다. 즉 변수의 상태가 `final`이거나 `effectively final`이어야 한다. 불변이 지켜지지 않으면 컴파일 에러가 발생한다.

<img width="80%" alt="스크린샷 2025-02-16 오후 8 16 16" src="https://github.com/user-attachments/assets/51ac54e5-8cf1-4360-a4e3-3f31d08fe65c" />

위의 경우는 변수 캡쳐된 `x` 변수가 처음에는 `effectively final` 상태(즉, 할당된 이후에 값의 변화가 발생하지 않으리라 기대하는 상태)여서 캡쳐가 허용됐으나 다음 라인에서 변수의 불변성이 깨지면서 컴파일 에러를 유발하는 사례다. `x` 변수는 원시 타입(`int`)이기 때문에 직접 참조되기 때문에 값의 변화가 곧 변수의 변화로 이어지기 때문에 변수 불변성이 지켜지지 않는 것이다.

이때까지는 원시 타입의 변수 캡처에 대한 설명이었고, 참조 타입의 변수 캡처는 설명이 조금 달라지게 된다.

### 2) 참조 타입의 변수 캡처

```java
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

위와 같은 클래스가 있고, 이 클래스를 활용해서 똑같이 원시 타입의 변수 캡처 예제에 적용해본다. 결과는 아까와 똑같을 것이다.

<img width="80%" alt="스크린샷 2025-02-17 오전 12 07 54" src="https://github.com/user-attachments/assets/655582c4-f04e-4700-89fe-1c9e50c3922a" />

여기서 `Example` 클래스 내부의 `setter` 메소드를 호출해서 인스턴스의 필드를 변화시켜본다. 원시 타입 예제에서는 컴파일 에러가 발생했으나 여기서는 조금 다르다.

<img width="80%" alt="스크린샷 2025-02-17 오전 12 12 15" src="https://github.com/user-attachments/assets/0d7f4267-94ed-414a-93c2-a04885de1227" />

아까와 다르게 컴파일 에러가 바뀌지 않고 인스턴스의 필드가 정상적으로 업데이트되는 것을 확인할 수 있다. 이렇게 원시 타입 변수 캡처와 참조 타입 변수 캡처가 다르게 동작하는 이유는 **캡처의 대상이 타입의 형식에 따라 다르기 때문**이다.

모든 변수 캡처의 전제 조건은 **변수의 불변성 준수**다. 원시 타입의 변수 캡처는 **값이 직접 캡처**되기 때문에 흡사 값이 복사돼서 람다식 내부에서 사용되는 것과 같다. 그렇기 때문에 다른 값이 할당되는 시점부터 바로 불변성이 깨지게 되는 것이다. 그러나 참조 타입의 변수 캡처는 **참조가 캡처**되기 때문에 변수의 참조가 그대로 유지되면 내부의 필드 변화가 이뤄지든 뭘 하든 아무런 문제가 없는 것이다. 중요한 것은 참조 타입의 변수 캡처는 **참조가 불변**이어야 된다는 것이다. 다른 참조를 할당하게 되면 아래처럼 컴파일 에러가 발생하게 된다.

<img width="80%" alt="스크린샷 2025-02-17 오전 12 18 54" src="https://github.com/user-attachments/assets/f1607fbe-b034-44a4-bd86-b4d5274ac670" />

### 3) 클로저

**클로저**는 람다식이나 익명 클래스가 외부 변수의 값을 캡처하고 그 변수를 계속 유지하는 성질이자, 함수와 그 함수가 참조하는 외부 변수들을 함께 묶은 객체을 뜻한다. 사실 뭔 말하는지 잘 모르겠다. 그만큼 어려운 개념이긴 하다. 하나씩 천천히 정리해보자면...

```java
import java.util.function.IntSupplier;

public class Closure {
    public static void main(String[] args) {
        IntSupplier counter = closure(); // 클로저 생성

        System.out.println("변수 값: " + counter.getAsInt());
        System.out.println("변수 값: " + counter.getAsInt());
        System.out.println("변수 값: " + counter.getAsInt());
    }

    // IntSupplier : 매개변수를 받지 않고 int 값을 반환하는 함수형 인터페이스
    static IntSupplier closure() {
        int[] count = {0};
        return () -> ++count[0]; // IntSupplier 타입 람다식이 외부 변수(배열)인 count 캡처
    }
}
```

다음과 같은 예제가 있다고 가정하자. 함수형 프로그래밍에서는 **함수가 일급 객체로 취급**된다. 그렇기 때문에 정적 메소드인 `closure()` 또한 람다식을 반환하는 함수형 객체라고도 볼 수 있다. 아까까지 봤던 변수 캡처가 `closure()` 메소드 내부에서 일어나고 있는 것을 볼 수 있다. `count`라는 배열 참조 타입의 변수가 `return`되는 람다식에 의해 캡처되고 있다.

이제 `counter` 변수의 추상 메소드의 구현체인 람다식을 `getAsInt()` 메소드 호출로 동작시켜보자.

<img width="80%" alt="스크린샷 2025-02-17 오전 1 47 50" src="https://github.com/user-attachments/assets/88bdab74-56e5-4dec-9614-fdfd28dcf93d" />

`counter` 변수로부터 호출한 메소드로 인해 람다식 `() -> ++count[0]`이 동작하면서 `count[0]`의 변수 값이 1씩 가산되는 것을 확인할 수 있다. 분명히 `closure()` 메소드 입장에서는 외부인 `main(String[] args)`에서 람다식이 호출됐음에도 불구하고 `count` 배열의 내부 값이 영향을 받아 변화하고 있다. 이것이 발생할 수 있는 이유가 바로 클로저 때문이다. 정리하자면,

- `counter` 변수는 `closure()` 메서드에서 반환된 람다식, `() -> ++count[0]`을 가리킴
- 람다식 내부에서 **(람다식 입장에서의) 외부 변수 `count`**를 캡처하고, `count[0]` 값을 변경
- 람다식은 `closure()` 메서드 내에서 정의된 `count` 배열을 캡처하고 있기 때문에, `counter`가 호출될 때마다 `count[0]`의 값이 가산 갱신
- `closure()` 메서드는 람다식이 (람다식 입장에서의) 외부 상태(여기서는 `count[0]`)를 기억하고 있기 때문에, 그 상태가 갱신됐던 값으로 계속 유지

참고로 `int`로 착각할 수 있는데, `count`는 배열인 참조 타입이다. 그렇기 때문에 그 내부 요소의 값이 변해도 참조는 유지되기 때문에 캡처가 유효한 것이다. 핵심은 람다식(`() -> ++count[0]`) 내부에서 외부 변수(`count`)의 값이 변경(`++count[0]`)되더라도, 그 변수는 람다식 외부에서 정의된 상태(**그 호출로 인한 변화**)를 기억할 수 있는 것이 클로저다.

~~와씨 드럽게 어렵네~~

#### 다른 언어에서의 클로저

클로저는 자바에 한정된 개념이 아닌, 웬만한 함수형 프로그래밍을 채택하는 언어에서 등장하는 성질이다. 자바는 함수형 프로그래밍이 익명 클래스와 람다식을 통해 실현될 수 있어서 객체지향 관점에서 자주 접하기 힘든 개념이지만, 스크립트 언어인 자바스크립트와 파이썬에서는 꽤나 쉽고 빈번하게 접하는 개념이다.

```js
// JavaScript

function closure() {
    let count = 0; // 외부 변수

    return function() {
        count++; // 외부 변수 값 변경
        return count;
    }
}

const counter = closure(); // 클로저 반환
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```
```py
# Python

def closure():
    count = 0  # 외부 변수

    def counter():
        nonlocal count  # 외부 변수에 접근
        count += 1  # 외부 변수 값 변경
        return count

    return counter

counter = closure()  # 클로저 반환
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3
```

## 3. 함수형 프로그래밍에서의 람다식, 그리고 클로저

사실 클로저는 함수형 프로그래밍에서 매우 중요한 개념으로 다뤄진다. 그 이유는, 클로저가 상태 캡처, 불변성, 함수 조합, 비동기 처리 등 여러 함수형 프로그래밍의 원칙들을 효과적으로 구현할 수 있게 해주는 역할로써 존재하기 때문이다. 그렇지만 자바의 태생적인 존재 이유가 **객체지향 프로그래밍**의 실현이라 봐도 무방하기 때문에 다른 언어들에 비해 함수형 프로그래밍의 실현도가 낮은 편이고, 더불어서 클로저의 중요도 비중도 낮은 편이다. 그럼에도 불구하고 자바 8 이후에 함수형 프로그래밍을 위한 다양한 도구들이 도입되면서 클로저 개념 역시 도입됐다는 것은 자바의 확장성을 꾀하겠다는 의도가 아닐까 싶다... ~~지만 벌써 LTS 21을 바라보고 있다.~~

그래도 자바에서의 클로저 존재 의의를 조금 더 살펴보자면, 위의 예제에서 봤던 것처럼 **무분별한 변수의 전역화를 방지할 수 있다는 점**이 가장 큰 것 같다. 그와 더불어 변수의 갱신 기억을 위해 별개의 클래스를 정의하지 않아도 기억이 가능하다는 점 정도? 클로저의 중요성과 함수형 프로그래밍에서의 정확한 동작 원리를 파악하고 활용도를 높이려면 다른 프로그래밍 언어 학습도 필요하다. 일단 자바스크립트 공부하던 시절에 너무 어려워서 무릎 꿇었던 클로저를 어느 정도 감 잡은 듯해서 나는 만족:D

클로저가 함수형 프로그래밍에서 중요한 이유는 다음과 같다.

#### (1) 상태를 기억하기 때문에 함수형 프로그래밍에서의 캡슐화 실현 가능

아까 위에서 본 클로저 예제를 보자.

```java
import java.util.function.IntSupplier;

public class Closure {
    // Controller (count 변수에 직접 접근할 수는 없지만, counter를 통해 조작할 수 있음)
    public static void main(String[] args) {
        IntSupplier counter = closure();  // count 변수를 감싸고 있는 클로저를 반환

        System.out.println("변수 값: " + counter.getAsInt());
        System.out.println("변수 값: " + counter.getAsInt());
        System.out.println("변수 값: " + counter.getAsInt());
    }

    // Service (count 변수를 직접 노출하지 않고, 클로저를 통해서만 접근 가능)
    static IntSupplier closure() {
        int[] count = {0};  // 외부에서 직접 접근 불가능한 상태 변수
        return () -> ++count[0];  // count 값을 증가시키는 클로저 반환
    }
}
```

`IntSupplier` 타입의 람다식을 반환하는 `closure()` 메소드를 서비스로 생각하고 `main(String[] args)` 정적 메소드를 컨트롤러로 생각해보면, 서비스의 변수라 할 수 있는 `count`는 컨트롤러에 직접 노출되지 않는다. 하지만 해당 변수를 캡처한 람다식을 통해 외부에서는 직접 접근할 수 없는 상태를 유지하면서도, 제공된 인터페이스를 통해 안전하게 값을 변경할 수 있다. 따라서 클로저를 활용하여 캡슐화를 실현하면서도 상태를 유지하는 함수를 만들 수 있다.

#### (2) 고차함수와의 응용성

클로저를 활용하면 동일한 고차함수여도 다양한 기준을 제시할 수 있는 등, 범용성있게 활용할 수 있다. 아래 예제를 보자.

```java
import java.util.function.IntPredicate;

public class ClosureHigherOrderFunction {

    // Service: 특정 값 이상만 허용하는 조건을 가진 클로저 반환 (고차 함수)
    static IntPredicate thresholdFilter(int threshold) {
        return value -> value >= threshold; // threshold 변수 캡처
    }

    // Controller
    public static void main(String[] args) {
        IntPredicate isOver10 = thresholdFilter(10);
        IntPredicate isOver20 = thresholdFilter(20);

        System.out.println("<isOver10> 15는 기준에 속할까? : " + isOver10.test(15));
        System.out.println("<isOver20> 15는 기준에 속할까? : " + isOver20.test(15));
    }
}
```
<img width="80%" alt="스크린샷 2025-02-17 오후 5 01 14" src="https://github.com/user-attachments/assets/d2ebb61c-d52d-4781-b11e-85747f38a029" />


같은 클로저를 반환했음에도 변수를 기억하는 성질 때문에 다양한 기능을 구현하면서 코드의 재사용성이 증가하고 다형성을 실현할 수 있게 된다. 이것을 활용하여 비동기 프로그래밍에서 클로저를 강력하게 활용할 수 있을 것 같은데 아직 예제를 찾아보진 못함 ㅎ;
