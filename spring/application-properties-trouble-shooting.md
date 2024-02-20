## 요약
```yml
aws:  
  s3:  
    bucket:  
      name: test  
      cloudFrontBaseUrl: "https://cloudfront.url"
    mock:  
      enabled: false  
      port: 8111
```

이렇게 작성되어 있는 application property 는 사실 아래와 의미가 동일하다.

```yml
aws:  
  s3:  
    bucket:  
      name: test  
      cloudFrontBaseUrl: "https://cloudfront.url"  
aws:
  s3:
    mock:  
      enabled: false  
      port: 8111
```

## 문제 상황
```kotlin
@Configuration  
@EnableConfigurationProperties(AwsS3Properties::class)  
@ComponentScan(basePackages = ["com.package.path"])  
class AwsS3Config(  
  private val awsS3Properties: AwsS3Properties,  
) {  
  
  @ConditionalOnProperty(  
    prefix = "aws.s3.mock",  
    name = ["enabled"],  
    havingValue = "false",  
    matchIfMissing = true  
  )  
  @Bean  
  fun s3Client(): AmazonS3 =  
    AmazonS3ClientBuilder  
      .standard()  
      .withRegion(Regions.AP_NORTHEAST_2)  
      .build()  
  
  
  @Profile(value = ["local"])  
  @ConditionalOnProperty(  
    prefix = "aws.s3.mock",  
    name = ["enabled"],  
    havingValue = "true",  
    matchIfMissing = false  
  )  
  @Bean  
  fun mockS3Client(): AmazonS3 {  
    return AmazonS3ClientBuilder.standard()  
      .withPathStyleAccessEnabled(true)  
      .withEndpointConfiguration(  
        AwsClientBuilder.EndpointConfiguration(  
          "http://localhost:${awsS3Properties.mockS3.port}",  
          Regions.AP_NORTHEAST_2.getName()  
        )  
      )  
      .withCredentials(AWSStaticCredentialsProvider(AnonymousAWSCredentials()))  
      .build()  
  }  
}
```

`s3Client` 의 `@ConditionalOnProperty` 설정은 아래와 같다.

- `prefix`: `aws.s3.mock` prefix 인 property 의
- `name`: `enabled` 이름의 설정이
- `havingValue`: `false` 인 경우에 `s3Client` bean 을 생성 후 주입한다.
- `matchIfMissing`: 만약 해당되는 property 가 없어도 비교를 진행한다.
	- (yml property 에서 해당되는 property 들이 없으면 모두 기본 false 로 판단)

`mockS3Client` 의 `@ConditionalOnProperty` 설정은 아래와 같다.

- `prefix`: `aws.s3.mock` prefix 인 property 의
- `name`: `enabled` 이름의 설정이
- `havingValue`: `true` 인 경우에 `mockS3Client` bean 을 생성 후 주입한다.
- `matchIfMissing`: 만약 해당되는 property 가 없으면 bean 생성을 포기한다.

또한 `mockS3Client` 에는 `@Profile(value = ["local"])` 설정이 포함되어 있으므로, local profile 로 실행될 때만 mockS3Client bean 주입을 시도한다.

위와 같은 코드 형태로 local profile 에서는 `mockS3Client` bean 을, 그 외 profile 에서는 실제 `s3Client` bean 을 주입받는 설정을 준비해두었다.

기존 `AwsS3Properties` 를 통해 불러오는 property 는 아래와 같았다.

```yml
aws:  
  s3:  
    bucket:  
      name: test  
      cloudFrontBaseUrl: "https://default.cloudfront.url"  
  
---  
  
spring.config.activate.on-profile: local  
aws:  
  s3:    
    bucket:  
      name: inara-web-management-local  
      cloudFrontBaseUrl: "https://local.cloudfront.url"  
    mock:  
      enabled: true  
      port: 8111
---  
  
spring.config.activate.on-profile: dev  
aws:  
  s3:  
    bucket:  
      name: inara-web-management-dev  
      cloudFrontBaseUrl: "https://dev.cloudfront.url"  
```

갑자기 어느 순간부터 dev profile 의 spring application 이 실행에 실패하고 있었다.

```
expected at least 1 bean which qualifies as autowire candidate for this dependency. Dependency annotations: {}
```

dev profile 설정에 의해 `s3Client` bean 이 정상적으로 로드되었어야했는데, `s3Client` bean 을 찾을 수 없다는 설정. 심지어는 `mockS3Client` bean 도 만들지 못한거 같았다.

## 원인

한참을 헤매다 application.yml 파일의 커밋로그를 살펴보니, 아래와 같이 변경사항이 추가되어 있었다. 

```yml
aws:  
  s3:  
    bucket:  
      name: test  
      cloudFrontBaseUrl: "https://default.cloudfront.url"
	mock:  # 추가된 부분
	  enabled: true  
	  port: 8111
  
---  
  
spring.config.activate.on-profile: local  
aws:  
  s3:    
    bucket:  
      name: inara-web-management-local  
      cloudFrontBaseUrl: "https://local.cloudfront.url"  
    mock:  
      enabled: true  
      port: 8111
---  
  
spring.config.activate.on-profile: dev  
aws:  
  s3:  
    bucket:  
      name: inara-web-management-dev  
      cloudFrontBaseUrl: "https://dev.cloudfront.url"  
```

오류가 발생하기 전까지만 해도, dev profile 의 spring application 은 application.yml 의 `spring.config.activate.on-profile: dev  ` 하위 영역만 spring application 이 읽고 해석을 진행할거라 생각중이었다.

그런데 application.yml 파일은 그 특성상, default 설정 외 profile 별로 직접 property 를 오버라이드 하지 않으면 default 설정을 그대로 따른다.

즉, dev profile 에서 인지한 `aws.s3` 설정은, default 값을 포함해서 아래와 같다.

```yml
spring.config.activate.on-profile: dev  
aws:  
  s3:  
    bucket:  
      name: inara-web-management-dev  
      cloudFrontBaseUrl: "https://dev.cloudfront.url" 
aws:
  s3:
    mock:
      enabled: true  
      port: 8111
```

`@ConditionalOnProperty` 설정에 의해서 `s3Client` bean 을 만들 수도, `@Profile` 설정에 의해 `mockS3Client` bean 을 만들 수도 없어서 bean 예외를 발생시키며 application 실행에 실패한 것이다.

## 해결 방법 및 예방 방법

default profile 에 추가된 설정을 지우니 정상적으로 application 이 구동되었다.

예방 할 수 있는 방법은 무엇이 있을까?

- `@ConditionalOnProperty` 을 부정형으로 사용하지 않는다.
	- 현재는 `s3Client` 의 `@ConditionalOnProperty` 가 mock s3 의 부정형으로 작성되어 있어 헷갈린다...
- default profile 에 property 를 명시한 후엔, 공통적으로 모든 profile 별로 오버라이딩 하거나 하지 않도록 통일하자.
	- local 환경에만 불필요하게 중복값이 오버라이드 되어있어 더 헷갈리고 인지에 오랜 시간이 소요되었다.