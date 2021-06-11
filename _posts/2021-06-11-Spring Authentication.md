---
title:  "Spring Authentication"
categories: programming
tags: Spring
---

이 글에서는 Spring Framework로 HTTP 세션을 처리하는 방법을 정리한다.

## 세션 생성 정책

Spring의 세션 생성 정책을 설정하기 위해서는 `@Configuration` 클래스 대신 `WebSecurityConfigurerAdapter`를 상속한 클래스를 생성하고 `@EnableWebSecurity` 어노테이션을 붙여준 클래스에서 설정 가능하다. 그 중 아래의 메서드를 오버라이딩 해야된다.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.sessionManagement()
        .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
}
```

정책에는 다음과 같이 4가지가 있다.

- always: 세션이 아직 존재하지 않으면 생성
- ifRequired: 요구하는 경우에만 세션을 생성(기본값)
- never: 프레임워크가 세션을 만들지는 안지만 이미 세션을 사용 중인 경우 그 세션을 사용
- stateless: 세션을 생성하지도 사용하지도 않는다.

## 동시 세션 제어

이미 인증 된 사용자가 다시 인증 요청을 하면 1. 기존 사용자의 세션을 무효화하고 새 세션으로 사용자를 다시 인증하거나 2. 두 세션이 동시에 존재하도록 허용할 수 있다. 

동시 세션을 지원하기 위해서는 다음과 같이 Bean을 정의해야된다.

```java
@Bean
public HttpSessionEventPublisher httpSessionEventPublisher() {
    return new HttpSessionEventPublisher();
}
```

동시 세션의 개수를 제한하기 위해서는 앞에 나왔던 `configure()`메서드에 다음과같이 설정해야된다.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.sessionManagement().maximumSessions(2)
}
```

## 세션 시간 초과

### 세션 시간 처리

사용자가 시간이 만료된 세션을 사용하여 요청을 보내면 다른 url로 리다이렉트되게 설정 가능하다. 아래의 예시는 시간이 만료된 세션을 보낼 경우 '/sessionExpired.html'로, 유효하지 않은 세션 ID를 보낼 경우 '/invalideSession.html'로 리다이렉트 된다.

### 세션 시간 설정

properties를 이용하여 세션 시간을 지정할 수 있다. 별도의 단위를 입력하지 않으면 초로 간주한다.

```
server.servelt.session.timeout=15m
```

## 세션 쿠키 보호

`httpOnly`와 `secure` 플래그를 이용하여 세션 쿠키를 보호할 수 있다.

- httpOnly: true면 브라우저의 스크립트가 쿠키에 접근할 수 없다.
- secure: true면 HTTPS 연결을 통해서만 전송된다.

아래와 같이 Java를 통해 설정가능하다.

```java
public class MainWebAppInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext sc) throws ServletException {
        // ...
        sc.getSessionCookieConfig().setHttpOnly(true);        
        sc.getSessionCookieConfig().setSecure(true);        
    }
}
```

Spring Boot의 경우는 properties 파일을 통해서 설정할 수 있다.

```
server.servlet.session.cookie.http-only=true
server.servlet.session.cookie.secure=true
```

## 세션 사용하기

`@Scope` 어노테이션을 통해 session 스코프로 Bean을 정의할 수 있다.

```java
@Component
@Scope("session")
public class Foo { .. }
```

다음과 같이 Controller 메서드에서 세션을 등록할 수 있다.

```java
@RequestMapping(..)
public void fooMethod(HttpSession session) {
    session.setAttribute(Constants.FOO, new Foo());
    //...
    Foo foo = (Foo) session.getAttribute(Constants.FOO);
}
```

## 참고 자료

<https://www.baeldung.com/spring-security-session>

<https://kimchanjung.github.io/programming/2020/07/02/spring-security-02/>

