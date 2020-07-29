# Mordern Java in Action

## 1장 - 자바 8, 9, 10, 11 : 무슨 일이 일어나고 있는가?

stream을 사용해서 병렬성을 쉽게 사용할 수 있구나!

함수형으로 코드를 구현할 때 항상 궁금했던게 switch 케이스는 어떻게 처리하는지였다. 이걸 **(구조적) 패턴 매칭** 이라고하는구나! 나중에 주의깊게 봐야겠다. Phil Wadler 가 제안한 용어인 expression problem을 참고하자.

## 2장 - 동작 파라미터화 코드 전달하기

## 3장 - 람다 표현식

## 8장 - 컬렉션 API 개선

다양한 기능이 많은데 이걸 모두 기억할 수 있을까? 없다면 어떻게 잘 사용할 수 있을까?

---

`List.of(E e1, E e2, E e3)`
`List.of(E... elements)`

내부적으로 가변 인수 버전은 추가 배열을 할당한 뒤 리스트로 감싸기 때문에 배열 할당, 초기화, GC 비용이 든다. 때문에 위와 같이 고정된 매개변수도 제공한다.

그런데 Map에서 Entry를 입력받을 때에는 가변 인수로 구현된 Map.ofEntries 팩토리 메소드를 추천하고 있다.

위에서는 비용문제로 피한 패턴을 아래에서 추천한 이유는 뭘까?

---

Map의 foreach는 BiConsumer를 지원해 기존의 EntrySet으로 처리하던 로직을 더 쉽게 구현할 수 있다.

```java
// AS-IS
for(Map.Entry<...> entry: map.entrySet()) {...}

// TO-BE
map.forEach((key, value) -> ...)
```