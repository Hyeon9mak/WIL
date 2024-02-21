## Querydsl 조회 결과를 DTO 에 매핑할 때, DTO 의 필드가 엔티티가 아닌 다른 클래스(VO, DTO)인 경우

- Querydsl 의 SELECT 조건 문에서 DTO 매핑에 활용할 수 있는 방법은 크게 3가지다.
    - `Projections.constructor` 생성자를 이용한 주입
    - `Projections.bean` getter/setter 이용한 주입
    - `Projection.fields` 필드 직접 주입
- 그러나 3가지 방법 모두 필드가 엔티티(Q객체)와 일치하는 경우에만 활용 가능하다.
- DTO의 필드를 엔티티가 아닌 다른 클래스로 가지고 있는 경우 부생성자를 만든 `Proejctions.constructor` 방식으로 해결이 가능하다!
- [http://querydsl.com/static/querydsl/4.4.0/apidocs/com/querydsl/core/types/Projections.html](http://querydsl.com/static/querydsl/4.4.0/apidocs/com/querydsl/core/types/Projections.html) 참고
