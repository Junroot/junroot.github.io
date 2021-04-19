---
title:  "Defensive Copy (방어적 복사)"
categories: programming
tags: Java OOP
---

## 방어적 복사란?

Java는 메서드나 생성자의 매개변수가 클래스인 경우, 기본적으로 얕은 복사가 이루어진다. 이로인해, 클래스를 만들 때 한 가지 문제가 발생한다.

```java
public class Cars {
    List<String> names;

    public Cars (List<String> names) {
        this.names = names;
    }

    public List<String> getNames() {
        return names;
    }
}
```

위와 같은 Cars라는 클래스가 있다고 가정한다.

```java
List<String> names = new ArrayList<>(Arrays.asList("a", "b"));
Cars cars = new Cars(names);
names.add("c");
System.out.println(cars.getNames().size());
```

위의 코드를 실행하면 `cars`를 생성한 후에 `names`에 추가했기때문에 `2`라고 예측할 수도 있지만, 결과는 `3`이다. 이와같이 의도치않게, 클래스 밖에서 클래스 안의 필드를 변경하게 되는 문제가 발생할 수 있다. 이 문제때문에 나온 개념이 방어적 복사이다.

## 방어적 복사 방법

### 생성자

아래와 같이, 생성자 내에서 리스트를 새로 생성하여 인스턴스 변수에 초기화한다. 이렇게되면 매개변수와 인스턴스 변수가 참조하고 있는 것이 달라지기 떄문에 문제가 발생하지 않는다.

```java
public Cars (List<String> names) {
    this.names = new ArrayList<>(names);
}
```

### getter 메서드

getter 메서드에도 반환된 인스턴스 변수가 의도치않게 변형될 수 있다. 이를 막기 위해 getter 메서드를 아래와 같이 수정한다.

```java
public List<String> getNames() {
    return new ArrayList<>(names);
}
```

### unmodifiable... 메서드의 문제점

방어적 복사로 `unmodifiable...`종류의 메서드를 사용하는 경우가 종종 보인다. 하지만 이는 멀티 스레드 환경에서는 문제가 발생한다. 이 메서드로 구현된 getter 메서드로 값을 받은 후, 다른 스레드에서 원본의 값을 수정하게 되면 값이 같이 변하게된다는 문제가 있다. 따라서, `unmodifiable...` 메서드를 사용하는 방법은 가능하면 피하는 것이 좋다.

## 관련 사이트

<https://johngrib.github.io/wiki/defensive-copy.md/#방어-기법-불변-객체의-사용>
