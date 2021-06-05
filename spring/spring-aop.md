## Spring AOP
AOP(Aspect Oriented Programming)은 관점 지향 프로그래밍으로, 어떤 로직에 대해 핵심적인 관점, 부가적인 관점을 나누어 보고 그 관점들을 기준으로 각각 모듈화를 하겠다는 것이다.
(모듈화 - 공통된 로직이나 기능을 하나의 단위로 묶는 것)

예를 들어 핵심적인 관점을 우리가 적용하고자 하는 핵심 비즈니스 로직.
부가적인 관점은 그에 필요한 데이터베이스 커넥션, 로깅, 파일I/O 등이 있다.

![image-20210514171338061](/Users/hyeon9mak/Library/Application Support/typora-user-images/image-20210514171338061.png)

(이미지 출처 - https://engkimbs.tistory.com/746)

AOP 관점에서 로직을 모듈화한다는 것은 코드들을 부분적으로 나누어 다른 부분에서 계속해서 반복되는 코드들을 발견하고 이것들을 모듈로 만들어 재사용하겠다는 것. "계속해서 반복되는 코드들"을 **흩어진 관심사(Crosscutting Concerns)** 라고 부른다.

<br>
