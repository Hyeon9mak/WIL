# data class vs value class

## data class

- 일반적으로 자바 개발자가 생각하는 DTO에 가장 유사한 형태의 클래스
- `equals()`, `toString()`, `hashCode()`, `copy()`, `componentN()`를 자동으로 구현해주므로 보일러 플레이트 코드가 사라짐
- 프로퍼티를 val, var 로 모두 구현할 수 있다.
- 상속을 허용하지 않으며, abstract, open, sealed, inner 클래스도 될 수 없음
- 주 생성자에 1개 이상의 프로퍼티가 무조건 선언되어야 함
- 비유하자면 `A9BJ3145CB` 라는 고유 번호를 가진 1,000원 권 지폐

<br>

## value class

- `equals()`, `toString()`, `hashCode()` 까지만 자동으로 구현해줌
- 프로퍼티를 val 로만 구현할 수 있다. 무조건적인 불변제약
- 동일성(identity) 개념이 없다. 동등성 개념만 존재. 즉, '값 그 자체'를 뜻하는 클래스
- 비유하자면 1,000원 이라는 금액 그 자체
- 현재는 프로퍼티를 1개만 가질 수 있지만, 곧 data class 처럼 여러개의 프로퍼티를 가질 수 있게 될 예정
- 향후 젯브레인사의 Valhalla 프로젝트가 적용되면, JVM이 컴파일단계에서 value 클래스를 JVM 기본 클래스로 구현할 수 있게 된다고 함 (엄청난 최적화 가능성)

<br>

## References

- [Kotlin 1.5에 추가된 value class에 대해 알아보자](https://velog.io/@dhwlddjgmanf/Kotlin-1.5%EC%97%90-%EC%B6%94%EA%B0%80%EB%90%9C-value-class%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90)
- [Kotlin 1.4.30의 새로운 언어 기능 테스트 버전](https://blog.jetbrains.com/ko/kotlin/2021/02/new-language-features-preview-in-kotlin-1-4-30/)
