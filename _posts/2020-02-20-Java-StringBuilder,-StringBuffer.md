---
title:  "Java StringBuilder, StringBuffer"
categories: programming
tags: Java
classes: wide
---

Java 코딩 시 출력과 관련된 부분에서 문자열을 합쳐서 출력해야 되는 경우가 많다. 하지만 이 때 문자열 덧셈으로 처리할 경우 성능이 떨어진다. 메모리 할당과 해제가 발생하기 때문이다.

```java
String a = "aa";
String b = "bb";

System.out.println(a.hashCode());
System.out.println((a + b).hashCode());
```

위의 코드를 실행하면 다음과 같은 결과가 나온다. 다음과 같이 새로운 객체를 생성한다.

```bash
3104
2986080
```

`StringBuilder`와 `StringBuffer`는 문자열을 연결할 떄도 객체로 새로 생성하지 않고 기존 객체에 연결한다. 그럼 이 두 클래스는 어떤 차이가 있을까. `StringBuilder`는 동기 처리가 되지 않아 멀티 스레드 환경에서 사용하면 안되지만, `StringBuffer`는 thread-safe 하다.

```java
String a = "aa";
String b = "bb";

StringBuilder stringBuilder = new StringBuilder(a);

System.out.println(stringBuilder.hashCode());
System.out.println(stringBuilder.append(b).hashCode());
```

실행 결과는 아래와 같다.

```java
2049926666
2049926666
```

## 참고

- <https://novemberde.github.io/2017/04/15/String_0.html>

- Effective Java 3rd Edition: Item 63 문자열 연결은 느리니 주의하라