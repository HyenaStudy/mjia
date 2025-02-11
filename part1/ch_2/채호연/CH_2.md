# Chapter 2: 코드 전달과 동작 파라미터화

## 2.1 변화하는 요구사항에 대응하기

소프트웨어를 개발할 때 요구사항은 끊임없이 변화한다. 특히 데이터를 필터링하는 기능은 자주 변경되는 요구사항 중 하나다. 예를 들어, 과일 목록에서 특정 조건을 만족하는 사과만 골라내는 기능을 생각해보자. 처음에는 녹색 사과만 필터링하면 됐지만, 시간이 지나면서 다른 조건이 추가될 수도 있다. 이런 변화에 어떻게 대응할 수 있을까?

### 첫 번째 시도 : 녹색 사과 필터링

가장 단순한 방식은 특정 조건을 만족하는 데이터를 직접 필터링하는 것이다. 예를 들어, 녹색 사과만 골라내는 코드를 작성할 수 있다.

```java
import java.util.*;

class Apple {
    private String color;
    private int weight;
    
    public Apple(String color, int weight) {
        this.color = color;
        this.weight = weight;
    }
    
    public String getColor() { return color; }
    public int getWeight() { return weight; }
    
    @Override
    public String toString() {
        return color + " 사과 (" + weight + "g)";
    }
}
```

```java
public class AppleFilter {
    public static List<Apple> filterGreenApples(List<Apple> inventory) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if ("green".equals(apple.getColor())) {
                result.add(apple);
            }
        }
        return result;
    }
    
    public static void main(String[] args) {
        List<Apple> apples = Arrays.asList(
            new Apple("green", 150),
            new Apple("red", 180),
            new Apple("green", 120)
        );
        System.out.println(filterGreenApples(apples));
    }
}
```

이 방식은 간단하지만, 만약 빨간 사과를 필터링하고 싶다면 새로운 메서드를 만들어야 한다. 이는 코드 중복을 초래할 뿐만 아니라 유지보수가 어렵게 된다.

### 개념: 하드코딩의 문제
이 방식의 문제는 특정 조건이 코드에 **하드코딩(hardcoded)** 되어 있어, 변경이 필요할 때마다 직접 코드 수정이 필요하다는 점이다. 따라서 요구사항이 변화할 때마다 새로운 메서드를 추가하는 것은 비효율적이다.

---

### 두 번째 시도 : 색을 파라미터화

중복을 피하기 위해 필터링할 색상을 파라미터로 받아 처리하는 방식으로 개선할 수 있다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, String color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (color.equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}
```

이제 `filterApplesByColor(apples, "red")` 와 같이 호출하면 빨간 사과를 필터링할 수 있다. 하지만 만약 무게를 기준으로 필터링하고 싶다면? 또다시 새로운 메서드를 만들어야 한다.

### 개념: 파라미터화를 통한 유연성 증가
이 방식은 **파라미터화를 통해 코드 재사용성을 높이는 기초적인 접근법**이다. 하지만 특정 속성(예: 색상)에만 한정된 방식이므로, **더욱 일반화된 방법**이 필요하다.

---

### 세 번째 시도 : 가능한 모든 속성으로 필터링

색상뿐만 아니라 무게도 포함하여 다양한 조건을 적용할 수 있도록 개선해보자.

```java
public static List<Apple> filterApples(List<Apple> inventory, String color, int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if ((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) {
            result.add(apple);
        }
    }
    return result;
}
```

이 방식은 유연성을 높이긴 했지만, 코드가 직관적이지 않고 유지보수가 어렵다. 더욱 일반적인 방식이 필요하다.

### 개념: 플래그 인수(flag argument)의 문제
이 방식에서는 `flag` 변수를 사용하여 조건을 변경하는데, 이는 코드 가독성을 저하시킨다. 즉, **단일 메서드가 너무 많은 역할을 하게 되는 문제**가 발생한다.

---

## 2.2 동작 파라미터화

### 네 번째 시도 : 추상적 조건으로 필터링

각기 다른 조건을 함수 형태로 전달하면 더욱 유연한 코드가 된다. 이를 위해 **인터페이스**를 활용할 수 있다.

```java
interface ApplePredicate {
    boolean test(Apple apple);
}
```

이제 원하는 필터링 기준을 정의하는 클래스를 만들 수 있다.

```java
class GreenApplePredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return "green".equals(apple.getColor());
    }
}
```

이제 조건을 적용하는 필터링 메서드를 만들자.

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (p.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}
```

### 개념: 동작 파라미터화(Behavior Parameterization)
이 방식은 **동작 파라미터화(Behavior Parameterization)** 의 핵심 개념을 적용한 것이다. 즉, 메서드 내부의 동작을 직접 정의하지 않고, **외부에서 원하는 동작을 주입(Inject)하여 코드의 유연성을 높인다**.

이를 통해, 필터링 조건이 변경되더라도 새로운 메서드를 만들 필요 없이 새로운 조건을 정의하는 클래스를 추가하는 방식으로 유지보수가 가능해진다. 이후에는 Java 8의 **람다 표현식**을 활용하여 더욱 간결하게 개선할 수 있다.

---

## 정리
- **하드코딩 방식** → 요구사항 변경 시 코드 수정이 필요.
- **파라미터화된 방식** → 코드 중복을 줄이지만 특정 속성에 한정됨.
- **플래그 인수 사용** → 유연하지만 가독성이 떨어짐.
- **동작 파라미터화** → 인터페이스를 활용하여 코드 유연성을 극대화함.

이 개념은 이후 다양한 API 설계에도 적용될 수 있으며, Java 8의 **람다 표현식**과 결합하면 더욱 간결하고 강력한 코드 작성이 가능하다.

