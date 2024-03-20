이름에서부터 느껴지는 강력함과 같이, `enforcedPlatform` 은 강제로 의존 라이브러리들의 버전을 모두 변경하는 기능을 가졌다.

> 아래 버전 이야기는 예시

만약 `spring-cloud-dependencies` 에서 `spring-boot-starter-web` 를 의존중이고, `spring-boot-starter-web` 의 버전이 `1.1.2` 였다고 가정해보자.
`implementation enforcedPlatform` 명령어와 함께 `spring-boot-starter-web:1.0.9` 를 의존하는 `spring-cloud-dependencies:1.0.0`을 명시할 경우, `spring-boot-starter-web:1.1.2` 를 명시해두었어도 강제로 `spring-boot-starter-web:1.0.9` 로 버전을 치환한다.

```groovy
# kotlin

dependencies {

  implementation 'org.springframework.boot:spring-boot-starter-web:1.1.2'

  implementation enforcedPlatform("org.springframework.cloud:spring-cloud-dependencies:1.0.0")
}
```

아래와 같이 강제로 치환

```groovy
# kotlin

dependencies {

  implementation 'org.springframework.boot:spring-boot-starter-web:1.0.9'

  implementation enforcedPlatform("org.springframework.cloud:spring-cloud-dependencies:1.0.0")
}
```