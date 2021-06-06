# valid annotation
## @NotNull, @NotEmpty, @NotBlank

`@NotNull` 은 말 그대로 Null만 허용하지 않는다. 이 때문에 `""`, `" "` 는 허용하게 된다.
Null이 들어오면 로직에 예상치 못한 오류가 발생하거나, 문제가 생길 경우에 사용된다.

`@NotEmpty` 는 Null과 `""` 를 허용하지 않는다. `@NotNull` 쪽에 `""` 검증이 추가된 것이다.

`@NotBlank`  는 Null과 `""`, `" "` 를 모두 허용하지 않는다.

즉, 3개 중 가장 검증 강도가 높다.



### validation 어노테이션을 꼭 사용해야만 하는가?

```java
    public static LineServiceDto from(final Long id, final LineRequest lineRequest) {
        if (Objects.isNull(id)) {
            throw new SomeException();
        }
        return new LineServiceDto(id, lineRequest.getName(), lineRequest.getColor());
    }
```

정적 팩터리 메서드를 통해서도 충분히 id Null에 대해서 예외처리를 해줄 수 있다.
validation 어노테이션이 편리함을 제공해주는건 맞지만, 맹목적으로 거기에 의존할 필요는 없다.
가장 중요한 것은 검증하는 방식이 아니라, 검증을 해야하는 이유(목적)과 검증 그 자체다.

<br>

## @Valid 어노테이션과 사용 가능한 종류
| Anotation                      | 제약조건                                          |
| ------------------------------ | ------------------------------------------------- |
| @NotNull                       | Null 불가                                         |
| @Null                          | Null만 입력 가능                                  |
| @NotEmpty                      | Null, 빈 문자열 불가                              |
| @NotBlank                      | Null, 빈 문자열, 스페이스만 있는 문자열 불가      |
| @Size(min=,max=)               | 문자열, 배열등의 크기가 만족하는가?               |
| @Pattern(regex=)               | 정규식을 만족하는가?                              |
| @Max(숫자)                     | 지정 값 이하인가?                                 |
| @Min(숫자)                     | 지정 값 이상인가                                  |
| @Future                        | 현재 보다 미래인가?                               |
| @Past                          | 현재 보다 과거인가?                               |
| @Positive                      | 양수만 가능                                       |
| @PositiveOrZero                | 양수와 0만 가능                                   |
| @Negative                      | 음수만 가능                                       |
| @NegativeOrZero                | 음수와 0만 가능                                   |
| @Email                         | 이메일 형식만 가능                                |
| @Digits(integer=, fraction = ) | 대상 수가 지정된 정수와 소수 자리 수 보다 작은가? |
| @DecimalMax(value=)            | 지정된 값(실수) 이하인가?                         |
| @DecimalMin(value=)            | 지정된 값(실수) 이상인가?                         |
| @AssertFalse                   | false 인가?                                       |
| @AssertTrue                    | true 인가?                                        |

<br>

## Response 방향의 DTO에는 Validation 어노테이션을 붙이지 않는다.
`'org.springframework.boot:spring-boot-starter-validation'` 의존성을 추가하면 DTO 단에 어노테이션을 붙이는 것으로 손쉽게 값 검증이 가능하다.

우선 검증을 진행하고 싶은 필드 위에 `@NotNull`, `@NotEmpty` 등의 어노테이션을 붙인다.
```java
public class MemberRequest {

    @NotNull(message = "INVALID_INPUT")
    private String email;
    @NotNull(message = "INVALID_INPUT")
    private String password;
    @NotNull(message = "INVALID_INPUT")
    private Integer age;

    public MemberRequest() {
    }

    public MemberRequest(String email, String password, Integer age) {
        this.email = email;
        this.password = password;
        this.age = age;
    }

    public String getEmail() {
        return email;
    }

    public String getPassword() {
        return password;
    }

    public Integer getAge() {
        return age;
    }
}
```

그 후 DTO가 인자로 사용되는 지점에 `@Valid` 어노테이션을 붙이면, 자동으로 검증이 진행된다.

```java
@RestController
@RequestMapping("/api/members")
public class MemberController {

    private final MemberService memberService;

    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }

    @PostMapping
    public ResponseEntity createMember(@Valid @RequestBody MemberRequest request) {
        MemberResponse member = memberService.createMember(request);
        return ResponseEntity.created(URI.create("/members/" + member.getId())).build();
    }
}
```

이렇게 Request 방향의 자동 검증은 지원하지만, Response 방향의 자동 검증은 지원하지 않는다. 
아무리 눈을 씻고 찾아봐도 마땅히 `@Valid` 어노테이션을 붙일 곳이 보이지 않는다.

'왜 Response 방향의 Validation 어노테이션을 붙일 곳이 없을까' 라는 고민하다가, 
'굳이 할 필요가 없으니까 없겠구나' 라는 결론에 도달했다.

검증 관련 문제가 발생할거라면 Request -> Domain 으로 변환되는 과정에서 먼저 발생했어야하고, 
그 이후의 데이터는 이미 서비스 도메인으로 변환이 성공됐기 때문에 믿고 쓴다는 느낌인거 같다.
