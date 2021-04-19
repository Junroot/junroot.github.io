---
title:  "Utility Class"
categories: programming
tags: Java OOP
---

## Utility Class란?

프로젝트 진행시 여러 클래스에서 공통적으로 사용되는 메서드가 발생할 수 있다. 이 때, 관련된 메서드들끼리 모아서 클래스로 만드는 경우에 중복된 코드가 발생하지 않고 효율적으로 관리할 수 있다. 이를 Utility Class(또는 Helper Class)라고 한다.

## Utility Class의 구조

기본적으로 Utility Class의 모든 메서드들은 static이다. 또한 이 클래스는 인스턴스화 될 필요가 없으므로, 이를 방지하기위해 기본 생성자를 private으로 만들기도한다. 또한 상속을 방지하기위해 클래스를 final로 선언한다.

```java
import java.util.Calendar;
import java.util.Date;

public final class DateUtils {
    
    private DateUtils() {
    }

    public static Date createDate(int year, int month, int date) {
        Calendar calendar = Calendar.getInstance();
        calendar.clear();
        calendar.set(year, month - 1, date);
        return calendar.getTime();
    }
}
```

## Utility Class의 문제점

Utility Class는 근본적으로 객체지향적인 프로그래밍 기법이 아니다. 기능분할에 익숙해져있는 절차적 프로그래밍에 가깝다. 또한, Utility Class는 다른 클래스로부터 종속성이 너무 커지며, 시간이 지남에따라, 기능을 추가하면서 Utility Class의 코드가 커지면서 단일 책임 원칙을 위반하기 될것이다.

## 관련 사이트

<https://www.vojtechruzicka.com/avoid-utility-classes/>

<https://www.mimul.com/blog/oop-alternative-to-utility-classes/>
