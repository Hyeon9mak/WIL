# 지식파편

## 너무너무 좋은 아티클 모음
- https://sandn.tistory.com/112
- https://hamait.tistory.com/612
- https://medium.com/@eastperson/spring-mvc%EC%97%90-http-request%EB%A5%BC-%EB%B3%B4%EB%82%B4%EB%A9%B4-%EC%96%B4%EB%96%A4-%EC%9D%BC%EC%9D%B4-%EC%9D%BC%EC%96%B4%EB%82%A0%EA%B9%8C-80467f8bc486
- [[AWS] 가장 쉽게 VPC 개념잡기 - 해리의유목코딩](https://medium.com/harrythegreat/aws-%EA%B0%80%EC%9E%A5%EC%89%BD%EA%B2%8C-vpc-%EA%B0%9C%EB%85%90%EC%9E%A1%EA%B8%B0-71eef95a7098)
- [Spring - Open Session In View - kingbbode](https://kingbbode.tistory.com/27)
- [JPA N+1 문제 및 해결방안 - 기억보단 기록을](https://jojoldu.tistory.com/165)
- [Bean 스코프란? - 놀기 뭐해서 공부라도 하려구요!](https://velog.io/@probsno/Bean-%EC%8A%A4%EC%BD%94%ED%94%84%EB%9E%80)
- [Difference between Docker Compose Vs Dockerfile](https://dockerlabs.collabnix.com/beginners/difference-compose-dockerfile.html)
- [Terraform의 tfstate를 원격으로 관리하기 :: Outsider's Dev Story](https://blog.outsider.ne.kr/1290)
- [AWS EC2 Container Service(ECS) (1) - 구조와 특징 - ㅍㅍㅋㄷ](https://bluese05.tistory.com/52)
- [AWS Client VPN Endpoint 사용하기 - 무스바 기술블로그](https://musma.github.io/2019/11/04/aws-client-vpn-endpoint.html)
- [Availability zone 이란? - Med&Tech](https://med-tech.tistory.com/5)
- [AWS MFA CLI 설정 변경 자동화하기 - swalloow.github.io](https://swalloow.github.io/aws-cli-mfa/)
- [SpringBoot - Kotlin에서 @Valid가 동작하지 않는 원인(JSR-303, JSR-380)](https://velog.io/@lsb156/SpringBoot-Kotlin%EC%97%90%EC%84%9C-Valid%EA%B0%80-%EB%8F%99%EC%9E%91%ED%95%98%EC%A7%80-%EC%95%8A%EB%8A%94-%EC%9B%90%EC%9D%B8JSR-303-JSR-380)
- [Coroutine IO Dispatcher의 Thread number가 최대 Thread 갯수를 초과하는 이슈](https://sandn.tistory.com/112)
- [쓰레드풀 과 ForkJoinPool](https://hamait.tistory.com/612)
- [RDS 성능 개선 도우미](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/USER_PerfInsights.Overview.html)

## intellij 에서 gradle 프로젝트를 정상적으로 찾지 못할 때
- build.gradle 에서 커넥 확인
- 

## ES 몇 가지..
- es 는 샤드를 기반으로 스코어링을 한다.
	- 때문에 특정 샤드에 몰리면 스코어링이 어려워진다.
	- 검색 시간이 느려진다는 뜻.
- 루씬 인덱스가 너무 많아지면 검색 히트율이 많이 낮아질 수 있다.

## query IN (null)
- 쿼리의 IN 혹은 NOT IN 은 null 이 포함되면 예상 외의 결과를 반환한다.
- JPA 는 파라미터가 null 일 때 이것을 걸러주지 못하고 자동적으로 `IN (null)` 쿼리를 생성한다.
- 이에 대한 대응책으로 queryDSL 로 직접 쿼리를 작성하거나, 별개 쿼리를 2벌 준비해서 사용하자.

## sub query NOT IN
- https://choihyuunmin.tistory.com/93
- 서브쿼리의 IN 은 하나라도 일치하면 반환
- 서브쿼리의 NOT IN 은 "모든 요소들과 일치하지 않는 값"을 체크하여 반환한다.
	- 만약 비교 컬럼이 nullable 하다면 예상외의 결과가 나올 수 있다.
	- null 은 무조건적으로 false 를 내뱉기 때문

## 모듈별 코드 옮기기 트러블 슈팅
### 겪은 문제
- A 모듈에서 F 모듈 configuration bean scan 에 실패함
- A 와 비슷한 형상의 B 모듈에서는 문제가 발생하지 않음

### 원인
- B 모듈과 F 모듈은 패키지 구조가 완전히 동일함
	- 때문에 B 모듈 Main 이 F 모듈 내부 bean scan 을 모두 성공
- A 모듈은 F 모듈과 패키지 구조가 다름
	- 때문에 실패

### 해결방법
- F 모듈 내부 bean scan 범위를 명시해주는 Config 추가
- 또 다른 해결 방법으로는 B 모듈 Main 에 탐색 패키지 명칭을 상위로 올리면 되는데, 이건 딱히 좋은 방법이 아닌 것 같음.

## presignedURL
- "우리가 어떠한 경로에 이미지를 올릴거야" 클라이언트에 올리라고 응답을 준다.
- 경로랑 파일명을 내가 이미 알고 있으니까, 거기에 등록되었을거라고 알아서 저장되도록
- 일단 컨텐츠 ID 를 0으로 저장한 후에 다시 변경하는 것으로.....

## mongo db and search
- mongo 검색 히트률이 낮아서 보통 별개의 검색용 서버를 둔다.
- 그리고 mongo 는 read model 만 갖고 있는다.
	- 검색 서버를 통해 ID 를 가져온다음, ID 로 mongo 에 질의해서 read model 조회

## @Transactional(propagation = Propagation.REQUIRES_NEW)`
- propagation requires new 는 새로운 스레드를 생성하는 것이 아니다.
- 하나의 스레드에서 새로운 커넥션을 얻는 것 뿐이다.
- 때문에 자식 트랜잭션에서 예외가 발생했을 때 부모측에서 try-catch 로 예외처리를 해주지 않으면 부모 트랜잭션까지 모두 롤백된다.
- 부모는 롤백되지 않고 자식만 롤백을 원할 경우 try-catch 로 예외를 관리해주어야 한다.

## spring actuator
- [spring actuator 는 기본적으로 자신이 의존중인 클라이언트의 서버 상태까지 모두 확인하여, 클라이언트 서버 중 하나라도 unhealth 상태면 unhealth 를 반환한다.](https://toss.tech/article/how-to-work-health-check-in-spring-boot-actuator)
- 이를 해소하기 위해선 spring actuator 관련 상태를 모두 비활성화하고, 커스텀하게 헬스체크 상태를 반환해주는 것이 좋음.
- 간단한 방법으로는 spring actuator 라이브러리 의존 자체를 제거하고, custom health check controller 구현
```kotlin
@RestController
fun HealthCheckController {

	@GetMapping("/actuator/health")
	fun healthCheck(): String {
		return "OK"
	}
	
	@GetMapping("/actuator/info")
	fun healthCheck(): String {
		return "OK"
	}
}
```
- spring actuator 의 다른 기능을 활용하면서 하는 방법으로는 Indicator 구현
```kotlin
@Component
class CustomHealthIndicator : HealthIndicator {

  override fun health(): Health = Health.up().build()
}

@Component
class CustomInfoContributor : InfoContributor {

  override fun contribute(builder: Info.Builder) {
    builder.withDetail("info", "OK")
  }
}
```

## ConstructorBinding
```kotlin
@ConstructorBinding  
@ConfigurationProperties("work-book")
```

https://codingdog.tistory.com/entry/spring-boot-constructorbinding-%EC%83%9D%EC%84%B1%EC%9E%90%EB%A1%9C-binding%EC%9D%84-%EC%8B%9C%ED%82%A8%EB%8B%A4


## 제네릭 공변성과 반공변성
- https://hungseong.tistory.com/30

## Redis 도입을 통[[hyeon9mak/wil/지식파편]]한 성능 개선

서비스의 성능은 여러 단계로 확인할 수 있다.

- 브라우저를 통해 직접 QA하며 전구간 테스트
- webpagetest 등을 통한 인터넷 구간만 테스트
- 부하 테스트로 (브라우저 렌더링을 제외한) 전구간을 테스트

nGrinder를 이용해 테스트를 진행한다.

## 테스트 시나리오 대상
가장 중요한 것은 **어떤 성능을 중점적으로 측정할 것인가?**  
사용자 트래픽이 많은 파트, 가장 중요한 파트를 측정하는게 효율적이다.
경쟁사이트 또는 유사 사이트의 성능을 조사해서 지표로 삼는 것도 좋다.
즉, 자기 서비스에 대한 이해가 우선 되어야 한다.

## 성능 목표
성능 목표를 지정하는 방법은 여러가지가 있지만, CU 강의에서 제시해주신 방법은 크게 3가지

1. DAU (1일 사용자 수)
2. 피크 시간대 집중률 (최대 트래픽 / 평소 트래픽)
3. 1명당 1일 평균 요청 수

3가지가 모두 합쳐져서 `Throughput: 1일 평균 rps ~ 1일 최대 rps`가 만들어진다.

계산 방법은 아래와 같다.

- 1일 사용자 수 (DAU) * 1명당 1일 평균 접속수 = 1일 총 접속수
- 1일 총 접속수 / 86,400 (초/일) = 1일 평균 rps
- 1일 평균 rps * (최대 트래픽/ 평소 트래픽) = 1일 최대 rps



## java reflection
### getMethods()
상속한 메서드를 포함해서, 접근제한자가 public 인 메서드들만 가져온다.

```java
Class<?> animalCLass = Student.class;
Method[] methods = animalClass.getMethods();

"toString", "getName", "getAge", 
"wait", "wait", "wait", "equals", "hashCode",
"getClass", "notify", "notifyAll"
```

별별 거 다 가져온다.

### getDeclaredMethods()
상속한 메서드를 제외하고, 접근제한자에 관계없이 직접 코드로 선언된 메서드들을 가져온다.

```java
Class<?> animalClass = Student.class;
Method[] methods = animalClass.getDeclaredMethods();

"getAge", "toString", "getName"
```

getConstructor, getDeclaredConstructor도 마찬가지.


## LogBack JPA query 설정
logback.xml 설정에서 JPA query는 application.yml 프로필의 hibernate 옵션 설정을 따른다.

## Querydsl 은 insert query 를 지원하지 않는다.
- https://junhyunny.github.io/java/jpa/query-dsl/crud-with-jpa-query-factory/#closing

## Spring Data Envers 는 navtive query 를 대상으로 동작하지 않는다.
- spring data envers 는 entitry 의 변경을 감지하여 동작한다.
- 때문에 native query 를 이용한 테이블 업데이트는 감지하지 못한다.
- 이에 대한 현상은 jpql 을 이용한 변경감지 불가, querydsl 을 이용한 변경감지 불가가 있다.
- https://hibernate.atlassian.net/browse/HHH-10318

## 성능 테스트 및 하드웨어 스펙 조절
- 보통은 스케일 아웃으로 대응하지 스케일 업으로 대응하진 않는다.
- 만약 2대에서 4대로 띄워서 못버티면, 10대 20대까지 띄워봐야한다.
- 트래픽이 늘어나면 스케일 아웃으로 대응하는 법을 먼저 생각해봐야한다.
- "pod 1대로 버틸 수 있는게 정해지고 코어를 올린 것인지?" 이런 기준이 중요함


## enum 에 없는 타입이 전달 되었을 때 예외를 발생시키지 않고 기본 enum 을 선택하게 할 수 있는 방법
`@JsonEnumDefaultValue` 을 이용한다.

## json 은 정규표현식 사용이 어렵다.
`{}` 를 이용한 depth 가 깊기 때문에 정규표현식 활용이 어렵다.
json parser 라이브러리를 적극 활용하자. 대부분 `readTree` 를 통해 해결 가능하다.

## 캐싱할 땐 항상 중복주의
해싱을 하기 때문에 같은 데이터도 별개 id 로 저장될 수 있다.

## CPU 가 기하급수적으로 튈 때
- db connection pool 부족 문제일 수 있다.
	- 아마 내부 원인은 db connection pool 을 빠른 속도로 주고 받으면서 부하가 발생하는 듯

## request dto no args constructor 왜 안됨?
intellij `settings` > `Build, Execution, Deployment` > `Build Tools` > `Gradle` 메뉴의 `Build and Run` 설정을 Gradle 로 변경
![](https://i.imgur.com/kxhPseA.png)


## Redis 공부
- https://velog.io/@injoon2019/NHN-FORWARD-2021-Redis-%EC%95%BC%EB%AC%B4%EC%A7%80%EA%B2%8C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-%EC%A0%95%EB%A6%AC

## JPA Bulk delete
- JPA 는 bulk delete 를 수행할 때 select -> delete 를 진행한다.
- 때문에 select 단계에서 엔티티가 조회되지 않으면 예외를 발생시킴.

## Request header is too large
https://goateedev.tistory.com/126

# CDC 가 추적하지 않는 것
CDC 는 truncate 를 추적하지 않는다.

# 디버깅이 너무 오래걸릴 때
- https://developer-c.tistory.com/16

## AWS VPC
### VPC  
-   AWS 서비스 내에서 정의한 논리적으로 격리된 가상 네트워크
-   집에서 공유기를 사용할 때 내 컴퓨터의 IP주소를 확인해보면(시스템 환경설정 > 네트워크) 192.168.x.x, 10.0.x.x 등인 것을 볼 수 있다. 이건 공유기를 통해 관리되는 내부망에서 할당된 IP 주소다. 네이버에 “IP 주소“를 검색해보면 위의 주소와 다른 IP가 나오는데, 이 주소는 우리집 네트워크에 할당된 IP 주소이다.
-   위 내용 중 192.168.x.x와 같이 우리집 안에서 할당된 네트워크를 사설 IP, 가상 IP, private IP라고 부른다. 반대로 외부에서 접속할 수 있는 IP주소를 공인 IP, public IP라고 부른다. 사설 IP는 특정 네트워크 내에서 사용되는 주소이므로, 다른 네트워크에서도 동일한 주소를 사용할 수 있다.
-   사설 IP? [https://namu.wiki/w/%EC%82%AC%EC%84%A4%20IP](https://namu.wiki/w/%EC%82%AC%EC%84%A4%20IP)

### Subnet  
-   VPC에 설정한 IP 중 일부를 나타낸다. VPC에서 할당할 수 있는 IP 중 일부를 어떤 가용 영역에서 사용할지 관리한다.
-   서브넷은 public, private 등으로 관리할 수 있다. public 서브넷은 외부에서 접근할 수 있고, private 서브넷은 외부에서 직접 접근할 수 없다.

-   더 궁금하면 인터넷 게이트웨이, NAT 게이트웨이를 찾아보자.

### Availability Zone (AZ, 가용 영역)  
-   데이터 센터라고 생각하면 된다. AWS도 서비스 제공을 위해 서버를 어딘가에 설치하고 제공할텐데, 해당 서버가 있는 구역이다.
-   현재 서울 리전에는 4개의 가용 영역이 있다. 하나의 가용 영역에서 서비스를 제공하면 데이터 센터에 자연재해, 화재 등의 피해가 발생했을 때 서비스가 중지될 수 있다.

### Security Group (SG, 보안 그룹)  
-   AWS 내에서 사용할 수 있는 방화벽이라고 생각하면 된다.
-   어떤 트래픽이 들어오고 나갈 수 있는지에 대한 규칙을 지정할 수 있다.



## Querydsl 에서 `Projections.constructor` 표현식에 nullable 한 인자 전달하기
```
java.lang.NullPointerException: Parameter specified as non-null is null: method com.kidsworld.main.domain.dto.PaperSubDto.<init>, parameter linkType
```
- 불가능함. 애초에 non-nullable 한 인자만 전달하기 떄문.
- 대체 가능한 해결 방법은 2가지

1. groupBy 절로 묶어서 Map 으로 가져와서 일일히 매핑한다.
2. 별개 DTO 를 만들어서 `@QueryProjection` 을 붙인다.

1번 방법의 경우 간단한 테이블에서는 문제가 해결된다.
그러나 1:N:N 으로 2depth 이상 계위가 들어가는 구조에서는 상당히 복잡하다.
때문에 엔티티가 붙는게 조금 껄끄럽지만, `@QueryProjection` 를 사용해서 문제 해결했음.


## AWS Gateway API
- aws 자체적으로 gateway 솔루션을 제공한다.

## [No Constructor" Deserializer module](https://github.com/FasterXML/jackson/wiki/Jackson-Release-2.13#no-constructor-deserializer-module)
New module -- `jackson-module-no-ctor-deser` -- now included in [jackson-modules-base](https://github.com/FasterXML/jackson-modules-base) -- added to support a very specific use case of POJOs that do not have either:

1.  No-arguments ("default") constructor, nor
2.  `@JsonCreator` annotated constructor or factory method

in which case module can force instantiation of values without using any constructor, using JDK-internal implementation (included to support JDK serialization itself). Note that this module may stop functioning in future, but appears to work at least until JDK 14.

jackson 라이브러리 2.13 버전부터는 default 생성자 없이도 `@RequestBody` 어노테이션을 이용하는 request DTO 생성이 가능하다. 언제 지원이 끊길지는 알 수 없으나, JDK 14 버전까지는 잘 동작하는 중.


## AutoConfigureTestDatabase

포트어댑터 패턴에서 집중해야 하는 것은 코드 재사용성이 아니라, 어댑터 모양에 알맞는 포트는 무엇이든 끼울 수 있다는 뜻의, 결국 포트의 규격만 맞춰서 어떻게 구현하든 상관없는, 다른 유스케이스에서 사용하는 혹은 다른 의존체의 모양을 신경쓸 필요 없이 나의 유스케이스에만 맞춰서 코드를 작성하면 되는 격리성에 집중해야하는거 아닐까?
