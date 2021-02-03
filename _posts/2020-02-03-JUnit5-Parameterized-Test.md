---
title:  "JUnit5 Parameterized Test"
categories: programming
tags: JUnit
classes: wide
---

## Parameterized Test란?

테스트 코드 작성시 각 다른 매개변수들로 한 테스트 메소드를 여러 번 실행해야되는 상황이 발생할 수 있다. 이 때 중복 코드를 발생시키지 않고 여러번 테스트를 구현가능 하도록해주는 것이 Parameterized Test 이다.

## 시작하기

기존의 테스트 코드에서 `@ParameterizedTest` annotation을 추가하면 사용할 수 있다.

예시:

```java
@ParameterizedTest
@ValueSource(ints = {1, 3, 5, -3, 15, Integer.MAX_VALUE}) // six numbers
void isOdd_ShouldReturnTrueForOddNumbers(int number) {
    assertTrue(Numbers.isOdd(number));
}
```

## 인자 넘기기

테스트 케이스로 사용할 인자를 넘기는 방법에는 여러가지 방법이 있다.

### @ValueSource

간단한 값들을 테스트 메소드에 넘길 때 사용할 수 있다.

예시:

```java
@ParameterizedTest
@ValueSource(strings = {"", "  "})
void isBlank_ShouldReturnTrueForNullOrBlankStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

매개 변수로 사용 아래의 자료형만 사용가능하다.

- *short* (with the *`shorts`* attribute)
- *byte* (with the *`bytes`* attribute)
- *int* (with the *`ints`* attribute)
- *long* (with the *`longs`* attribute)
- *float* (with the *`floats`* attribute)
- *double* (with the *`doubles`* attribute)
- *char* (with the *`chars`* attribute)
- *java.lang.String* (with the *`strings`* attribute)
- *java.lang.Class* (with the *`classes`* attribute)

### @CsvSource

입력 값과 예상하는기대 값을 모두 매개 변수로 사용해야되는 경우 CSV파일 형식으로 사용할 수도 있다.

예시:

```java
@ParameterizedTest
@CsvSource({"test,TEST", "tEst,TEST", "Java,JAVA"})
void toUpperCase_ShouldGenerateTheExpectedUppercaseValue(String input, String expected) {
    String actualValue = input.toUpperCase();
    assertEquals(expected, actualValue);
}
```

## 관련 사이트

<https://www.baeldung.com/parameterized-tests-junit-5>