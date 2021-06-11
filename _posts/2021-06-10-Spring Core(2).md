---
title:  "Spring Core(2)"
categories: programming
tags: Spring
---

[Spring Test](https://junroot.github.io/programming/Spring-Test/) 내용을 정리하면서 아직 Spring에 관련된 개념이 정리가 되지 않았다는 것을 꺠달았다. 이를 한번 정리하기 위해 Spring Core(2)를 작성하게 되었다.

## Spring IoC 컨테이너 및 Bean 소개

스프링에서는 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트를 Bean이라고 부른다. 그리고 스프링에서는 빈의 생성과 관계설정 같은 제어를 담당하는 IoC 오브젝트를 `BeanFactory`또는 `ApplicationContext`라고 부른다. `Applicationcontext`는 `BeanFactory`에 엔터프라이즈 애플리케이션을 개발하는데 필요한 여러 가지 컨테이너 기능이 추가되어있다. `ApplicatoinContext`는 `BeanFactory`의 서브 인터페이스다. `ApplicationContext`는 다음이 추가 되어있다.

- Spring의 AOP 기능과 쉽게 통합
- 메시지 리소스 핸들링
- 이벤트 발행
- 웹 애플리케이션 사용을 위한 `WebApplicationContext` 같은 애플리케이션 레이어의 컨텍스트

`WebApplicationContext`는 Spring 애플리케이션에서 가장 많이 사용되는 `ApplicationContext`다. `WebApplicationContext`는 `ApplicationContext`의 서브 인터페이스다. 미리 `DispatcherServlet`과 `WebApplicationContext`를 만들어두고, 요청이 서블릿으로 들어올 떄마다 `getBean()`으로 필요한 빈을 가져와 정해진 메서드를 호출한다. 

## Configuration

`ApplicationContext` 또는 `BeanFactory`가 IoC를 적용하기 위해 사용하는 메타정보를 말한다. 컨테이너에 어떤 기능을 세팅하거나 조정하는 경우에도 사용하지만, IoC 컨테이너의 의해 관리되는 빈을 생성하고 구성할 떄 사용된다.

## Spring Bean Scope

Spring은 기본적으로 모든 Bean을 singletone으로 생성하여 관리한다. 따라서, 일반적으로 우리가 주입받은 Bean은 같은 객체라고 믿을 수 있었다. 하지만 singleton 뿐만 아니라 다른 Scope도 존재한다.

|Scope|설명|
|--|---------|
|singletone|하나의 Bean 정의에 대해서 Spring IoC 컨테이너 내에 하나의 객체만 존재한다.|
|prototype|하나의 Bean 정의에 다수의 객체가 존재할 수 있다.|
|request|하나의 HTTP 요청에 하나의 Bean 객체가 존재할 수 있다.|
|session|하나의 HTTP session에 하나의 Bean 객체가 존재할 수 있다.|
|application|하나의 `ServletContext`에 하나의 Bean 객체가 존재할 수 있다.|
|websocket|하나의 `WebSocket`에 하나의 Bean 객체가 존재할 수 있다.|


## 참고 자료

<https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core>

토비 스프링 3.1 (이일민)