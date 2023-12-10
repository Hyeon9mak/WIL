> 이전 https://hyeon9mak.github.io/spring-circuit-breaker/ 서킷 브레이커 글을 참고하면 좋다.

spring circuit breaker([Spring Cloud Circuit Breaker](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-circuitbreaker.html#page-title)) 에 대해 알아보았으니, 이번엔 Spring Retry 에 대해서 알아보자.
우선 두 기술은 모두 fault-tolerance(장애허용)을 위해 사용하는 기술이다. 때문에 동작이 상당히 유사하다. 
그러나 자세히 살펴보면 아래와 같은 코어 컨셉  차이를 가지고 있다.

- circuit breaker: 요청들의 추이를 지켜보다가 지속적으로 실패하는 경우, 잠시 요청을 **차단**해서 장애를 전파하지 않도록 한다.
- retry: 하나의 요청에 대해, 요청이 실패했을 경우 해당 요청을 다시 시도한다. 계속 실패할 경우 다른 응답을 반환한다.

circuit breaker 는 요청 자체를 차단하는 것에, retry 는 다시 시도하는 것에 중점을 두고 있다. 이번 글에서는 retry 에 대해 중점적으로 다뤄보자.

> 최초 Retry 기능은 Spring Batch 라이브러리에 포함되어 있었다. 
> [그러나 Spring Batch 버전이 2.2.0 이 되는 시점에서 Spring Retry 라는 별개 라이브러리로 분화되었다.](https://docs.spring.io/spring-batch/reference/retry.html)

<br>

## retry example

클라이언트가 고양이 사진을 요청하면, DB 로부터 고양이 사진을 가져와 응답해주는 서비스가 있다.
클라이언트와 서버, DB 간 동작 순서를 간단히 나타내면 아래와 같다.

![](https://i.imgur.com/zZy97DH.png)

갑작스럽게 서버에서 DB 로 요청을 보내는 과정에 네트워크 순단이 발생하여 고양이 사진을 가져오는데 실패했다고 가정해보자. 
이 경우 클라이언트는 고양이 사진을 받지 못하고, 고양이 사진을 가져오지 못했다는 로그만 전달 받게 된다.

![](https://i.imgur.com/h7aiEBi.png)

일시적으로 발생한 문제기 때문에, 클라이언트는 잠시 후 다시 요청을 보내면 고양이 사진을 받을 수 있을 것이다.
즉, 서버가 네트워크 순단을 인지하고 DB 에게 재요청을 보낼 수 있다면, 클라이언트는 문제상황을 인지하지 않고 고양이 사진을 받을 수 있을 것이다.

![](https://i.imgur.com/luh6WQs.png)

대부분의 상황에서 클라이언트에게 고양이 사진을 전달하는데 성공할 것이므로, 자연스럽게 서비스 신뢰도 하락을 방어할 수 있다.

<br>

## retry test

> 테스트에 사용된 코드는 https://github.com/Hyeon9mak/spring-retry-playground 를 참고하자.

### dependent libraries

의존성 관리 파일에 `spring-retry` 라이브러리를 추가하는 것으로 편리하게 사용할 수 있다.

```groovy
implementation 'org.springframework.retry:spring-retry'
```

물론 Spring AOP 를 활용하므로 `spring-aspects` 라이브러리 의존 또한 필요하다.

```groovy
implementation 'org.springframework:spring-aspects'
```

### enable Spring Retry

Spring Retry 를 활성화 시키기 위해선 `@EnableRetry` 어노테이션을 추가한다.

```koltin
@EnableRetry
@SpringBootApplication
class SpringRetryTestApplication

fun main(args: Array<String>) {
	runApplication<SpringRetryTestApplication>(*args)
}
```

### `@retryable` annotation

간단하게 고양이 이미지를 가져오는 API 를 만들어보자.

```kotlin
@RestController
class SpringRetryTestController(
    private val springRetryTestService: SpringRetryTestService,
) {
    @GetMapping("/cats/{id}/image")
    fun catImage(@PathVariable id: Long): String = springRetryTestService.catImage(id = id)
}
```

`@Retryable` 어노테이션을 통해 간편하게 retryable 한 메서드를 정의할 수 있다.

```kotlin
@Service
class SpringRetryTestService {

    private var counter = 0

    @Retryable(
        maxAttempts = 3,
        backoff = Backoff(delay = 1000),
        include = [RuntimeException::class]
    )
    fun catImage(id: Long): String {
        counter += 1
        logger.info("counter: $counter")

        if (counter % 2 == 0) {
            return "cat_${id}_image.png"
        }

        throw RuntimeException("Failed to get cat image.")
    }

    companion object {
        private val logger = LoggerFactory.getLogger(SpringRetryTestService::class.java)
    }
}
```

`@Retryable` 어노테이션의 속성은 아래와 같다.

- `maxAttempts`: 최대 재시도 횟수
- `backoff`: 재시도 사이 시간 간격 (ms)
- `include`: 재시도 대상 Exception
  - `include` 는 `value` 와 동일한 속성이다.

위 코드상에서는 고양이 이미지 호출시 `counter` 가 홀수면 `RuntimeException` 발생 1초 후 retry 가 일어나면서 정상적으로 고양이 이미지를 반환할 것이다.

```
curl -X 'GET' 'http://localhost:8080/cats/1/image'

cat_1_image.png
```
```
2023-12-10T21:45:28.726+09:00  SpringRetryTestService : counter: 1
2023-12-10T21:45:29.728+09:00  SpringRetryTestService : counter: 2
```

### `@recover` annotation

`@Recover` 어노테이션을 통한 재시도가 최종적으로 실패했을 경우 대체 응답을 반환할 수 있다.

```kotlin
@Service
class SpringRecoverTestService {

  @Retryable(
    maxAttempts = 3,
    backoff = Backoff(delay = 1000),
    include = [RuntimeException::class],
    recover = "recover"
  )
  fun catImage(id: Long): String {
    throw RuntimeException("Failed to get cat image.")
  }

  @Recover
  fun recover(e: RuntimeException, id: Long): String {
    logger.info("recover: $e")
    return "very_cute_cat_image.png"
  }

  companion object {
    private val logger = LoggerFactory.getLogger(SpringRecoverTestService::class.java)
  }
}
```

고양이 이미지 호출 재시도 3번을 모두 실패하면, `@Recover` 어노테이션을 통해 지정된 메서드가 응답을 대체한다.
`@Recover` 어노테이션의 메서드는 `@Retryable` 어노테이션의 메서드와 동일한 파라미터, 반환타입을 가져야 한다.
파라미터 첫 번째 인자로는 `@Retryable` 어노테이션의 메서드에서 발생한 Exception 이 전달된다.

### RetryTemplate

`@Retryable` 어노테이션을 통해 간편하게 retryable 한 메서드를 정의할 수 있지만, 직접 `RetryTemplate` 을 사용하는 방법도 있다.

```kotlin
@Configuration
class RetryTemplateConfig {

    @Bean
    fun retryTemplate(): RetryTemplate = RetryTemplate.builder()
        .maxAttempts(3)
        .fixedBackoff(1000)
        .retryOn(RuntimeException::class.java)
        .build()
}
```
```kotlin
@Service
class SampleRetryService {

    fun catImage(id: Long): String {
        logger.info("exception will be thrown.")
        throw RuntimeException("Failed to get cat image.")
    }

    fun recover(id: Long): String {
        return "very_cute_cat_image.png"
    }

    companion object {
        private val logger = LoggerFactory.getLogger(SampleRetryService::class.java)
    }
}
```
```kotlin
@Service
class RetryTemplateService(
    private val sampleRetryService: SampleRetryService,
    private val retryTemplate: RetryTemplate,
) {
    fun catImage(id: Long): String {
        return retryTemplate.execute<String, RuntimeException>(
            { sampleRetryService.catImage(id = id) },
            { sampleRetryService.recover(id = id) }
        )
    }
}
```

`execute` 메서드의 첫 번째 인자는 `RetryCallback` 이다. 
반복해서 수행할 타겟 메서드를 지정한다.
두 번째 인자는 `RecoveryCallback` 이다.
retry 실패 후 최종적으로 수행할 메서드를 지정한다.

만약 retry 실패 후 최종적으로 수행할 메서드가 없다면, 두 번째 인자는 생략해도 된다.

### RetryListener

`RetryListener` 인터페이스를 구현한 후 spring bean 으로 등록하면
아래 3가지 타이밍에 추가 콜백을 제공할 수 있다.

- retry 전체 프로세스 시작 전
- retry 중 발생한 예외상황
- retry 전체 프로세스(recover 포함)를 모두 마무리 한 직후

```java
public interface RetryListener {
    
	<T, E extends Throwable> boolean open(RetryContext context, RetryCallback<T, E> callback);

	<T, E extends Throwable> void close(RetryContext context, RetryCallback<T, E> callback, Throwable throwable);

	<T, E extends Throwable> void onError(RetryContext context, RetryCallback<T, E> callback, Throwable throwable);

}
```
```kotlin
@Component("catImageListener")
class CatImageListener : RetryListener {

    override fun <T : Any?, E : Throwable?> open(context: RetryContext?, callback: RetryCallback<T, E>?): Boolean {
        logger.info("open")
        return true
    }

    override fun <T : Any?, E : Throwable?> close(
        context: RetryContext?,
        callback: RetryCallback<T, E>?,
        throwable: Throwable?
    ) {
        logger.info("close")
    }

    override fun <T : Any?, E : Throwable?> onError(
        context: RetryContext?,
        callback: RetryCallback<T, E>?,
        throwable: Throwable?
    ) {
        logger.info("onError")
    }

    companion object {
        private val logger = LoggerFactory.getLogger(CatImageListener::class.java)
    }
}
```
```kotlin
@Service
class SpringRetryListenerTestService {

    @Retryable(
        maxAttempts = 3,
        backoff = Backoff(delay = 1000),
        include = [RuntimeException::class],
        listeners = ["catImageListener"]
    )
    fun catImage(id: Long): String {
        throw RuntimeException("Failed to get cat image.")
    }

    @Recover
    fun recoverListener(e: RuntimeException, id: Long): String {
        logger.info("recover: $e")
        return "very_cute_cat_image.png"
    }

    companion object {
        private val logger = LoggerFactory.getLogger(SpringRetryListenerTestService::class.java)
    }
}
```

`@Retryable` 어노테이션에 `listeners` 속성을 등록하는 것으로 편하게 `RetryListener` 를 사용할 수 있다.
만약 `RetryTemplate` 을 사용한다면, `RetryTemplate` 에 `RetryListener` 를 등록해야 한다.

```kotlin
    @Bean
    fun retryTemplate(): RetryTemplate = RetryTemplate.builder()
        .maxAttempts(3)
        .fixedBackoff(1000)
        .retryOn(RuntimeException::class.java)
        .withListeners(listOf(CatImageListener()))
        .build()
```

실제 동작을 테스트 해보면 아래와 같다.

```
CatImageListener     : open
CatImageListener     : onError
CatImageListener     : onError
CatImageListener     : onError
SpringRetryListenerTestService   : recover: java.lang.RuntimeException: Failed to get cat image.
CatImageListener     : close
```

1. retry 전체 프로세스가 시작되며 `RetryListener` 의 `open` 메서드가 호출된다.
2. retry 중 발생한 예외상황이 발생하면 `RetryListener` 의 `onError` 메서드가 호출된다.
3. retry 가 모두 실패하면 recover 메서드가 호출된다.
4. recover 를 포함한 retry 전체 프로세스가 모두 마무리되면 `RetryListener` 의 `close` 메서드가 호출된다.

<br>

## when to use the retry

- retry 는 서비스의 신뢰도를 높이기 위해 사용한다.
  - 잘 활용한다면 일시적인 장애 상황을 사용자에게 전달하지 않고, 서비스 이용을 성공할 수 있도록 도와준다.
- retry 는 일시적인 장애를 대응하기 위해 사용한다.
  - 일시적인 장애: 네트워크 순단, DB 장애 등
  - 일시적인 장애가 아닌 경우, retry 를 통해 반복적인 장애를 전파하게 되면 서비스 신뢰도가 하락할 수 있다.
    - 매 요청마다 뒷단 서버, 서비스에 반복된 요청을 시도하므로 서비스 부하가 증가할 수 있다.
  - 일시적인 장애가 아닌 경우, circuit breaker 를 통해 장애를 전파하지 않도록 차단하는 것이 좋다.
- retry 횟수와 주기를 너무 적고 짧게 설정하면, 장애 상황이 지속되는 경우 서비스 부하가 증가할 수 있다.
- retry 횟수와 주기를 너무 많고 길게 설정하면, 사용자가 API 를 한번 호출했을 때 느끼는 응답시간이 길어질 수 있다.

<br>

## References
- https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-circuitbreaker.html#page-title
- https://docs.spring.io/spring-batch/reference/retry.html
- https://www.baeldung.com/spring-retry
- https://github.com/spring-projects/spring-retry