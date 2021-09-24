---
title:  "Spring Configuration"
categories: programming
tags: Spring
---

## Spring을 구성하는데 어노테이션과 XML 중 어느 것이 좋을까?

각각의 장단점이 있다. 어노테이션은 선언에 많은 컨텍스트를 제공하여 더 짧고 간결할 수 있다. XML은 소스 코드를 건드리거나 다시 컴파일하지 않고도 설정 정보(Configuration)을 연결할 수 있다는 장점이 있다. 일부 개발자는 어노테이션이 달린 클래스는 더 이상 POJO가 아니고 설정 정보가 분산되고 관리하기가 더 힘들다는 주장도 있다.

## 어노테이션 기반 컨테이너 설정 정보

### `@Required`

빈 속성의 setter 메서드에 적용하는 어노테이션이다.

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

이 어노테이션은 구성 시간에 Bean 정의의 명시적인 속성 값 또는 자동 와이어링을 통해 채워져야 됨을 나타낸다. 만약 채워지지 않을 경우 컨테이너에서 예외가 발생한다.

### `@Autowired`

빈을 자동으로 주입하기 위한 어노테이션이다.

아래와 같이 생성자에 어노테이션을 사용할 수 있다. 참고로 클래스의 생성자가 하나만 정의된 경우 생략이 가능하다.

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

다음과 같이 임의의 이름과 여러 인자가 있는 메서드에 어노테이션을 붙일 수 있다.

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

다음과 같이 해당 메서드의 파라미터로 배열 등을 사용하여 `ApplicationContext`로부터 모든 Bean을 제공 받을 수도 있다.

``` java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

키 타입이 `String`인 `Map` 인스턴스로 주입받을 경우, key에는 Bean의 이름을 갖는 형식으로 주입 받을 수 있다.

```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

`Required`와 `Autowired`를 동시에 사용해야되는 경우에는 `@Autowired`의 `required` 속성을 이용하면된다. 비필수 항목으로 표기하고 싶으면 `required` 속성을 `false`로 지정하면된다.

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

`@Autowired`에 `required`가 `true`인 것은 Bean 클래스의 한 생성자에만 지정할 수 있다. 만약 생성자가 하나면 `@Autowired`의 `required`의 기본 값은 `true`이다. 만약 `@Autowired`가 붙어있는 생성자가 여러 개라면, 매칭되는 빈이 가장 많은 생성자를 사용하게 된다.

아래와 같이 `Optional`과 `@Nullable` 어노테이션을 이용해 필수가 아닌 Bean을 지정할 수 있다.

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

### `@Primary`

자동 와이어링에 여러 후보가 있을 경우 선택할 수 있는 방법 중 한가지다. `@Primary`는 타입을 단일 값 의존성에 만족하는 Bean이 여러 개 있을 때 우선적으로 주입되도록 한다.

다음과 같이 사용할 수 있다.

```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

앞의 설정 정보에서 `MovieRecommender`는 `firstMovieCatalog`가 주입된다.

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

### `@Qualifier`

`@Qualifier` 어노테이션에 값을 기입하여 매칭되는 Bean을 주입하도록 좀 더 정교한 설정을 할 수 있다.

```java
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

생성자나 메서드의 파라미터로 지정할 수 있다.

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

`@Qualifier`의 값은 고유할 필요가 없다. 여러 Bean에 `@Qualifier("action")`을 지정하여 `Set<MovieCatalog>`로 모든 "action"을 주입하는 것도 가능하다.

다음과 같이 사용자 지정 `@Qualifier`를 만들 수도 있다.

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```

```java
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```

`value` 속성 대신 사용자 정의 필드를 추가할 수도 있다. 자동 와이어링을 위해서는 여러 속성 값이 모두 일치해야된다.

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```

```java
public enum Format {
    VHS, DVD, BLURAY
}
```

```java
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...
}
```

### `@Resource`

Bean의 필드 또는 setter 메서드에 붙여서 주입을 할 수 있다. 이는 Spring이 아닌 Java EE의 공통 패턴이다. Spring은 Spring 관리 객체에 대해서도 이 방법을 지원한다.

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

이름을 명시하지 않은경우 필드 이름 또는 setter 메서드를 기반으로 정해진다. 필드의 경우 필드의 이름으로 결저오디고, setter 메서드의 경우에는 Bean 속성 이름으로 결정된다. (생성자의 타입)

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

명시적인 이름을 지정하지 않은 경우, bean의 이름 대신 일치하는 타입을 찾아 주입한다. `context` 필드의 경우 `ApplicationContext`라는 타입을 기반으로 주입된다.

```java
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context; 

    public MovieRecommender() {
    }

    // ...
}
```

### `@Value`

일반적으로 외부 속성을 삽입하는데 사용한다.

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

`application.properties` 파일에는 다음과 같은 내용이 있었다.

```
catalog.name=MovieCatalog
```

속성 값이 존재하지 않는 경우 속성 이름으로 값이 삽입된다. (예: `${catalog.name}`) 존재하지 않는 값에 대한 업격한 제어를 위해서는 다음 과같이 `PropertySourcesPlaceholderConfigurer` Bean을 선언해야된다.

```java
@Configuration
public class AppConfig {

     @Bean
     public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
           return new PropertySourcesPlaceholderConfigurer();
     }
}
```

다음과 같이 기본값을 지정할 수도 있다.

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {
        this.catalog = catalog;
    }
}
```

사용자 정의 타입 캐스팅을 지정하려면 다음과 같이 `ConversionService`를 Bean으로 등록해야된다.

```java
@Configuration
public class AppConfig {

    @Bean
    public ConversionService conversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
        conversionService.addConverter(new MyCustomConverter());
        return conversionService;
    }
}
```

다음과 같이 SpEL expression을 이용해서 값을 동적으로 처리할 수 있다.

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("#{systemProperties['user.catalog'] + 'Catalog' }") String catalog) {
        this.catalog = catalog;
    }
}
```

SqEL은 또한 복잡한 데이터 구조를 처리할 수도 있다.

```java
@Component
public class MovieRecommender {

    private final Map<String, Integer> countOfMoviesPerCatalog;

    public MovieRecommender(
        @Value("#{{'Thriller': 100, 'Comedy': 300}}") Map<String, Integer> countOfMoviesPerCatalog) {
        this.countOfMoviesPerCatalog = countOfMoviesPerCatalog;
    }
}
```

## Java 기반 컨테이너 설정 정보

### 기본 개념: `@Bean`과 `@Configuration`

`@Bean`은 Spring IoC 컨테이너에 의해서 관리될 새로운 객체를 인스턴스화, 구성, 초기화 하는 메소드를 가리키기 위해 사용한다. `@Bean`은 `@Component`에 존재할 수 있지만 보통은 `@Configuration` Bean에 사용된다.

클래스에 `@Configuration`을 추가하면 기본적인 목적인 Bean 정의라는 것을 가리킨다. 게다가 `@Configuration` 클래스는 클래 스 내부에 다른 `@Bean`을 호출하는 Bean 정의가 가능하다.

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

### Full `@Configuration` vs lite `@Bean` mode

`@Configuration`이 아닌 `@Component`에 `@Bean` 메서드를 선언한 경우 라이트 모드로 처리된다. 라이트 모드의 `@Bean`메서드는 다른 `@Bean` 메서드의 의존이 불가능하다. 대신에 컴포넌트의 내부 상태에따라 작동하게 할 수 있다. `@Bean` 메서드가 `@Configuration` 클래스에 선언된 경우 풀 모드가 사용되고 교차 메서드 참조가 가능하다.

### `AnnotationConfigApplicationContext`을 사용한 Spring 컨테이너 인스턴스화

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

`AnnotationConfigApplicationContext`는 `@Configuration`에만 국한되지 않는다. 다음의 예는 `MyServiceImpl`, `Dependency1`, `Dependency2`가 자동 와이어링에 사용된다고 가정한다.

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

`register()` 메서드를 사용하여 컨테이너를 구성할 수도 있다.

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

컴포넌트 스캐닝을 하기위해서는 `scan()`메서드를 사용할 수 있다. 다음과같은 `@Configuration` 클래스가 있다고 가정한다.

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}
```

스캐닝을 하기 위해서는 아래와 같이 `scan()` 메서드를 호출하면 된다.

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

### `@Bean`

기본적으로 Bean 이름은 메서드 이름과 동일하다.

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

다음과 같이 반환 타입을 인터페이스로 메서드를 선언할 수 있다.

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

`@Bean`메서드도 의존성 주입이 가능하다. 기존의 생성자 기반 의존성 주입과 거의 동일하다.

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

Bean 이름을 메서드 이름이 아닌 별도의 이름으로 지정이 가능하다.

```java
@Configuration
public class AppConfig {

    @Bean(name = "myThing")
    public Thing thing() {
        return new Thing();
    }
}
```

또한 Bean이 여러 개의 이름을 갖는 것도 가능하다.

```java
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

Bean에 자세한 설명을 기입하는 것이 가능하다. 보통 모니터링을 목적으로 사용한다.

```java
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```

### `@Configuration`

`@Configuration`은 Bean 정의의 소스인 객체임을 나타내는 클래스 수준 어노테이션이다. 

Bean이 서로 의존성을 가질 때 다음과 같이 Bean 메서드가 다른 메서드를 호출하면된다.

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

아래와 같이 Bean이 두 번 호출되어도 싱글톤으로 관리되기 때문에 같은 인스턴스를 호출하게 된다. 

```java
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

`CGLIB`에 의해 모든 `@Configuration` 클래스는 서브 클래스화 된다. 부모 메서드를 호출하고 새로운 인스턴스를 만들기 전에 자식 클래스의 자식 메서드가 컨테이너에 캐시되어 있는 Bean이 있는지 확인한다.

### Java 기반 설정 정보 조합하기

`@Import` 어노테이션으로 다른 설정 정보 클래스를 로드할 수 있다.

```java
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

아래와 같이 `ConfigB`만으로도 `A` Bean을 가져올 수 있다.

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

다른 설정 정보의 Bean을 의존하는 것도 가능하다.

```java
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

`@Configuration` 클래스도 하나의 빈이므로 `@Autowired`의 주입이 가능하다.

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

해당 Bean이 어느 설정 정보에 있는지 명확하게 하게위해 `@Configuration` 클래스를 직접 주입하기도 한다.

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```

하지만 이렇게하면 `RepositoryConfig`와 `ServiceConfig`의 결합도가 높아지기 때문에 인터페이스나 추상 클래스를 사용할 수도 있다.

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

## 환경 추상화

`Environment` 인터페이스는 컨테이너에 통합된 추상화다. 또한 애플리케이션 환경의 두 가지 측면인 프로필과 프로퍼티를 모델링한다. 프로필은 활성화된 프로필의 Bean만 컨테이너에 등록되도록하는 Bean 정의의 논리적인 그룹이다. 프로필과 관련된 `Environment` 객체는 현재 활성화 된 프로필과 기본적으로 활성화 되어야하는 프로필을 결정한다. 프로퍼티는 대부분의 애플리케이션에서 중요한 역할을 한다. properties 파일, JVM 시스템 properties, 시스템 환경 변수, JNDI, 서블릿 컨텍스트 파라미터 등이 있다. 프로퍼티와 관련된 `Environment` 객체의 역할은 프로퍼티를 구성할 수 있는 편리한 인터페이스를 제공하는 것이다. 

### `@Profile`

한 개 이상의 지정된 프로필이 활성화 될 때 설정 정보가 등록될 자격이 있음을 표시할 수 있다.

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```

프로필 값에는 간단하게 프로필 이름이나 프로필 표현식을 사용할 수 있다. 프로필 표현식은 복잡한 논리 연산을 할 수 있다. 아래의 연산자를 사용할 수 있다. 예) `production & us-east`

- `!`: not
- `&`: and
- `|`: or

여러개의 연산자를 사용할 경우 무조건 괄호가 있어야된다. 예) `production & (us-east | eu-central)`

`@Profile`은 Bean 메서드 단위로도 선언할 수 있다.

```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

Java 코드로 프로필을 활성화 하는 방법은 `ApplicationContext`에 있는 `Environment` API를 사용하는 방법이다.

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

또한 `spring.profiles.active` 프로퍼티를 통해 프로필을 활성화 활수도 있다. 여러개의 프로필 설정도 가능하다

```java
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

다음과 같이 `spring.profiles.active`로 지정할 수도 있다.

```java
spring.profiles.active="profile1,profile2"
```

기본 프로필은 다음과 같이 지정할 수 있다. 또한 `setDefaultProfiles()`나 `spring.profiles.default` 프로퍼티를 사용하여 선언할 수도 있다.

```java
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

### `PropertySource`

`Environment`는 프로퍼티의 계층형을 구성을 탐색하도록 제공해준다. 아래와 같이 `my-property`이 현재 환경에 정의되어 있는지 확인할 수 있다. 이 경우 `Environment` 객체는 `PropertySource` 집합에 검색을 한다. `PropertySource`는 key-value 페어의 추상화다.

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```

다음과 같이 사용자 정의 `PropertySource`를 추가 할 수도 있다. 이 때 `MyPropertySource`는 가장 높은 우선순위로 추가된다.

```java
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

### `@PropertySource`

스프링의 `Environment`에 `PropertySource`를 추가하기 위한 어노테이션이다. 

`app.properties`에 `testbean.name=myTestBean`이라는 key-value pair가 있을 때, `testBean.getName()`을 호출하면 `myTestBean`이 반환된다.

```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

`@PropertySource`에 `${...}` placeholder를 사용할 수 있다. `Environment`에 이미 등록되어 있다면 그 값을 사용하고, 아니면 기본 값을 사용한다. 아래의 예시는 기본값으로 `default/path`를 사용하는 경우이다. 만약 기본값을 사용하지 않았는데 값을 확인할 수 없다면 `IllegalArgumentException`이 던져진다.

```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

## 참고 자료

<https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core>

토비 스프링 3.1 (이일민)
