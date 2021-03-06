## @EnableWebMvc
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```
`DelegatingWebMvcConfiguration` 클래스를 import 하고 있다.

## 계층을 나누는 이유 - 유지보수 편의성
주요 로직 레이어(서비스 레이어-DAO 등)과 컨트롤러가 분리되어 있지않은 경우  
→ API 명세가 달라졌을 때 전체 서비스가 영향을 받는다

주요 로직 레이어가 컨트롤러와 분리되어 있느 경우  
→ API 명세가 달라졌으 때 컨트롤러 쪽만 영향을 받게 된다.

이런 계층 분리는 리팩토링의 범위를 획기적으로 줄여준다.

Dao 가 만일 주요 로직을 함께 포함하고 있다면, DB 변경과 DAO영역이 함께 수정의 영향을 받게 된다.

서비스 레이어와 DAO레이어를확실하게 나누면 DB를 수정할 때 영향을 최소화 할 수 있다.

즉 레이어를 나누는 가장 큰 이유는 변경의 영향을 최소화 하기 위함이다!

<br>

## 즉, 레이어 별 역할을 명확하게 나누면 더 좋겠다!
레이어 아키텍쳐가 완벽한 아키텍쳐는 아니지만, 유지보수에도 좋고 이해하기 쉬운 개념의 아키텍쳐다!
그렇다면 핵심 비즈니스 로직은 어디에 위치해야할까   
→ 구간 추가 기능을 구현할 때 주요 핵심 로직은 어디에 위치해야할까

### 도메인? vs 서비스?
도메인을 가볍게 구현 vs 도메인을 풍부하게 구현하는 차이라고 볼 수 있다

도메인을 가볍게 만들 경우 서비스 레이어에 절차지향식 코드가 만들어지게 된다.

도메인을 풍부하게 만들 경우 서비스 레이어의 부담이 줄어들고, 서비스 레이어가 굉장히 단순해진다.  
→ 즉, 도메인을 엔티티로서만 사용하려하지말고, 풍부하게 비즈니스 로직을 섞어라.

<br>

## 모든 레이어 별로 테스트를 진행해야하는가? 
→ 결국 내가 검증하고 싶은게 무엇인지 파악하고 필요한 만큼 진행하는게 가장 중요하겠다. 정해진 룰은 없다.