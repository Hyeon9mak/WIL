# @ManyToOne을 사용할 때 @JoinColumn 생략

## summary
JPA를 학습하면서 `@ManyToOne` 어노테이션을 통해 일대다 연관관계를 매핑할 때 `@JoinColumn`을 항시 같이 사용했다.

```java
@Entity
public class School {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    protected School() {
    }

    public School(String name) {
        this(null, name);
    }

    public School(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    // getter, equals, hashCode
```
```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "school_id")
    private School school;

    protected Student() {
    }

    public Student(String name, School school) {
        this(null, name, school);
    }

    public Student(Long id, String name, School school) {
        this.id = id;
        this.name = name;
        this.school = school;
    }

    // getter, equals, hashCode
```

그런데 김영한님의 JPA 프로그래밍 책 5장 객체연관 매핑을 천천히 살펴보니 아래와 같은 구절이 보였다.

> `@JoinColumn`: 조인 컬럼은 외래 키를 매핑할 때 사용한다. name 속성에는 매핑할 외래 키 이름을 지정한다. 
회원과 팀 테이블은 TEAM_ID 외래 키로 연관관계를 맺으므로 이 값을 지정하면 된다. **이 어노테이션은 생략할 수 있다.**

당연히 `@JoinColumn` 어노테이션을 사용해야 Student 엔티티를 조회할 때 
School 엔티티와 조인을 해서 정보를 가져올거라 생각했는데, 생략하고도 이것이 가능하다는 이야기였다.
크루들에게 물어보니 조앤이 "생략은 가능하지만 조인을 하지 않고 Student-School 사이에 중간테이블을 만들지 않을까?" 라는 의견을 제시해주었다.
나 역시도 `@JoinColumn`이 엔티티간 조인을 이루어준다고 굳게 믿어 왔기 때문에 
조앤의 의견이 맞을 것 같았다.
그렇지만 JPA 책에 굳이 생략이 가능하다고 적혀있는 이유가 너무 궁금해서 직접 테스트를 해보기로 했다.

<br>

## 단건 조회 테스트
```java
@SpringBootTest
public class StudentSchoolTest {

    @Autowired
    private StudentRepository studentRepository;

    @Autowired
    private SchoolRepository schoolRepository;

    @Test
    void test() {
        // given
        School school = schoolRepository.save(new School("구구 학교"));
        Student student = studentRepository.save(new Student("현구막", school));

        // when
        Student foundStudent = studentRepository.findById(student.getId())
            .orElseThrow(IllegalArgumentException::new);

        // then
        assertThat(foundStudent).isEqualTo(student);
    }
}
```

준비된 테스트 코드는 위와 같다. 저장된 Student를 ID를 통해 조회했을 때 
School을 어떻게 가져오는지 하이버네이트가 작성하는 쿼리를 확인해볼 것이다.

### 테스트1 - @JoinColumn 사용
```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "school_id")
    private School school;
```
```
Hibernate: 
    select
        student0_.id as id1_3_0_,
        student0_.name as name2_3_0_,
        student0_.school_id as school_i3_3_0_,
        school1_.id as id1_1_1_,
        school1_.name as name2_1_1_ 
    from
        student student0_ 
    left outer join
        school school1_ 
            on student0_.school_id=school1_.id 
    where
        student0_.id=?
```

기대했던 대로 school을 조인해서 데이터를 모두 가져온다.

### 테스트2 - @JoinColumn 생략
```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    // @JoinColumn(name = "school_id")
    private School school;
```
```
Hibernate: 
    select
        student0_.id as id1_3_0_,
        student0_.name as name2_3_0_,
        student0_.school_id as school_i3_3_0_,
        school1_.id as id1_1_1_,
        school1_.name as name2_1_1_ 
    from
        student student0_ 
    left outer join
        school school1_ 
            on student0_.school_id=school1_.id 
    where
        student0_.id=?
```

중간 테이블을 만들거 같다는 예상과는 다르게, `@JoinColumn` 어노테이션을 사용할 때와 완전히 동일하게 동작하고 있다.
김영한님이 책에서 말씀하신대로 생략이 가능했다. 
`@JoinColumn`은 엔티티간 조인과는 관계없이  외래키 이름 지정을 위해서만 사용하는 것이었고, 
생략이 될 경우 알아서 `@ManyToOne`의 대상이 되는 엔티티의 `이름_id`를 대상으로 삼는 것 같았다.

### 테스트3 - @JoinColumn에 다른 이름 지정
```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "awesome_column_name")
    private School school;
```
```
Hibernate: 
    select
        student0_.id as id1_3_0_,
        student0_.name as name2_3_0_,
        student0_.awesome_column_name as awesome_3_3_0_,
        school1_.id as id1_1_1_,
        school1_.name as name2_1_1_ 
    from
        student student0_ 
    left outer join
        school school1_ 
            on student0_.awesome_column_name=school1_.id 
    where
        student0_.id=?
```

이번 테스트에서 `@JoinColumn`의 역할이 명확히 드러났다. `@JoinColumn` 어노테이션은 현재 엔티티(테이블) 기준으로 조인의 대상으로 사용할 컬럼의 이름을 지정하는 어노테이션이었다.

### 테스트4 - @ManyToOne에 Lazy Loading 적용
```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "awesome_column_name")
    private School school;
```
```
Hibernate: 
    select
        student0_.id as id1_3_0_,
        student0_.name as name2_3_0_,
        student0_.school_id as school_i3_3_0_ 
    from
        student student0_ 
    where
        student0_.id=?
```

`@ManyToOne(fetch = FetchType.LAZY)`로 Lazy Loading을 적용하자 단순 조회에서 School이 조인되지 않았다.
그렇다면 아무런 옵션을 적용하지 않은 `@ManyToOne`에서는 왜 조인이 됐을까? 
이유에 대해서는 킹갓제네럴 인비가 도움을 줘서 더 쉽게 찾을 수 있었는데, `@ManyToOne` 어노테이션의 기본 로딩 정책이 Eager Loading이라 그렇다. 

```java
@Target({METHOD, FIELD}) 
@Retention(RUNTIME)

public @interface ManyToOne {

    ...

    FetchType fetch() default EAGER;

    ...
}
```

<br>

## 다수건 조회 테스트
`@ManyToOne` 어노테이션의 기본 로딩 정책이 EAGER임을 알아냈다. 그렇다면 다수건을 조회할 때도 
School 엔티티를 한꺼번에 묶어서 조회해줄까?
이를 확인해보기 위해 테스트 코드를 수정했다.

```java
@SpringBootTest
public class StudentSchoolTest {

    @Autowired
    private StudentRepository studentRepository;

    @Autowired
    private SchoolRepository schoolRepository;

    @Test
    void test() {
        // given
        School school1 = schoolRepository.save(new School("구구 학교"));
        School school2 = schoolRepository.save(new School("또구 학교"));
        Student student1 = studentRepository.save(new Student("현구막", school1));
        Student student2 = studentRepository.save(new Student("또구막", school2));

        // when
        List<Student> foundStudents = studentRepository.findAll();

        // then
        assertThat(foundStudents).containsExactly(student1, student2);
    }
}
```

### 테스트1 - 기본정책 사용
```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "school_id")
    private School school;
```
```
Hibernate: 
    select
        student0_.id as id1_3_,
        student0_.name as name2_3_,
        student0_.school_id as school_i3_3_ 
    from
        student student0_
Hibernate: 
    select
        school0_.id as id1_1_0_,
        school0_.name as name2_1_0_ 
    from
        school school0_ 
    where
        school0_.id=?
Hibernate: 
    select
        school0_.id as id1_1_0_,
        school0_.name as name2_1_0_ 
    from
        school school0_ 
    where
        school0_.id=?
```

기본정책을 사용했을 땐 Student 하나마다 매번 School을 조회해왔다. 
즉, N+1 문제가 발생했다.

### 테스트2 - Eager Loading 사용
```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "school_id")
    private School school;
```
```
Hibernate: 
    select
        student0_.id as id1_3_,
        student0_.name as name2_3_,
        student0_.school_id as school_i3_3_ 
    from
        student student0_
Hibernate: 
    select
        school0_.id as id1_1_0_,
        school0_.name as name2_1_0_ 
    from
        school school0_ 
    where
        school0_.id=?
Hibernate: 
    select
        school0_.id as id1_1_0_,
        school0_.name as name2_1_0_ 
    from
        school school0_ 
    where
        school0_.id=?
```

여전히 N+1이 발생한다. `@ManyToOne`의 기본 정책이 Eager Loading이라고 했으니 당연한 결과겠다. Lazy Loading을 사용하면 당장은 N+1이 발생하지 않겠지만, School을 사용하면 곧바로 N+1이 발생할 것이다.

> 해결을 위해선 BatchSize를 조절하거나, JPQL에 fetch join 을 사용하자!!

<br>

## 결론
`@JoinColumn`은 컬럼 이름 매핑에 사용되는 어노테이션이지, 연관관계에는 아무런 영향이 없다.
사실 생략해도 된다.
그럼에도 사용하는 이유는 보다 더 명시적이기 때문이 아닐까?

이번 테스트 덕분에 어떤 역할을 하는 어노테이션인지도 알게 됐고,
`@ManyToOne` 단방향 연관관계에서 Many에 해당하는 엔티티도 N+1을 발생시킬 수 있음을 새로 알게 됐다.
단순히 `@OneToMany` 관계에서 One에 해당하는 엔티티가 Many를 참조할 때 N+1을 발생시킨다고 생각했는데, 
`List<?>`를 조작할 일이 생기는 `@ManyToOne` 단방향 연관관계에서도 N+1이 얼마든지 발생할 수 있음을 인지하고 성능 최적화를 시도해야겠다.

처음 시작할때 생각한 것보다 훨씬 더 유익하고 재밌는 테스트였다!

<br>

## References
- [킹갓제네럴마이티 인비](https://github.com/taehee-kim-dev)
- [킹갓제네럴마이티 욘](https://github.com/thisisyoungbin)
- [킹갓제네럴마이티 조앤](http://github.com/seovalue)
- [킹갓제네럴마이티 크로플](https://github.com/perenok)
