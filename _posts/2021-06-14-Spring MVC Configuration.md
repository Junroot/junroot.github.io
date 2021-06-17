---
title:  "Spring MVC Configuration"
categories: programming
tags: Spring
---

## MVC Configuration 활성화

다음과 같이 `@EnableWebMvc` 어노테이션을 붙여서 MVC Configuration을 활성화 할 수 있다. 아래는 Spring MVC infrastructure로 등록 하고 classpath로 사용할 수 있는 의존성이 적용된다.

```java
@Configuration
@EnableWebMvc
public class WebConfig {
}
```

## MVC Configuration API

아래와 같이 `WebMvcConfigurer`를 구현하여 설정 정보를 만들 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    // Implement configuration methods...
}
```

## Type conversion

기본적으로 다양한 숫자와 날짜 타입의 포매터가 설치되어있다. 이 포매터는 `@NumberFormat`과 `@DateTimeFormat`의 필드를 통해 커스터마이징 할 수 있다. 만약, 어노테이션을 사용하지않고 전체적으로 포맷을 지정해주고 싶다면 다음과 같은 방법을 사용하면 된다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
        registrar.setUseIsoFormat(true);
        registrar.registerFormatters(registry);
    }
}
```

## Validation

기본적으로 classpath에 Bean Validation이 있으면, 컨트롤러 메서드의 인자에 `@Valid`와 `Validated`를 사용하여 전역 `Validator`를 사용할 수 있도록 `LocalValidatorFactoryBean`이 등록되어있다. Configuration을 이용하여 전역 `Validator`를 커스터마이징 할 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public Validator getValidator() {
        // ...
    }
}
```

만약 특정 컨트롤러에서만 등록하고 싶다면 아래와 같이 `@IntiBinder` 어노테이션을 사용하면 된다.

```java
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }
}
```

## Interceptor

다음과 같이 수신된 request에 적용할 수 있는 인터셉터를 등록할 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleChangeInterceptor());
        registry.addInterceptor(new ThemeChangeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }
}
```

## Content Type

Spring MVC가 request에서 요청 된 미디어 타입을 결정하는 방법을 정할 수 있다. (Accept 헤더, URL 경로 확장, 쿼리 매개 변수 등) 다음을 참고하면 좋다.

<https://www.baeldung.com/spring-mvc-content-negotiation-json-xml>

기본적으로, URL 경로 확장이 먼저 `json`, `xml`, `rss`, `atom`으로 확인된다. 그 다음 Accept 헤더가 확인된다.

만약, 기본값을 Accept 헤더만으로 변경하고 URL 기반 미디어 타입을 사용해야된다면, 쿼리 파라미터 전략을 사용해보는 것을 고려해야된다. 쿼리 파라미터 전략을 사용할 때 아래와 같이 컴텐츠 타입을 정의할 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.mediaType("json", MediaType.APPLICATION_JSON);
        configurer.mediaType("xml", MediaType.APPLICATION_XML);
    }
}
```

## Message Converter

스프링이 기본적으로 제공해주는 메시지 컨버터 구성을 사용하지 않고 직접 메시지 컨버터를 구성하고싶다면 `configureMessageConverters()`를 사용하면 된다. 만약, 디폴트 메시지 컨버터를 유지한 채로 메시지 컨버터를 추가하고 싶다면 `extendMessageConverters()`를 사용하면 된다.

다음 예는 커스터마이징된 `ObjectMapper`를 사용하여 XML과 JSON 컨버터를 추가하는 방법이다.

```java
@Configuration
@EnableWebMvc
public class WebConfiguration implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        converters.add(new MappingJackson2XmlHttpMessageConverter(builder.createXmlMapper(true).build()));
    }
}
```

## View Controller

호출될 때 바로 전달되는 `ParameterizableVieController`를 정의하기 위한 방법이다. 뷰가 response를 만들기 전에 실행할 Java 컨트롤러의 로직이 없을 경우에 사용할 수 있다.

다음의 예는 `/` 요청을 `home`이라는 뷰로 전달한다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

## View Resolver

View Resolver의 등록을 할 수 있다. 아래는 JSON 렌더링을 위한 기본 `View`로 JSP와 Jackson을 사용하도록 구성하는 예시다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.jsp();
    }
}
```

## Static resource

`Resource`기반의 경로의 리스트로부터 정적 리소스를 제공하기위한 방법을 정의할 수 있다.

아래의 예시는 `/resources`로 시작하는 요청을 `/static` 아래의 classpath 또는 웹 애플리케이션 아래의 `/public`의 상래 경로로 정적 리소스를 찾는다. 리소스는 1년의 기한으로 브라우저 캐싱을 지원한다. Last-Modified 정보는 HTTP conditional request가 "Last-Modified" 헤더를 지원하기위해 `Resource#lastModified`로부터 추론된다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
            .addResourceLocations("/public", "classpath:/static/")
            .setCacheControl(CacheControl.maxAge(Duration.ofDays(365)));
    }
}
```

리소스 핸들러는 `ResourceResolver`와 `ResourceTransformer`의 구현체의 체인도 원한다. 이는 최적화된 리소스로 작업하기 위한 toolchain을 만들 수 있게 해준다. 버전이 있는 URL을 사용하기 위한 `VersionResourceResolver`를 사용할 수 있다. `ContentVersionStrategy`(MD5 hash)는 이를 처리할 수 있는 좋은 방법이다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public/")
                .resourceChain(true)
                .addResolver(new VersionResourceResolver().addContentVersionStrategy("/**"));
    }
```

## Default Servlet

Spring MVC는 `DispatcherServlet`을 `/`에 매핑(컨테이너의 디폴트 서블릿의 매핑을 무시)하는 동시에 컨테이너의 기본 서블릿에 의해 정적 리소스 요청이 처리되도록 해준다. 이는 `DefaultServletHttpRequestHandler`가 `/**` URL 매핑에 가장 낮은 우선순위로 지정되도록 구성해준다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

`/` 서블릿 매핑을 오버라이딩 할 때 주의할 점은 디폴트 서블릿인 경로가 아닌 이름으로 검색이 된다는 점이다. `DefaultservletHttpRequestHandler`는 시작시 대부분의 주요 서블릿 컨테이너(Tomcat, Jety, GlassFish, JBoss, Resin, WebLogic, WebSpehre)에 알려진 이름 목록을 사용하여 컨테이너에 대한 디폴트 서블릿을 자동으로 탐지한다. 디폴트 서블릿 이름을 알 수 없는 다른 서블릿 컨테이너를 사용할 경우, 아래와 같이 기본 서블릿 이름을 명시해야된다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable("myCustomDefaultServlet");
    }
}
```

## Path Matching

URL의 경로 매칭과 처리와 관련된 커스터마이징을 할 수 있도록 해준다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer
            .setPatternParser(new PathPatternParser())
            .addPathPrefix("/api", HandlerTypePredicate.forAnnotation(RestController.class));
    }

    private PathPatternParser patternParser() {
        // ...
    }
}
```

## 참고 자료

<https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config>

<https://browndwarf.tistory.com/48>

<https://www.baeldung.com/spring-mvc-content-negotiation-json-xml>

토비 스프링 3.1 (이일민)