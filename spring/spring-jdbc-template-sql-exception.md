spring `JdbcTemplate` 은 sql exception 을 어떻게 처리할까?

`JdbcTemplate` 을 사용하면 필연적으로 SqlException  이 발생한다.

```java
public void deleteAll() throws SQLException {
    // ...
}

// getConnection, prepareStatement, executeUpdate 모두 SQLException 발생
```

그러나 spring `JdbcTemplate` 을 활용하면 SqlException 을 볼 수 없다.

```java
public void deleteAll() {
    // ...
}
```

스프링의 `JdbcTemplate` 내부 코드를 들어가보면 모든 메서드가 다음과 같이 `RuntimeException` 을 상속한 `DataAccessException` 을 사용한다.

```java
public abstract class DataAccessException extends NestedRuntimeException { 
    public DataAccessException(String msg) { super(msg); } 
    
    public DataAccessException(@Nullable String msg, @Nullable Throwable cause) { super(msg, cause); } 
}
```

