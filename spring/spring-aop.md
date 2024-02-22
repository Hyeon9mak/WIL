## Spring AOP
AOP(Aspect Oriented Programming)은 관점 지향 프로그래밍으로, 어떤 로직에 대해 핵심적인 관점, 부가적인 관점을 나누어 보고 그 관점들을 기준으로 각각 모듈화를 하겠다는 것이다.
(모듈화 - 공통된 로직이나 기능을 하나의 단위로 묶는 것)

예를 들어 핵심적인 관점을 우리가 적용하고자 하는 핵심 비즈니스 로직.
부가적인 관점은 그에 필요한 데이터베이스 커넥션, 로깅, 파일I/O 등이 있다.

AOP 관점에서 로직을 모듈화한다는 것은 코드들을 부분적으로 나누어 다른 부분에서 계속해서 반복되는 코드들을 발견하고 이것들을 모듈로 만들어 재사용하겠다는 것. "계속해서 반복되는 코드들"을 **흩어진 관심사(Crosscutting Concerns)** 라고 부른다.

<br>

## 프록시와 AOP
Spring AOP를 이해하기 전에 프록시 패턴에 대해 알면 좋다. 
프록시 패턴의 시작점은 아래와 같다.

```java
public class Apple implements Fruit {

    @Override
    public void color() {
        System.out.println("사과는 빨간색");
    }
}
```
```java
public class Banana implements Fruit {

    @Override
    public void color() {
        System.out.println("바나나는 노란색");
    }
}
```

`사과는 빨간색`, `바나나는 노란색` 문장을 출력하는 `Fruit` 인터페이스 구현체들이다. 
이 문장 앞에 `바구니 속 과일 ` 이라는 문장을 이어 붙이고 싶다면 어떻게 해야할까?

가장 간단한 방법은 `color()` 메서드 앞부분에 출력문을 추가하는 방법이 있다. 

```java
public class Apple implements Fruit {

    @Override
    public void color() {
        System.out.println("바구니 속 ");
        System.out.println("사과는 빨간색");
    }
}
```

`Banana` 클래스도 코드 수정이 일어나야 한다.

```java
public class Banana implements Fruit {

    @Override
    public void color() {
        System.out.println("바구니 속 ");
        System.out.println("바나나는 노란색");
    }
}
```

우여곡절 끝에 `Banana` 클래스도 코드 수정이 마무리 되었다. 그런데 `바구니 속` 문장을 `과일 가게에 있는`으로 바꾸고 싶어진다면?

이런 식으로 코드의 유지보수성이 떨어지고, 중복이 발생하는 것을 막고자 해서 등장한 것이 프록시 객체다.

<br>

```java
public class PrefixOfFruit implements Fruit {

    private Fruit fruit;

    public PrefixOfFruit(Fruit fruit) {
        this.fruit = fruit;
    }

    @Override
    public void color() {
        System.out.println("과일 가게에 있는 ");
        fruit.color();
    }
}
```
실제 구현 코드인 `Apple`과 `Banana`의 부가적인 공통 로직을 분리하고, 그 로직을 가진 객체가 `Apple`과 `Banana`를 의존하게 되었다.
기존 코드를 변경하지 않고 공통 로직을 동작(추가) 할 수 있으며, 코드의 중복도 방제했다. `Apple`과 `Banana`는 자신의 색깔이 무엇인지 표현하는 핵심 로직만 갖고 있으면 된다.
이처럼 핵심 기능은 다른 실구현체에게 위임하고, 부가적인 공통 로직을 제공하는 객체를 **프록시 객체**라 부른다.

<br>

## Spring AOP
AOP의 기본 개념은 핵심 기능에 공통 기능을 삽입하는 것이다.

> 엄밀히 말하면 공통(부가)기능이 핵심 기능을 의존하도록 코드를 구성한다.

AOP에서 핵심 기능에 공통 기능을 삽입하는 방법은 3가지가 있다.

1. 컴파일 시점에 코드에 공통 기능을 삽입
2. 클래스 로딩 시점에 바이트 코드에 공통 기능을 삽입
3. 런타임에 프록시 객체를 생성해서 공통 기능을 삽입

`AspectJ` 와 같은 AOP 전용 도구를 사용하면 1,2 번 방법을 사용할 수 있다.

Spring AOP에서는 프록시를 이용한 3번 방법을 사용한다. 위 `Fruit` 인터페이스 예제 코드를 다시 떠올려 보자. 
Spring AOP는 `Fruit` 인터페이스의 프록시 객체를 자동으로 만들어준다. 따라서 `PrefixOfFruit`와 같은 프록시 클래스를 직접 구현할 필요가 없다. 
단지 공통 기능(로직)을 구현한 클래스만 따로 구현하고, AOP용 객체로 등록해주면 끝난다.

<br>

## AOP 주요 용어
### AOP 주요 용어와 의미
- Advice : 공통 관심 기능(Aspect)을 핵심 로직에 적용하는 **시점**.
- Joinpoint : Advice가 적용 **가능한 지점**.
- Porintcut : Advice가 실제로 **적용되는 지점**.
- Weaving : Advice를 적용하는 행위 자체.
- Aspect : 공통 기능.

### Advice 주요 용어와 의미
- Before Advice: 대상 메서드 호출 전에 실행
- After Returning Advice: 대상 메서드 동작 수행 후 실행
- After Throwing Advice: 대상 메서드 동작 수행 중 예외 발생시 실행
- After Advice: Returning + Throwing
- Around Advice: Before + Returning + Throwing

`Around Advice`가 가장 널리, 자주 사용된다.

<br>

## Spring AOP 구현
- Aspect로 사용할 클래스에 `@Aspect` 어노테이션을 붙인다.
- `@Pointcut` 어노테이션으로 공통 기능을 적용할 지점을 정의한다.
- `@Around` 어노테이션으로 공통 기능을 구현한 메서드를 지정한다.

### Aspect
```java
@Aspect
public class BeforeAndAfterOfFruit implements Fruit {

    @Pointcut("execution(public * chap07..*(..))")
    private void publicTarget() {
    }

    @Around("publicTarget()")
    public Object color(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("과일 가게에 있는 ");
        try {
            joinPoint.proceed();
        } finally {
            System.out.println("너무 맛있어요!");
        }
    }
}
```

`@Aspect` 어노테이션을 적용해서 Aspect로 사용할 클래스임을 명시했다.

`@Pointcut` 어노테이션으로 공통 기능을 적용할 대상을 지정했다. 

`@Around` 어노테이션으로 핵심로직 전후, 예외상황에서 `color` 메서드가 수행될 것임을 명시한다. `@Around` 어노테이션에 기입된 값이 `publicTarget()`인데, 바로 위 `publicTarget()` 메서드의 어노테이션 `@Pointcut` 에 지정되어 있는 핵심로직들을 대상으로 `color` 메서드가 `Around`하게 동작할 것임을 의미한다.

`ProceedingJoinPoint` 타입 파라미터는 프록시 대상 객체의 메서드를 호출할 때 사용한다.
`proceed()` 메서드를 사용해서 대상 객체의 메서드를 호출한다.

### Configuration
```java
@Configuration
@EnableAspectJAutoProxy
public class AppCtx {

    @Bean
    public BeforeAndAfterOfFruit beforeAndAfterOfFruit() {
        return new BeforeAndAfterOfFruit();
    }

    @Bean
    public Fruit fruit() {
        return new Apple();
    }
}
```

`@Aspect` 어노테이션을 붙인 클래스를 공통 기능으로 사용하려면 `@EnableAspectJAutoProxy` 어노테이션을 `@Configuration` 설정 클래스에 붙여야 한다. 이 어노테이션이 추가되면 
스프링은 `@Aspect` 어노테이션이 붙은 Bean 객체를 찾아 `@Pointcut`, `@Around` 설정을 읽어들인다.

### 동작 결과
```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = 
            new AnnotationConfigApplicationContext(AppCtx.class);

            Fruit fruit = ctx.getBean("fruit", Fruit.class);
            fruit.color();

            System.out.println(fruit.getClass().getName());
            ctx.close();
    }
}
```
```
과일 가게에 있는 
사과는 빨간색
너무 맛있어요!
com.sun.proxy.$Proxy17
```

`과일 가게에 있는`, `너무 맛있어요!`는 `BeforeAndAfterOfFruit` 클래스의 `color` 메서드가 수행된 결과다. `사과는 빨간색` 만이 `Fruit.color` 메서드의 수행 결과다.

`com.sun.proxy.$Proxy17`를 보면 `Fruit`타입이 `Apple`이 아니고 `$Proxy17`임을 알 수 있다. 즉, `Fruit.color`의 동작 수행을 `Apple`이 아닌 프록시 객체가 진행했다.

이 때 프록시 객체는 `Fruit` 인터페이스를 기반으로 생성된 것이지, `Apple`을 기반으로 생성된 것이 아니다. 
실제로 `@Configuration` 설정 클래스에서 반환 타입을 `Fruit`가 아닌 `Apple`로 지정할 경우 익셉션이 발생한다.

```java
@Configuration
@EnableAspectJAutoProxy
public class AppCtx {

    @Bean
    public BeforeAndAfterOfFruit beforeAndAfterOfFruit() {
        return new BeforeAndAfterOfFruit();
    }

    @Bean
    public Apple fruit() {  // -- 반환 타입이 Apple이 될 경우 예외 발생!
        return new Apple();
    }
}
```

굳이 인터페이스가 아닌 구현체로 프록시 객체를 생성되게 하고 싶다면 
`@EnableAspectJAutoProxy` 어노테이션에 속성을 추가해주면 된다.

```java
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class AppCtx {

    @Bean
    public BeforeAndAfterOfFruit beforeAndAfterOfFruit() {
        return new BeforeAndAfterOfFruit();
    }

    @Bean
    public Apple fruit() {  // -- 프록시 객체가 Apple을 기반으로 만들어지므로
        return new Apple(); // -- 예외 발생 없음!
    }
}
```

<br>

## Advice 순서 지정
하나의 `Pointcut`에 여러 `Advice`를 적용할 수도 있다. 그런데 이 때 `Advice`가 
동작되는 순서에 따라 결과가 달라질 수 있다.
이런 상황을 대비해서 `Advice`들의 순서를 지정하는 어노테이션이 바로 `@Order` 다.

```java
@Aspect
@Order(1) // @Order(2), @Order(3), ...
public class BeforeAndAfterOfFruit implements Fruit {

    @Pointcut("execution(public * chap07..*(..))")
    private void publicTarget() {
    }

    @Around("publicTarget()")
    public Object color(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("과일 가게에 있는 ");
        try {
            joinPoint.proceed();
        } finally {
            System.out.println("너무 맛있어요!");
        }
    }
}
```

<br>

## Aspect 객체에서 JoinPoint 객체의 정보 얻어내기
`ProceedingJoinnPoint` 인터페이스는 다음 메서드들을 제공한다.

- `Signature getSignature()`: 호출되는 메서드의 정보 반환
- `Object getTarget()`: 대상 객체 반환
- `Object[] getArgs()`: 파라미터 목록 반환

`org.aspectj.lang.Signature` 인터페이스는 다음 메서드들을 제공한다.

- `String getName()`: 호출되는 메서드의 이름 반환
- `String toLongString()`: 호출되는 메서드를 완전하게 표현한 문장(리턴 타입, 파라미터 타입 등) 반환
- `String toShortString()`: 호출되는 메서드를 축약해서 표현한 문장 반환

<br>

## AOP 는 자기 자신의 호출(내부 호출)에 응답할 수 없다.

```kotlin
@Service  
class GetSplashesUseCase {  
  @Cacheable(  
    value = [CacheConstant.SPLASH],  
    cacheManager = "ONE_DAY_CACHE_MANAGER",  
  )  
  fun getSplashes(): GotSplashes {  
    return ...
  }  
}
```

위와 같이 `@Cacheable` 을 이용해 `GotSplashes` 응답 결과를 캐싱해두는 로직이 있다.

```kotlin
@Transactional  
@Service  
class UpdateSplashUseCase {  
  fun updateSplash(command: SplashUpdateCommand): UpdatedSplash {  
    // ...
    clearAllCaches()
    return updatedSplash
  }
  
  @CacheEvict(value = [CacheConstant.SPLASH])  
  fun clearAllCaches() {  
    logger().info("clear all splash caches.")  
  }  
}
```

위와 같이 스플래시 업데이트 직후 내부 메서드를 호출하여 `@CacheEvict` 를 실행하면 어떤 일이 벌어질까?

결론은 아무런 일도 발생하지 않는다.
분명 public 한 메서드임에도 spring AOP 가 동작하지 않는 것이다.

그 원리는 당연하게도, 자기 자신의 메서드(타겟)을 직접 호출하므로, 메서드(타겟)을 감싼 프록시를 호출할 기회를 잃어버린 것이다.

![](https://i.imgur.com/dApV1cv.png)

해결 방법은 크게 3가지가 있다.

1. 자기 자신을 주입
	- 이 방식을 이용하면 생성자 주입시 무한 순환참조 문제가 발생한다.
	- 애초에 설계상으로도 좋아보이는 구조가 아니다.
	- 이 방식은 사용하지 말자.
2. 지연 조회
	- 생성자 주입을 이용하지 않고 수정자 주입을 이용하는 것 (setter, autowired)
	- 단 불필요한 spring application context 관련 코드가 많아진다.
	- 관심사 분리 이용을 위해서 사용하는 AOP 컨셉과 어울리지 않는다.
3. 설계 구조 변경
	- 애초에 AOP 를 이용하는 컨셉에 맞춰서 아예 별개의 클래스로 분리하여 관리한다.

### 설계 구조 변경 예시

```kotlin
@Transactional  
@Service  
class UpdateSplashUseCase(
  private val splashCacheClearUseCase: SplashCacheClearUseCase,
) {  
  fun updateSplash(command: SplashUpdateCommand): UpdatedSplash {  
    // ...
    splashCacheClearUseCase.clearAllCaches()
    return updatedSplash
  }
}

@Service
class SplashCacheClearUseCase {

  @CacheEvict(value = [CacheConstant.SPLASH])  
  fun clearAllCaches() {  
    logger().info("clear all splash caches.")  
  }  
}
```

위와 같이 별도로 의존성을 주입 받아서 AOP 메서드를 실행시킴으로서 해결할 수 있다.

![](https://i.imgur.com/isH7nYj.png)