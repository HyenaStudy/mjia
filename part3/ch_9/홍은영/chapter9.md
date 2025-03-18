# CHAPTER 9 리팩터링, 테스팅, 디버깅

## ✅ 리팩터링
### 1. 익명 클래스를 람다 표현식으로 리팩터링
- 익명 클래스는 코드를 장황하게 만들고 쉽게 에러를 일으킴   

**문제점**
1. this : 익명 클래스에서는 자신을 가리키지만 람다에서는 람다를 감싸는 클래스를 가리킴
2. 섀도 변수 : 익명 클래스는 감싸고 있는 클래스 변수를 가릴 수 있다
3. 익명 클래스를 람다로 바꿀 때 콘텍스트 오버로딩에 따른 모호함 초래
<br>

### 2. 람다 표현식을 메서드 참조로 리팩터링하기
- 메서드 참조의 메서드명으로 코드의 의도를 명확하게 알릴 수 있음
- 람다 표현식과 저수준 리듀싱 연산을 조합하는 것보다 정적 헬퍼 메서드나 내장 컬렉터를 사용하는 것이 좋다
<br>

### 3. 명령형 데이터 처리를 스트림으로 리팩터링하기
- 스트림 api는 데이터 처리 파이프라인의 의도를 명확히 보여줌
- 멀티코어 아키텍처 활용 가능
- 병렬화 가능
<br>

### 4. 람다로 객체지향 디자인 패턴 리팩터링하기
#### 1) 전략(strategy)
- 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법
- 직접 코드를 수정하지 않고 전략만 변경해서 유연하게 확장 가능
- 람다를 사용하면 코드조각(전략)을 캡슐화
<br>

#### 2) 템플릿 메서드(template method)
- 알고리즘의 구조를 유지하면서 하위클래스에서 구조변경없이 재정의하는 것
- 상속받지 않고 람다로 세부 동작 추가 가능
<br>

#### 3) 옵저버(observer)
- 이벤트가 발생했을 때 알림을 보내고 동작을 수행하는 것
- 옵저버가 상태를 가지거나 복잡하면 람다보다 기존 방식 추천
<br>

#### 4) 의무 체인(chain of responsibility)
- 한 객체가 작업 결과를 다른 객체로 전달하고 다른 객체도 작업 결과를 또 다른 객체로 전달하는 것
- UnaryOperator<String> 형식 인스턴스로 표현 가능
<br>

#### 5) 팩토리(factory)
- 인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 사용
- 람다식으로 사용할 때는 생성자를 메소드 참조처럼 접근
- 생성자로 여러 인수를 전달할 때는 람다식을 적용하기 어렵다
<br><br>

## ✅ 테스팅
### 1. 보이는 람다 표현식의 동작 테스팅
- 람다를 필드에 저장해서 메서드를 호출하는 것처럼 사용
<br>

### 2. 람다를 사용하는 메서드 동작에 집중
- 람다 표현식을 공개하지 않고 람다를 사용하는 메서드를 테스트해서 람다 표현식 검증
<br>

### 3. 복잡한 람다를 개별 메서드로 분할
- 람다를 메서드 참조로 바꿔서 일반 메서드 테스트하듯 람다 테스트 가능
<br>

### 4. 고차원 함수 테스팅
- 고차원 함수 : 함수를 인수로 받거나 반환하는 메서드
<br>

#### 1) 함수를 인수로 받을 때
-  동작이 다른 람다를 전달해서 원하는 동작이 수행되는지 테스트
```java
import java.util.function.Function;

public class LambdaTest {
    // 람다를 인수로 받음
    public static int applyFunction(Function<Integer, Integer> function, int value) {
        return function.apply(value);
    }

    public static void main(String[] args) {
        // 람다를 전달해서 테스트
        int result1 = applyFunction(x -> x * 2, 5);
        int result2 = applyFunction(x -> x + 10, 5);

        System.out.println(result1);
        System.out.println(result2);
    }
}

```
<br>

#### 2) 함수를 반환할 때
- 반환된 함수가 예상대로 동작하는지 테스트
```java
import java.util.function.Function;

public class LambdaTest2 {
    public static Function<Integer, Integer> createMultiplier(int factor) {
        return x -> x * factor; // 람다를 반환
    }

    public static void main(String[] args) {
        // 받환받은 람다 테스트
        Function<Integer, Integer> doubleFunction = createMultiplier(2);
        Function<Integer, Integer> tripleFunction = createMultiplier(3);

        System.out.println(doubleFunction.apply(5));
        System.out.println(tripleFunction.apply(5));
    }
}

```
<br>

## ✅ 디버깅
### 1. 스택 트레이스
- 람다 표현식과 관련된 스택 트레이스는 이해하기 어렵다
<br>

### 2. peek
- 스트림 파이프라인 연산 중간값을 확인할 때 사용
- 스트림의 각 요소를 소비한 **것처럼** 동작 실행. 실제 소비 X
- 확인한 요소를 다음 연산으로 그대로 전달