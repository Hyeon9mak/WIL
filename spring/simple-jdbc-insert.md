# SimpleJdbcInsert 실습
아래는 새로 생성된 Station 객체를 데이터베이스의 STATION 테이블에 삽입하는 코드다.

```java
    private final JdbcTemplate jdbcTemplate;

    public StationDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public void insert(final Station station) {
        String sql = "INSERT INTO station(name) VALUES(?)";
        jdbcTemplate.update(sql, station.getName());
    }
```

JdbcTemplate를 사용하면서 데이터베이스에 값을 직접 삽입하다보면, 삽입 직후 조회(READ) 과정없이 곧장 
삽입된 데이터의 primary key 값을 얻고 싶을 때가 있다.  

JdbcTemplate만을 사용해서도 이를 충분히 구현할 수 있지만, 코드의 가독성이 기하급수 적으로 떨어진다.

## KeyHolder 사용 방식

```java
    private final JdbcTemplate jdbcTemplate;

    public StationDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public Station insert(final Station station) {
        String sql = "INSERT INTO station(name) VALUES(?)";

        KeyHolder keyHolder = new GeneratedKeyHolder();
        this.jdbcTemplate.update(connection -> {
            PreparedStatement ps = connection
                .prepareStatement(sql, new String[]{"id"});
            ps.setString(1, station.getName());
            return ps;
        }, keyHolder);

        return new Station(keyHolder.getKey().longValue(),
            station.getName());
    }
```

try-catch 키워드까지 추가되면 가독성이 많이 떨어진다.

```java
    private final JdbcTemplate jdbcTemplate;

    public StationDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public Station insert(final Station station) {
        try {
            String sql = "INSERT INTO station(name) VALUES(?)";

            KeyHolder keyHolder = new GeneratedKeyHolder();
            this.jdbcTemplate.update(connection -> {
                PreparedStatement ps = connection
                    .prepareStatement(sql, new String[]{"id"});
                ps.setString(1, station.getName());
                return ps;
            }, keyHolder);

            return new Station(keyHolder.getKey().longValue(),
                station.getName());
        } catch (DuplicateKeyException e) {
            ...
        }
    }
```

<br>

## SimpleJdbcInsert 사용 방식
```java
    private final JdbcTemplate jdbcTemplate;
    private final SimpleJdbcInsert jdbcInsert;

    public StationDao(JdbcTemplate jdbcTemplate, DataSource source) {
        this.jdbcTemplate = jdbcTemplate;
        this.jdbcInsert = new SimpleJdbcInsert(source)
            .withTableName("STATION")
            .usingGeneratedKeyColumns("id");
    }


    public Station insert(Station station) {
        try {
            Map<String, String> params = new HashMap<>();
            params.put("name", station.getName());
            Long id = jdbcInsert.executeAndReturnKey(params).longValue();

            return new Station(id, station.getName());
        } catch (DuplicationKeyException e) {
            ...
        }
    }
```

Map 자료구조로 필요한 파라미터 값들을 미리 대입시키고, `SimpleJdbcInsert.executeAndReturnKey(Map).longValue()` 를 수행하면 곧장 primary key 값을 얻어낼 수 있다.

이 동작이 가능하게 한 핵심은 생성자 쪽의 `withTableName()`과 `usingGeneratedKeyColumns`다.

```java
    public StationDao(JdbcTemplate jdbcTemplate, DataSource source) {
        this.jdbcTemplate = jdbcTemplate;
        this.jdbcInsert = new SimpleJdbcInsert(source) // 데이터소스로 DB에 접근해라.
            .withTableName("STATION")                  // 'STATION' 테이블에 삽입해라.
            .usingGeneratedKeyColumns("id");           // 'id' 컬럼의 값을 key로 반환해라.
    }
```
