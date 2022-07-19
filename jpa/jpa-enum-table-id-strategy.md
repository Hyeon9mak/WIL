# 테이블로 직접 관리되는 Enum의 ID 전략 선택

간혹 레거시 코드, DB 테이블 중에 Enum 값을 테이블로 직접 만들어서 관리하는 경우가 있다.

<img width="803" alt="image" src="https://user-images.githubusercontent.com/37354145/179743926-fa09fd77-df96-47d6-9671-2c4d762921e4.png">

이러한 Enum을 가진 엔티티를 통합 테스트하고자 할 때, Enum 엔티티의 PK 값이 발목을 붙잡는 경우가 있었다.
id 값이 깔끔하게 1, 4, 3 등으로 선택 되어야 Enum 값 활용이 가능한데, AUTO_INCREMENT 옵션에 의해서 
유효하지 않은 값이 선택되거나 없는 값이 선택되는 경우다.
때문에 테스트 코드를 작성하는데 있어 굉장히 애를 먹었는데, 생각보다 해결 방법이 간단했다.
Enum 엔티티에 선언되어 있는 `@GeneratedValue(strategy = GenerationType.IDENTITY)`를 제거하는 것이다.

```kotlin
// as-is
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
@Column(name = "id", nullable = false)
var id: Long = 0
    private set

// to-be
@Id
@Column(name = "id", nullable = false)
var id: Long = 0
    private set
```

Enum 엔티티는 새로운 데이터가 추가되는 일이 거의 없고, local, stage, production 환경에 차별없이 
모두 동일한 ID 값을 사용해야하기 때문에 `@GeneratedValue(strategy = GenerationType.IDENTITY)`를 애초에 쓸 필요가 없던 것이다.
