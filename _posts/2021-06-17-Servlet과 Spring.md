---
title:  "Servlet과 Spring"
categories: programming
tags: Java Spring
---

Spring Framework의 문서를 정리해보면서 이해가 되지 않는 부분이 상당히 많다. 그 중에서 서블릿과 관련된 많은 부분이 이해가 되지 않았다. 슬슬 Spring뿐만 아니라 Servlet에 대한 이해가 필요하지 않을까하여, 이번 기회에 정리해보고자 한다.

## 서블릿(Servlet)

Java를 사용하여 동적인 웹페이지를 생성하는 서버측의 프로그램이다. JSP와 많이 비교가 되는데, 서블릿은 Java 코드 안에 HTML이 포함되어 있어서 View가 변경될 때마다 코드를 수정해야되기 때문에 웹 디자이너 입장에서 사용하기 편하도록 HTML 안에 Java 코드를 삽입하는 형태로 만들어진 스크립트 언어다. 서블릿은 Java EE의 스펙으로 많은 웹 서비스에서 사용되고 있다.

### HttpServlet

서블릿은 `HttpServlet`을 상속하여 정의할 수 있다. 기본적으로 request를 처리하여 response를 반환한다. 예를들어, 서블릿을 이용하여 HTML form으로 받은 유저의 입력을 수집하여 DB에 저장해둔 뒤, 웹 페이지를 동적으로 생성할 때 활용할 수 있다. 이 `HttpServelt` 클래스에는 서블릿의 생명 주기를 관리하기 위한 `init()`, `service()`, `destory()` 메서드가 있고, 각 요청을 처리하기 위한 `doGet()`, `doPost()`, `doPut()`, `doDelete()` 등의 메서드가 있다.

생명 주기를 관리하기 위한 메서드에 대한 자세한 설명은 아래와 같다.

- `init()`: 서블릿 인스턴스가 생성되고 초기화될 로직을 담는 메서드. 기본적으로 아무 처리도 하지 않는다.
- `service()`: HTTP request의 유형을 해석하여, `doGet`, `doPost`, `doPut`, `doDelete` 등 적절한 메서드를 호출한다.
- `destroy()`: 더 이상 해당 서블릿의 서비스를 사용하지 않을 때 호출한다. 기본적으로 아무 처리도 하지 않는다.

## 서블릿 컨테이너

앞서 말한 서블릿의 생명 주기는 서블릿 컨테이너에 의해 관리된다. 즉, 개발자가 직접 서블릿을 생성하고 파괴할 필요가 없다. 서블릿 컨테이너의 대표적인 예가 Apache Tomcat이다. 서블릿 컨테이너는 요청이 들어왔을 때 해당 컨테이너가 로드되어 있지않으면 `init()` 메서드를 호출하여 서블릿을 인스턴스화 한다. 그리고 서블릿의 `service()` 메서드를 호출하여 처리한다. 이후에 서블릿 컨테이너가 더 이상 request를 받지 않을 때 `destroy()` 메서드를 호출하여 서블릿을 제거한다.

request의 URL과 파라미터 매핑은 `web.xml` 설정 파일이나 `@WebServlet` 어노테이션을 통해 설정할 수 있다.

## Front Controller Pattern

각 요청 URL마다 별개의 서블릿을 만들면 어떤 문제가 있을지 생각해보자. 새로운 요청이 생길 때마다 새로운 `HttpServlet`을 상속한 클래스를 정의해야되고, `web.xml`에 매핑 내용을 추가해야된다. 이 과정에서 많은 코드 중복이 발생할 것이다. 이를 해결하기 위해 웹 애플리케이션에서 사용하는 소프트웨어 디자인 패턴이 프론트 컨트롤러 패턴이다. 이 패턴은 모든 request를 하나의 프론트 컨트롤러가 받아서 이 프론트 컨트롤러가 적절한 컨트롤러에게 처리를 위임하는 구조다.

![front-controller](/assets/images/front-controller.png)

## Spring MVC의 DispatcherServlet

Spring MVC의 `DispatcherServlet`도 프론트 컨트롤러 패턴을 따르고 있다. 아래 사진과 같이 서블릿을 하나만 두고 모든 요청을 다 받은 후, 그 요청을 처리하기 위해 다른 클래스의 메서드를 호출하는 구조로 이루어져있다.

![dispatcher-servlet](/assets/images/dispatcher-servlet.png)

![dispatcher-servlet2](/assets/images/dispatcher-servlet2.png)

## Spring Container

위 사진에 나와있는 `HandlerMapping`, `HandlerAdapter`, `ViewResolver`는 `DispatcherServlet`이 Spring Container로부터 Bean으로 주입을 받아서 처리되기 때문에, 웹 애플리에키션 개발자는 컨트롤러 부분만 직접 구현하면된다. 이 Spring Container로는 Servlet `WebApplicationContext`와 Root `WebApplicationContext` 2가지가 있는데 이 둘은 계층 구조를 갖는다. 이 Root `WebApplicationContext`는 여러개의 `DispatcherServlet`을 등록하면 공유하는 Bean을 관리하기 위해 따로 존재한다.

![webapplicatoincontext](/assets/images/webapplicatoincontext.png)

## 참고 자료

<https://www.youtube.com/watch?v=calGCwG_B4Y>

<https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94_%EC%84%9C%EB%B8%94%EB%A6%BF>

<https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94%EC%84%9C%EB%B2%84_%ED%8E%98%EC%9D%B4%EC%A7%80>

<https://www.baeldung.com/intro-to-servlets>

<https://www.baeldung.com/java-servlets-containers-intro>

<https://mangkyu.tistory.com/14>

<https://www.javaguides.net/2018/08/front-controller-design-pattern-in-java.html>

<https://velog.io/@lacomaco/Spring-MVC-%ED%99%9C%EC%9A%A9>

<https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-config>