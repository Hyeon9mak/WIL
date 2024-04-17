## 문제 상황
spring + postgresql 을 이용하는 환경에서 jdbcTemplate 을 이용하여 `jdbcTemplate.update(...)` 을 호출하는 경우, null parameter 를 주입하면 내부 assertion 에 걸리게 된다.

java assertion 특성상 실제 프로그램이 동작할 때는 문제가 되지 않지만, 테스트를 통해 동작을 하게 되는 경우 `java.lang.AssertionError` 이 발생하게 된다.

```
java.lang.AssertionError: Simple mode does not support describe requests.
```

## 정확한 원인
stack trace 를 타고 가보면 에러가 발생하는 지점은 아래와 같았다.

```java
// org.postgresql.core.v3.QueryExecutorImpl.sendOneQuery

private void sendOneQuery(SimpleQuery query, SimpleParameterList params, int maxRows,  
    int fetchSize, int flags) throws IOException {  
  boolean asSimple = (flags & QueryExecutor.QUERY_EXECUTE_AS_SIMPLE) != 0;  
  if (asSimple) {  
    assert (flags & QueryExecutor.QUERY_DESCRIBE_ONLY) == 0  
        : "Simple mode does not support describe requests. sql = " + query.getNativeSql()  
        + ", flags = " + flags;  
    sendSimpleQuery(query, params);  
    return;  
  }

  // ...
}
```

AssertionError 를 발생 시킨 직접적 원인을 제공한 서비스 코드는 아래와 같다.

```java
String query = String.format("UPDATE %s SET status = ?, version = ? WHERE id = ?", TABLE_NAME);  
  
jdbcTemplate.update(  
        query,  
        object.getStatus(),  
        object.getVersion() == null ? null : object.getVersion().value()
);
```

`version` 이 `null` 이면 그대로 `null` 을, 아니라면 값을 채우는 코드다.
그런데 `jdbcTemplate.update` 의 파라미터로 null 을 넘기게 되면, null 값이 생략되어 버린다.

![](https://i.imgur.com/PokIokP.png)

(아마도) 다른 jdbc 구현체에서는 이를 핸들링 할 수 있으나, postgresql 에서는 이를 별도로 핸들링해주지 않아 assertion 에 걸려 error 를 발생시키는 것으로 보인다.

## 해결책

query 자체를 새로 작성하는 것으로 문제를 해결했다.

```java
String versionOrNull = object.getVersion() == null ? null : "'" + object.getVersion().value() + "'";

String query = String.format("UPDATE %s SET status = ?, version = %s WHERE id = ?", TABLE_NAME, versionOrNull);  
  
jdbcTemplate.update(  
        query,  
        object.getStatus()
);
```
