# 로깅 (By 검프 테코톡)

## 로깅 기본
## 로깅의 종류
- 서비스 동작 상태
    - 시스템 로딩
    - HTTP 통신
    - 트랜젝션
    - DB SQL 요청
    - (의도된) 예상된 Exception
- 장애
    I/O Exception
    NullPointerException
    (의도치 않은) 예상하지 못한 Exception

### 로깅은 언제?
- 프로젝트의 성격에 맞게, 팀에 맞게

### 로깅을 어떻게?
- System.out.println
- System.err.println
- 로깅 프레임 워크 사용 - SLFJ, LogBack 등...

### 로깅 vs System.out.println()
- 출력 형식을 지정할 수 있다.
- 로그 레벨에 따라 남기고 싶은 로그를 별도로 지정할 수 있다.
- 콘솔뿐만 아니라 파일이나 네트워크 등 로그를 별도의 위치에 남길 수 있다.

로깅 짱짱맨이다!

<br>

## 로그레벨
- Fatal: 매우 심각한 에러. 프로그램이 종료되는 경우
- Error: 의도하지 않은 에러가 발생. 프로그램이 종료되진 않음.  
------------------- 개발자가 의도한 예외 -------------------
- Warn: 에러가 될 수 있는 잠재적 가능성이 존재
- Info: 명확한 의도가 있는 에러, 요구사항에 따라 시스템 동작을 보여줄 때
- Debug: Info 레벨보다 더 자세한 정보가 필요한 경우. Dev환경
- Trace: Debug 레벨보다 더 자세함. Dev환경에서 버그를 해결하기 위해 사용

ex. 회원가입시 DB에 동일한 email을 가진 회원이 있을 때 
`DuplicateException`을 던진다면 이 이벤트의 로그는 어떤 레벨을 적용할까? 
-> Info. 개발자가 의도한 예외이기 때문.

<br>

## 로깅 vs 디버깅
- 프로그래밍의 절반은 디버깅이다.
- 디버깅을 할 수 없는 상황에서는 로깅이 최선의 선택이다. (e.g. 실 서버 구동 중)
- 디버깅을 쓸 수 있다면 디버깅을 최대한 활용한다.

변수의 값이나 메모리 주소 등을 BreakPoint를 통해 잡아낼 수 있기 때문에 예외상황을 가장 쉽고 빠르게 찾아내는 방법은 디버깅이다.

<br>

## SLF4J
Simple Logging Facade for Java
다양한 로깅 프레임 워크에 대한 추상화(인터페이스) 역할. 단독으로 사용 불가능. 최종 사용자가 배포시 원하는 구현체를 선택한다.

<br>

## SLF4J - 동작 과정
![image](https://user-images.githubusercontent.com/37354145/128099558-b7e1905d-2d24-4b83-9b25-33711720cc7a.png)


- 개발시 SLF4J API를 사용한 로깅 코드 작성
- 배포시 SLF4J와 바인딩 된 Logging framework가 실제 로깅 코드를 수행

이 과정은 SLF4J에서 제공하는 3가지 모듈인
 `Bridge`, `SLF4J API`, `Binding` 모듈을 통해 수행될 수 있다.

### Bridge
- SLF4J 이외의 다른 로깅 API로의 Logger 호출을 SLF4J 인터페이스로 연결하여 SLF4J API가 대신 처리하도록 하는 일종의 어댑터 역할 라이브러리.
- 이전의 레거시 로깅 프레임워크를 위한 라이브러리
- 여러개 사용 가능
- Binding 모듈에서 사용될 프레임워크와 달라야한다.

### SLF4J API
- 로깅에 대한 추상 레이어(인터페이스)를 제공한다.
- 로깅 동작에 대한 추상 메서드를 제공한다고 생각하면 된다.
- 추상 클래스이기 때문에 SLF4J API만 단독적으로 사용할 순 없다.
- 반드시 하나의 API에 하나의 Binding 모듈을 둬야한다는 점이다.

### Binding
- SLF4J API를 로깅 구현체(Logging Framework)와 연결한다. 어댑터 역할.
- 하나의 API 모듈에 하나의 Binding 모듈이 연결되어야 한다.

<br>

## SLF4J + Logback을 이용한 Java 로깅
```
# build.gradle

plugins {
    id 'java'
}

group 'org.livenow'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    implementation('org.slf4j:slf4j-api:1.7.31')
    implementation('ch.qos.logback:logback-core:1.2.3')

    implementation('ch.qos.logback:logback-classic:1.2.3') {
        exclude group: 'org.slf4j', module: 'slf4j-api'
        exclude group: 'ch.qos.logback', module: 'logback-core'
    }
}

test {
    useJUnitPlatform()
}
```

`logback-classic` 의존 추가하는 것만으로도 알아서 필요한 의존성(`slf4j-api`, `logback-core`)들을 추가하지만,
명시적으로 추가해주는게 더 좋은 방법이라고 한다.

```java
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(LabApplication.class);

        for (int count = 0; count < 10; count++) {
            logger.error("에러다 -- {}", count);
            logger.warn("워닝이다 -- {}", count);
            logger.info("인포다 -- {}", count);
            logger.debug("디버그다 -- {}", count);
            logger.trace("트레이스다 -- " + count);
        }
    }
```

위 코드에서 `logger.trace`는 출력되지 않는다. `LoggerFactory` 등록시 기본 로깅 레벨이 DEBUG기 때문.

또한 자세히 보면 `logger.trace`에서 `+` 연산자를 사용하고 있는데, 이 경우 기본 로깅 레벨 DEBUG 때문에 출력이 되진 않지만 메모리 상에서 덧셈 연산을 진행하므로 좋지 않다고 한다. 로깅 레벨을 확인하고 `{}`를 적극 활용하자.

### 로깅 레벨 관리 - logback.xml
```xml
# logback.xml

<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%d{HH:mm} [%-5level] %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="trace">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

<br>

## Logback
- SLF4J의 구현체
- Log4J를 토대로 만든 프레임워크다.
    - 현재는 Logback을 더 많이 사용한다.

스프링에서도 SLF4J와 Logback을 채택하고 있다.

## Logback - 구조
![image](https://user-images.githubusercontent.com/37354145/128099578-e9b16535-a218-48aa-9972-b3dec651221e.png)

### Logback-core
- 다른 두 모듈을 위한 기반 역할
- Appender와 Layout 인터페이스가 이 모듈에 속한다.

### Logback-classic
- logback-core에서 확장된 모듈
- logback-core를 가지며 SLF4J API를 구현함. 
- Logger 클래스가 이 모듈에 속한다.

### logback-access
- Servlet container와 통합되어 HTTP 액세스에 대한 로깅 기능을 제공.
- 웹 애플리케이션 레벨이 아닌 컨테이너 레벨에서 설치되어야 한다.

<br>

## Logback - 설정요소
![image](https://user-images.githubusercontent.com/37354145/128099748-66050e84-33cd-4a19-b00b-f5b108eaa70d.png)

### Logger
- 실제 로깅을 수행하는 구성 요소.
- 출력 레벨 조정
    - TRACE < DEBUG < INFO < WARN < ERROR (default DEBUG)

### Appender
- 로그 메세지를 출력할 대상 결정
    - ConsoleAppender: 콘솔 출력
    - FileAppender: 파일 출력
    - RollingFileAppender: 파일을 일정 조건에 맞게 나눠 저장

### Layout (= Encoder)
- Encoder(Layout): 로그 이벤트를 바이트 배열로 변환, 해당 바이트배열을 OutputStream에 쓰는 역할.
    - Appender의 종류에 따라 사용자가 지정한 형식으로 표현될 로그 메세지를 변환하는 역할 담당.
- FileAppender와 하위 클래스는 더 이상 layout을 필요로 하지 않는다.
    - 이제 Layout은 사용하지 않는다(거의 인터페이스 용도). Encoder를 사용한다.

<br>

## SLF4J + Logback을 이용한 SpringBoot 로깅
`spring-boot-starter-data-jpa` 의존성을 추가하면 자동으로 `slf4j`, `logback` 관련 의존성이 포함된다.

![image](https://user-images.githubusercontent.com/37354145/129984457-44bf71ef-0d57-4436-8165-d9c8f0c547b0.png)

```xml
# logback-spring.xml

<?xml version="1.0" encoding="UTF-8"?>


<configuration>
    <timestamp key="BY_DATE" datePattern="yyyy-MM-dd"/>
    <property name="LOG_PATTERN"
              value="[%d{yyyy-MM-dd HH:mm:ss}:%-4relative] %green([%thread]) %highlight(%-5level) %boldWhite([%C.%M:%yellow(%L)]) - %msg%n"/>

    <springProfile name="!prod">
        // 콘솔에 출력하기 위한 설정
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder> // 위쪽의 property "LOG_PATTERN"에 맞춰서 출력
                <pattern>${LOG_PATTERN}</pattern>
            </encoder>
        </appender>

        // root: 등록되어 있는 Logger들의 최상위 클래스
        <root level="INFO"> // 이 설정에 의해 등록된 로거들은 모두 INFO 레벨
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>

    <springProfile name="prod"> <!-- 어떤 프로필(yml)에 동작할 것인지 -->
        <!-- appender class(name)에 따른 각각의 지정을 돕는 설정 -->
        <appender name="FILE-INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>./log/info/info-${BY_DATE}.log</file> <!-- 일자별로 파일 관리하겠다 -->
            <filter class = "ch.qos.logback.classic.filter.LevelFilter"> 
                <level>INFO</level> <!-- filter는 if문과 비슷하다 -->
                <onMatch>ACCEPT</onMatch> <!-- INFO와 매칭되면? ACCEPT 해라 -->
                <onMismatch>DENY</onMismatch> <!-- 매칭 안되면? DENY 해라 -->
            </filter>
            <encoder>
                <pattern>${LOG_PATTERN}</pattern>
            </encoder>
            <!-- 로그파일이 새로 생성되었을 때, 기존 로그파일 백업 정책 설정 -->
            <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                <fileNamePattern> ./backup/info/info-%d{yyyy-MM-dd}.%i.log</fileNamePattern> <!-- 백업경로 -->
                <maxFileSize>100MB</maxFileSize> <!-- 하나의 파일 크기가 100MB 이상일 때 -->
                <maxHistory>30</maxHistory> <!-- 혹은 저장된 시간이 30일이 넘었을 때 -->
                <!-- 단, 여기서 maxHistory는 <file>의 단위에 영향을 받는다. BY_DATE가 바뀔 경우 maxHistory도 그 단위를 따른다. -->
                <totalSizeCap>3GB</totalSizeCap> <!-- 혹은 전체 사이즈가 3GB가 넘을 때 -->
                <!-- 오래된 파일 중 1개를 삭제하는 정책 설정 -->
            </rollingPolicy>
        </appender>

        ...(생략)...

        <root level="INFO">
            <appender-ref ref="FILE-INFO"/>
            <appender-ref ref="FILE-WARN"/>
            <appender-ref ref="FILE-ERROR"/>
        </root>
    </springProfile>

</configuration>
```

### property 설정
`property` 설정은 encoder(=layout)의 '어떻게 출력할까?'에 해당되는 설정이다.

- `%d`: 시간을 표현한다. `%-4relative`는 밀리초 단위를 4자리까지 표현한다는 의미.
- `%green`: 색상 설정. `%red`, `%blue` 등 다양하게 설정 가능하다.
- `%thread`: 기능이 동작할 때 실행된 스레드를 나타낸다.
- `%highlight`: 로그 레벨별 색상 자동 지정
- `%-5level`: 로그 레벨을 최대 5글자까지로 나타낸다.
- `%boldWhite`: 두꺼운 흰색!
- `%C.%M:%yellow(%L)`: Class, Method, Line을 나타낸다.
- `%msg`: 로그에 지정된 메세지를 출력한다.

### DB 쿼리는 logback.xml과 관계없다
DB 쿼리의 경우 `logback.xml` 설정과 관계없이 `application.yml` 파일의 아래 설정에 따라  출력된다.

```yml
  jpa:
    hibernate:
      ddl-auto: create-drop
    properties: // jpa.show-sql에 의해 출력
      hibernate:
        format_sql: true
    show-sql: true
    generate-ddl: true
```

### 로그 설정 파일 분리하기
xml 파일 하나가 너무 길지 않은가? 이를 `include`, `included` 태그를 통해서 분리할 수 있다.

```xml
# logback.xml

<?xml version="1.0" encoding="UTF-8"?>

<configuration>
    <timestamp key="BY_DATE" datePattern="yyyy-MM-dd"/>
    <property name="LOG_PATTERN"
      value="[%d{yyyy-MM-dd HH:mm:ss}:%-4relative] %green([%thread]) %highlight(%-5level) %boldWhite([%C.%M:%yellow(%L)]) - %msg%n"/>

    <springProfile name="!prod">
        <include resource="console-appender.xml"/>

        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>

    <springProfile name="prod">
        <include resource="file-info-appender.xml"/>
        <include resource="file-warn-appender.xml"/>
        <include resource="file-error-appender.xml"/>


        <root level="INFO">
            <appender-ref ref="FILE-INFO"/>
            <appender-ref ref="FILE-WARN"/>
            <appender-ref ref="FILE-ERROR"/>
        </root>
    </springProfile>

    <springProfile name="test">
        <include resource="console-appender.xml"/>

        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
</configuration>
```
```xml
# console-appender.xml

<included>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>
</included>
```
```xml
# file-info-appender.xml

<included>
    <appender name="FILE-INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>./log/info/info-${BY_DATE}.log</file>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>./backup/info/info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>3GB</totalSizeCap>
        </rollingPolicy>
    </appender>
</included>

```

<br>

## References
- [[10분 테코톡] ☂️ 검프의 Logging(로깅) #1](https://youtu.be/1MD5xbwznlI)
- [[10분 테코톡] ☂️ 검프의 Logging(로깅) #2](https://youtu.be/JqZzy7RyudI)
- [[Logging] SLF4J란? :: 경험의 연장선](https://livenow14.tistory.com/63)
- [[Logging] Logback이란? :: 경험의 연장선](https://livenow14.tistory.com/64)
- [Chapter 6: Layouts - logback.qos.ch](http://logback.qos.ch/manual/layouts.html#coloring)
