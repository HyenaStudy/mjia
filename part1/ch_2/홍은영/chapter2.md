# CHAPTER 2 동작 파라미터화 코드 전달하기

### 동작 파라미터화
- 어떻게 실행할지 결정하지 않은 코드 블록
- 자주 바뀌는 요구사항에 효과적으로 대응가능
<br><br>

### 예제. 사과 필터링하기
#### 1) 첫 번째 과정
```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
  List<Apple> result = new ArrayList<>();
  for(Apple apple : inventory) {
    if(GREEN.equals(apple.getColor()){
      result.add(apple);
    }
  }
  return result;
}
```
필터링 할 사과 색을 추가하려면 메서드 복붙해서 if문 조건만 변경<br>
-> 비슷한 코드가 반복 존재하면 그 코드 추상화
<br><br>

#### 2) 두 번째 과정
- 메서드에 파라미터 추가해서 색을 파라미터화 하기
```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, **Color color**) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if (apple.getColor().equals(color)) {
      result.add(apple);
    }
  }
  return result;
}
```

- 무게 별 분류 추가
```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, **int weight**) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if (**apple.getWeight() > weight**) {
      result.add(apple);
    }
  }
  return result;
}
```
-> 무게 관련 코드를 제외하면 색깔 필터링 코드와 중복되는 부분이 많음<br>
-> DRY(Don't Repeat Yourself) 원칙에서 어긋남<br>
<br><br>

#### 3) 세 번째 과정
- 하나의 메서드에서 flag로 색과 무게 구분하기
```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    // true는 색깔, false는 무게
    if ((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) {
      result.add(apple);
    }
  }
  return result;
}
```
-> 크기, 모양, 출하지 등 분류체계가 추가되면 메서드가 거대하고 복잡해짐