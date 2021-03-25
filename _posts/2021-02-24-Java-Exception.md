---
title:  "Java Exception"
categories: programming
tags: Java
classes: wide
---

## Exception의 상속 구조

![exception-hierarchy](/assets/images/exception-hierarchy.png)

Exception은 위 그림과 같은 상속 구조를 가지고 있다. 최고의 조상은 `Exception`클래스이며, 모든 예외처리는 이 클래스를 상속받고 있다.

## Exception의 분류

`Exception`은 크게 두 개의 부류로 나눌 수 있다. 

- `RuntimeException`과 그의 자손 클래스

Unchecked Exception이라고도 부르며, 컴파일 시점에 Exception을 catch하는지 확인하지 않는다. 컴파일 시점에 Exception이 발생할 것인지의 여부를 판단할 수 없다. 또한 Exception이 발생하는 메소드에 `throws`를 통해 처리할 필요가 엇다. 하지만 처리해도 무방하다.

- `Exception`과 그의 자손 클래스

Checked Exception 또는 Compile Time Exception 이라고 한다. 컴파일 시점에 Exception을 catch하는지 확인한다. 컴파일 시점에 Exception에 대한 처리(`try-catch`)를 하지 않을 경우 컴파일 에러가 발생한다. 또는 Exception이 발생하는 메소드에서 `throws`를 활용해 Exception을 호출메소드에 전달할 수 있다.

## Checked Exception과 Unchecked Exception의 선택 방법

기본적으로 치명적인 예외인 경우 Checked Exception을 사용한다. 또한 호출하는 메소드가 Exception을 통해 무엇인가 의미 있는 작업을 할 수 있다면 Checked Exception을 사용한다. 만약 호출하는 메소드가 Exception을 catch해 예외 상황을 해결하거나 문제를 해결할 수 없다면 Unchecked Exception을 사용한다. 둘 다 명확하지 않는 상황이라면 Unchecked Exception을 사용하도록 한다.

## Exceptoin 처리

- 메소드에서 여러 개의 Exception을 throw하는 경우 아래와 같이 `,`로 구분할 수 있다. 또는 부모 클래스를 throw하여 부모 클래스를 포함한 전체 Exception을 throw할 수 있다.

```jsx
public void send() throws ObjectStreamException, UnknownHostException {
}
```

```jsx
public void send() throws Exception {
}
```

- 여러 개의 Exception을 catch할 때 아래의 방법들을 사용할 수 있다.

```jsx
try {
		new Student("A");
		fail("이름이 형식에 맞지 않아 Exception이 발생해야 한다.");
} catch (StudentNameFormatException e) {
		assertEquals("A 이름은 형식에 맞지 않습니다.", e.getMessage());
} catch (NoSuchElementException e) {

}
```

또는

```jsx
try {
		new Student("A");
		fail("이름이 형식에 맞지 않아 Exception이 발생해야 한다.");
} catch (StudentNameFormatException e) {
		assertEquals("A 이름은 형식에 맞지 않습니다.", e.getMessage());
} catch (NoSuchElementException e) {

}
```

또는

```jsx
try {
		new Student("A");
    fail("이름이 형식에 맞지 않아 Exception이 발생해야 한다.");
} catch (Exception e) {
    assertEquals("A 이름은 형식에 맞지 않습니다.", e.getMessage());
}
```

- 예외 다시 전달하기

catch안에 다시 Exception을 throw할 수 있다.

```jsx
try {
		new Position("a");
} catch (InvalidPositionException e) {
    throw e;
}
```

- Stack Trace

예외가 발생할 경우 예외가 발생한 원인을 찾기 위해 예외의 발생 경로를 추적하는 것이 가능하다.

```jsx
try {
		new Position("a");
    fail("Position 인자가 형식에 맞지 않아 Exception이 발생해야 한다.");
} catch (InvalidPositionException e) {
    e.printStackTrace();
}
```

- finally

예외발생 유무와 관계없이 항상 실행해야되는 코드가 있을 수 있다. 예를 들어 파일을 open하면 반드시 close 해야된다.

```jsx
try {
    // Exception이 발생하는 코드
} catch ( NullPointerException e ) {
    // NullPointerException이 발생하는 경우에 대한 처리
} finally {
    // 무조건 실행
}
```

## 표준 예외

자주 사용하는 표준 예외인 `IllegalArgumentException`과 `IllegalStateException`을 소개한다.

- `IllegalArgumentException`: 부적절한 인자를 메소드에게 건네준 것을 나타내는 Exception

- `IllegalSteaException`: 부적절한 상황에 메소드가 호출된 경우를 나타내는 Exception

즉 `IllegalArgumentException`는 메소드를 호출했을 때의 인자 값에 대한 예외, `IllegalSteaException`는 메소드 호출 타이밍에 대한 예외를 나타낸다.

자바로 개발할 때, 커스텀 예외를 만들 것인지 표준 예외를 활용할 것인지에대한 고민을 해볼 수도 있다. 그에 대한 정보는 이 [링크](https://woowacourse.github.io/javable/post/2020-08-17-custom-exception/)를 확인하면 좋다.

## 참고 자료

[https://5balloons.info/introduction-to-exception-handling/](https://5balloons.info/introduction-to-exception-handling/)

<http://cris.joongbu.ac.kr/course/java/api/java/lang/IllegalArgumentException.html>

우아한테크코스 수업 내용