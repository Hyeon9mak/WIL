# MySQL(MariaDB) Replication with JPA

## summary
### 개요
복제(Replication)는 한 서버에서 다른 서버로 데이터가 동기화 되는 것을 말하며, 원본 데이터를 가진 서버를 소스(Source/Master) 서버, 복제된 데이터를 가지는 서버를 레플리카(Replica/Slave) 서버라고 부른다. 소스 서버에서 데이터 및 스키마에 대한 변경이 최초로 발생하며, 레플리카 서버에서는 이러한 변경 내역을 소스 서버로부터 전달 받아 자기 데이터에 반영한다. 복제에는 여러가지 목적이 존재한다.

- 스케일 아웃: 서버 분리로 트래픽도 분산 시킨다.
- 데이터 백업: 레플리카 서버를 이용해 백업한다.
- 데이터 분석: 분석용 쿼리만 별도로 실행되는 서버(DB) 구축이 가능하다.
- 데이터의 지리적(물리적) 분산: 물리적 거리만큼의 통신속도를 개선할 수 있다.

### 복제 아키텍쳐
MySQL 서버에서 발생하는 모든 변경 사항은 별도의 로그 파일(Binary Log)에 순서대로 기록된다.
바이너리 로그에는 **데이터의 변경 내역뿐만 아니라 데이터베이스나 테이블의 구조 변경과 계정, 권한 변경 정보까지 모두** 저장된다.
바이너리 로그에 기록된 각 변경 정보를 이벤트(Event)라고 부른다. 

MySQL 서버의 복제는 이 바이너리 로그를 기반으로 구현된다. 소스 서버에서 생성된 바이너리 로그가 레플리카 서버로 전송되고,
레플리카 서버에서는 해당 내용을 저장(이렇게 저장된 파일을 Relay Log라 부름)한 뒤 자신이 가진 데이터에 반영한다.
소스서버의 바이너리 로그에 기록된 변경 내역들을 식별하는 방식에 따라 **로그 파일 위치 기반 복제(Binary Log File Position Based Replication)** 과 **글로벌 트랜잭션 ID 기반 복제(Global Transaction Identifiers Based Replication)** 으로 나뉜다.
이 글에서는 로그 파일 위치 기반 복제를 적용해본다.

<br>

## Master 서버 생성 및 설정
우선 Master 서버에서 CUD 작업 수행시 Slave 서버에 반영되도록 하는 연결 설정을 진행한다.
가장 먼저 Master 서버 터미널에서 vim을 통해 master db 서버의 설정 파일을 열어 주석처리된 설정을 해제한다.

```
$ vim /etc/mysql/mariadb.conf.d/50-server.cnf

server-id              = {id로 사용하고 싶은 정수 값}
log_bin                = /var/log/mysql/mysql-bin.log
expire_logs_days       = 10
max_binlog_size        = 100M

bind-address		= 0.0.0.0
```

`server_id`는 복제에 참여한 MySQL 서버들이 고유하게 가지고 있는 식별 값이다. 이 값은 바이너리 로그에 이벤트별로 이벤트가 최초로 발생한 MySQL을 식별하기 위해 사용되며, 기본값이 모두 1이다. 이 때문에 Master 서버와 Slave 서버를 구별할 수 있도록 Slave 서버의 `server_id`를 바꾸어 주는 것이다.

만일 Master 서버와 Slave 서버의 `server_id`값이 동일할 경우 Slave서버는 바이너리 로그 파일에 기록된 이벤트를 동기화 할 수 없다. 
Slave 서버는 자신이 발생시킨 이벤트로 간주해서 동기화를 시도하지 않기 때문이다. 
이 때문에 복제에 참여한 모든 MySQL 서버가 고유한 `server_id`를 갖도록 설정해야 한다.

`log_bin`은 앞서 복지 아키텍쳐에서 이야기한 바이너리 로그 파일의 형식을 뜻한다.

`bind-address`를 전체 개방해서, Slave 서버의 접근을 허용하도록 한다. (동시에 WAS 서버의 접근도 허용됨)

설정이 끝났다면 저장 후 `$ service mysqld restart`를 통해 설정을 적용해준다.
이후 Master 서버의 관리자 계정으로 데이터베이스 서버에 접속해서 Slave 서버가 사용할 계정을 생성한다.

```sql
$ mysql -u root -p
mysql> USE mysql;

mysql> CREATE USER '{유저 이름}'@'{% 또는 Slave IP}' IDENTIFIED BY '{유저 비밀번호}';  
mysql> GRANT REPLICATION SLAVE ON {* 또는 스키마 이름}.{* 또는 테이블 이름} TO '{유저 이름}'@'{% 또는 Slave IP}'; 
mysql> FLUSH PRIVILEGES;
```

`GRANT REPLICATION SLAVE` 명령에서 `SLAVE ON *.*`는 SLAVE 디비에게 모든 스키마에 속한 모든 테이블의 권한을 부여하겠다는 것이며, `'replication_user'@'%'`는 replication_user 계정으로 접근한다면 모든 IP에 대해 요청을 허락하겠다는 뜻이 된다.

> 강의자료에서는 유저 생성 과정에서 비밀번호 등록을 `IDENTIFIED WITH mysql_native_password BY '{유저 비밀번호}'`를 사용하도록 되어 있으나, mariadb 10.6 버전 기준으로 `mysql_native_password` 명령어를 해석하지 못하는 에러가 발생했다.
>
> ```
> ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your > MariaDB server version for the right syntax to use near 'BY '{유저 비밀번호}'' at line 1
> ```
>
> [MariaDB 공식문서](https://mariadb.com/kb/en/authentication-plugin-mysql_native_password/)를 살펴보니 별도로 mysql_native_password 플러그인을 명시하지 않아도 자동으로 사용되기 때문인 것으로 보인다.

```
mysql> SHOW master STATUS\G;
*************************** 1. row ***************************
            File: {바이너리 로그 파일명}
        Position: {POS 값}
    Binlog_Do_DB:
Binlog_Ignore_DB:
```

최종적으로 `SHOW master STATUS` 명령을 통해 바이너리 로그 파일을 제대로 관리 중인지 확인한다.
`File`은 바이너리 로그파일, `Position`은 바이너리 로그 파일 내부의 위치를 의미한다.
곧이어 Slave 서버에서 사용할 것이므로 기억해두자.

<br>

## Slave 서버 생성 및 설정
Slave 서버 터미널에서 역시 vim을 통해 slave db 서버의 설정 파일을 열어 주석처리된 설정을 해제한다.
이 때 `server-id` 값은 앞서 이야기했던 대로 Master, Slave들과 겹치지 않는 고유한 값으로 지정해준다.

```
$ vim /etc/mysql/mariadb.conf.d/50-server.cnf

server-id              = {id로 사용하고 싶은 정수 값}
log_bin                = /var/log/mysql/mysql-bin.log
expire_logs_days       = 10
max_binlog_size        = 100M
```

역시나 설정이 끝났다면 저장 후 `$ service mysqld restart`를 통해 설정을 적용해준다.
곧이어 관리자 계정으로 데이터베이스 서버에 접속 후 Master와의 연결을 위한 설정을 진행한다.

```sql
% mysql -u root -p
mysql> USE mysql;

mysql> CHANGE MASTER TO MASTER_HOST='{MasterDB IP}', MASTER_PORT={MasterDB 포트번호}, MASTER_USER='{만들어둔 유저 이름}', MASTER_PASSWORD='{만들어둔 유저 비밀번호}', MASTER_LOG_FILE='{바이너리 로그 파일명}', MASTER_LOG_POS={POS 값}; 
```
 
- `MASTER_USER`와 `MASTER_PASSWORD`는 앞서 Master 서버에서 생성해둔 Replication 전용 계정, 비밀번호를 입력한다.
- `MASTER_LOG_FILE`는 앞서 `SHOW master STATUS`에서 확인한 바이너리 로그 파일의 이름을 명시한다.
- `MASTER_LOG_POS`는 앞서 `SHOW master STATUS`에서 확인한 바이너리 로그 파일 내부 위치를 명시한다.

설정이 완료되었다면 `START slave` 이후 `SHOW slave STATUS\G` 명령을 통해 상태를 확인해본다.

```sql
mysql> START slave;
mysql> SHOW slave STATUS\G;
*************************** 1. row ***************************
                Slave_IO_State: Waiting for master to send event
                   Master_Host: {MasterDB IP}
                   Master_User: replication_user
                   Master_Port: {MasterDB 포트번호}
                 Connect_Retry: 60
               Master_Log_File: {바이너리 로그 파일명}
           Read_Master_Log_Pos: {POS 값}
                Relay_Log_File: mysqld-relay-bin.000001
                 Relay_Log_Pos: 4
         Relay_Master_Log_File: {바이너리 로그 파일명}
              Slave_IO_Running: Yes
             Slave_SQL_Running: Yes
             ...(생략)...
```

![image](https://user-images.githubusercontent.com/37354145/138378553-82556311-ae33-4d3c-a37e-da903975ec19.png)

Master 서버 쪽에 데이터를 추가하면 Slave 서버 쪽에도 곧장 반영되는 것을 확인할 수 있다.
위와 같이 출력될 경우 연결에 성공한 것이다. 
곧바로 Springboot Datasource 설정을 진행하자.

<br>

## Master-Slave 연결 실패
만일 `Slave_IO_State`, `Slave_IO_Running`, `Slave_SQL_Running` 등이 다르게 출력될 경우 연결에 실패한 것이다.
가장 큰 경우의 수는 Master 서버의 IP, 포트번호를 잘못 입력했거나 replication용 계정 비밀번호를 잘못 입력했을 가능성이 있다.
쉽고 빠르게 확인해볼 수 있는 방법으로는 Slave 서버 터미널에서 아래 명령을 통해 replication용 계정으로 직접 Master 서버 DB에 접속해보는 것이다.

```
$ mysql -h {Master 서버 IP} -u {replication 계정} -p
Enter password: {replication 비밀번호}
```

위 명령어로 접근이 불가능하다면 Master 서버의 bind-address 설정이나 방화벽, replication 계정 권한 등을 다시 확인해보아야 한다.

원인이 파악되었다면 아래 명령을 통해 Slave 서버에서 Master 서버로 연결 설정을 다시 진행할 수 있다.

```sql
mysql> STOP slave;

mysql> CHANGE MASTER TO MASTER_HOST='{MasterDB IP}', MASTER_PORT={MasterDB 포트번호}, MASTER_USER='{만들어둔 유저 이름}', MASTER_PASSWORD='{만들어둔 유저 비밀번호}', MASTER_LOG_FILE='{바이너리 로그 파일명}', MASTER_LOG_POS={POS 값}; 

mysql> START slave;
```

또한 연결에 성공했음에도 연결 성공시점의 Master 서버와 Slave 서버의 스키마, 테이블 구성이 다를 경우
Slave 서버는 Master 서버가 실행한 쿼리를 읽었다가 에러를 일으키고 replication 수행을 중지하게 된다.
(Master 서버의 상태를 복제하는것이 아닌, Master 서버가 수행한 쿼리를 따라서 수행하기 때문)

이럴 땐 `mysqludump`를 이용해서 Master 서버의 스키마를 덤핑한 다음, Slave 서버 쪽에 적용해준 후
Slave-Master 서버 연결 설정을 다시 진행해보자.

```
# Master 서버 터미널
$ mysqldump -u {DB 계정 이름} -p {덤프할 스키마 이름} > {덤프 파일명}.sql

# scp 명령을 통해 Slave 서버로 전송 후
# Slave 서버 터미널
$ sudo mysql -u root -p {스키마 이름} < {덤프 파일명}.sql
```

<br>

## Springboot Datasource 설정
```yml
# 기존 
spring:
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://localhost:53306/babble
    username: {계정명}
    password: {비밀번호}
```

기존 스프링 프로필 파일에서 `spring.datasource` 관련 여러가지 설정을 통해 자동으로 datasource 가 등록 되었다.
하지만 replication을 이용하려면 여러 개의 datasource를 동시에 등록해야하므로, 자동 등록 기능을 이용할 수 없다.
직접 각각의 datasource들을 적용해주어야만 한다.

```yml
spring:
# 새로운 커스텀
# 어떤 양식을 이용하던지 본인 자유
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://127.0.0.1:53306/babble
    username: root
    password: masterpw

    slaves:
      slave1:
        name: slave1
        driver-class-name: org.mariadb.jdbc.Driver
        url: jdbc:mariadb://127.0.0.1:63306/babble
        username: root
        password: slavepw
```

또한 자동 등록 기능을 이용하지 않기 때문에 hiernate 설정도 수동 등록해주어야 한다.

```yml
spring:
# 새로운 커스텀
# 어떤 양식을 이용하던지 본인 자유
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://127.0.0.1:53306/babble
    username: root
    password: masterpw

    slaves:
      slave1:
        name: slave1
        driver-class-name: org.mariadb.jdbc.Driver
        url: jdbc:mariadb://127.0.0.1:63306/babble
        username: root
        password: slavepw

# 여기부턴 JPA가 읽고 해석하므로 자유 아님
  jpa:
    properties:
      hibernate:
        show_sql: false
        generate-ddl: false
        format_sql: true
        physical_naming_strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
        jdbc:
          lob:
            non_contextual_creation: true
```

`physical_naming_strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy`는 네이밍전략 설정이다. 설정을 생략할 경우 테이블/칼럼 명이 자바에서 사용하는 camel case 그대로 사용된다. (datasource 자동 등록시 얼마나 많은 기본설정들이 포함되어 있는지 알 수 있는 대목)

`non_contextual_creation` 설정은 `createClob()` 메서드를 구현하지 않았다는 Hibernate의 에러로그를 보여주지 않는 설정이다.

스프링 프로필 작성이 완료되었다면, 임의로 작성한 DataSource를 `@ConfigurationProperties` 어노테이션을 통해 객체로 매핑해주자.

### Properties 클래스
```java
@Setter
@Getter
@ConfigurationProperties(prefix = "spring.datasource")
public class ReplicationDataSourceProperties {

    private String driverClassName;
    private String url;
    private String username;
    private String password;
    private final Map<String, Slave> slaves = new HashMap<>();

    @Setter
    @Getter
    public static class Slave {

        private String name;
        private String driverClassName;
        private String url;
        private String username;
        private String password;
    }
}
```

필드 변수명 하나하나가 yml 파일에서 설정한 이름과 매칭되어야 함을 주의하자.

> `@ConfigurationProperties` 어노테이션을 처음 사용하면 에러를 마주할 수 있는데, 
[공식 문서를 확인해보면 의존성 추가를 통해 해결할 수 있음](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-configuration-metadata.html#configuration-metadata-annotation-processor)을 알려준다.
> 
> ```
> dependencies {
>     annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"
> }
> ```

다음은 `@Transactional(readOnly=true)` 분기 처리를 위한 로직을 작성해야한다.
서비스 로직을 수행할 때 메서드에 붙은 어노테이션이 `@Transactional(readOnly=true)`인 경우 slave datasource로, 나머지는 master datasource로 분기 처리를 하기위한 RoutingDataSource 클래스를 생성한다.

### RoutingDataSource 클래스
```java
public class ReplicationRoutingDataSource extends AbstractRoutingDataSource {

    private static final String DATASOURCE_KEY_MASTER = "master";
    private static final String DATASOURCE_KEY_SLAVE = "slave";

    private DataSourceNames<String> slaveNames;

    @Override
    public void setTargetDataSources(Map<Object, Object> targetDataSources) {
        super.setTargetDataSources(targetDataSources);

        List<String> replicas = targetDataSources.keySet()
            .stream()
            .map(Object::toString)
            .filter(string -> string.contains(DATASOURCE_KEY_SLAVE))
            .collect(toList());

        this.slaveNames = new DataSourceNames<>(replicas);
    }

    // 요청에서 사용할 DataSource Key 값 반환
    @Override
    protected Object determineCurrentLookupKey() {
        boolean isReadOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly();

        if (isReadOnly) {
            logger.info("Connection Slave");
            return slaveNames.getNext();
        }

        logger.info("Connection Master");
        return DATASOURCE_KEY_MASTER;
    }

    public static class DataSourceNames<T> {

        private final List<T> values;
        private int index = 0;

        public DataSourceNames(List<T> values) {
            this.values = values;
        }

        public T getNext() {
            if (index + 1 >= values.size()) {
                index = -1;
            }
            return values.get(++index);
        }
    }
}
```

`ReplicationRoutingDataSource` 클래스는 `AbstractRoutingDataSource`를 상속하여 구현해줘야한다. `AbstractRoutingDataSource`는 spring-jdbc 모듈에 포함되어 있는 클래스로, 
복수의 datasource를 등록하고 상황에 맞게 원하는 datasource를 사용할 수 있도록 추상화한 클래스이다.

`determineCurrentLookupKey()` 메서드 오버라이딩을 통해 현재 요청에서 필요한 Master/Slave 분기를 진행하고 사용할 datasource의 key값을 반환해준다.

여러 개의 Slave 서버를 골고루 사용해서 부하를 분산시킬 수 있도록 원형 연결리스트 형태의 클래스도 사용한다.
이를 통해 요청마다 인덱스가 증가/감소하면서 모든 Slave 서버를 순회할 수 있게 된다.

다음은 실제 datasource를 스프링부트에 등록하기 위한 config 클래스를 생성한다.

### DataSourceConfig 클래스
```java
// @EnableAutoConfiguration(exclude = DataSourceAutoConfiguration.class)
@EnableConfigurationProperties(ReplicationDataSourceProperties.class)
@Configuration
public class ReplicationDataSourceConfig {

    private final ReplicationDataSourceProperties dataSourceProperties;
    private final JpaProperties jpaProperties;

    public ReplicationDataSourceConfig(ReplicationDataSourceProperties dataSourceProperties,
                                       JpaProperties jpaProperties) {
        this.dataSourceProperties = dataSourceProperties;
        this.jpaProperties = jpaProperties;
    }

    // DataSourceProperties 클래스를 통해 yml 파일에서 읽어들인 DataSource 설정들을 실제 DataSource 객체로 생성 후 ReplicationRoutingDataSource에 등록
    @Bean
    public DataSource routingDataSource() {
        DataSource masterDataSource = createDataSource(
            dataSourceProperties.getDriverClassName(),
            dataSourceProperties.getUrl(),
            dataSourceProperties.getUsername(),
            dataSourceProperties.getPassword()
        );

        Map<Object, Object> dataSources = new LinkedHashMap<>();
        dataSources.put("master", masterDataSource);

        for (Slave slave : dataSourceProperties.getSlaves().values()) {
            DataSource slaveDatSource = createDataSource(
                slave.getDriverClassName(),
                slave.getUrl(),
                slave.getUsername(),
                slave.getPassword()
            );

            dataSources.put(slave.getName(), slaveDatSource);
        }

        ReplicationRoutingDataSource replicationRoutingDataSource = new ReplicationRoutingDataSource();
        replicationRoutingDataSource.setDefaultTargetDataSource(masterDataSource);
        replicationRoutingDataSource.setTargetDataSources(dataSources);

        return replicationRoutingDataSource;
    }

    // DataSource 생성
    public DataSource createDataSource(String driverClassName, String url, String username, String password) {
        return DataSourceBuilder.create()
            .type(HikariDataSource.class)
            .url(url)
            .driverClassName(driverClassName)
            .username(username)
            .password(password)
            .build();
    }

    // 매 쿼리 수행마다 Connection 연결
    @Bean
    public DataSource dataSource() {
        return new LazyConnectionDataSourceProxy(routingDataSource());
    }

    // JPA에서 사용하는 EntityManagerFactory 설정. hibernate 설정을 직접 주입한다.
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        EntityManagerFactoryBuilder entityManagerFactoryBuilder = createEntityManagerFactoryBuilder(jpaProperties);
        return entityManagerFactoryBuilder.dataSource(dataSource())
            .packages("gg.babble.babble")
            .build();
    }

    private EntityManagerFactoryBuilder createEntityManagerFactoryBuilder(JpaProperties jpaProperties) {
        JpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        return new EntityManagerFactoryBuilder(vendorAdapter, jpaProperties.getProperties(), null);
    }

    // JPA에서 사용할 TransactionManager 설정
    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
        JpaTransactionManager tm = new JpaTransactionManager();
        tm.setEntityManagerFactory(entityManagerFactory);
        return tm;
    }
}
```

> 가장 첫 줄을 보면 주석 처리가 되어있다. 
> exclude 옵션을 이용해 DataSource 자동설정(`DataSourceAutoConfiguration`)을 제외시킬 수 있으나,
>Spring 공식문서에서 `DataSourceAutoConfiguration`에 대해 침투적이지 않다(Auto-configuration is > non-invasive)라고 평가한다. 
> 때문에 굳이 제외시킬 필요 없다고 생각해서 포함하지 않았다.

위 코드에서 눈에 띄는 부분은 `LazyConnectionDataSourceProxy()`일 것이다.
우선 Spring은 트랜잭션에 진입하는 순간 이미 설정된 DataSource의 커넥션을 가져온다.
TransactionManager가 트랜잭션을 식별하면 DataSource의 커넥션을 가져오고, 트랜잭션의 동기화가 시작되어버린다.
이럴 경우 다중 DataSource 환경에서는 DataSource를 선택하는 분기가 불가능하다.

따라서 미리 DataSource를 정하지 않도록 `LazyConnectionDataSourceProxy`를 사용하여 실제 쿼리가 실행될 때 Connection을 가져오도록 한 것이다.

설정을 모두 마쳤으면 애플리케이션을 실행해보자. 다른 이슈가 없다면 정상적으로 서비스가 진행되며 로거를 통해 Master/Slave 데이터베이스를 번갈아서 사용하고 있음을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/37354145/138382656-b750552c-8225-4607-96c0-d08a9dc4cdb5.png)

<br>

## 요약 및 이슈 내용
### 스프링 DataSource 설정 요약
1. 스프링 프로필 파일에 작성된 datasource 정보들을 DataSourceProperties 클래스를 통해 수동으로 매핑한다.
2. `isReadOnly` 분기를 통해 Master/Slave 서버 선택을 유도한다.
3. 리스트를 순환하면서 Slave 서버를 선택하도록해서, Slave 서버 부하를 분산시킨다.
4. 매핑된 DataSource 설정들을 실제 DataSource 객체로 생성 후 ReplicationRoutingDataSource에 등록한다.
5. JPA가 사용할 EntityManagerFactory를 수동으로 설정한다.
6. JPA가 사용할 TransactionManager를 수동으로 설정한다.
7. 서비스로직 메서드마다 datasource가 바뀌어야하므로, `LazyConnectionDataSourceProxy`를 통해 proxy datasource를 연결하도록 설정한다.

### Master DataSource는 왜 컬렉션으로 관리하지 않는가?
현재 글에서 구성된 Master:Slave 는 1:N으로, Master 서버가 1개인 상황을 가정하고 Slave는 컬렉션으로 관리하는 형태의 Replication이 구성되어 있다. 이 때 ‘왜 Master는 1개여야 하지? Master도 여러개 준비하고 컬렉션으로 관리하면 안되나?’ 라는 생각이 들었고, 아래 그림과 같이 Master와 Master 간에도 서로 복제 관계를 가지는 것을 생각했다.

![image](https://user-images.githubusercontent.com/37354145/138393560-3d6fa33c-53f4-4cc3-8c9e-cd338c63bf3c.png)

그러나 이 생각에 대해 우테코 질문 채널에 크루들의 의견을 물어보니, 완태가 아래와 같은 이야기와 함께 링크를 공유해주었다.

> https://bcho.tistory.com/1062  
> "Slave에서 처럼 binlog를 사용해서 data를 주고 받으면 data gap이 발생하기 때문에, Master끼리 data동기화를 하는 데에 data 불일치성이 발생할 수 있을듯 뇌피셜 팡팡!"

완태가 공유해준 링크의 글을 읽다보니, 내가 Master와 Master 간 복제 관계를 통해 해결하고자 했던 문제는 사실 Replication보다는 클러스터링을 통해 해결하는 것이 더 잘 어울리는 것 같았고, Replication을 통해서는 Master-Slave를 1:N 관계로 만드는 것으로 마무리 하기로 했다.

![image](https://user-images.githubusercontent.com/37354145/138393572-358d98d6-f937-43f9-877d-9a35918b04df.png)


<br>

## Refernces
- [Authentication Plugin - mysql_native_password - MariaDB](https://mariadb.com/kb/en/authentication-plugin-mysql_native_password/)
- [[SpringBoot] Spring 에서 자동설정의 이해와 구현 (AutoConfiguration)](https://ddingg.tistory.com/75)
- [[DB, Spring] Replication 적용하기 - velog.io/@max9106](https://velog.io/@max9106/DB-Spring-Replication)
- [SpringBoot AutoConfiguration을 대하는 자세 - Tecoble](https://tecoble.techcourse.co.kr/post/2021-10-14-springboot-autoconfiguration/)
- [MySQL – Replication 구조 - Rain.i](http://cloudrain21.com/mysql-replication)
- [LazyConnectionDataSourceProxy 알아보기](https://sup2is.github.io/2021/07/08/lazy-connection-datasource-proxy.html)
- 스페샬 땡스투 [완태](https://github.com/wannte), [에어](https://github.com/KJunseo)
