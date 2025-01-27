# Java 8 취지

## 1. 병렬성의 극대화
- 기존의 자바(자바 8 이전)은 기능, 성능의 병렬 처리를 멀티스레딩 수작업을 통해 수행
- 데이터 분할, 스레드 생성 및 작업 분배의 책임이 개발자에게만 위임
- 코드 레벨에서의 멀티스레드를 활용한 병렬 처리의 극대화가 필요
- 기존의 코드 레벨 병렬 처리의 책임을 개발자로부터 포크조인 프레임워크(parellelstream)와 스트림(stream)에게 넘김

## 2. 함수형 프로그래밍의 도입

### 1) 불변성, 고차함수

```js
function App() {

  // 불변성 : count 변수는 변하지 않음
  const [count, setCount] = useState(0)

  // 함수 정의
  const increase = (value) => value + 1

  // 함수를 인자로 전달받는 고차함수
  const onClick = () => setCount(increase(count))
  
  // 결과
  return <button onClick={onClick}>{count}</button>
}
```
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

int sum = numbers
        .stream() // 불변성 : numbers 변수는 변하지 않고 스트림으로 변환
        .filter(number -> number % 2 == 0) // '짝수 검증 함수'를 고차함수의 인자로 전달
        .mapToInt(number -> number * 2) // '두 배 증가 함수'를 고차함수의 인자로 전달
        .sum();

System.out.println(sum); // 결과
```

비 함수형 프로그래밍에서는 '무엇을' + '어떻게' 전부 개발자가 책임\
함수형 프로그래밍에서는 '무엇을' 까지만 개발자가 책임을, '어떻게'는 라이브러리(스트림 api)가 책임

다만 함수 인자 전달은 함수형 프로그래밍의 수많은 특징 중 하나일뿐

### 2) 람다
- 고차함수의 핵심이자 다형성 실현 수단

>1. 데이터와 데이터 처리 공간이 있다.
>2. 데이터 처리 수단을 마련해야 한다.
>3. 외부에서 함수를 제공한다.
>4. 데이터를 처리한다.
>5. 데이터를 다르게 처리하고 싶어졌다.
>6. 함수만 바꾸면 된다.

### 3) 모듈
- 단일 책임 원칙 강화

### 4) 디폴트 메소드
- 인터페이스의 재사용성 강화, 메소드 체이닝 활용성 향상
