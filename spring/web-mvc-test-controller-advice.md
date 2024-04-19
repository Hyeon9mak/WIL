## 문제 상황

`@WebMvcTest` 를 이용하여 controller 를 테스트 하고자 할 때, 테스트 대상이 아닌 controller 와, 그 controller 들이 의존하고 있는 모든 spring bean 을 불러오려고 하는 문제가 발생핬다.

> `@WevMvcTest` 는 대상 controller 만 spring bean 을 주입받고, 나머지들은 mock 처리를 함으로서 슬라이싱 테스트가 가능토록 해주는 편의를 제공해준다.

## 원인
`@WebMvcTest` 어노테이션을 사용하면 spring bean 전체 자동 구성이 비활성화 되고,
대신 MVC 테스트와 관련된 구성(`@Controller`, `@ControllerAdvice`, `@JsonComponent`, `Converter/GenericConverter`, `Filter`, `WebMvcConfigurer`,  `HandlerMethodArgumentResolver` been, `@Component`, `@Service`, `@Repository` been)만 적용된다.

문제는 현재 프로젝트 구성에서 하나의 controller class 에 `@RestController` 와 `@RestControllerAdvice` 어노테이션을 모두 사용하고 있었다. 

해당 controller 에 대한 테스트를 진행하기 위해 `@WebMvcTest(xxxController.class)` 명시하게 될 경우, `@RestController` 와 `@RestControllerAdvice` 가 모두 불러와지게 되는데, `@ControllerAdvice` 특성상 자신과 같은 패키지 + 하위 패키지에 있는 모든 핸들러들을 탐색하려고 시도하고, 그 때문에 같은 패키지에 있는 controller 들이 모두 대상으로 잡히면서 spring bean 을 요구하게 된 것이다.

## 해결

2가지 해결 방식이 존재한다.

### 1. excludeFilters

```java
@WebMvcTest(
  controllers = xxxController.class, 
  excludeFilters = { 
    @ComponentScan.Filter(
      type = FilterType.ANNOTATION, 
      classes = ControllerAdvice.class) 
  }
)
```

### 2. controller advice 분리

`@RestController` 와 `@RestControllerAdvice` 를 하나의 클래스 파일에서 사용하지 않는 것.

개인적으로 2번 방식이 더 깔끔한 듯.