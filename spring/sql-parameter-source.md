# SqlParameterSource 인터페이스 실습
`SqlParameterSource`는 `SimpleJdbcInsert`를 사용할 때 함께 활용하기 좋은 인터페이스다.
스프링에서 `SqlParameterSource`의 구현체를 다수 제공하고 있다. 그 중에서 자주 사용되는 2가지를 알아보자.

<br>

## SqlParameterSource - MapSqlParameterSource
```java
    public Station insert(Station station) {
        try {
            SqlParameterSource params = new MapSqlParameterSource()
                .addValue("name", station.getName())
                .addValue(..., ...);
            Long id = jdbcInsert.executeAndReturnKey(params).longValue();

            return new Station(id, station.getName());
        } catch (DuplicationKeyException e) {
            ...
        }
    }
```
`MapSqlParameterSource`는 Map 과 비슷하지만, `addValue` 메서드를 통해 체인으로 연결할 수 있어 
더 편리하다.

<br>

## SqlParameterSource - BeanPropertySqlParameterSource
```java
    public Station insert(Station station) {
        try {
            SqlParameterSource params = new BeanPropertySqlParameterSource(station);
            Long id = jdbcInsert.executeAndReturnKey(params).longValue();

            return new Station(id, station.getName());
        } catch (DuplicationKeyException e) {
            ...
        }
    }
```

공식 문서에는 Java Bean 호환(생성자, getter/setter를 가진) 객체라면 `BeanPropertySqlParameterSource` 를 사용할 수 있다고 명시되어 있다.

`BeanPropertySqlParameterSource`는 객체의 필드 값을 추출하는데 같은 이름으로 대응되는 getter 메서드를 필요로 한다.

---

남는 의문점

- `class` 값은 무엇일까?
- null로 채워진 필드값은 어떻게 회피하고 데이터베이스에 삽입하지 않는가?
    - `BeanPropertySqlParameterSource.getValue()` 메서드 쪽에 @Nullable 어노테이션이 붙어있긴 했음
    - DAO 생성자에서 지정한 `SimpleJdbcInsert.usingGeneratedKeyColumns("id")` 덕분에 `id` 필드를 primary key 값으로 생각하고 회피하는건가? 

```java
            SqlParameterSource params = new BeanPropertySqlParameterSource(station);
            System.out.println("################################# START");
            System.out.println(Arrays.toString(params.getParameterNames()));
            System.out.println("################################# END");
```
```
################################# START
[class, id, name]
################################# END
```

출력 테스트에서도 `id, name` 필드 값이 넘어오는 걸 확인할 수 있었다.

6월 8일 내용 추가.

리뷰어 재연링께 질문 드리고 힌트를 얻을 수 있었다.

> 안녕하세요 현구막!  
> 저도 BeanPropertySqlParameterSource 에 대해 잘 몰라 찾아봤는데 ㅎㅎ  
>   ```java
> 	String sql = "insert into station(name) values (:name)";
> 	SqlParameterSource param = new BeanPropertySqlParameterSource(station);
> 	KeyHolder generatedKeyHolder = new GeneratedKeyHolder();
> 	namedParameterJdbcTemplate.update(sql, param, generatedKeyHolder);
> 	generatedKeyHolder.getKey().longValue();
>   ```

> !! 알려주신대로면 SimpleJdbcInsert 없이도 사용할 수 있겠군요..!  
> 위처럼 별도의 Sql 쿼리문을 지정해서 사용할 수 있다는 이야기는 SimpleJdbcInsert 의 설정 때문인게 확실시 되는거 같기도 하구요!  
> 감사합니다!!!

아무래도 DAO 생성자에서 지정한 `SimpleJdbcInsert.usingGeneratedKeyColumns("id")` 덕분에 `id` 
필드를 primary key 값으로 지정하고 회피하는 것이 맞는 것 같다.

<br>