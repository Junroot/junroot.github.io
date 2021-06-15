---
title:  "Spring JDBC"
categories: programming
tags: Java Spring
---

Spring JDBC 는 JDBC를 사용할 때 개발자가 처리해야되는 것의 일부를 대신 처리해준다.

|동작|Spring JDBC|You|
|----|--|--|
|connection parameters 정의| |x|
|connection 열기|x| |	
|SQL statement 명시| |x|
|파라미터 선언 및 파라미터의 값 제공| |x|
|statement 준비 및 실행|x| |	
|결과를 iterate하도록 루프 설정|x| |	
|iteration에 대한 작업|	|x|
|예외 처리|x| |	
|트랜잭션 처리|x| |	
|connection, statement, resultset 닫기|x| |

## JDBC 데이터베이스 엑세스를 위한 접근법 선택

- `JdbcTemplate`: 가장 많이 사용되고 고전적인 접근법. 다른 모든 접근 방식도 내부적으로 `JdbcTemplate`를 사용한다.
- `NamedParameterJdbcTemplate`: `JdbcTemplate`를 래핑하고 있다. JDBC의 `?` placeholder 대신에 이름이 있는 파라미터를 제공한다. 이 접근 방식은 SQL문에 여러 개의 파라미터가 있을 때 더 좋은 문서화와 사용성을 제공해준다.
- `SimpleJdbcInsert`, `SimpleJdbcCall`: 필요한 구성의 개수를 제한하는 데이터베이스의 메타 데이터를 최적화한다. 이 접근법은 테이블 또는 프로시저의 이름만 제공하고 열 이름과 일치하는 변수 맵을 제공하면되기 때문에 코딩을 단순화 시켜준다. 이는 데이터베이스가 적절한 메타 데이터를 제공하는 경우메나 작동한다. 만약 데이터베이스가 메타 데이터를 제공하지 않으면 파라미터의 명시적인 구성을 제공해야된다.
- `MappingSqlQuery`, `SqlUpdate`, `StoreProcedure`: 다음과 같은 RDBMS 객체는 data access layer의 초기화 중에 재사용 가능 및 스레드 안전한 객체를 만들어야된다. 이 접근 방식은 쿼리 문자열을 정의하고 매개 변수를 선언하고 쿼리를 컴파일하는 JDO 쿼리를 모델로한다. `execute(...)`, `update(...)`, `findObject(...)` 메서드들을 사용하면 다양한 파라미터 값으로 여러번 호출될 수 있다.

## JdbcTemplate 사용

`JdbcTemplate`는 리소스 생성 및 해제를 처리해주므로 connection close를 잊는 것과 같은 일반적인 실수를 방지할 수 있다. 이는 아래의 내용을 수행합니다.

- SQL query 실행
- statments 업데이트 및 프로시저 호출
- `ResultSet`인스턴스 반복 및 반환 된 매개 변수 값 추출을 수행한다.
- JDBC 에외를 포착하여 `org.springframework.dao`패키지에 정의된 일반적이고 좀 더 많은 정보를 제공하는 계층 구조로 변환한다.

DAO 구현에 직접 `Datasource` 를 참조하는 인스턴스를 만들어 `JdbcTemplate`를 사용하거나 Spring IoC 컨테이터에서 설정하여 Dao에 Bean 참조를 제공할 수 있다.

### Querying(SELECT)

- 다음 쿼리는 row의 개수를 가져옵니다.

```java
int rowCount = this.jdbcTemplate.queryForObject("select count(*) from t_actor", Integer.class);
```

- 다음 쿼리는 변수를 바인딩합니다.

```java
int countOfActorsNamedJoe = this.jdbcTemplate.queryForObject(
        "select count(*) from t_actor where first_name = ?", Integer.class, "Joe");
```

- 다음 쿼리는 `String`을 구합니다.

```java
String lastName = this.jdbcTemplate.queryForObject(
        "select last_name from t_actor where id = ?",
        String.class, 1212L);
```

- 다음 쿼리는 단일 도메인 객체를 찾아서 채웁니다.

```java
Actor actor = jdbcTemplate.queryForObject(
        "select first_name, last_name from t_actor where id = ?",
        (resultSet, rowNum) -> {
            Actor newActor = new Actor();
            newActor.setFirstName(resultSet.getString("first_name"));
            newActor.setLastName(resultSet.getString("last_name"));
            return newActor;
        },
        1212L);
```

- 다음과 같이 도메인 객체를 찾아서 채우는 과정을 단일 필드로 추출할 수도 있습니다.

```java
private final RowMapper<Actor> actorRowMapper = (resultSet, rowNum) -> {
    Actor actor = new Actor();
    actor.setFirstName(resultSet.getString("first_name"));
    actor.setLastName(resultSet.getString("last_name"));
    return actor;
};

public List<Actor> findAllActors() {
    return this.jdbcTemplate.query( "select first_name, last_name from t_actor", actorRowMapper);
}
```

- 행이 여러개인 경우 `Map`의 `List` 형태로 반환 시킬 수 있다.

```java
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate(dataSource);
}

public List<Map<String, Object>> getList() {
    return this.jdbcTemplate.queryForList("select * from mytable");
}
```

### Updating(INSERT, UPDATE, DELETE)

매개 변수 값을 변수 인자로 제공하거나 객체의 배열로 제공할 수 있습니다.

- 다음은 새 항목을 삽입합니다

```java
this.jdbcTemplate.update(
        "insert into t_actor (first_name, last_name) values (?, ?)",
        "Leonor", "Watling");
```

- 다음은 기존 항목을 업데이트합니다.

```java
this.jdbcTemplate.update(
        "update t_actor set last_name = ? where id = ?",
        "Banjo", 5276L);
```

- 다음은 항목을 삭제합니다.

```java
this.jdbcTemplate.update(
        "delete from t_actor where id = ?",
        Long.valueOf(actorId));
```

- 자동 생성된 키 검색

```java
final String INSERT_SQL = "insert into my_test (name) values(?)";
final String name = "Rob";

KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.update(connection -> {
    PreparedStatement ps = connection.prepareStatement(INSERT_SQL, new String[] { "id" });
    ps.setString(1, name);
    return ps;
}, keyHolder);

// keyHolder.getKey() now contains the generated key
```

### 기타 작업

`execute(...)`메서드를 사용하여 임의의 SQL을 실행할 수 있습니다. 이 메서드는 DDL 문에 자주 사용됩니다. 콜백 인터페이스, 변수 배열 바인딩 등의 처리로 인해 과부하게 많이 걸린다.

- 아래는 테이블을 생성한다.

```java
this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
```

- 다음은 저장되어 있는 프로시저를 호출한다.

```java
this.jdbcTemplate.update(
        "call SUPPORT.REFRESH_ACTORS_SUMMARY(?)",
        Long.valueOf(unionId));
```

### JdbcTemplate의 모범 사례

`JdbcTemplate`클래스의 인스턴스는 스레드로부터 안전하다. 이 말은 `JdbcTemplate`의 인스턴스를 하나만 구성하고 여러 개의 DAO에 주입하는 인스턴스를 공유해도 문제가 없다는 의미다. `JdbcTemplate` 는 상태 보존적이다. 따라서 DataSource의 참조를 유지한다.

`JdbcTemplate` 클래스를 사용할 때 일반적인 관행은 `DataSource`빈을 DAO 클래스에 주입하는 형태다. `DataSource`만 있으면 다른 JDBC 객체도 만들 수 있기 때문이다.

```java
public class JdbcCorporateEventDao implements CorporateEventDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    // JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

어노테이션을 사용하는 방법도 있다. DAO 클래스에 `@Repository`를 추가하고 DataSource의 setter 메서드에 `@Autowired`를 지정한다.

```java
@Repository 
public class JdbcCorporateEventDao implements CorporateEventDao {

    private JdbcTemplate jdbcTemplate;

    @Autowired 
    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource); 
    }

    // JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

## NamedParameterJdbcTemplate 사용

- `JdbcTemplate`를 래핑하고 있다. JDBC의 `?` placeholder 대신에 이름이 있는 파라미터를 제공한다. 아래 예시와 같이 `MapSqlParameterSource`나 `Java.util.Map`을 통해 파라미터 값을 설정할 수 있다.

```java
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActorsByFirstName(String firstName) {

    String sql = "select count(*) from T_ACTOR where first_name = :first_name";

    SqlParameterSource namedParameters = new MapSqlParameterSource("first_name", firstName);

    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
}
```

```java
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActorsByFirstName(String firstName) {

    String sql = "select count(*) from T_ACTOR where first_name = :first_name";

    Map<String, String> namedParameters = Collections.singletonMap("first_name", firstName);

    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters,  Integer.class);
}
```

- 또 다른 `SqlParameterSource`의 구현으로 `BeanPropertySqlParameterSource`가 있다. 이 클래스는 임의의 JavaBean을 래핑하고 래핑된 JavaBean의 속석을 명시된 매개변수 값의 소스로 사용한다.

```java
public class Actor {

    private Long id;
    private String firstName;
    private String lastName;

    public String getFirstName() {
        return this.firstName;
    }

    public String getLastName() {
        return this.lastName;
    }

    public Long getId() {
        return this.id;
    }

    // setters omitted...

}
```

```java
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActors(Actor exampleActor) {

    // notice how the named parameters match the properties of the above 'Actor' class
    String sql = "select count(*) from T_ACTOR where first_name = :firstName and last_name = :lastName";

    SqlParameterSource namedParameters = new BeanPropertySqlParameterSource(exampleActor);

    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
}
```

## SQLExceptionTranslator 사용

`SQLExceptionTranslator`는 `SQLException`과 Spring 자체의 `org.springframework.dao.DataAccessException`사이를 변환 시킬 수 있는 클래스에 의해 구현되는 인터페이스다. `SQLErrorCodeSQLExceptionTranslator` 가 `SQLExceptionTranslator`의 기본적인 구현이다.

`SQLErrorCodeSQLExceptionTranslator`는 다음과 같은 순서로 규칙을 적용한다.

1. 하위 클래스에 의해 구현된 변환. 일반적으로 제공된 콘크리트 `SQLErrorCodeSQLExceptionTranslator`를 사용하므로 이 규칙은 적용되지 않는다. 만약 서브클래스 구현을 제공하면 이 규칙이 적용된다.
2. `SQLExceptionTranslator` 인터페이스의 커스텀 구현은 `SQLErrorCodes`의 프로퍼티인 `customSqlExceptionTranslator`에 제공된다.
3. `CustomSQLErrorCodesTranslation`(`SQLErrorCodes`의 `customTranslation`프로퍼티에 제공된)의 인스턴스의 리스트에 일치되는 항목을 검색한다.
4. 에러 코드 매칭이 적용된다.
5. fallback translator 사용한다. `SQLExceptionSubclassTranslator`가 기본 fallback translator다. 만약 변환을 사용할 수 없는 경우 다음 fallback traslator는 `SQLStateSQLExceptionTranslator`.

`SQLErrorCodeSQLExceptionTranslator`는 다음 예제와 같이 확장할 수 있다.

```java
public class CustomSQLErrorCodesTranslator extends SQLErrorCodeSQLExceptionTranslator {

    protected DataAccessException customTranslate(String task, String sql, SQLException sqlEx) {
        if (sqlEx.getErrorCode() == -12345) {
            return new DeadlockLoserDataAccessException(task, sqlEx);
        }
        return null;
    }
}
```

위의 예제는 오류코드 -12345가 변환되고, 다른 오류는 기본 변환기 구현에 의해 처리된다. 아래는 `JdbcTemplate`에서 커스텀 변환기를 사용하는 법이다.

```java
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {

    // create a JdbcTemplate and set data source
    this.jdbcTemplate = new JdbcTemplate();
    this.jdbcTemplate.setDataSource(dataSource);

    // create a custom translator and set the DataSource for the default translation lookup
    CustomSQLErrorCodesTranslator tr = new CustomSQLErrorCodesTranslator();
    tr.setDataSource(dataSource);
    this.jdbcTemplate.setExceptionTranslator(tr);

}

public void updateShippingCharge(long orderId, long pct) {
    // use the prepared JdbcTemplate for this update
    this.jdbcTemplate.update("update orders" +
        " set shipping_charge = shipping_charge * ? / 100" +
        " where id = ?", pct, orderId);
}
```

## 참고 자료

[https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc)