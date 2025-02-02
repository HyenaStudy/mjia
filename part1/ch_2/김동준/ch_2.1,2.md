# 1. 동작의 파라미터화

스트림 API 도입의 취지 중 하나는 **동작의 파라미터화**다.\
말이 좀 어려울 수도 있는데, 각각의 일련 과정들에 대해 정리해보자.

*책의 예제 코드는 사과를 주제로 했는데, 이해하기 쉽게 사람으로 구현할 예정*

```java
public class Person {
    private final String name;
    private final int height;
    private final Gender gender;

    // ... + 생성자, 각 필드들의 getter
    // Gender는 열거형 타입
```

위와 같은 **사람** 객체가 있다. 사람 객체는 **이름**, **신장**, **키**라는 필드 특성을 지니고 있다.\
수많은 사람들이 있을 때, 이 사람들을 분류해서 처리하기 위한 기능이 필요하다.

```java
List<Person> people = List.of(
        new Person("김동준", 184, Gender.MALE),
        new Person("송아름", 200, Gender.FEMALE),
        new Person("채호연", 300, Gender.MALE),
        new Person("허예림", 400, Gender.FEMALE),
        new Person("홍은영", 500, Gender.FEMALE)
);
```

## 1) 데이터 처리, 즉 동작은 자바에서 메소드로 구현

위에서 언급한 **기능**은 곧 사람이라는 데이터를 처리해야 하고 그것은 자바에서 메소드로 구현된다. 예를 들어서 아래와 같은 요구사항이 있다고 치자.

> 1. **키가 2m 50cm 초과인 사람들을 찾아야 함**

별 생각 없이 위의 기능을 메소드로 구현하고자 하면 이렇게 메소드로 구현할 수 있을 것이다.\
또한 나름의 확장성을 위해 키의 기준을 다양하게 설정할 수 있도록 `height`로 파라미터화했다.

```java
public List<Person> filterPerson(List<Person> people, int height) {
    List<Person> result = new ArrayList<>();

    for (Person person: people) {
        if (person.getHeight() > height) result.add(person);
    }

    return result;
}
```

여기까지는 꽤나 심플하고 무난하게 생각할 법하다.

## 2) 같은 데이터에 대하여 다양한 동작이 요구될 가능성 존재

하지만 분류 기준이 추가되는 요구사항이 들어오면 어떨까? 아래와 같은 요구사항이 추가됐다.

> 1. **키가 2m 50cm 초과인 사람들을 찾아야 함**
> 2. **성별이 여자인 사람들을 찾아야 함**

이미 코드는 작성돼서 잘 돌아가고 있는 와중에 추가 요구사항이 들어오면서 리팩토링이 필요해졌다.

### (1) 메소드 추가하기?

조건이 2개나 요구되면서 일단 기존에 작성했던 메소드 구조에서 착안을 얻어 또 다른 기능을 구현한다.

```java
// 키를 기준으로 분류하는 메소드
public List<Person> filterPersonByHeight(List<Person> people, int height) {
    List<Person> result = new ArrayList<>();

    for (Person person: people) {
        if (person.getHeight() > height) result.add(person);
    }

    return result;
}

// 성별을 기준으로 분류하는 메소드
public List<Person> filterPersonByGender(List<Person> people, Gender gender) {
    List<Person> result = new ArrayList<>();

    for (Person person: people) {
        if (person.getGender().equals(gender)) result.add(person);
    }

    return result;
}
```

이 코드의 문제는 **유사한 로직 구조의 중복**이다. 소프트웨어 공학의 DRY 원칙을 어기는 코드다.\
리스트 타입의 `people` 파라미터는 어차피 제공되는 단위 데이터는 동일할 것이기 때문에 처리 부분의 조건만 달라졌음에도 파라미터, 반복문, 반환의 중복 처리가 이뤄지고 있다. 이 상황에서 만약 또 다른 요구사항이 들어오면 유사한 로직 구조의 메소드가 거기에 비례해서 증가하고 관리가 어려워질 것이다.

### (2) 메소드 수정하기?

그래서 아예 하나의 메소드 내에서 구현하는 것으로 결정했다. 물론 상황마다 메소드 요구사항이 다를 수 있다는 것을 고려해야 한다.\
무슨 말이냐면 어떤 서비스에서는 성별 분류를 요구할 수도 있고, 어떤 서비스에서는 신장 분류를 요구할 수 있기 때문에 이것을 반영해야 한다는 뜻이다.

일단은 분류 기준이 신장과 성별만 있으니, 이걸 `flag`라는 논리 타입으로 구별해서 메소드 동작을 수정해보자.

```java
// flag = true : 성별 분류, false : 신장 분류
public List<Person> filterPerson(List<Person> people, int height, Gender gender, boolean flag) {
    List<Person> result = new ArrayList<>();

    for (Person person: people) {
        if ((flag && person.getGender().equals(gender)) || (!flag && person.getHeight() > height)) result.add(person);
    }

    return result;
}
```

일단 메소드 관리는 하나의 메소드 내에서 이뤄지므로 관리는 편하지만, 문제는 기능 하나만 추가했는데도 파라미터가 급증하고 심지어 내부 로직도 상당히 보기 불편해졌다.\
지금이야 분류 기준이 **신장**과 **성별** 뿐이지만, 만약 여기서 **이름** 문자열을 갖고 또 분류하는 기능 요구사항이 추가된다면 그 때는 `flag` 파라미터로도 부족해진다. 뭐 굳이 여기서 더 생각하자면 `int` 타입 파라미터 등을 활용해 어떤 기능 요구사항인지를 명시하는 방법도 있겠지만... 상상만 해도 끔찍하다. 거기에 덤으로 내부 로직도 더 복잡해지게 될 것이다.

심지어 사용하는 예시를 보면 아래와 같다.

```java
// 키가 250 넘는 사람들 분류
filterPerson(people, 250, null, false);

// 성별이 여자인 사람들 분류
filterPerson(people, 0, Gender.FEMALE, true);
```

벌써부터 코드 독해에 머리가 아파지는 구조다.

## 3) 문제 해결 과정

객체지향 프로그래밍의 원칙을 기반으로 생각하는 과정인데 사담이 섞여서 틀릴 수도 있음

### (1) 코드가 상당히 정적이고 확장성이 떨어진다 - OCP 위반

코드는 원칙적으로 **확장에는 열려있되 변경에는 닫혀 있어야 한다**는 OCP 원칙을 준수해야 한다. 다만 현재의 문제 상황에서 제시한 두 개의 해결책들은 코드를 중복 추가(메소드 분리)하거나 기존의 코드를 수정(메소드 통합)해야 한다. 즉, 기존 코드의 변경을 요구하면서 확장성이 떨어지는 구조라는 것이다.

### (2) 객체와 기능이 결합되어 있다

객체, 즉 데이터에 대응되는 `Person` 인스턴스와 동작, 즉 기능에 대응되는 `무슨무슨 사람 찾기`가 메소드 내부에서 결합되어 있다. 코드를 다시 봐보면

```java
// 메소드 분리
for (Person person: people) {
    if (person.getHeight() > height) result.add(person); // 데이터(person)와 동작이 결합
}

for (Person person: people) {
    if (person.getGender().equals(gender)) result.add(person); // 데이터(person)와 동작이 결합
}

// 메소드 결합
for (Person person: people) {
    // 데이터(person)와 동작이 결합
    if ((flag && person.getGender().equals(gender)) || (!flag && person.getHeight() > height)) result.add(person);
}
```

반복문 내부의 조건 분기에서 필터링하는 코드 라인에 `Person` 인스턴스 데이터와 필터링 기능이 결합되어 있다. 결합됐다는 말은 **하드코딩**되어있다는 말과 일맥상통한다. 즉, 데이터를 필터링하기 위해 데이터 내부에 보유하고 있는 필드 값들을 직접 비교하면서 동작을 검증하는 로직이 결합되어 있기 때문에 분리가 어려운 구조를 따라가게 되면서 유연성이 떨어지고 중복 등의 비효율성이 증가하게 된다.

### (3) 문제 원인 정리 및 해결책 파악

위의 두 과정을 통해 현재 코드의 핵심 문제는 **데이터 내부로 직접 들어가서 조건을 검증**하면서 결국 객체와 기능이 결합하게 되는 것이 주요 원인임을 알아냈다. 그렇다면 해결책은 이 객체와 기능을 분리하는 것이 될 것이다. 정리하자면

>1. 현재 코드는, **객체는 멈춰있고 로직이 직접 객체 속에 들어가 조건을 검증하고 나오고를 반복**하는 형태다.
>2. 해결 방향은, **객체가 흘러가게 하면서 검증 로직을 거쳐 자동으로 분리**하는 형태를 추구한다. 즉 기존 로직의 흐름을 역전시킨다.

이렇게 될 것인데 왜 1번 방향처럼 코드가 작성됐는지 다시 살펴보면 외부에서 개입되는 기능 관련 요건, 즉 **파라미터가 단순히 기준값으로만 주어지고 검증을 메소드 내부로 책임을 위임**한 것이 원인이다. 즉 데이터가 흘러가는 과정을 **컨베이어 벨트**로 비유할 때, 검증하고자 하는 기능을 마치 **검출기**처럼 취급하되, 컨베이어 벨트는 건드리지 않고 우리가 원하는 검출기를 컨베이어 벨트 위에 교체하듯 얹는 방식으로 코드 리팩토링을 생각해봐야 한다.

<img src="https://github.com/user-attachments/assets/65249927-5182-4088-a535-26e60a527f91" width="60%" />

약간 이런 느낌?

## 4) 다양한 동작을 전부 캡슐화하기 위한 인터페이스, 익명 구현 객체

## 5) 이것조차 간편하게 할 수 있는 스트림, 람다
