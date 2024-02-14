## 왜 SET 쿼리를 없애야 하는가?
일반적으로 JpaRepository 사용시 아래와 같은 쿼리가 실행된다.

```sql
SET session transaction read only
SET autocommit = 0

...(내가 원하는 쿼리)

commit
SET autocommit = 1
SET session transaction read write
```

실제 쿼리 외에도 커밋과 트랜잭션 관련 `SET` 쿼리가 5건 더 호출되는 셈.
때문에 DB 로 들어가는 실제 요청 수는 `쿼리 * 6` 이 된다.

5건의 `SET` 쿼리를 제거하면 DB 리소스 소비량도 줄이고, 자연스럽게 애플리케이션의 최종 응답속도도 더 빨라진다.

그렇다면 대게 `@Transactional` 같은 어노테이션을 명시하지 않으면 해결될텐데, 이 설정을 알아두어야 하는 이유가 무엇일까?
간혹 `SimpleJpaRepository` 를 사용하게 되는 경우가 있는데, `SimpeJpaRepository` 가 자체적으로 `@Transactional(readOnly=true)` 설정을 내포하고 있다.

![](https://i.imgur.com/xJaP2T1.png)

이렇게 반 강제적으로 진행되는 트랜잭션을 무시하기 위함

## 확인을 위한 준비

db 엔진에 접속 후 아래와 같은 명령을 이용해 로그가 보이도록 변경하면 확인이 가능하다. 아래는 mysql 기준
(DBMS 마다 명령어가 다르므로 참고)

```
1. 로그 ON
mysql> set global general_log=ON

2. 로그 파일 위치 확인
mysql> show global variables like 'general_log_file';

3. 로그 파일을 통해 내용 확인
$ tail -f {파일 위치}

4. 확인 후 최종적으로 설정 변경
mysql> set global general_log=OFF
```

그리고 `@EnableJpaRepositories` 의 `enableDefaultTransactions` 옵션을 변경한다.

```kotlin
@EnableJpaRepositories(enableDefaultTransactions = false)
```

## 실험 결과

### enableDefaultTransactions = true 일 때

query 실행 과정에서 JpaRepository 의 동작 방식에 따라 트랜잭션 여부를 결정한다.
만약 SimpeJpaRepository 를 사용하는 경우, `@Transactional(readOnly=true)` 옵션에 따라서 트랜잭션이 시작된다. (= SET 쿼리가 5번 따라 호출된다.)

### enableDefaultTransactions = false 일 때

별도로 트랜잭션을 명시하지 않은 경우, 트랜잭션을 시작하지 않는다.
(= SET 쿼리가 사라진다.)

## 원리

`RepositoryAnnotationTransactionAttributeSource`  클래스 내부 `computeTransactionAttribute` 메서드를 확인해보면 된다.

> 자세한 탐색을 원한다면 `@Transactional` > `AnnotationTransactionAttributeSource` > `AbstractFallbackTransactionAttributeSource#getTransactionAttribute` > `AbstractFallbackTransactionAttributeSource#computeTransactionAttribute` > `RepositoryAnnotationTransactionAttributeSource`(`AbstractFallbackTransactionAttributeSource` 구현) 을 확인해보면 된다.

## 주의점
말 그대로 트랜잭션을 강제로 없애는 것이므로, 영속성 컨텍스트를 활용할 수 없다.
트랜잭션이 필요한 구간에는 명시적으로 트랜잭션을 시작해주도록 해야한다.

## References
- [https://velog.io/@seovalue/using-enableDefaultTransactions-remove-set](https://velog.io/@seovalue/using-enableDefaultTransactions-remove-set)