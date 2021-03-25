---
title:  "Static Factory Method (정적 팩토리 메서드)"
categories: programming
tags: Java OOP
classes: wide
---

## Static Factory Method란?

기본적으로 Java에서 객체를 생성하기 위해서는 `new`라는 키워드를 사용한다. 이를 사용하지않고, 객체를 생성하는 정적 메서드를 따로 만드는 기법을 Static Factory Method라고한다.

자바에서 흔히 사용하는 `valueOf`메서드가 대표적인 에이다.

```java
public static Car valueOf(final String name) {
    return new Car(name);
}
```

## 장점

### 메서드에 이름이 있어, 가독성이 높아진다

메서드에 이름이 존재하기 때문에 현재 클래스의 역할이 무엇인지 쉽게 이해할 수 있다.

아래는 "Effective Java"의 "Joshua Bloch"가 제안한 Static Factory Method의 네이밍 컨벤션이다.

- `from`: 하나의 매개 변수를 받아 해당 타입의 인스턴스를 반환한다.
- `of`: 여러 개의 매개 변수를 받아 적합한 타입의 인스턴스를 반환한다.
- `valueOf`: `from`과 `of`의 더 자세한 버전.
- `getInstance`: 매게 변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다. 기존에 생성했던 인스턴스를 다시 반환할 수도 있다.
- `newInstance`: `getInstance`와 비슷하지만 기존에 생성했던 인스턴스와 다른 인스턴스라는 것을 보장해준다.
- `get"Type"`: `getInstance`와 유사하지만 메소드가 다른 클래스에 있을 때 사용된다. `"Type"`에는 반환될 객체의 자료형이 들어간다.
- `new"Type"`: `newInstance`와 유사하지만 메소드가 다른 클래스에 있을 때 사용된다. `"Type"`에는 반환될 객체의 자료형이 들어간다.
- `"type"`: `get"Type"`과 `new"Type"`의 간결한 버전

### 캐시를 사용할 수 있다

자주 사용되는 인스턴스의 경우에 클래스내에 캐시를 만들어 매번 `new`와 같은 비싼 연산을 할 필요 없게 할 수 있다. 아래는 그의 예시다.

`java.lang.Integer.valueOf` 메소드는 아래와 같이 작성되어있다. `Integer` 클래스 안에 `IntegerCache`라는 캐시를 만들어두고, 캐시에 있는 값이면 새로 인스턴스를 만들지 않고 캐시에 있는 인스턴스를 반환한다.

```java
@HotSpotIntrinsicCandidate
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}

```

### 다양한 서브 클래스를 반환할 수 있다

매개 변수의 값에 따라 다른 서브 클래스를 반환 해야될 때, 클라이언트가 이런 관계를 이해할 필요없이 사용할 수 있는 코드를 작성할 수 있다.

```java
public class UserFactory {

    public static User newUser(UserEnum type){
        switch (type){
            case ADMIN: return new Admin();
            case STAFF: return new StaffMember();
            case CLIENT: return new Client();
            default:
                throw new IllegalArgumentException("Unsupported user. You input: " + type);
        } 
    }
}
```

## 관련 사이트

<https://johngrib.github.io/wiki/static-factory-method-pattern/>

<https://velog.io/@ljinsk3/정적-팩토리-메서드는-왜-사용할까>
