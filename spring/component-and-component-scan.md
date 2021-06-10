# 컴포넌트 스캔

## @Component 
클래스 선언부 위 쪽에 `@Component` 어노테이션을 붙이면 
`@Configuration` 클래스에 별도로 Bean 등록 과정을 거치지 않아도 
스프링이 패키지 전체를 탐색해서 Bean으로 등록 할 수 있다.

```java
@Component
public class AppleDao {
    ...
}
```

별도의 Bean 등록 메서드가 필요 없기 때문에 설정 코드가 크게 줄어든다.

`@Component` 어노테이션으로 Bean의 이름도 지정해줄 수 있다.  
(Bean 등록 메서드의 `@Qualifier("~")`와 비슷한 동작)

```java
@Component("switBananaDao")
public class BananaDao {
    ...
}
```

앞선 `AppleDao`의 경우 별도로 이름을 지정해주지 않았는데, 이 경우 자동으로 `appleDao`의 이름을 갖게 된다.
즉, 이름을 설정하지 않을 경우 **클래스 이름의 첫 글자만 소문자로 바꾼 Bean 이름**이 사용된다.

<br>

## @ComponentScan
### @ComponentScan - basePackages
`@Component` 어노테이션이 스캔 되는 범위를 지정하려면 설정 클래스에 `@ComponentScan` 어노테이션을 추가해야 한다.

```java
@Configuration
@ComponentScan(basePackages = {"spring"})
public class AppCtx {
    ...
}
```

`@ComponentScan(basePackages = {"spring"})` 설정을 통해 스캔 대상 패키지 목록을 지정한다. 
배열 형태이기 때문에 여러 패키지를 동시에 지정할 수도 있다. 지정된 패키지와 그 하위 패키지에 속한 클래스를 대상으로 스캔을 진행한다.

> `basePackages` 를 지정하지 않을 경우 전체 패키지를 스캔한다. 
> 이 경우 `@ComponentScan` 어노테이션을 사용하지 않는 것과 별반 다름 없다.

### @ComponentScan - excludeFilters - pattern
`excludeFilters` 속성을 사용하면 특정 대상을 스캔 대상에서 제외할 수 있다.

```java
@Configuration
@ComponentScan(basePackages = {"spring"},
    excludeFilters = @Filter(type = FilterType.REGEX, pattern = "spring\\..*Dao"))
public class AppCtx {
    ...
}
```

위 코드는 `@Filter` 어노테이션의 type 속성으로 `FilterType.REGEX`(정규표현식)을 선택했다.

```java
@Configuration
@ComponentScan(basePackages = {"spring"},
    excludeFilters = @Filter(type = FilterType.ASPECTJ, pattern = "spring.*Dao"))
public class AppCtx {
    ...
}
```

위 코드는 `@Filter` 어노테이션의 type 속성으로 `FilterType.ASPECTJ`를 선택했다. 
AspectJ 패턴을 사용하려면 build.gradle 파일에 aspectjweaver 모듈을 추가해주면 된다.

`pattern` 속성 역시도 `basePackges` 처럼 배열 형태를 사용해서 여러 패턴을 지정해줄 수 있다.

### @ComponentScan - excludeFilters - classes
특정 어노테이션을 붙이고 있는 Bean들을 제외 시키고 싶다면 `FilterType.ANNOTATION`을 사용할 수 있다.

```java
@Configuration
@ComponentScan(basePackages = {"spring"},
    excludeFilters = @Filter(type = FilterType.ANNOTATION, 
    classes = {NoProduct.class, ManualBean.class})
public class AppCtx {
    ...
}
```

`FilterType.ANNOTATION`을 사용하면 `classes` 속성에 필터로 사용할 어노테이션의 값을 지정해줄 수 있다.

특정 클래스나 그 하위 클래스를 스캔 대상에서 제외하려면 `FilterType.ASSIGNABLE_TYPE`을 사용할 수 있다.

```java
@Configuration
@ComponentScan(basePackages = {"spring"},
    excludeFilters = @Filter(type = FilterType.ASSIGNABLE_TYPE, 
    classes = MemberDao.class})
public class AppCtx {
    ...
}
```

역시나 배열 형태를 사용해서 여러 클래스들을 지정할 수 있다.

### @ComponentScan - excludeFilters - 필터 여러개
만일 여러 `@Filter`를 등록하고 싶다면 아래와 같이 이용하면 된다.

```java
@Configuration
@ComponentScan(basePackages = {"spring"},
    excludeFilters = {
        @Filter(type = FilterType.ANNOTATION, classes = ManualBean.class)
        @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = MemberDao.class)
    })
public class AppCtx {
    ...
}
```

<br>

## 스캔 대상
`@Component` 뿐만 아니라 다른 어노테이션을 붙인 클래스도 스캔 대상이 될 수 있다.

- `@Component`
- `@Controller`
- `@Service`
- `@Repository`
- `@Configuration`
- `@Aspect`

이 때 `@Aspect`를 제외한 나머지 어노테이션들은 모두 `@Component` 어노테이션을 내포하면서 특수한 기능들을 가진 어노테이션들이다.

<br>

## 컴포넌트 스캔 충돌 처리
### Bean 이름 충돌
```java
@Configuration
@ComponentScan(basePackages = {"spring", "spring2"})
public class AppCtx {
    ...
}
```

위와 같이 서로 다른 패키지에 같은 이름을 가진 Bean 클래스가 2개 존재하고, 2개의 클래스 모두 `@Component` 어노테이션을 갖고 있을 경우 Exception이 발생한다. 이 때문에 `@Component` 어노테이션에 
이름을 지정하는 등의 방법으로 충돌을 피해야 한다.

### @Configuration 클래스에 수동 등록한 Bean과 충돌
```java
@Component
public class MemberDao {
    ...
}
```
```java
@Configuration
@ComponentScan(basePackages = {"spring"})
public class AppCtx {
    
    @Bean
    public MemberDao memberDao() {
        return new MemberDao();
    }
}
```

스캔할 때 사용하는 Bean 이름과 수동 등록 Bean 이름이 같은 경우에도 충돌이 발생한다. 
이 때는 Exception이 발생하지 않고 수동으로 등록된 Bean이 우선권을 가진다.

> Bean 주입에서는 `@Autowired`가 수동보다 우선권을 가졌지만, Bean 등록에서는 반대의 양상을 보인다.
