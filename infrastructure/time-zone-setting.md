# 타임 존 설정

## 도커 컨테이너 타임 존 설정
도커 컨테이너 위에서 동작하면서 별도의 타임 존 설정이 없는 프로그램들의 경우 
`docker-compose.yml` 파일의 타임 존 설정에 따라 시간이 설정된다.

```yml
# cat docker-compose.yml

# 버전 정의 (yml 버전)
version: '3.7'

# service 정의
# docker-compose로 생성 할 container의 옵션을 정의
services:
    # 생성 할 container 변수 지정
    nignx:
        # container 이름 지정
        container_name: test-nginx
        # container 생성시 사용 할 이미지 이름 지정 및 등록([이미지 이름]:[버전])
        image: reverse-proxy:0.0.4
        
        ...(생략)

        environment:
                - TZ=Asia/Seoul
```

`envoironment - TZ` 설정을 해두면 컨테이너의 타임 존이 설정된다.
NGINX, DB 등의 타임 존을 이 방법을 통해 쉽게 관리할 수 있다.

<br>

## 스프링 부트 타임 존 설정
스프링 부트는 크게 2가지 방법으로 타임 존을 맞출 수 있다.

### 1. 실행시 타임 존 지정
```
$ java -jar -Duser.timezone=Asia/Seoul app.jar
```

`-Duser.timezone=Asia/Seoul` 명령을 추가 기입해서 타임 존을 맞출 수 있다.
모든 배포시마다 해당 명령을 통해 시간을 맞춰야한다는 단점이 있으나, 
대부분의 경우 배포 스크립트를 이용하기 때문에 금새 해결 가능한 단점이다.

### 2. @PostContruct 어노테이션을 사용한 타임 존 지정
```java
@SpringBootApplication
public class TimeZoneApplication {

    @PostConstruct
    void started() {
        TimeZone.setDefault(TimeZone.getTimeZone("Asia/Seoul"));
    }

    public static void main(String[] args) {
        application.run(TimeZoneApplication.class, args);
    }
}
```

`@PostConstruct`는 Bean 초기화가 완료된 후 단 한 번만 수행되는 동작이다. (보통 초기화 직후 별도의 작업이 필요할 때 사용) 위 코드의 경우 애플리케이션이 실행될 때 타임 존이 딱 한 번만 설정되게 되므로, 배포시마다 자동으로 타임 존이 맞춰진다는 장점이 있다.

> 그러나 타임 존 설정을 위한 코드가 프로덕션 코드에 포함된다는 것이 석연치 않았다. 
때문에 나는 실행시 타임 존 지정 방법을 선호한다.


<br>

## References
- [[SpringBoot] TimeZone 설정 후 출력하기 - 어제보다 더 나은 내일을](https://dbjh.tistory.com/74)