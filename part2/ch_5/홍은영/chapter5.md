# CHAPTER 5 스트림 활용

### 1. 필터링
- `filter(Predicate<T>)`  
조건을 만족하는 요소만 포함하는 스트림 반환

- `distinct()`  
중복 제거 후 고유 요소만 남겨서 반환
<br>

### 2. 슬라이싱
- `takeWhile(Predicate<T>)`  
정렬된 스트림에서 조건을 만족하는 동안 요소를 가져오고, 불만족하는 순간 조회 종료

- `dropWhile(Predicate<T>)`  
정렬된 스트림에서 조건을 만족하는 요소를 건너뛰고, 불만족하는 요소부터 반환

- `limit(n)`  
앞에서부터 n개 요소 반환

- `skip(n)`  
처음 n개 요소를 제외한 나머지 반환
<br>

### 3. 매핑
- `map(Function<T, R>)`  
스트림 각 요소에 함수를 적용해 새로운 요소로 변환
```java
List<String> words = Arrays.asList("Hello", "modern", "in", "java");
List<Integer> wordLengths = words.stream()
    .map(String::length)
    .collect(toList());	 // [5, 6, 2, 4]
```
<br>

- `flatMap(Function<T, Stream<R>>)`  
여러 개의 스트림을 하나의 평면화된 스트림으로 변환
<img src="https://github.com/user-attachments/assets/1a81bc28-b07c-44cc-9ab1-b039d4c112d0" width="500">
<br><br><br>

### 4. 검색과 매칭
- `anyMatch`
적어도 하나의 요소가 조건을 만족하는지 확인(OR 연산)

- `allMatch`
모든 요소가 조건을 만족하는지 확인(AND 연산)

- `noneMatch`
모든 요소가 조건을 만족하지 않는지 확인(전부 거짓이면 true)
<br>

> ✅**쇼트서킷 기법**
> - 논리 연산자(`&&`, `||`)를 활용해 전체를 처리하지 않아도 결과 반환 가능
> - ex) AND 연산에서 하나라도 거짓이면 나머지는 검사하지 않고 거짓 반환
<br>

- `findAny`
순차 스트림에서는 사실상 첫번째 요소 반환(보장 X). 병렬 스트림에서는 어떤 요소가 반환될지 모름

- `findFirst`
첫 번째 요소 반환.
