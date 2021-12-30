# @MappeedSuperclass
`@MappedSuperclass` 어노테이션은 데이터베이스 테이블은 건드리지 않고,
객체(애플리케이션) 입장에서 공통되는 필드를 추상화하여 관리하고 싶을 때 사용하는 어노테이션이다.

![image](https://user-images.githubusercontent.com/37354145/147714014-dcbc637d-945d-4dfb-bfd2-f4aa44f726aa.png)

즉, 상속관계와는 엄연히 다르다.

<br>

## 사용 방법
```java
@Entity
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    
    private LocalDateTime createdAt;
    private LocalDateTime modifiedAt;
}

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private LocalDateTime createdAt;
    private LocalDateTime modifiedAt;
}
```

위와 같은 클래스 구성에서 중복되는 필드를 추출하고 싶을 때 `@MappedSuperclass`를 사용한다.

```java
@MappedSuperclass
public abstract class BaseEntity {

    private LocalDateTime createdAt;
    private LocalDateTime modifiedAt;
}

@Entity
public class Team extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
}

@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
}
```

이외에도 공통으로 사용하는 컬럼에 공통 옵션을 적용하고 싶은 경우에도 사용하면 좋다.

```java
@MappedSuperclass
public abstract class BaseEntity {

    @Column(name = "CREATED_DATE_TIME")
    private LocalDateTime createdAt;
    @Column(name = "MODIFIED_DATE_TIME")
    private LocalDateTime modifiedAt;
}
```

<br>

## 정리
- 상속관계와는 전혀 관계가 없다.
- 테이블과 별도 매핑되지 않고, 때문에 엔티티도 아니다.
    - 단순히 엔티티가 공통으로 사용하는 필드를 모으는 역할
- 부모 클래스를 상속 받는 자식 클래스에 매핑 정보만 제공한다.
- 부모 클래스를 이용한 조회, 검색이 불가능하다.
- 직접 생성해서 사용할 일이 없으므로 추상 클래스를 권장한다.
