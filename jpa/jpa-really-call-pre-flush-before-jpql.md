# JPA 는 JPQL 을 DB 로 던지기 전 정말로 flush 를 먼저 호출할까?

우리가 JpaRepository interface 를 통해 만드는 `findByName(...)` 과 같은 Query Method 는 
[JPA 가 제공하는 Query Creation 기능](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html#jpa.query-methods.query-creation)을 통해
자동으로 JPQL 을 생성한다. 

그리고 JPQL 은 영속성 컨텍스트를 거치지 않고 곧바로 DB 에 조회를 시도하기 때문에, 
JPA 는 JPQL 이 영속성 컨텍스트에 남아있는 정보를 확인하지 못하는 불상사를 방지하기 위해 
먼저 flush 를 호출하고 이후에 JPQL Query를 DB 로 던진다고 한다.

진짜일까? 결론은 그렇다.

<br>

## Query Creation by JpaQueryExecution

`org.springframework.data.jpa.repository.query.JpaQueryExecution` 은 
Method 이름을 기반으로 JPQL 을 생성하고 DB 에 던지는 역할을 담당한다.

`findById` 같이 `SimpleJpaRepository` 가 제공해주는 기본 Method 는 별도로 `JpaQueryExecution` 을 이용하지 않는다.

 `findByName` 과 같이 커스텀하게 만든 Query Method 는 `JpaQueryExecution` 을 이용해 JPQL 을 생성하여 던지려는 모습을 볼 수 있다.

![image](https://github.com/user-attachments/assets/102f5ba9-4003-4cb3-a0f6-c3844231754b)

JPQL 생성 이후에는 실제로 Query 를 DB 에게 던지기 위해 `HikariDbConnection` 얻는걸 볼 수 있다.

![image](https://github.com/user-attachments/assets/88749190-a46f-4cf2-a215-509b8c1f1a47)


<br>

## Pre Flush before JPQL in Transaction

그럼 실제로 Transaction 이 진행되는 동안 JPQL 을 위해 Flush 가 호출되는지 확인해보자.

테스트를 위해 준비한 코드는 아래와 같다.

```kotlin
@Entity
class Student(
    @Column(nullable = false, length = 10)
    val name: String,

    @ManyToOne(fetch = FetchType.LAZY)
    val classroom: Classroom,
) {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = INITIAL_ID
}
```
```kotlin
@Entity
class Classroom(
    @Column(nullable = false, length = 50)
    val name: String,
) {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = INITIAL_ID

    @OneToMany(
        mappedBy = "classroom",
        fetch = FetchType.LAZY,
        cascade = [CascadeType.PERSIST, CascadeType.REMOVE],
        orphanRemoval = true,
    )
    val students: MutableList<Student> = mutableListOf()

    fun enrollStudent(student: Student) {
        students.add(student)
    }
}
```
```kotlin
@Transactional
@Component
class StudentEnroller(
    private val classroomRepository: ClassroomRepository,
) {
    fun enrollStudent(studentName: String, classroomId: Long): ClassInfo {
        val classroom = classroomRepository.findByIdOrNull(classroomId)
            ?: throw IllegalArgumentException("Classroom not found with id $classroomId")
        
        val student = Student(name = studentName, classroom = classroom)
        classroom.enrollStudent(student = student)

        return ClassInfo.fromClassroom(classroom = classroom)
    }
}
```

학생을 나타내는 `Student` 와 강의실을 나타내는 `Classroom` Entity 는 서로 N:1 관계에 있다.
`StudentEnroller` 코드는 한 Transaction 내에서 `Classroom` 을 조회하고, `Student` 를 생성하여 `Classroom` 에 등록한다.

여기에 `SimpleJpaRepository` 를 이용하는 `classroomRepository.findById` method 와 Query Creation 을 이용하는
`classroomRepository.findByName` method 를 호출하는 코드를 추가해보자.

```kotlin
@Transactional
@Component
class StudentEnroller(
    private val classroomRepository: ClassroomRepository,
) {
    fun enrollStudent(studentName: String, classroomId: Long): ClassInfo {
        val classroom = classroomRepository.findByIdOrNull(classroomId)
            ?: throw IllegalArgumentException("Classroom not found with id $classroomId")
        
        val student = Student(name = studentName, classroom = classroom)
        classroom.enrollStudent(student = student)

        // 추가된 코드
        val foundClassroomById = classroomRepository.findByIdOrNull(id = classroom.id)
        val foundClassroomByName = classroomRepository.findByName(classroom.name)

        return ClassInfo.fromClassroom(classroom = classroom)
    }
}
```

그리고 강의실에 학생을 등록하는 일련의 과정을 디버깅 해보면?

`classroomRepository.findByIdOrNull` 가 호출된 시점에는 별도로 `HikariDbConnection` 을 얻지도, Flush 를 호출하지도 않는다.

`classroomRepository.findByName` 가 호출된 시점에는 `HikariDbConnection` 을 얻고, 황급하게 Flush 를 호출하는 것을 확인할 수 있다.

![image](https://github.com/user-attachments/assets/88749190-a46f-4cf2-a215-509b8c1f1a47)

![image](https://github.com/user-attachments/assets/56d21e88-24ea-44d8-b45c-507a0258f875)

(hibernate 가 구현한 `package org.hibernate.internal.SessionImpl.autoPreFlush` 가 호출된다.)

![image](https://github.com/user-attachments/assets/eea13d3c-99c7-416e-9422-af7568643ba4)

아직 트랜잭션이 종료되지 않았음에도 INSERT Query를 던지고, findByName 에 의한 조회까지 다시 하는 걸 확인할 수 있다.

<br>

## What happen in DB?

> Postgres 같은 DB 는 READ UNCOMMITTED 를 지원하지 않는다.
> 
> > The SQL standard defines one additional level, READ UNCOMMITTED. In PostgreSQL READ UNCOMMITTED is treated as READ COMMITTED.
> 
> 때문에 직접 눈으로 확인해보고 싶다면 READ UNCOMMITTED 를 지원하는 MySQL 등의 DB 를 이용하자.

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

위 명령어로 MySQL 의 Transaction Isolation Level 을 READ UNCOMMITTED 로 변경하고 테스트를 진행해보면,
실제로 `classroomRepository.findByName` 가 호출된 직후 DB 에 아직 Commit 되지 않은 학생 정보가 삽입된 것을 확인할 수 있다.

![image](https://github.com/user-attachments/assets/107fd18d-4ce5-4f5e-b07c-e26a0bb19c9c)
