---
title:  "Spring Test"
categories: programming
tags: Spring
---

Spring에서 제공하는 라이브러리를 이용하여 단위 테스트 및 통합 테스트 하는 방법을 알아본다.

## 개요

애플리케이션 서버 또는 배포 환경에 의존하지 않고 테스트를 할 수 있도록해준다. 아래와 같은 사항을 테스틀 할 수 있다.

- Spring IoC 컨테이너 컨텍스트의 올바른 연결.
- JDBC 또는 ORM 도구를 사용한 데이터 접근. 여기에는 SQL 문의 정확성, Hibernate 쿼리, JPA 엔티티 매핑 등이 포함될 수 있다.

## Spring Test의 목표

Spring Test는 다음과 같은 주요 목표가 있다.

- 테스트가 Spring IoC 컨테이너 캐싱을 관리한다.
- 테스트 픽스처 인스턴스의 의존성 주입을 제공한다.
- 통합 테스트에 적합한 트랜잭션 관리를 제공한다.
- 통합 테스트 작성을 도와주는 Spring 기반 클래스를 제공한다.

### 컨테이너 관리 및 캐싱

Spring TestContext Framework는 Spring의 `ApplicationContext` 인스턴스 및 `WebApplicatoinContext` 인스턴스를 일관성 있게 로드하고 이 컨텍스트를 캐싱해준다. 로드한 컨텍스트의 캐싱은 중요하다. Spring 컨테이너에 있는 객체들을 인스턴스화 하는데 시간이 많이 걸리기 때문이다. 기본적으로 로드되면 `ApplicationContext`가 각 테스트에 재사용된다. 만약 컨텍스트를 수정하여 다시 로드해야되는 경우, TestContext 프레임워크를 구성하여 다음 테스트을 실행하기 전에 다시 로드하고 컨텍스트를 다시 빌드 할 수 있다.

### 테스트 픽스처의 의존성 주입

TestContext 프레임워크가 애플리케이션 컨텍스트를 로드할 때 선택적으로 의존성 주입을 사용하여 테스트 클래스의 인스턴스를 구성할 수 있다. 이는 사전에 애플리케이션 컨텍스트에 구성된 Bean을 사용하여 테스트 픽스처를 설정시켜준다. 애플리케이션 컨텍스트를 재사용하므로 각 테스트 케이스에따라 복잡한 테스트 픽스처를 복제할 필요가 없다.

### 트랜잭션 관리

실제 데이터베이스에 접근하는 테스트의 문제 중 하나는 Persistance Store의 상태에 영향을 미친다. 개발용 데이터베이스를 사용하는 경우에도 상태 변경이 다음 테스트에 영향을 준다. 또한 영구 데이터 삽입 또는 수정과 같은 많은 작업은 트랜잭션 외부에서 수행 할 수 없다.

TestContext 프레임워크가 이 문제를 해결한다. 기본적으로 이 프레임워크는 각 테스트에 대해 트랜잭션을 만들고 롤백한다. 따라서, 트랜잭션의 존재를 가정하고 테스트 코드를 작성할 수 있다. 각 테스트에서 트랜잭션으로 객체를 프록시한다. 그리고 트랜잭션 내에서 실행 되는 동안 테스트 메서드가 해당 테이블에 내용을 추가, 수정, 삭제를 하면 트랜잭션은 기본적으로 롤백되고 데이터베이스는 테스트 실행 이전의 상태로 돌아간다. 트랜잭션 지원은 테스트의 애플리케이션 컨텍스트에 정의된 `PlatformTransactionManager` Bean을 사용하여 테스트에 제공한다.

트랜잭션을 커밋하려면(일반적으로는 그러지 않지만) `@Commit`주석을 사용하여 트랜잭션이 커밋되도록 TestContext 프레임워크에 지시할 수 있다.

### 통합 테스트를 위한 지원 클래스

TestContext Framework는 `abstract` 통합 테스트를 단순화 하기위해 몇몇의 `abstract` 클래스를 제공한다. 이런 기본 테스트 클래스는 테스트 프레임워크에대한 잘 정의된 후크와 인스턴스 변수 및 메서드를 제공하여 다음에 접근할 수 있다.

- `Application Context`: 명시적인 Bean 검색을 수행하거나 전체 컨텍스트의 상태를 테스트한다.
- `JdbcTemplate` SQL문을 싱행하여 데이터베이스를 쿼리한다. 이러한 쿼리를 사용하여 데이터베이스 관련 애플리케이션 코드 실행 전후에 데이터베이스 상태를 확인할 수 있다. 또한 Spring은 이러한 쿼리가 애플리케이션 코드와 동일한 트랜잭션 범위에서 실행되도록 보장한다.

## JDBC 테스트 지원

`org.springframework.test.jdbc` 패키지는 `JdbcTestUtils`를 포함한다. 이는 기본적인 데이터베이서 테스트 시나리오를 간단하게 하기위해서 JDBC 관련 유틸리티 함수 모음이 포함되어있다. 

- `countRowsInTable(..)`: 지정한 테이블의 row 개수를 센다.
- `countRowsInTableWhere(..)`: `WHERE` 절을 포함하여 지정한 테이블의 row 개수를 센다.
- `deleteFromTables(..)`: 지정한 테이블의 모든 row를 삭제한다.
- `deleteFromTableWhere(..)`: `WHERE` 절을 포함하여 지정한 테이블의 row들을 삭제한다.
- `dropTables(..)`: 지정한 테이블을 드랍한다.

## 어노테이션

### `@ContextConfiguration`

테스트에서 `ApplicationContext`를 어떻게 로드하고 구성할 것인지 결정하는 클래스 수준 메타 데이터를 정의한다. 이 어노테이션은 컨텍스트를 로드하는데 사용되는 애플리케이션 컨텍스트 리소스(`locations`) 또는 컴포넌트(`classes`)를 선언한다.

리소스는 일반적으로 classpath에 있는 XML configuration file 또는 Groovy script이며, 컴포넌트는 `@Configuration` 클래스들이다.

다음은 XML 파일을 참조하는 예제다.
```java
@ContextConfiguration("/test-config.xml") 
class XmlApplicationContextTests {
    // class body...
}
```

다음은 클래스를 참조하는 예제다.
```java
@ContextConfiguration(classes = TestConfig.class) 
class ConfigClassApplicationContextTests {
    // class body...
}
```

다음 예제처럼 `ApplicationContextInitializer`를 선언할 수도 있다. `ApplicationContextInitializer`는 컨텍스트가 생성된 후에 초기화 작업이 필요한 객체를 만들 때 사용하는 인터페이스다.

```java
@ContextConfiguration(initializers = CustomContextIntializer.class) 
class ContextInitializerTests {
    // class body...
}
```

다음 예제처럼 `ContextLoader` 전략을 선언할 수도 있다. `ContextLoader`를 상속하여 ApplicationContext를 생성하고 ServletContext에 추가할 수 있다.(`ContextLoaderListner`)

```java
@ContextConfiguration(locations = "/test-context.xml", loader = CustomContextLoader.class) 
class CustomLoaderXmlApplicationContextTests {
    // class body...
}
```

### `@ContextHierarchy`

클래스 수준 어노테이션이다. 테스트 할 때 `ApplicationContext` 인스턴스의 계층을 선언할 수 있다. 아래는 하나의 테스트 클래스에서 사용하는 예시다. `@ContextHierarchy`는 테스트 클래스 계층안에서도 사용이 가능하다.

```java
@ContextHierarchy({
    @ContextConfiguration("/parent-config.xml"),
    @ContextConfiguration("/child-config.xml")
})
class ContextHierarchyTests {
    // class body...
}
```

만약 테스트 클래스 계층 안에서 컨텍스트 계층 수준의 configuration을 오버라이드하려면 같은 `name`속성을 명시하여 지정할 수 있다.

### `@ActiveProfiles`

클래스 수준 어노테이션이다. `ApplicationContext`를 로드할 때 어떤 Bean 정의 프로필이 active 일지 선언하기위해 사용한다.

다음 예시는 `dev`프로필이 active인 경우다.

```java
@ContextConfiguration
@ActiveProfiles("dev") 
class DeveloperTests {
    // class body...
}
```

다음과 같이 동시에 여러개의 프로필을 active할 수도 있다.

```java
@ContextConfiguration
@ActiveProfiles({"dev", "integration"}) 
class DeveloperIntegrationTests {
    // class body...
}
```

### `@TestPropertySource`

클래스 수준 어노테이션이다. `ApplicationContext`의 `Environment`에서 `PropertySources` 집합에 추가할 properties 파일이나 inlined properties를 설정할 때 사용한다.

다음과 같이 classpath로 properties 파일을 선언할 수 있다.

```java
@ContextConfiguration
@TestPropertySource("/test.properties") 
class MyIntegrationTests {
    // class body...
}
```

다음과 같이 inlined properties를 선언할 수 있다.

```java
@ContextConfiguration
@TestPropertySource(properties = { "timezone = GMT", "port: 4242" }) 
class MyIntegrationTests {
    // class body...
}
```

### `@DynamicPropertySource`

메서드 수준 어노테이션이다. `ApplicationContext`의 `Environment`에서 `PropertySources` 집합에 추가할 properties를 동적으로 등록할 수 있다. 동적 properties는 properties를 미리 모르는 경우에 유용하다.

다음은 예제다.

```java
@ContextConfiguration
class MyIntegrationTests {

    static MyExternalServer server = // ...

    @DynamicPropertySource 
    static void dynamicProperties(DynamicPropertyRegistry registry) { 
        registry.add("server.port", server::getPort); 
    }

    // tests ...
}
```

### `@DirtiesContext`

`ApplicationContext`가 테스트를 실행하는 동안 더럽혀 졌음을 나타낸다. `ApplicationContext`가 더티로 표시되면 테스트 프레임워크의 캐시가 지워지고 닫힌다. 결과적으로 기존 Spring 컨테이너는 동일한 configuration 메타데이터가 있는 컨텍스트를 필요하면 새로 빌드를 한다.

`@DirtiesContext`는 같은 클래스 또는 클래스 계층구조에서 클래스 수준과 메서드 수준 어노테이션으로 사용할 수 있다. 

- class mode가 `BEFORE_CLASS`인 경우 현재 테스트 클래스 이전.
  ```java
  @DirtiesContext(classMode = BEFORE_CLASS) 
    class FreshContextTests {
        // some tests that require a new Spring container
    }
  ```
- class mode가 `AFTER_CLASS`인 경우 현재 테스트 클래스 이후. (default class mode)
    ```java
    @DirtiesContext 
    class ContextDirtyingTests {
        // some tests that result in the Spring container being dirtied
    }
    ```
- class mode가 `BEFORE_EACH_TEST_METHOD`인 경우 현재 테스트 클래스의 각 테스트 메서드 이전.
    ```java
    @DirtiesContext(classMode = BEFORE_EACH_TEST_METHOD) 
    class FreshContextTests {
        // some tests that require a new Spring container
    }
    ```
- class mode가 `AFTER_EACH_TEST_METHOD`인 경우 현재 테스트 클래스의 각 테스트 메서드 이후.
    ```java
    @DirtiesContext(classMode = AFTER_EACH_TEST_METHOD) 
    class ContextDirtyingTests {
        // some tests that result in the Spring container being dirtied
    }
    ```
- method mode가 `BEFORE_METHOD`인 경우 현제 테스트 이전.
    ```java
    @DirtiesContext(methodMode = BEFORE_METHOD) 
    @Test
    void testProcessWhichRequiresFreshAppCtx() {
        // some logic that requires a new Spring container
    }
    ```
- method mode가 `BEFORE_METHOD`인 경우 현재테스트 이후(default method mode)
    ```java
    @DirtiesContext 
    @Test
    void testProcessWhichDirtiesAppCtx() {
        // some logic that results in the Spring container being dirtied
    }
    ```

만약 `@DirtiesContext`를 `@ContextHierarchy`를 사용한 컨텍스트 계층 구조의 일부에 사용했다면, `hierarchyMode`를 사용하여 컨텍스트 캐시를 어떻게 지울건지 조절할 수 있다. 기본적으로, 현재 레벨뿐만아니라 현재 테스트에서 공통적으로 사용되는 상위 컨텍스트를 모두 지운다(`EXHAUSTIVE`). `CURRENT_LEVEL`의 경우 현재 레벨 컨텍스트의 하위 계층을 지운다.

```java
@ContextHierarchy({
    @ContextConfiguration("/parent-config.xml"),
    @ContextConfiguration("/child-config.xml")
})
class BaseTests {
    // class body...
}

class ExtendedTests extends BaseTests {

    @Test
    @DirtiesContext(hierarchyMode = CURRENT_LEVEL) 
    void test() {
        // some logic that results in the child context being dirtied
    }
}
```

### `@TestExecutionListeners`

`TestContextManager`에 등록해야 하는 `TestExecutionListener`의 구현을 구성하기 위한 클래스 레벨 메타데이터를 정의한다. 일반적으로 `@TestExecutionListener`는 `@ContextConfiguration`과 함께 사용한다.

다음은 2개의 `TestExecutionListener`를 등록하는 예시다.
```java
@ContextConfiguration
@TestExecutionListeners({CustomTestExecutionListener.class, AnotherTestExecutionListener.class}) 
class CustomTestExecutionListenerTests {
    // class body...
}
```

### `@RecordApplicationEvents`

클래스 수준 어노테이션이다. 하나의 테스트를 실행하는 중에 발생한 모든 애플리케이션 이벤트를 기록하도록 Spring TestContext 프레임워크에 지시하는데 사용된다.

기록된 이벤트는 테스트 안에서 `ApplicationEvents` API를 통해 접근할 수 있다.

### `@Commit`

트랜잭션 테스트 메서드에서 테스트 메서드가 완료된 후 트랜잭션이 커밋되는 것을 나타내는 어노테이션이다. `@Rollback(false)`를 대체하여 코드의 의도를 더 명확하게 나타낼 수 있다.

```java
@Commit 
@Test
void testProcessWithoutRollback() {
    // ...
}
```

### `@Rollback`

테스트 메서드가 완료된 후 트랜젝션을 롤백해야하는지 여부를 나타낸다. `true`인 경우 롤백되고 아니면 커밋된다. 명시적으로 선언되지 않으면 기본값은 `true`로 롤백된다.

클래스 수준으로 선언되면 `@Rollback`은 테스트 클래스 계층 구조안의 모든 메서드는 기본적으로 롤백이된다는 것을 정의한다. 그리고 메서드 단위로 `@Rollback`을 오버라이딩 할 수 있다.

```java
@Rollback(false) 
@Test
void testProcessWithoutRollback() {
    // ...
}
```

### `@BeforeTransaction`

`@Transaction` 주석을 사용하여 트랜잭션 내에서 실행되는 테스트 메서드를 실행하기 전에 `@BeforeTransaction`이 달린 메서드를 먼저 실행한다.

```java
@BeforeTransaction 
void beforeTransaction() {
    // logic to be run before a transaction is started
}
```

### `@AfterTransaction`

`@Transaction` 주석을 사용하여 트랜잭션 내에서 실행되는 테스트 메서드를 실행한 후에 `@AfterTransaction`이 달린 메서드를 실행한다.

```java
@AfterTransaction 
void afterTransaction() {
    // logic to be run after a transaction has ended
}
```

### `@Sql`

테스트 클래스 또는 테스트 메서드에 주석을 명시할 수 있다. 테스트 클래스에 명시할 경우 각 테스트 메서드 이전에 실행된다.

```java
@Test
@Sql({"/test-schema.sql", "/test-user-data.sql"}) 
void userTest() {
    // run code that relies on the test schema and test data
}
```

### `@SqlConfig`

`@Sql`로 구성된 SQL 스크립트를 파싱하고 실행하는 방법을 결정하는데 사용하는 메타데이터를 정의한다. 

아래는 주석의 prefix와 separator를 설정하는 예시다.

```java
@Test
@Sql(
    scripts = "/test-user-data.sql",
    config = @SqlConfig(commentPrefix = "`", separator = "@@") 
)
void userTest() {
    // run code that relies on the test data
}
```

### `@SqlMergeMode`

메서드 수준 `@Sql`선언과 클래스 수준 `@Sql`선언이 합쳐지는지 여부를 지정할 수 있다. 명시하지 않은 경우는 `OVERRIDE` 모드가 사용된다. `OVERRIDE`모드는 메서드 수준 `@Sql`선언이 클래스 수준 `@Sql`선언을 효율적으로 재정의한다.

`@SqlMergeMode`는 메서드 수준이 클래스 수준을 오버라이드할 수 있다.

아래는 모든 테스트 메서드가 `MERGE` 모드로 설정되는 예시다.

```java
@SpringJUnitConfig(TestConfig.class)
@Sql("/test-schema.sql")
@SqlMergeMode(MERGE) 
class UserTests {

    @Test
    @Sql("/user-test-data-001.sql")
    void standardUserProfile() {
        // run code that relies on test data set 001
    }
}
```

아래 예시는 특정 테스트 메서드에서 `MERGE` 코드로 설정되는 예시다.

```java
@SpringJUnitConfig(TestConfig.class)
@Sql("/test-schema.sql")
class UserTests {

    @Test
    @Sql("/user-test-data-001.sql")
    @SqlMergeMode(MERGE) 
    void standardUserProfile() {
        // run code that relies on test data set 001
    }
}
```

### `@SqlGroup`

여러 `@Sql` 어노테이션을 집계하는 컨테이너 어노테이션이다. Java 8의 경우는 Repeatable Annotations 때문에 선택적으로 사용하게된다. 여기서 `@Sql`은 동일한 클래스 또는 메서드에서 여러번 선언되어 이 컨테이너 어노테이션을 생성할 수 있다.

```java
@Test
@SqlGroup({ 
    @Sql(scripts = "/test-schema.sql", config = @SqlConfig(commentPrefix = "`")),
    @Sql("/test-user-data.sql")
)}
void userTest() {
    // run code that uses the test schema and test data
}
```

## 표준 어노테이션 지원

다음 어노테이션은 테스트에 한정되지 않고 모든 곳에서 사용할 수 있다.

- `@Autowired`

- `@Qualifier`

- `@Value`

- `@Resource` (javax.annotation) if JSR-250 is present

- `@ManagedBean` (javax.annotation) if JSR-250 is present

- `@Inject` (javax.inject) if JSR-330 is present

- `@Named` (javax.inject) if JSR-330 is present

- `@PersistenceContext` (javax.persistence) if JPA is present

- `@PersistenceUnit` (javax.persistence) if JPA is present

- `@Required`

- `@Transactional` (org.springframework.transaction.annotation) with limited attribute support

## WebTestClient

`WebTestClient`는 서버 애플리케이션을 테스트하기위해 설계된 HTTP 클라이언트다. 이는 Spring의 `WebClient`를 래핑하고 요청을 수행하지만 응답 확인을 위한 퍼사드를 노출한다. `WebTestClient`는 end-to-end HTTP test를 할 때 사용할 수 있다. 

### Setup

`WebTestClient`를 Setup하려면 바인딩할 서버 Setup을 선택해야된다. 이 때 mock 서버를 사용할 수도 있고 라이브 서버일 수도 있다.

#### Controller 바인딩

서버 실행 없이 mock request와 response 객체를 통해 특정 컨트롤러들을 테스트할 수 있다.

```java
WebTestClient client =
        WebTestClient.bindToController(new TestController()).build();
```

#### ApplicationContext 바인딩

configuration을 로드하고 서버 실행 없이 mock request와 response 객체를 통해 테스트할 수 있다.

```java
@SpringJUnitConfig(WebConfig.class) 
class MyTests {

    WebTestClient client;

    @BeforeEach
    void setUp(ApplicationContext context) {  
        client = WebTestClient.bindToApplicationContext(context).build(); 
    }
}
```

#### 서버 바인딩

실행중인 서버에 연결한다.

```java
client = WebTestClient.bindToServer().baseUrl("http://localhost:8080").build();
```

#### Client 설정

기본 URL, 기본 헤더, 클라이언트 필터 등을 포함한 클라이언트 옵션을 구성할 수 있다. `bindToServer()`에 이어서 사용할 수 있다. 다른 경우는 `configureClient()`를 사용하여 서버에서 클라이언트 구성으로 전환해야된다.

```java
client = WebTestClient.bindToController(new TestController())
        .configureClient()
        .baseUrl("/test")
        .build();
```

### 테스트 작성

`WebTestClient`는 `exchange()`를 사용하여 요청을 수행하는 시점까지 `WebCLient`와 동일한 API를 제공한다.

응답 상태와 헤더를 테스트하는 방법은 다음과 같다.

```java
client.get().uri("/persons/1")
            .accept(MediaType.APPLICATION_JSON)
            .exchange()
            .expectStatus().isOk()
            .expectHeader().contentType(MediaType.APPLICATION_JSON)
```

응답 바디는 다음 중 하나로 선택할 수 있다.

- `expectBody(Class<T>)`: 하나의 객체로 디코딩
- `expectBodyList(Class<T>)`: `List<T>`로 디코딩
- `expectBody()`: JSON을 `byte[]`로 디코딩

```java
client.get().uri("/persons")
        .exchange()
        .expectStatus().isOk()
        .expectBodyList(Person.class).hasSize(3).contains(person);
```

만약 built-in assertion이 비효율적이면, 다음과 같이 사용할 수도 있다.

```java
import org.springframework.test.web.reactive.server.expectBody

client.get().uri("/persons/1")
        .exchange()
        .expectStatus().isOk()
        .expectBody(Person.class)
        .consumeWith(result -> {
            // custom assertions (e.g. AssertJ)...
        });
```

`EntityExchangeResult` 로 결과를 얻을 수도 있다.

```java
EntityExchangeResult<Person> result = client.get().uri("/persons/1")
        .exchange()
        .expectStatus().isOk()
        .expectBody(Person.class)
        .returnResult();
```

#### No Content

만약 응답에 content가 미어있다고 예상되면 다음 방법이 있다.

```java
client.post().uri("/persons")
        .body(personMono, Person.class)
        .exchange()
        .expectStatus().isCreated()
        .expectBody().isEmpty();
```

만약 응답 content르 무시하고 싶으면 다음 방법이 있다.

```java
client.get().uri("/persons/123")
        .exchange()
        .expectStatus().isNotFound()
        .expectBody(Void.class);

```

#### JSON Content

full JSON content를 검증하는 방법은 다음이 있다.

```java
client.get().uri("/persons/1")
        .exchange()
        .expectStatus().isOk()
        .expectBody()
        .json("{\"name\":\"Jane\"}")
```

JSON content의 읿부 경로를 검증하는 방법은 다음이 있다.

```java
client.get().uri("/persons")
        .exchange()
        .expectStatus().isOk()
        .expectBody()
        .jsonPath("$[0].name").isEqualTo("Jane")
        .jsonPath("$[1].name").isEqualTo("Jason");
```

## MockMvc

컨트롤러를 인스턴스화하여 의존성을 주입하고 메서드를 호출하여 Spring MVC에대한 단위 테스트를 할 수 있다. 이 Spring MVC 테스트 프레임워크는 서버 실행 없이 Spring MVC Controller에 완벽한 테스트를 하는 것을 목표로한다. `MockMvc`는 `DispatcherServlet`을 호출하고 Servlet API의 mock 구현을 전달한다. 이 mock 구현은 서버 실행 없이 완전한 Spring MVC 요청 처리를 복제하는 `spring-test` 모듈로부터 얻을 수 있다.

### Static Imports

MockMvc를 사용하여 요청을 수행할 경우 다음의 static import가 필요하다.

- `MockMvcBuilders.*`
- `MockMvcRequestBuilders.*`
- `MockMvcResultMatchers.*`
- `MockMvcResultHandlers.*`

### Setup 방법 선택

MockMvc는 두 가지 방법으로 setup할 수 있다. 하나는 테스트할 컨트롤러를 직접 가리키고 Srping MVC infrastructure를 프로그래밍 방식으로 구성하는 것이다. 두 번째는 Spring MVC와 컨트롤러 infrastructre가 포함된 configuration을 가리키는 방법이다. 

특정 컨트롤러를 테스트하기 위한 MockMvc Setup 방법이다.

```java
class MyWebTests {

    MockMvc mockMvc;

    @BeforeEach
    void setup() {
        this.mockMvc = MockMvcBuilders.standaloneSetup(new AccountController()).build();
    }

    // ...

}
```

Spring configuration을 통해 MockMvc Setup 방법이다.

```java
@SpringJUnitWebConfig(locations = "my-servlet-context.xml")
class MyWebTests {

    MockMvc mockMvc;

    @BeforeEach
    void setup(WebApplicationContext wac) {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
    }

    // ...

}
```

### Setup Features

어떤 MockMvc Builder를 사용하든 `MockMbcBuilder`의 구현은 몇가지 유용한 기능을 제공한다. 

예를들어, 다음과 같이 모든 요청에 대해 `Accept` 헤더를 선언하고 모든 응답이 `200`인 상태와 `Content-Type` 헤더의 예상을 할 수 있다.

```java
// static import of MockMvcBuilders.standaloneSetup

MockMvc mockMvc = standaloneSetup(new MusicController())
    .defaultRequest(get("/").accept(MediaType.APPLICATION_JSON))
    .alwaysExpect(status().isOk())
    .alwaysExpect(content().contentType("application/json;charset=UTF-8"))
    .build();
```

### 요청 보내기

MockMvc가 요청을 보내고 응답을 검증하는 방법을 보여준다.

HTTP 메서드를 사용하여 요청을 보내는 방법은 다음과 같다.

```java
// static import of MockMvcRequestBuilders.*

mockMvc.perform(post("/hotels/{id}", 42).accept(MediaType.APPLICATION_JSON));
```

내부적으로 `MockMultipartHttpServletRequest`를 사용하여 파일의 업로드 요청을 보낼 수도 있다.

```java
mockMvc.perform(multipart("/doc").file("a1", "ABC".getBytes("UTF-8")));
```

URI template sylte로 쿼리 파라미터를 명시할 수도 있다.

```java
mockMvc.perform(get("/hotels?thing={thing}", "somewhere"));
```

form 파라미터으로도 표현할 수 있다.

```java
mockMvc.perform(get("/hotels").param("thing", "somewhere"));
```

여기서 URI template로 명시하는 파라미터는 디코딩이 되지만, 그냥 보내는 request 파라미터는 디코딩 되어있는 것으로 처리한다.

대부분의 경우 컨텍스트 경로와 서블릿 경로를 따로 명시하는 것이 좋다.

- 컨텍스트 경로: 웹 애플리케이션의 경로
- 서블릿 경로: 웹 애플리케이션에서 서블릿의 경로

전체 요청 URI로 테스트해야 되는 경우 다음과 같이 컨텍스트 경로와 서블릿 경로를 지정해준다.

```java
mockMvc.perform(get("/app/main/hotels/{id}").contextPath("/app").servletPath("/main"))
```

모든 요청을 보낼 때의 `contextPath`와 `servletPath`을 지정하고 싶으면 다음과 같이 setUp 할 수 있다.

```java
class MyWebTests {

    MockMvc mockMvc;

    @BeforeEach
    void setup() {
        mockMvc = standaloneSetup(new AccountController())
            .defaultRequest(get("/")
            .contextPath("/app").servletPath("/main")
            .accept(MediaType.APPLICATION_JSON)).build();
    }
}
```

### 기댓값 정의

`.andExpect(..)`를 사용하여 기댓값을 지정할 수 있다.

```java
// static import of MockMvcRequestBuilders.* and MockMvcResultMatchers.*

mockMvc.perform(get("/accounts/1")).andExpect(status().isOk());
```

`MockMvcResultmatchers.*`는 많은 기댓값을 지정할 수 있도록 제공한다.

기대값은 두 가지로 나눌 수 있다. 첫 번째는 응답의 속성을 확인하는 것이고, 두 번째는 응답을 넘어선 확인을 한다. 요청을 처리한 컨트롤러 메서드, 예외 발생 및 처리 여부, 모델의 내용, 선택된 뷰, 추가 된 플래스 속성 등과 같은 Spring MVC의 특정 수치를 검사할 수 있다.

다음 예시는 바인딩 또는 유효성 검사가 실패했다고 예측한다.

```java
mockMvc.perform(post("/persons"))
    .andExpect(status().isOk())
    .andExpect(model().attributeHasErrors("person"));
```

요청의 결과를 덤프할 수도 있다.

```java
mockMvc.perform(post("/persons"))
    .andDo(print())
    .andExpect(status().isOk())
    .andExpect(model().attributeHasErrors("person"));
```

경우에 따라 결과에 직접 접근하여 확인할 수 없는 항목들을 확인할 수 있다.

```java
MvcResult mvcResult = mockMvc.perform(post("/persons")).andExpect(status().isOk()).andReturn();
// ...
```

공통되는 예측이 있다면, `MockMvc`를 빌드할 때 예측을 Setup 할 수 있다. 이 때 예측은 오버라이딩 할 수 없다.

```java
standaloneSetup(new SimpleController())
    .alwaysExpect(status().isOk())
    .alwaysExpect(content().contentType("application/json;charset=UTF-8"))
    .build()
```

JSON 응답일 경우 JsonPath 표현식을 사용하여 결과를 검증할 수 있다.

```java
mockMvc.perform(get("/people").accept(MediaType.APPLICATION_JSON))
    .andExpect(jsonPath("$.links[?(@.rel == 'self')].href").value("http://localhost:8080/people"));
```

XML 응답일 경우 XPath 표현식을 사용하여 결과를 검증할 수 있다.

```java
Map<String, String> ns = Collections.singletonMap("ns", "http://www.w3.org/2005/Atom");
mockMvc.perform(get("/handle").accept(MediaType.APPLICATION_XML))
    .andExpect(xpath("/person/ns:link[@rel='self']/@href", ns).string("http://localhost:8080/people"));
```

### MockMvc vs End-to-End 테스트

MockMvc는 `spring-test` 모듈의 Servlet API mock 구현을 기반으로 실행중인 컨테이너에 의존하지 않는다. 따라서 실제 클라이언트와 실행중인 라이브 서버를 사용한 end-to-end 테스트와 차이가 있다.

각 접근 방식에는 장단점이 있다. Spring MVC Test에서 제공하는 것은 고전적인 단위 테스트 범주위 속하지는 않지만 단위 테스트에 가까운 것이 있다. 예를 들어, 컨트롤러에 Mock 서비스를 주입하여 웹 게층을 분리 할 수 있다. 이 경우 데이터 접근 게층을 위의 게층과 분리하여 테스트할 수 있다.

Spring MVC Test를 사용할 때 또 다른 종요한 차이점은 이런 테스트는 서버 사이드이므로 어떤 핸들러가 사용되었는지, `HandlerExcetpionResolver`로 예외가 처리되었는지 ,모델의 내용이 무엇인지, 어떤 바인딩 오류가 있는지 확인할 수 있는 것이다. 따라서 단위 테스트는 작성, 추론, 디버깅이 더 쉽다. 하지만 응답이 확인해야 할 가장 중요한 것이라는 점에서 통합 테스트도 필요하다.

## 참고 자료
<https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#integration-testing>

<https://www.concretepage.com/spring-5/dirtiescontext-example-spring-test#hierarchyMode>