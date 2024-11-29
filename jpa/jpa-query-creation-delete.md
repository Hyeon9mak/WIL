# JPA query creation delete 메서드는 flush 를 일으키지 않는다.

JPA query creation delete 메서드는 flush 를 일으키지 않는다.

```kotlin
interface StudentRepository : JpaRepository<Student, Long> {
    fun deleteByName(name: String): Long
}

@Trasactional
fun deleteStudent() {
    studentRepository.deleteByName("name")
    studentRepository.save(Student("name"))
}
```

위와 같은 코드를 작성했다고 가정해보자.
`deleteByName` 메서드는 JPA query creation 을 통해 JPQL 을 생성하고, JPQL 은 곧바로 DB 로 향할 것이라 생각된다.
JPA 는 JPQL 이 동작하기 전 flush 를 발생시켜준다. 과연 기대처럼 동작할까?

결론은 아니다.

트랜잭션 내부에서 delete query 메서드는 select 문으로 변경된다.
select 문으로 엔티티를 찾아온 후, delete by id 를 수행하기 때문에 맨 마지막 더티체킹에서 쿼리 수행을 준비한다.
즉, JPA 내부 쿼리 최적화(INSERT-UPDATE-DELETE 순)를 타기 때문에 실제로는 delete 가 더 나중에 진행된다.

참으로 그지 같은 프레임워크다.
