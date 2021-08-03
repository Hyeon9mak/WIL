# @NotNull 어노테이션 예외처리 핸들링

## summary
lombok에서 지원하는 `@NonNull` 어노테이션을 통해 엔티티의 필드를 검증하던 중, 
`@NonNull` 어노테이션이 필드에 Null 값이 주입될 경우 `NullPointerException`이 던져지는 것을 발견했다.
프로젝트의 ControllerAdivce 구조상 `RuntimeException`을 한꺼번에 처리하고 있었기 때문에, 
`RumtimeException`을 상속한 `NullPointerException` 대신 custom exception이나 별도의 exception이 던져지길 원했다. 
(꼭 `@NonNull`이 아니어도 충분히 다른 이유에서 `NullPointerException`이 던져질 수도 있었다.)

```java
@RestControllerAdvice
public class BabbleAdvice {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ExceptionDto> unexpectedException(Exception e) {
        return ResponseEntity.badRequest().body(new ExceptionDto("unexpected exception"));
    }

    @ExceptionHandler(RuntimeException.class) // 여기에 걸려버린다!
    public ResponseEntity<ExceptionDto> unexpectedRuntimeException(RuntimeException e) {
        return ResponseEntity.badRequest().body(new ExceptionDto("unexpected runtime exception"));
    }

    @ExceptionHandler(BabbleException.class)
    public ResponseEntity<ExceptionDto> babbleException(BabbleException e) {
        return ResponseEntity.status(e.status())
            .body(new ExceptionDto(e.getMessage()));
    }

    @MessageExceptionHandler
    public ResponseEntity<ExceptionDto> handleException(BabbleException e) {
        return ResponseEntity.status(e.status())
            .body(new ExceptionDto(e.getMessage()));
    }
```

엔티티의 생성자에 검증 메서드를 직접 작성할까 고민하던 중, `javax.validation.constraints`의 `@NotNull` 어노테이션을 엔티티에도 붙여 사용할 수 있음을 알게 되었다. `@NotNull` 어노테이션은 `MethodArgumentNotValidException`와 같은 예외를 던지므로, 별도의 예외 핸들링 메서드를 구현해 문제를 해결하고자 했다.

<br>

## MethodArgumentNotValidException - DTO 예외

```java
// Request DTO
@Getter
@AllArgsConstructor
@NoArgsConstructor
public class UserRequest {

    @NotNull(message = "[Request] 유저 이름은 Null 일 수 없습니다.")
    private String name;
}
```
```java
// Controller
    @PostMapping
    public ResponseEntity<UserResponse> create(@Valid @RequestBody UserRequest userRequest) {
        return ResponseEntity.ok(userService.save(userRequest));
    }
```

`MethodArgumentNotValidException` 예외는 주로 DTO 필드에 붙은 `@NotNull` 어노테이션과 
컨트롤러 파라미터 앞에 붙은 `@Valid` 어노테이션을 통해 던져진다. `@NotNull` 어노테이션을 DTO 필드에 붙여두어도, 
해당 DTO를 받아내는 메서드 파라미터 위치에 `@Valid` 어노테이션을 추가하지 않으면 검증이 동작하지 않는다.

우선 아래와 같은 핸들링 메서드를 작성해서 `"[Request] 유저 이름은 Null 일 수 없습니다."` 메세지가 출력되는지 테스트 해보았다.

```java
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ExceptionDto> methodArgumentValidException(MethodArgumentNotValidException exception) {
        return ResponseEntity.badRequest().body(new ExceptionDto(exception.getMessage()));
    }
```
```
"message": "Validation failed for argument [0] in public ...(생략)... : [Field error in object 'userRequest' on field 'nickname': rejected value [null]; ...(생략)... default message [nickname]]; default message [[Request] 유저 이름은 Null 일 수 없습니다.]] "
```

도저히 알아 볼 수 없는 형태의 긴 문장이 출력된다. 수 많은 데이터들 중에서 `"[Request] 유저 이름은 Null 일 수 없습니다."` 메세지만 보고 싶은 경우 아래와 같은 파싱 작업이 필요하다.

```java
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<List<ExceptionDto>> methodArgumentValidException(MethodArgumentNotValidException e) {
        return ResponseEntity.badRequest().body(extractErrorMessages(e));
    }

    private List<ExceptionDto> extractErrorMessages(MethodArgumentNotValidException e) {
        return e.getBindingResult()
            .getAllErrors()
            .stream()
            .map(DefaultMessageSourceResolvable::getDefaultMessage)
            .map(ExceptionDto::new)
            .collect(Collectors.toList());
    }
```
```
{
    "message": "[Request] 유저 이름은 Null 일 수 없습니다."
}
```

`extractErrorMessages` 파싱 메서드를 통해 보고 싶은 메세지만 예쁘게 뽑아낸 것을 볼 수 있다. 
자세히 살펴보면 반환 값이 `ExceptionDto`에서 `List<ExceptionDto>`로 변화한 것을 알 수 있는데, 
이는 `@Valid` 어노테이션을 통해 하나의 `@NotNull` 검증 예외만 잡아 내는 것이 아니라, 해당 DTO의 필드에 붙은 
모든 검증 어노테이션의 예외를 한꺼번에 전달하는 용도로 파악된다.

<br>

## ConstraintViolationException - 엔티티 예외
엔티티 역시 같은 `MethodArgumentNotValidException`을 던질것이라 예상했지만, 별개로 `ConstraintViolationException`을 던지고 있었다. 

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotNull(message = "[Entity] 유저 이름은 Null 일 수 없습니다.")
    private String name;
    
    ...
}
```

엔티티의 필드에 붙은 `@NotNull` 어노테이션은 `@Valid` 어노테이션 없이 동작한다. JPA(Hibernate)가 
`@NotNull` 어노테이션을 읽어 동작하기 때문이다.

```java
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<List<ExceptionDto>> constraintViolationException(ConstraintViolationException e) {
        return ResponseEntity.badRequest().body(extractErrorMessages(e));
    }
```
```
"message": "Validation failed ...(생략)... interpolatedMessage='[Entity] 유저 이름은 Null 일 수 없습니다.', ...(생략)... [Entity] 유저 이름은 Null 일 수 없습니다.'}\n]"
```

이번에도 역시 알아보기 힘든 문장이 출력된다. 파싱을 진행하자.

```java
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<List<ExceptionDto>> constraintViolationException(ConstraintViolationException e) {
        return ResponseEntity.badRequest().body(extractErrorMessages(e));
    }

    private List<ExceptionDto> extractErrorMessages(ConstraintViolationException e) {
        return e.getConstraintViolations()
            .stream()
            .map(ConstraintViolation::getMessage)
            .map(ExceptionDto::new)
            .collect(Collectors.toList());
    }
```
```
{
    "message": "[Entity] 유저 이름은 Null 일 수 없습니다."
}
```

엔티티의 `@NotNull` 역시 깔끔하게 에러 메세지를 얻어낼 수 있게 되었다!

> 엔티티의 `@NotNull` 어노테이션에 대해서는 [@NotNull vs @Column(nullable = false)](https://github.com/Hyeon9mak/WIL/blob/main/jpa/not-null-vs-column-nullable-false.md) 글을 참고하자.

최종적으로 `RumtimeException` 핸들링 메서드에 포함되지 않는 별도의 검증 어노테이션으로 `@NotNull`을 사용할 수 있게 되었다.

```java
@RestControllerAdvice
public class BabbleAdvice {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ExceptionDto> unexpectedException(Exception e) {
        return ResponseEntity.badRequest().body(new ExceptionDto("unexpected exception"));
    }

    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<ExceptionDto> unexpectedRuntimeException(RuntimeException e) {
        return ResponseEntity.badRequest().body(new ExceptionDto("unexpected runtime exception"));
    }

    @ExceptionHandler(BabbleException.class)
    public ResponseEntity<ExceptionDto> babbleException(BabbleException e) {
        return ResponseEntity.status(e.status())
            .body(new ExceptionDto(e.getMessage()));
    }

    @MessageExceptionHandler
    public ResponseEntity<ExceptionDto> handleException(BabbleException e) {
        return ResponseEntity.status(e.status())
            .body(new ExceptionDto(e.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<List<ExceptionDto>> methodArgumentValidException(MethodArgumentNotValidException e) {
        return ResponseEntity.badRequest().body(extractErrorMessages(e));
    }

    private List<ExceptionDto> extractErrorMessages(MethodArgumentNotValidException e) {
        return e.getBindingResult()
            .getAllErrors()
            .stream()
            .map(DefaultMessageSourceResolvable::getDefaultMessage)
            .map(ExceptionDto::new)
            .collect(Collectors.toList());
    }

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<List<ExceptionDto>> constraintViolationException(ConstraintViolationException e) {
        return ResponseEntity.badRequest().body(extractErrorMessages(exception));
    }

    private List<ExceptionDto> extractErrorMessages(ConstraintViolationException e) {
        return e.getConstraintViolations().stream()
            .map(ConstraintViolation::getMessage)
            .map(ExceptionDto::new)
            .collect(Collectors.toList());
    }

```

<br>

## References
- [ParameterValidationException (Spring Shell Documentation ...](https://docs.spring.io/spring-shell/docs/current/api/org/springframework/shell/ParameterValidationException.html)