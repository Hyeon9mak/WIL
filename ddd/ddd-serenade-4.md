# 10월 4일 강의

## 도메인 주도 설계 아키텍처 

도메인 주도 설계 아키텍처라는건 사실 없다. 도메인 주도 설계에선 결국
Context 에 따라 관심사를 분리했을 뿐. 그 내부에서는 계층형 아키텍쳐 등
여러가지 아키텍처가 사용되고 있다.

다만, 도메인 주도 설계 아키텍처라 불리는 것을 구성하기 위해선
'어떤 아키텍처가 필요하다.' 정도는 이야기 할 수 있다.

<br>

## 계층형 아키텍처

### 비즈니스 로직 수행은 어느 곳에서 하는게 좋을까?

![image](https://www.petrikainulainen.net/wp-content/uploads/spring-web-app-architecture.png)

당연히 도메인 모델 영역이라 생각하겠지만, 미션을 진행하면서 application layer 에 대한 편집을 하고 싶지 않던가?
그런 욕망이 강할 수록, 도메인 모델 영역에서 비즈니스 로직을 수행하기보다,
application layer 에서 비즈니스 로직을 구현하는데 많이 익숙한 것이다.

### 네 개의 영역

![image](https://dzone.com/storage/temp/4277164-layered-architecture-overview.png)

- 표현 영역 또는 UI 영역은 사용자의 요청을 받아 응용 영역에 전달하고 응용 영역의 처리 결과를 다시 사용자에게 보여주는 역할을 한다.
- 응용 영역은 시스템이 사용자에게 제공해야 할 기능을 구현한다.
- 도메인 영역은 도메인 모델을 구현한다.
- 인프라스트럭처 영역은 구현 기술에 대한 것을 다룬다.

### 표현 영역

- 사용자가 시스템을 사용할 수 있는 (화면) 흐름을 제공하고 제어
- 사용자가의 요청을 알맞은 응용 서비스에 전달하고 결과를 사용자에게 제공한다.
- 사용자의 세션을 관리한다.

### 응용 서비스

- 사용자의 요청을 처리하기 위해 리포지터리로부터 도메인 객체를 구하고, 도메인 객체를 사용한다.
- 로직을 직접 수행하기보다는 도메인 모델에 로직 수행을 위임한다.
- 도메인 객체 간의 실행 흐름을 제어
- 트랜잭션 처리
- 도메인 영역에서 발생시킨 이벤트를 처리

```java
public Result doSomeFunc(SomeReq req) {
    // 1. 리포지터리에서 애그리거트를 구한다.
    SomeAgg agg = someAggRepository.findById(req.getId());
    checkNull(agg);
    
    // 2. 애그리거트의 도메인 기능을 실행한다.
    agg.doFunc(req.getValue());
    
    // 3. 결과를 리턴한다.
    return createSuccessResult(agg);
}
```

아래는 도메인 서비스일까, 응용 서비스일까?

```java
// 은행권에서 Transcation 이란 '거래'를 의미한다.
public class TransactionValidator {
    public boolean isValid(Money amount, Account from, Account to) {
        if (!from.getCurrency().equals(amount.getCurrency())) {
            return false;
        }
        if (!to.getCurrency().equals(amount.getCurrency())) {
            return false;
        }
        if (from.getBalance().isLessThan(amount)) {
            return false;
        }
        if (amount.isGreaterThan(someThreshold)) {
            return false;
        }
        return true;
    }
}
```

비즈니스 로직이자, 비즈니스 정책에 따라 변할 수 있는 내용으로 당연히 도메인 서비스다.

### 응용 서비스의 구현 - 메서드 파라미터와 값 리턴

- 응용 서비스에 데이터로 전달할 파라미터가 두 개 이상 존재하면 데이터 전달을 위한 별도 클래스를 사용하는 것이 편리하다.
- 응용 서비스는 표현 영역에서 필요한 데이터만 리턴하는 것이 기능 실행 로직의 응집도를 높이는 확실한 방법이다.

### 값 검증

- 값 검증은 표현 영역과 응용 서비스 두 곳에서 모두 수행할 수 있다.
- 원칙적으로 모든 값에 대한 검증은 응용 서비스에서 처리한다.
- 표현 영역에서 필수 값과 값의 형식을 검사하면 실질적으로 응용 서비스는 아이디 중복 여부와 같은 논리적 오류만 검사하면 된다.

> 도메인 계층에서 검증로직이 중요한 이유가 무엇일까? 검증로직 자체가 모델링의 결과물를 코드로 표현한 것이라 그렇다.
> 자바 개발자들은 `Validation` 이라는 용어를 사용하지만, 다른 언어에서는 '계약조건', '요구사항'으로 불린다.
> 즉, 아래 로직에서 검증 로직은 방어코드 수준이 아니라 당연히 지켜져야 하는 것이다.

VALUE OBJECT
```java
public class PhoneNumber {
    private final String phoneNumber;

    public PhoneNumber(String phoneNumber) {
        Objects.requireNonNull(phoneNumber, "phoneNumber must not be null"); // 
        var sb = new StringBuilder();
        char ch;
        for (int i = 0; i < phoneNumber.length(); ++i) {
            ch = phoneNumber.charAt(i);
            if (Character.isDigit(ch)) { // 
                sb.append(ch);
            } else if (!Character.isWhitespace(ch) && ch != '(' && ch != ')' && ch != '-' && ch != '.') { // 
                throw new IllegalArgument(phoneNumber + " is not valid");
            }
        }
        if (sb.length() == 0) { // 
            throw new IllegalArgumentException("phoneNumber must not be empty");
        }
        this.phoneNumber = sb.toString();
    }

    @Override
    public String toString() {
        return phoneNumber;
    }
}
```

JSR-303 Validation
```java
public class Customer {
    @NotNull 
    private CustomerNo customerNo;

    @NotBlank 
    @Size(max = 50) 
    private String name;

    @NotNull
    private PostalAddress address;
}
```

### 인터페이스는 어느 시점에 생성하는 것이 적절한가?

[안정된 의존관계 원칙과 안정된 추상화 원칙에 대하여 - 우아한형제들 기술 블로그](https://techblog.woowahan.com/2561/)

<br>

## DTO(Data Transfer Object)

### 응용 서비스의 출력

- 응용 프로그램 서비스를 설계 할 때 중요한 한 가지 결정은 사용할 데이터와 반환 할 데이터를 결정하는 것

1. 도메인 모델을 직접 사용하거나, - Domain Model Everywhere
2. 별도의 DTO(Data Transfer Object) 사용하기 - Pure Domain Model

### DTO(Data Transfer Object)

- 프로세스 간에 데이터를 전달하는 객체
- 구조체

> VALUE OBJECT는 DTO가 아니다. - Martin Fowler

DTO는 OOP 관점에서 Object 가 아니다. 객체로서 존중 받을 가치가 없다.
그저 단순한 구조체일 뿐이다. 그런데 Object 로 불리는 이유는? Java 에는 구조체라는 개념이 없기 때문에.
그래서 Object 라는 명칭이 붙었다.

### Domain Model Everywhere

- 도메인 모델을 모든 계층에서 사용
  - 이미 가지고 있는 클래스를 사용할 수 있다.
  - 도메인 모델과 DTO 간에 변환이 필요 없다.
- OSIV(Open Session In View) 사용
  - [Spring - Open Session In View - Kingbbode](https://kingbbode.tistory.com/27)
- Getter 메서드, Setter 메서드 사용
- DTO는 나쁘다!

VO는 불변이고 단순히 값을 의미하므로 DTO를 대체하여 사용할 수 있다.
백엔드 개발자 입장에서는 VO를 RequestDto, ResponseDto 대신 사용한다면
외부 클라이언트로부터 입력값을 변환할 필요 없이 바로 사용할 수 있다는 장점이 있다.
그러나 외부 클라이언트와 백엔드 개발자 모두 VO에 대한 이해가 필요하다는 단점도 있다.
결국 trade off. 그러나 충분히 시도해 볼법은 하다.

### Pure Domain Model

- 클라이언트와 도메인 모델 간의 분리
- 애플리케이션의 외부 요구 사항과 내부 요구 사항을 분리

내부 요구사항은? 우리의 도메인. 우리가 알고 있는 비즈니스 지식.

외부 요구사항은? 
백엔드 개발자라면? 모바일/프론트 엔드 개발자가 원하는 데이터 형태가 있다.
그런 요구사항들에 대해 맞추는 것.

외부 클라이언트들과 소통이 필요없다면 DTO를 만들 필요가 있을까?
만약 그렇다면 Domain Model Everywhere 가 정말 편했을 것이다.
그러나 외부의 요구사항에 맞는 (fit한) 데이터 형태를 만들기 위해서 DTO 를 사용하게 된다.

> 항상 DTO를 만드는 것은 실용적이지 않다.

<br>

## 의존 역전 원칙

의존 역전 원칙이란?
고수준 모듈과 저수준 모듈 관계에서 저수준 모듈의 변경에 따라 고수준 모듈이 영향을 받는다.
또는 고수준 모듈만 별도로 테스트 작성이 어렵다.

이럴 때 저수준 모듈이 고수준 모듈에 의존하도록 바꾼다고 해서 의존 역전 원칙이라고 부른다.

### 고수준 모듈

- 어떤 의미 있는 단일 기능을 제공하는 모듈
- e.g. 가격 할인 계산

### 저수준 모듈

- 고수준 모듈의 기능을 구현하기 위해 필요한 하위 기능의 실제 구현
  - 고객 정보를 구한다. = RDBMS에서 JPA로 구한다.
  - 룰을 이용해서 할인 금액을 구한다. = Drools로 룰을 적용한다.

### 고수준 모듈의 의존 문제

- 고수준 모듈이 저수준 모듈을 의존
- 저수준 모듈의 변경에 따라 고수준 모듈이 영향을 받는다.
- 고수준 모듈만 테스트하기 어렵다.
- 다른 구현 기술을 사용하려면 코드의 많은 부분을 고쳐야 한다.

### DIP
- 저수준 모듈이 고수준 모듈에 의존한다고 해서 이를 DIP(Dependency Inversion Principle, 의존 역전 원칙)라고 부른다.
- 인프라스트럭처 영역에 의존할 때 발생했던 구현 교체가 어렵다는 문제와 테스트가 어려운 문제를 해소할 수 있다.
- 고수준 모듈은 더 이상 저수준 모듈에 의존하지 않고 구현을 추상화한 인터페이스에 의존한다.

![image](https://user-images.githubusercontent.com/37354145/193806657-05a45ba9-850a-4adf-9475-0ced27f36d22.png)

### DIP 주의사항

- DIP를 적용할 때 하위 기능을 추상화한 인터페이스는 고수준 모듈 관점에서 도출한다. 
- 추상화한 인터페이스는 저수준 모듈이 아닌 고수준 모듈에 위치한다.

> 가령 특정 데이터를 조회할 때 `OOOORequest` 라는 네이밍이 아닌, `getOOOO` 와 같은, 비즈니스 레벨에 맞춘 네이밍을 갖추는 것.

### 육각형, 양파, 클린 아키텍처

육각형 아키텍처(hexagonal architecture), 양파 아키텍처(onion architecture) 및 클린 아키텍처(clean architecture)의 기본 규칙

![image](https://herbertograca.files.wordpress.com/2018/11/100-explicit-architecture-svg.png)

<br>

## 전략적 설계 - ANTICORRUPTION LAYER

### Anti-Corruption Layer (부패방지 규칙)

대부분의 개발자들은 깃 커밋 ID, 커밋 해쉬를 `commit hash` 라는 명칭을 사용한다.
그러나 정작 깃허브 개발자들은 이를 `SHA`라고 부른다.
만약 별도로 계층이 존재하지 않는다면, 깃허브 개발자들이 도메인 용어를 바꿀 때마다 개발자들의 서비스도 모두 영향을 받을 것이다.
외부 서비스로부터 우리의 도메인이 오염되는 것을 방지하기 위해서 사용하는 통역, 번역(오버스펙 내에서 필요한 내용만 추출)하는 
계층을 Anti-Corruption Layer (부패방지 규칙) 라고 부른다. 

![image](https://docs.microsoft.com/en-us/azure/architecture/patterns/_images/anti-corruption-layer.png)

<br>

## 제이슨의 꿀팁

> 컨텍스트 기준으로 패키지를 나누는 것은 쉽게 자를 수 있으나,
도메인 기준으로 패키지를 나누는 것은 굉장히 어렵다.
(도메인끼리 서로 의존하는 곳이 굉장히 많기 때문)
그럴 땐 초반에 레이어 단위로 패키지를 나누었다가, 컨텍스트가 명확히 나뉘었을 때 
그때가서 컨텍스트 단위로 패키지를 나누는 것이 유연하고 편리할 것이다.

<br>

## Aggregate Modeling

![image](https://user-images.githubusercontent.com/37354145/193812049-a5e3bd1f-62c6-4c0f-b745-b9274af53353.png)

NEXTSTEP 의 수강신청 규칙. 요구사항을 토대로 모델링을 한다면 어떻게 진행할까?
아마 아래와 같은 방식으로 엔티티를 구성하게 될 것이다.

![image](https://user-images.githubusercontent.com/37354145/193812299-bec47da8-dbb3-42a2-80e9-a1604a76fa7a.png)

이 때 서비스가 동작하기 시작하면 어떤 흐름을 타게 될까?

![image](https://user-images.githubusercontent.com/37354145/193812400-1f6a52da-dc77-4820-9daa-658a56c113f4.png)

강의의 모집 상태를 확인하고, 수강생 Repository 에 save를 진행할 것이다.

만약 정원은 30명인데 수강생은 60명이 되는 문제가 발생한다고 쳐보자.
무엇이 문제일까?

- 다중 사용자 환경에서 동시성 제어 실패
- 설계 모델과 구현 모델 간의 불일치
- 도메인 전문가와 개발자와의 멘탈 모델 불일치

기술적인 문제를 제외한다면, 모델링이 잘못되어(정원과 수강생에 대한 이해) 문제가 발생하게 된다.

### 애그리거트

- 시스템이 기대하는 책임을 수행하며 일관성을 유지하는 단위
- 일관성은 항상 참이어야하는 속성(불변성)을 유지함으로써 달성된다.
- 명령을 수행하기 위해 함께 조회하고 업데이트 해야하는 최소 단위.

### 강의 라는 애그리거트를 만들면 어떨까?

![image](https://user-images.githubusercontent.com/37354145/193813344-05d2aa3e-9e2d-4444-bf10-052e5aaac285.png)

- 강의 ID 를 전역 식별자(global identity) 로 사용한다. 외부에서 참조할 때 사용하는 ID.
- 강의는 `버전` 을 갖고 있다. 비관적 락 또는 낙관적 락을 사용해서 우선순위를 주고, 비즈니스 규칙을 보호하기 위한 수단으로 사용된다.

![image](https://user-images.githubusercontent.com/37354145/193813706-fda4a7a3-283b-4ec4-a1c3-862371f1f9fd.png)

- 수강생, 수강 대기자가 누군지를 식별하기 위한 `회원 ID`를 지역 식별자(local identity) 로 사용한다.
- 회원 ID로 만으로는 어떤 강의의 수강생인지 알 수 없다. 강의 ID와 함께 할 때 의미가 있다.

### 수정된 시나리오

![image](https://user-images.githubusercontent.com/37354145/193813974-7f77dd87-315c-4b8f-99b3-30e425e2b8bf.png)
![image](https://user-images.githubusercontent.com/37354145/193814048-fa84eff6-d3ee-4c16-b736-c1ddbeeff5ae.png)
![image](https://user-images.githubusercontent.com/37354145/193814072-15742757-75a6-4dc6-b6e9-d95811d0f502.png)

이 모든 시나리오가 전부 강의라는 애그리거트를 필요로 하게 된다.
이랬을 때 어떤 문제가 발생할까?
바로 `버전` 에 대한 트랜잭션 오류가 발생한다.

수강생이 버전 10을 사용할 때, 강의 정보 수정이 발생하면서 버전이 11으로 올라가면 수강생의 요청이 트랜잭션 오류로 실패하게 된다.

가장 중요한 비즈니스, 돈이 발생하는 비즈니스는 어떤 것일까? 바로 수강 신청이다.
강의 정보 수정, 수강생 정보 수정이 조금 실패하면 어떤가? 수강 신청을 통해서 돈을 버는데.
그래서 수강 신청이 가장 우선순위를 가지고 버전을 조작하도록 만들 수 있다.
이런게 바로 도메인 지식이다.

### 도메인 지식

- 여러 강사가 동시에 강의 정보를 수정하지 않는다.
- 수강생들은 동시에 수강 신청할 수 있다.
- 강의 정보 수정과 수강생 정보 수정으로 인해 수강 신청을 막아서는 안 된다.

개발자는 수강생 정보를 변경하려면 모든 수강생을 조회해야 한다는 고민이 생긴다.
그래서 수강생을 별도의 애그리거트로 분리하자는 결론을 얻게 된다.
만약 분리한다면... 기존의 불변식은 어떻게 보장할 것인가?

![image](https://user-images.githubusercontent.com/37354145/193814891-ad5f3483-b9c7-42ea-8077-4cd953e3365d.png)

`강의 수강생` 이라는, 관계를 나타내는 VO(혹은 Entity)를 구성하게 된다.
일관성을 보장하는 도구. 강의 수강생이 먼저 생성된 후 실제 수강생 애그리거트가 생성된다.
강의 수강생이 일관성을 유지해주기 때문에 수강생에 대한 정보 수정도 쉬울 것이다.

> 수강생의 이름, 이메일, 승인여부등은 수강생이 정보를 수정하거나 조회할 때 필요한 것들이고, 
> 강의 입장에서는 고유한 회원인지, 승인여부가 어떤지, 몇 명인지가 중요한거니까 VO로만 관리하고...

수강생 애그리거트 없이 강의 수강생만 생긴 시점에서는 불변식이 깨져있따. 어떻게 관리할 것인가?
배치, 혹은 이벤트처리.

### 큰 애그리거트와 작은 애그리거트

- 큰 애그리거트
  - 다중 사용자 환경에서 실행되면 정기적으로 트랜잭션 오류
  - 성능 및 확장성이 떨어진다.
- 작은 애그리거트
  - 통제 불능으로 성장할 수 있다.

### 결론

- 누구의 문제인지, 어떤 것이 문제인지, 왜 문제인지에 대한 질문을 던져 보자.
- 계층형 아키텍처의 계층을 나누는 것만큼 해결 영역을 나누는 것도 중요하다.
- 데이터베이스의 동시성 제어보다 애플리케이션의 동시성 제어가 더 한 눈에 들어온다.
- 도메인 전문가와 함께 모델을 검증하는 것이 코드를 작성하고 테스트하는 것보다 빠를 수 있다.

<br>

## QnA

Q. `DisplayName` 까지는 외부 인터페이스(비속어 검증)를 의존하더라도, 그(`Menu`~) 이상은 외부 인터페이스 의존이 퍼지지 않도록 하는거네용?  
A. 그렇다고 볼 수도 있으나, 더 중요한건 외부 인터페이스의 트랜잭션과 `Menu`가 생성되는 트랜잭션을 격리시킨다는 의미가 조금 더 크다.

Q. `Menu`의 금액과 `Product Price`의 합을 어떻게 검증할까요? `MenuProduct`는 `productId`만 갖고 있을 텐데...  
A. `MenuProduct`가 꼭 `productId`만 갖고 있어야 하나요? `Product`와 관계없는 VO로 갖고 있으면 안되나요? 일시적으로 불변식을 꺠뜨리고 추후에 맞추면 안되나요?
