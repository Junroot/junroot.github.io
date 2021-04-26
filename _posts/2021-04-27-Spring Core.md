---
title:  "Spring Core"
categories: programming
tags: Java Spring
---

## Components(컴포넌트)

Spring은 `@Compopnent`, `@Repository`, `@Service`, `@Controller` 같은 어노테이션을 제공한다. 그 중에 `@Repository`, `@Service`, `@Controller`는 `@Component`를 특수화 시킨 케이스다.

### 메타 어노테이션과 합성 어노테이션

메타 어노테이션이란 다른 주석에 적용할 수 있는 어노테이션이다. Spring에서는 많은 어노테이션들이 메타 주석으로 사용되고 있다. 예를들어 `@Service` 어노테이션은 `@Component`로 메타 주석 처리가 된다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {

    // ...
}
```

메타 어노테이션으로 만들어진 어노테이션을 합성 어노테이션이라고 부른다. 예를 들어, `@RestController` 어노테이션은 `@Controller`와 `@ResponseBody`의 합성 어노테이션이다.

### 클래스 자동 탐지와 빈 등록

Spring은 클래스들을 자동으로 탐지하고, 해당하는 `BeanDefinition` 인스턴스를 `ApplicationContext`에 등록할 수 있다. 예를들어, 아래 두 클래스는 자동 탐지된다.

```java
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

```java
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}

```

이 클래스들을 자동으로 탐지하고 해당하는 빈에 등록하려면 `@Configuration` 클래스에 `@ComponentScan`을 추가해야된다. 아래의 예시에 `basePackages` 속성은 위 두 클래스의 공통적인 패키지 경로여야한다. 또는 여러 패키지를 쉼표 또는 세미콜론올 구분하여 목록을 지정할 수도 있다.

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

### Customize Scanning Components 

기본적으로, `@Component`와 이를 사용하는 합성 어노테이션(`@Repository`, `@Service`, `@Controller`, `@Configuration` 등)은 컴포넌트로 탐지한다. 만약 스캔하고 싶은 컴포넌트를 수정 또는 확장하고싶다면 `@ComponentScan` 어노테이션을 사용할 수 있다. `@ComponentScan` 어노테이션에 `includeFilters`와 `excludFilters` 속성을 추가하면 된다. 아래는 `filter`의 속성들이다.

|filter type|예시|설명|
|-----|----|----|
|annotation|`org.example.SomeAnnotation`|컴포넌트에서 사용하고있거나 포함하고 있는 어노테이션|
|assignable|`org.example.SomeClass`|컴포넌트를 할당할 수 있는 클래스 또는 인터페이스|
|aspectj|`org.example..*Service_+`|매칭되는 컴포넌트를 찾기 위한 AspectJ 타입 표현식|
|regex|`org\.example\.Dafault.*`|매칭되는 컴포넌트를 찾기 위한 정규 표현식|
|custom|`org.example.MyTypeFilter`|`org.springframework.core.type.TypeFilter` 인터페이스의 커스텀 구현|

아래는 `@Repository` 어노테이션들을 전부 무시하고, "stub"의 레포지토리들을 사용하는 예시다.

```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```

### Component에 Bean Metadata 정의

컴포넌트 안에서 컨테이너에 Bean을 정의할 수 있다. 이 방법도 `@Configuration` 주석 처리된 클래스 내에서 `@Bean`을 사용하는 방법과 똑같이 동작한다. 아래는 예시다.

```java
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

### 컴포넌트 자동 탐지 네이밍

스캐닝 과정의 일부로 스캐너가 탐지되면 `BeanNameGenerator`에 의해 빈의 이름이 생성된다. 스프링의 스테레오 타입의 어노테이션(`@Component`, `@Repository`, `@Service`, `@Controller`)들도 어노테이션 안에 `value` 속성이 없다면 이 방식으로 빈의 이름을 생성한다. 

```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```

### 어노테이션을 이용한 Qualifier 

Qualifier의 경우, 여러 개의 타입이 일치하는 bean 객체가 있을 경우, `@Qualifier` 어노테이션을 통해 조건을 만족하는 객체를 주입하게 한다. 또는 `@Qualifier`의 합성 어노테이션을 사용할 수도 있다. 아래는 그 예시다.

```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

`@Qualifier`의 주입 방법은 인스턴스 변수, 생성자의 인자, 메서드의 파라미터로 사용할 수 있다.

```java
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}

```

## Denpendency Injection(의존성 주입)

Denpendency Injection(DI)은 객체 인스턴스를 생성자나 팩터리 메서드의 인자, setter 메서드를 통해서만 객체의 의존성을 정의하는 과정이다. 컨테이너는 빈을 생성할 때 이런 의존성을 주입한다. 이 과정은 기본적으로 클래스를 직접 인스턴스화하거나, 서비스 로케이터 패턴을 사용하여 종속성을 스스로 제어하는 방법의 반전된 형태이다.

DI는 객체간의 의존성을 낮춘다. 또한, 인터페이스나 추상 클래스의 의존성이 있을 때는 클래스를 테스트하기 쉬워지며, 이를 통해 stub 또는 mock 객체를 사용하여 단위 테스트를 할 수 있게된다.

### 생성자 기반 의존성 주입

생성자 기반 의존성 주입은 의존성을 나타내는 인자들이 있는 생성자를 컨테이너가 호출한다. 

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private final MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

### Setter 기반 의존성 주입

Setter 기반 의존성 주입은 인자가 없는 생성자 또는 인자가 없는 정적 팩터리 메서드를 호출한 후에, setter 메서드를 호출하는 컨테이너에 의해 처리된다.

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

생성자 기반 의존성 주입을 받은 경우에도 Setter 기반 의존성 주입이 가능하다.

### 생성자 기반? Setter 기반?

생성자 기반과 Setter 기반을 혼합해서 사용할 수 있다. 따라서 필수 의존성은 생성자를 사용하고, 선택적 의존성은 Setter를 사용하는 것이 좋다. 

하지만 Spring 팀은 컴포넌트들을 불변 객체로 만들 수 있고, 의존들이 `null`이 아니므로 생성자 기반을 선호한다. 또한 생성자가 주입한 컴포넌트들은 완전히 초기화된 상태이다.

### 순환 의존성

주로 생성자 주입을 할 경우 순환 의존성이 발생할 수 있다.

예를 들어 클래스 A에는 주입을 위한 클래스 B 인스턴스가 필요하고 클래스 B에는 주입을 위한 클래스 A 인스턴스가 필요한 경우에 해당한다. 클래스 A와 B가 서로 주입되도록 빈을 구성할 경우, Spring IoC 컨테이너가 순환 참조를 발견하고 `BeanCurrentlyInCreationException`을 던진다.

해결할 수 있는 근본적인 방법은 순환의전을 제거하기 위해재 설계를 하는 것이다. 순환 참조는 좋은 설계가 아니다. 두 번째 해결 방법은 생성자 주입 대신에 Setter 주입을 사용하면 객체를 생성하고 이후에 주입을 하기 때문에 예외 발생을 해결할 수 있다.

## 참고 자료

<https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core>

<https://engkimbs.tistory.com/683>