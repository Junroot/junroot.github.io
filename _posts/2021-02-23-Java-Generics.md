---
title:  "Java Generics"
categories: programming
tags: Java
---

## Java Generics(제네릭)이란?

Java에서 클래스나 메서드에 사용할 내부 데이터의 타입을 외부에서 지정할 수 있도록하는 기멉이다. 여러가지 타입을 받을 수 있는 클래스나 메서드를 만들어야 될 때, 특정 타입으로 제한함으로써 타입 안정성을 제공한다. 타입에 대한 검증이 컴파일 시에 확인이 되기 떄문이다. 또한 타입 체크와 형변환을 생략할 수 있으므로 코드가 간결해진다.

## 제네릭의 사용

아래와 같이 제네릭을 사용한 메서드와 클래스를 정의할 수 있다.

클래스:

```java
public class Box<T> {
    List<T> items = new ArrayList();

    public void add(T item) {
        items.add(item);
    }
}
```

메서드:

```java
public <T> T foo(List<T> list {});
```

만약 이 `Box`라는 클래스를 제네릭이 아닌 `List<Object>`로 관리한다면 어떻게 될까? 이 리스트에는 어떤 객체든 담을 수 있게되고, 개발자가 요소들을 사용할 때 타입에 대한 검증을 직접 해줘야된다. 실수로 검증을 제대로 못하면 런타인 에러가 발생할 것이다.

## 멀티 타입 파라미터

`class Box<K,V>` 와 같은 형식으로 복수개의 파라미터를 받을 수도 있다.

## 타입 매개변수의 제한

제네릭으로 담을 수 있는 타입을 제한시키고 싶을 때가 있을 것이다.

```java
public class Box<T extends Fruit> {
    List<T> items = new ArrayList();

    public void add(T item) {
        items.add(item);
    }
}
```

이렇게되면 `Box`에는 과일을 상속받는 객체만 담을 수 있게된다.

## 와일드카드

타입 파라미터로 와일드카드를 표현하는 `<?>`를 받으면 어떤 파라미터를 받을 수 있다는 뜻이 된다. `<T>`를 사용하는 것과의 차이점은 타입 파라미터의 참조 가능 여부다. `List<?>`의 경우는 와일드 카드를 사용한 경우 모든 요소는 `Object`타입으로 받을 수 있다. 또한 List에 어떤 타입이 올지알 수 없기때문에 `null`만 삽입할 수 있다.

- 상한 경계 와일드 카드: `<? extends T>`

T를 상속받은 클래스만 파라미터로 받을 수 있도록 제한한다. 이 경우에도 마찬가지로 `List<? extends T>`의 경우 모든 요소를 `Object`로 받을 수 있고, `null`만 삽입할 수 있다. `List<Dog>`를 파라미터로 받았는데, `Cat`을 삽입하는 상황을 막기 위해서다.

- 하한 경계 와일드카드: `<? super T>`

T가 상속중인, 즉 T의 상위 클래스를 파라미터로 받을 수 있도록 제한한다. 이 경우에는 `List<? extends T>`의 경우 모든 요소를 `Object`로 받을 수 있고, T또는 T의 하위 클래스를 삽입할 수 있다.

## 참고 자료

<https://www.youtube.com/watch?v=n28M8iryFPw>