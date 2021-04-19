---
title:  "Spring Framework 개요"
categories: programming
tags: Java Spring
---

## Spring Framework란?

Java 엔터프라이즈 애플리케이션을 개발할 수 있는 프레임워크다. JVM에서 대체언어로 Groovy 또는 Kotlin을 지원한다. 주로 웹서비스 개발 시 사용되고 있지만, 이 외에도 다양한 애플리케이션 개발에서 사용할 수 있다.

## Spring Boot

Spring Framework를 이용한 애플리케이션 개발에는 많은 설정이 필요하다. Spring Boot를 사용하면 최소한의 기능으로 프로젝트를 설정할 수 있도록 도와준다. Spring Framework를 처음 사용해본다면 Spring Boot를 사용하여 빠르게 시작해볼수 있다.

## IoC 컨테이너

Spring Framework는 IoC(Inversion of Control) 컨테이너다. 이름 그대로 개발자가 아닌 프레임워크에서 흐름을 제어한다. 이 프레임워크는 IoC를 위해 DI(Dependency Inejection)을 이용한다. 즉, 생성자나 팩터리 메서드의 인자, setter를 통해서만 의존성을 정의한다. 그러면 컨테이너가 빈(Bean)을 생성할 때 의존성을 주입한다. 이 과정은 서비스 로케이터 패턴같은 매커니즘이나 클래스의 생성자를 이용한다.

- 컨테이너: 빈을 인스턴스화, 구성, 조립을 담당한다. 컨테이너는 구성 메타 데이터를 읽어 인스턴스화, 구성, 조립에 대한 지침 얻는다. 구성 메타 데이터는 XML, Java 어노테이션, Java 코드로 표현이 가능하다.
- 빈: 애플리케이션의 핵심을 이루는 객체이며, 컨테이너에의해 인스턴스화, 관리, 생성된다. 빈은 컨테이너에서 제공하는 구성 메타 데이터에 의해 생성된다. 컨테이너 내에서는 `BeanDefinition` 객체로 표현되며, 아래의 메타 데이터를 포함하고 있다.
    - 패키지 규정 클래스 이름
    - 빈이 컨테이너에서 작동해야하는 방식을 나타내는 Bean 작동 구성 요소 (스코프, 라이프 사이클 등)
    - 빈이 작업을 수행하는 데 필요한 다른 빈의 참조. 이를 의존성이라 부른다.
    - 새로 생성된 객체에 설정할 기타 구성 설정 (Connection Pool의 zmrl, 커넥션의 개수 제한 등)

## 출처

[https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview)

[https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)