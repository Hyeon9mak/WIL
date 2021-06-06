# Exception을 한 단계 추상화하여 관리하기

```java
@RestControllerAdvice
public class ExceptionController {

    @ExceptionHandler(DuplicateKeyException.class)
    public ResponseEntity<ExceptionResponse> duplicateExceptionResponse(DuplicateKeyException e) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
            .build(new ExceptionResponse(e.getMessage());
    }

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ExceptionResponse> notFoundExceptionResponse(NotFoundException e) {
        return ResponseEntity.notFound()
            .build(new ExceptionResponse(e.getMessage());
    }

    @ExceptionHandler(EmptyResultDataAccessException.class)
    public ResponseEntity<ExceptionResponse> voidLineDeleteExceptionResponse(
        EmptyResultDataAccessException e) {
        return ResponseEntity.notFound()
            .build(new ExceptionResponse(e.getMessage());
    }

    @ExceptionHandler(InvalidDistanceException.class)
    public ResponseEntity<ExceptionResponse> invalidDistanceExceptionResponse(InvalidDistanceException e) {
        return ResponseEntity.badRequest()
            .build(new ExceptionResponse(e.getMessage());
    }

    @ExceptionHandler({NullIdException.class, NullNameException.class, NullColorException.class})
    public ResponseEntity<ExceptionResponse> nullExceptionResponse(NullException e) {
        return ResponseEntity.badRequest()
            .build(new ExceptionResponse(e.getMessage());
    }

    @ExceptionHandler(DuplicateException.class)
    public ResponseEntity<ExceptionResponse> duplicatedStationExceptionResponse(DuplicateException e) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
            .build(new ExceptionResponse(e.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ExceptionResponse> methodArgumentNotValidExceptionResponse(
        MethodArgumentNotValidException e) {
        return ResponseEntity.badRequest()
            .build(new ExceptionResponse(e.getMessage());
    }

    @ExceptionHandler(InvalidSectionOnLineException.class)
    public ResponseEntity<ExceptionResponse> alreadyExistedStationsOnLineExceptionResponse(
        InvalidSectionOnLineException e) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .build(new ExceptionResponse(e.getMessage());
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ExceptionResponse> illegalArgumentExceptionResponse(IllegalArgumentException e) {
        return ResponseEntity.notFound()
            .build(new ExceptionResponse(e.getMessage());
    }

    @ExceptionHandler(UnauthorizedException.class)
    public ResponseEntity<String> unauthorizedException(UnauthorizedException unauthorizedException) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(unauthorizedException.getMessage());
    }
}
```

위와 같은 예외 핸들링 방식을 사용하면, 커스텀 예외가 추가될 때마다 핸들러 메서드도 하나씩 추가되어야 하는 
크나큰 단점(코드 중복)이 발생하게 된다.

이를 해결하기 위해서 커스텀 예외를 한 단계 추상화하면, 핸들러 메서드의 수를 기하급수적으로 줄일 수 있다.

```java
public abstract class SubwayException extends RuntimeException {

    public SubwayException(String message) {
        super(message);
    }

    public abstract HttpStatus status();

    public abstract String error();
}

```
```java
public class InvalidEmailException extends SubwayException {

    public InvalidEmailException() {
        super("올바르지 않은 이메일 형식입니다.");
    }

    @Override
    public HttpStatus status() {
        return HttpStatus.BAD_REQUEST;
    }

    @Override
    public String error() {
        return "INVALID_EMAIL";
    }
}
```

각 예외마다 상태코드와 상세 메세지를 미리 지정해주면 핸들러 클래스가 아래와 같이 변한다.

```java
@RestControllerAdvice
public class ExceptionAdvice {

    @ExceptionHandler(SubwayException.class)
    public ResponseEntity<ExceptionResponse> exceptionResponse(SubwayException e) {
        return ResponseEntity.status(e.status())
            .body(new ExceptionResponse(e.error()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ExceptionResponse> exceptionResponse(MethodArgumentNotValidException e) {
        return ResponseEntity.badRequest()
            .body(new ExceptionResponse(e.getMessage()));
    }
}
```
