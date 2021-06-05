## @ControllerAdvice, @RestControllerAdvice vs @ResponseStatus

> Q. @RestControllerAdvice 어노테이션을 통해서 exception들을 핸들링 하곤 했는데, 대부분 ResponseEntity로 Http 상태코드만 반환하곤 했습니다. 그러다 오늘 @ResponseStatus 라는 어노테이션을 알게되었는데, exception 클래스 위에 붙여서 자동으로 Http 상태코드를 반환할 수 있는 것 같더라구요!
별도로 Advice 클래스를 만들지 않아도 자동으로 Http 상태코드 response를 반환해준다는게 굉장한 장점인 것 같은데, 현업에서는 어떤 방식이 더 선호되는지 궁금합니다!

> A. 당연한 얘기일 수 있지만 프로젝트 성격과 프로젝트 참여하는 사람 취향에 따라 달라질 것 같아요! 섞어서 쓰는 경우도 많을 것 같고요!
예시를 전달하기 위해 저희 서비스를 말씀드리면 예외에 따라 Body와 Header가 크게 변경되는 부분이 있고, 예외 상황에 따라 별도의 비즈니스 로직을 처리하는 경우가 있어 대부분은 ExceptionHandler를 사용하는 것 같습니다!
답변드리며 든 생각인데..! ExceptionHandler와 ResponseStatus를 섞어쓰는 경우는 있었어도, ResponseStatus 만으로 서비스를 했던 기억은 없네요 ㅎㅎ

@ControllerAdvice만 쓰거나 @ResponseStatus를 섞어쓰는 경우는 있어도, @ResponseStatus만 사용하는 경우는 없다.
