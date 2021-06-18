---
title:  "Spring vs Spring Boot"
categories: programming
tags: Java Spring
---

## Spring이란

Spring이 웹 애플리케이션 프레임워크로만 알고 있는 경우가 많다. 하지만 Spring은 보안, 웹 앱, 빅 데이터 등 여러가지 Java기반 애플리케이션을 개발할 때 필요한 Infrastructure를 제공하는 프로젝트들을 말하는 것이다. 해당 [링크](https://spring.io/projects)에서 현재 Spring에 존재하는 프로젝트들을 확인할 수 있다. 그 중 Spring Framework와 Spring Boot가 우리가 주로 이용하는 프로젝트다. 

## Spring Framework란

Spring Framework는 자바 기반 엔터프라이즈 애플리케이션을 개발할 때 필요한 Infrastructure를 제공해주는 프레임워크다. 이를 통해 개발자들은 특정 배포 환경과 불필요한 시도 없이 애플리케이션 수준의 비즈니스 로직에 집중할 수 있다. 우리가 흔히 사용하는 Spring MVC와 Spring JDBC 등은 Spring 프로젝트가 아니라 Spring Framework의 하위 모듈이다. 

Spring Framework는 IoC와 DI를 통해 결합도가 낮은 객체지향 프로그래밍이 가능하도록 도와주고, 이를 통해 유지보수 및 테스트하기 쉽도록 해준다.

## Spring Boot란

Spring Boot는 Spring 기반 애플리케이션 쉽게 만들 수 있도록 도와주는 프로젝트다. Spring Framework는 많은 초기 설정이 필요하다. 이 복잡하고 많은 설정을 해결해주기 위해 만들어진 프로젝트가 Spring Boot다. Spring Boot가 Spring 기반 애플리케이션을 만들 때 어떤 것을 대신 처리해주는지 알아보자.

### 의존성 버전 관리

Spring Framework을 사용하여 애플리케이션을 개발할 때 수동으로 추가해줘야되는 의존성들이 있다. 하지만 이 의존성들은 서로 호환성이 맞는 버전을 잘 선택해줘야된다. Spring Boot는 Spring Framework를 사용하는데 필요한 의존들을 버전에 맞춰 자동으로 구성해주는 기능이 있다. [Spring Initialzr](https://start.spring.io/)를 사용하여 애플리케이션이 필요한 의존성을 빠르게 가져와 개발 시간을 줄일 수 있다. 

Spring Boot는 Gradle 기준으로 `org.springframework.boot`라는 그룹의 의존들을 추가하면 된다. 해당 그룹에 있는 의존들은 [링크](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using.build-systems.starters)에 있으니 확인해서 필요한 의존을 추가해서 사용할 수 있다.

### 자동 Configuration

Spring Boot는 추가한 의존성을 기반으로 Spring 애플리케이션을 자동으로 설정해준다. Spring 애플리케이션에서 인-메모리 데이터베이스(H2)를 사용하는 경우를 예로 설명해본다. 만약, Spring Framework 만을 사용한다면 아래와 같이 `Datasource`를 직접 Bean으로 등록해줘야된다.

```java
@Configuration
public class DBConfiguration {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .addScript("schema.sql")
            .setScriptEncoding("UTF-8")
            .build();
    }
}
```

하지만 Spring Boot를 사용하면 직접 Bean을 등록할 필요없이 설정 값에 따라 자동으로 Bean을 등록해준다. 아래와 같이 `application.yml` 파일에 설정 값을 지정해주면 자동으로 `DataSource`가 등록된다.

```java
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
    username: sa
    password:
  h2:
    console:
      enabled: true
```

이런 자동 설정이 가능한 이유는 `@SpringBootApplication` 어노테이션때문이다. 이 어노테이션을 다음의 어노테이션을 포함하고 있다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    //...
}
```

- `@ComponentScan`: `@Component` 어노테이션이 붙은 클래스를 Bean으로 등록시켜준다. `@ComponentScan`은 `@Configuration` 어노테이션과 함께 써야되는데 `@SpringBootConfiguration`이 그 역할을 해준다.
- `@SpringBootConfiguration`: Spring Boot에서 사용하는 `@Configuration`임을 나타내는 어노테이션이다. Spring Boot의 `@*Test` 어노테이션은 테스트를 시작할 때 `@SpringBootConfiguration`가 있는 클래스를 찾는다.
- `@EnableAutoConfiguration`: 이 어노테이션이 자동 설정을 가능하게 해주는 어노테이션이다.

### 내장 WAS

Spring Boot는 `jar`파일을 패키징하여 임베디드 HTTP 서버를 사용하고 있기 때문에 IDE에서 웹 애플리케이션을 실행시키는 것이 가능하다. 기존에 Spring Framework만 사용할 때는 `war`파일을 패키징하고, WAR파일을 배포해야만 실행시켜볼 수 있었다. 

따라서, 아래와 같이 `java -jar` 명령어로 애플리케이션을 실행하는 것이 가능해진다.

```bash
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

또한 Gradle을 사용할 경우 `bootRun`을 이용하여 애플리케이션을 실행할 수도 있다.

```bash
$ gradle bootRun
```

### 모니터링

`spring-boot-starter-actuator`라는 의존을 추가하여 애플리케이션 관리 및 모니터링이 가능해진다. 이 기능은 서비스 중에 필요한 모니터링 기능을 엔드 포인트로 만들어서 제공해준다. 엔드포인트 종류는 [링크](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#actuator.endpoints)에서 확인할 수 있다. `health` 엔드포인트는 `/actuator/health` 경로로 매핑된다.

이 기능을 사용할 경우에 서비스의 민감한 정보를 많이 제공하기 때문에 보안에 신경써줘야된다. 또한 이 기능의 데이터는 영구 저장소에 저장하지 않으면 삭제될 수 있으니 따로 보관 방법을 마련해둬야된다.

## 참고 자료

<https://www.youtube.com/watch?v=Y11h-NUmNXI>

<https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/>

<https://spring.io/projects>

<https://medium.com/swlh/spring-vs-spring-boot-826fcfe21522>