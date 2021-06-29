---
title:  "템플릿 콜백 패턴과 JdbcTempalte"
categories: programming
tags: Java Spring
---

## 템플릿 콜백 패턴

[전략 패턴](https://junroot.github.io/programming/%EC%A0%84%EB%9E%B5-%ED%8C%A8%ED%84%B4/)의 기본 구조에 익명 내부 클래스를 활용한 방식이다. 클라이언트가 템플릿 메소드를 호출하면서 콜백 오브젝트를 전달하는 방식으로 메소드 레벨에서 DI가 일어난다.

- 템플릿: 어떤 목적을 위해 미리 만들어둔 모양이 있는 틀을 가리킨다. 템플릿 메소드 패턴은 고정된 틀의 로직을 가진 템플릿 메소드를 슈퍼클래스에 두고, 바뀌는 부분을 서브 클래스의 메소드에 두는 구조로 이루어진다.
- 콜백: 실행되는 것을 목적으로 다른 오브젝트의 메소드에 전달되는 오브젝트를 말한다. 자바에선 메소드 자체를 파라미터로 넘기지 못하기 때문에 메소드가 담긴 오브젝틀르 넘긴다. 그래서 펑셔널 오브젝트라고도 한다.

데이터베이스의 User 테이블에 접근하는 `UserDao`클래스에 이 패턴을 적용해보자.

DB 커넥션에 필요한 공통적인 로직들을 처리하기 위한 `JdbcContext` 클래스를 만든다.

```java
public class JdbcContext {
    private final DataSource dataSource;

    public JdbcContext(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {

        try (Connection c = dataSource.getConnection();
            PreparedStatement ps = stmt.makePreparedStatement(c)) {

            ps.execute();
        }
    }
}
```

`StatementStrategy`는 `PreparedStatement`를 만들기 위한 전략으로 함수형 인터페이스다. 따라서 `StatementStrategy`가 템플릿 메서드인 `JdbcContext`의 `workWithStatementStrategy`의 콜백에 해당한다.

```java
public interface StatementStrategy {

    PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}
```

`UserDao`는 `JdbcContext`의 클라이언트가 되어 콜백을 전달하게된다.

```java
public class UserDao {

    private final JdbcContext jdbcContext;

    public UserDao(JdbcContext jdbcContext) {
        this.jdbcContext = jdbcContext;
    }

    public void add (User user) throws SQLException {
        jdbcContext.workWithStatementStrategy(
            (StatementStrategy) c -> {
                PreparedStatement ps = c.prepareStatement("insert into user(id, name, password) values(?, ?, ?)");

                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());

                return ps;
            }
        );
    }
}
```

이렇게 템플릿 콜백 패턴은 전략 패턴과 DI의 장점을 익명 클래스 사용 전략과 결합한 구조라고 이해할 수 있다. 이 패턴을 통해 ***변하는 것과 변하지 않는 것을 분리하고 변하지 않는 건 유연하게 재활용 할 수 있는 설계를 할 수 있게 된다.***

## JdbcTemplate

Spring은 JDBC를 이용하는 DAO에서 사용할 수 있도록 다양한 템플릿과 롤백을 지원한다. JDBC용 기본 템플릿이 `JdbcTemplate`다.

### PreparedStatementCreator

위 템플릿 콜백 패턴 예시에서 `StatementStrategy` 인터페이스 대신 `JdbcTemplate`에서는 `PreparedStatementCreator`를 사용한다.

```java
@Repository
public class UserDao {

    private final JdbcTemplate jdbcTemplate;

    public UserDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public void add (User user) {
        jdbcTemplate.update(new PreparedStatementCreator() {
            @Override
            public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
                PreparedStatement ps = con.prepareStatement("insert into user(id, name, password) values(?, ?, ?)");

                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());

                return ps;
            }
        });
    }
}
```

`JdbcTemplate`에서는 이처럼 `PreparedStatement`를 만드는 것과 파라미터를 바인딩하는 것을 좀 더 쉽게 처리하기 위해 다음의 메서드도 제공한다.

```java
jdbcTemplate.update("insert into user(id, name, password) values(?, ?, ?)", user.getId(), user.getName(), user.getPassword());
```

### ResultSetExtractor

`ResultSet`을 이용해서 원하는 값을 추춘하는 작업도 템플릿 콜백 패턴으로 제공하고 있다. 그 인터페이스가 `ResultSetExtractor`다. 여기서 `ResultSetExtractor`는 제네릭스 타입 파라미터를 갖는 다는 점을 눈여겨볼 수 있다.

```java
@Repository
public class UserDao {

    private final JdbcTemplate jdbcTemplate;

    public UserDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public void getCount (User user) {
        jdbcTemplate.query(new PreparedStatementCreator() {
            @Override
            public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
                return con.prepareStatement("select count(*) from user");
            }
        }, new ResultSetExtractor<Integer>() {
            @Override
            public Integer extractData(ResultSet rs) throws SQLException, DataAccessException {
                rs.next();
                return rs.getInt(1);
            }
        });
    }
}
```

`JdbcTemplate`에서는 이런 기능을 가진 콜백을 내장하고 있는 `queryForInt()`라는 편리한 메소드를 제공한다.

```java
jdbcTemplate.queryForInt("select count(*) from user");
```

### RowMapper

`ResultSet`을 이용해서 `Integer`같은 산순한 값이 아니라 복잡한 객체로 매핑하는 작업도 템플릿 콜백 패턴으로 제공하고 있다. 그 인터페이스가 `RowMapper`다. 예시 코드는 다음과 같다.

```java
@Repository
public class UserDao {

    private final JdbcTemplate jdbcTemplate;

    public UserDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public User get (String id) {
        jdbcTemplate.queryForObject("select * from user where id = ?",
            new Object[]{id},
            new RowMapper<User>() {
                @Override
                public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                    User user = new User();
                    user.setId(rs.getString("id"));
                    user.setName(rs.getString("name"));
                    user.setPassword(rs.getString("password"));

                    return user;
                }
            });
    }
}
```

## 참고 자료

토비의 스프링 3.1 (이일민)