# 우아한멀티모듈 by 우아한형제들 권용근님

> - 유튜브 영상 [https://youtu.be/nH382BcycHc](https://youtu.be/nH382BcycHc)
> - 블로그 포스트 [https://techblog.woowahan.com/2637/](https://techblog.woowahan.com/2637/)

## MSA? 멀티 모듈?
### 모놀리틱(Monolithic) 아키텍쳐
![image](https://user-images.githubusercontent.com/37354145/150033824-7033043a-2897-4a64-8457-f778b4e13d7a.png)

하나의 서비스에서 API, ADMIN, BATCH, WEB, DB 등이 관리되는 구조를 모놀리틱 아키텍쳐라 부른다.

![image](https://user-images.githubusercontent.com/37354145/150034040-762fb952-25c3-4fa1-81b4-8c1193101bdf.png)
![image](https://user-images.githubusercontent.com/37354145/150034056-bcbb0904-fa34-482d-bce0-7adfebae0400.png)

모놀리틱 아키텍쳐는 크게 단일 모듈 멀티 프로젝트와 멀티 모듈 단일 프로젝트로 나뉜다.

### MSA
![image](https://user-images.githubusercontent.com/37354145/150033896-d6317086-f649-485e-a7d4-09eaf432bb61.png)
![image](https://user-images.githubusercontent.com/37354145/150033909-d0944088-6b38-49db-a411-3e614ffe3ca9.png)

서비스가 내부적으로 받는 도메인 단위를 상세하게 분리하고 그 안에 격리되는 집군들(서비스)을 구상한 방식을 MSA라고 부른다.

### 멀티 모듈과 MSA가 함께 거론되는 이유
![image](https://user-images.githubusercontent.com/37354145/150034625-5a79810c-b153-4437-9c55-beb8faade073.png)
![image](https://user-images.githubusercontent.com/37354145/150034778-d122e083-a9f6-4422-bdee-4581f03a9e64.png)
![image](https://user-images.githubusercontent.com/37354145/150034810-43e89ba1-6fbc-45ef-9d4c-ef97350ebecd.png)

멀티모듈의 역할, 의존성 분리를 통해 시스템의 분리와 통합을 유연하게 만들어주는 
좋은 아키텍쳐를 만들 수 있다.
즉, 멀티 모듈을 활용해 역할과 의존성을 잘 분리할수록 
모놀리틱 <-> MSA간 전환이 용이하며, 용이해야지만 좋은 아키텍쳐라고 볼 수 있다.
때문에 멀티 모듈과 MSA가 함께 자주 거론된다.

<br>

## 멀티 모듈
### 단일 모듈 멀티 프로젝트
![image](https://user-images.githubusercontent.com/37354145/150032925-63690065-118c-404a-9d81-21c593d7f126.png)

- 각각의 프로젝트 단위
    - IDE를 쓴다면 3개의 IDE 화면을 띄워둔 상태로 개발을 진행하는 형태
- `Member` 라는 클래스가 공유(중복)되고 있다.
    - member-internal-api에서 `Member`가 수정되면 나머지 프로젝트에도 복붙
    - 잘못 옮길 경우 곧바로 장애로 이어짐
- "사람에게 굉장히 의존하는 형태구나" 하고 느낌

### 단일 모듈 멀티 프로젝트 + 내부 Maven 저장소
![image](https://user-images.githubusercontent.com/37354145/150033154-62a61d83-ff7e-4ae0-a484-6af4395e06bd.png)

- Nexus라는 사설 Maven Repository를 만들어서 각각의 프로젝트에서 공유하고 있는 DTO나 도메인 클래스들을 분리 후 프로젝트화 시켜서 Nexus에 업로드

![image](https://user-images.githubusercontent.com/37354145/150033318-902f10ba-9a1c-4590-a831-08c89c319db0.png)

- 이렇게 하니 확실히 시스템적으로는 일관성을 보장 받음
- 문제는 개발 사이클이 너무 번거로움
    - IDE를 4개 켜놓고 member-internal-api를 수정해서 Nexus에 배포를 하고
      member-domain에서는 Nexus에 배포된 내용을 다운로드하고...
    - 기능 하나를 개발하는데 프로젝트를 왔다갔다 하는 리소스가 너무 많이 들고 개발에 집중하기 어려움


### 멀티 모듈 단일 프로젝트
![image](https://user-images.githubusercontent.com/37354145/150033550-f1f4d4ae-3cc9-4b52-b00e-d3cc003f04b0.png)

- 프로젝트는 하나고, 그 안에 여러 개의 모듈을 설치 가능한 방법을 찾게 됨
- IDE도 1개만 사용하고 시스템적으로 보장되는 일관성, 빠른 개발 사이클을 확보할 수 있었음.
- 이후 멀티 모듈을 확립해 나가는 과정에서 겪은 문제점들이 생김.

<br>

## 실패한 멀티 모듈 프로젝트
![image](https://user-images.githubusercontent.com/37354145/150035143-f875b716-d948-46df-885b-a97caff50aec.png)
![image](https://user-images.githubusercontent.com/37354145/150035239-abab4c58-dee2-43ba-870e-226ce43cba35.png)
![image](https://user-images.githubusercontent.com/37354145/150035246-34b4c6c1-002c-41e8-aed5-5d0aee39da2a.png)

- 처음엔 "멀티 모듈을 통해 공통되는 코드를 제거할 수 있겠다!" 라는 단순한 접근
- 공통되는 로직을 분리해서 common(core)라는 모듈을 만듬

![image](https://user-images.githubusercontent.com/37354145/150035318-71fd875d-48fc-4567-8569-545ee2bbdf37.png)
![image](https://user-images.githubusercontent.com/37354145/150035336-7a3f6b3c-ef14-4744-b9b0-113bd16c2f0b.png)

- 꼭 external-api, batch, internal-api 모두에서 중복이 발생하지 않더라도
  중복이 발생하면 분리해서 common 모듈에 포함시킴
- '코드를 모듈화 한다 = 중복을 최소화한다' 라는 생각으로 
  common에 추가 코드를 몰아 넣음

![image](https://user-images.githubusercontent.com/37354145/150035486-3ae50e4c-a58f-42e2-87d5-ec4372004165.png)
![image](https://user-images.githubusercontent.com/37354145/150035565-51e515aa-1b1e-45c5-8c92-61f861df0146.png)

- common이 점차 커지면 common 안에서 비즈니스가 흐르기 시작함
- 원래는 다른 모듈에 플로우를 추가해야하지만, 어쩔 수 없이 
  common에 플로우를 추가하는 사태가 발생
- 결국 다른 애플리케이션은 굉장히 날씬하고, common만 굉장히 큰 프로젝트로 구성됨
- 이후 지옥이 펼쳐짐

### 스파게티 코드
![image](https://user-images.githubusercontent.com/37354145/150036655-d16b76a0-e3d8-433b-ad1b-10186e372f99.png)
![image](https://user-images.githubusercontent.com/37354145/150036751-f7be3f30-8bc9-4bcf-b4c6-73319d420ba8.png)
![image](https://user-images.githubusercontent.com/37354145/150036761-7f21fe6f-810c-4fc3-9690-10b6a7d2d9b5.png)
![image](https://user-images.githubusercontent.com/37354145/150036778-d2e0c436-2c0c-48d9-811a-ed47c370154c.png)

- 200~300 개가 넘는 클래스가 서로를 의존하는 스파게티 코드
- common을 분해하고 싶어도 분해가 불가능함
- 특정 기능이 사라졌음에도 의존도가 너무 높아서 관련 클래스를 제거할 수 없음
    - A를 제거하려고 시도하면 common 뿐만 아니라 서비스 전체가 오염됨

### 의존성 덩어리
![image](https://user-images.githubusercontent.com/37354145/150036889-a609ed5a-ad81-43aa-bead-4be789a9b9b8.png)
![image](https://user-images.githubusercontent.com/37354145/150037022-ed2ffa03-10f1-46d4-a522-bf2975cd516f.png)
![image](https://user-images.githubusercontent.com/37354145/150037295-b5a4871c-8bc5-4266-95b9-a37e0a81c1ba.png)

- common 모듈에서는 애플리케이션들이 사용할 수 있는 의존성을 모두 품게됨
- 사용하지 않는 것에 대해서도 의존을 하고 있어야함
- 또 아래의 스프링부트 설정 등에 의해서 의도하지 않은 설정 값이 트리거로 발동되어서
  예기치 못한 에러를 발생시키는 상황이 발생함

> **SpringBoot AutoConfiguration 동작에 대한 설명**
> - `@ConditionalOnBean`: 특정 Bean이 있을 때
> - `@ConditionalOnMissingBean`: 특정 Bean이 없을 때
> - `@ConditionalOnClass`: 특정 Class가 있을 때

![image](https://user-images.githubusercontent.com/37354145/150037408-02ce36f9-b39c-487a-8e7f-d673ee72d829.png)

- 예시 상황
    - common 모듈에 dynamodb 의존성을 추가해 둔 상태.
    - batch 모듈은 webflux로 구현되어 있음
    - webflux는 기본적으로 netty를 띄움
    - 당연히 nettydb가 뜰 것이라 생각했지만,  
      common 모듈의 daynamodb 의존성에 의해 jetty가 떠 있었음.

### 공통 설정
![image](https://user-images.githubusercontent.com/37354145/150037675-ed5f1890-8d68-43b9-b839-0c93f22ebf4a.png)
![image](https://user-images.githubusercontent.com/37354145/150037802-f47730e6-1f68-4d42-af6d-b31a3cb20941.png)
![image](https://user-images.githubusercontent.com/37354145/150037849-2921d6ab-0b67-49b2-bff9-fb773a7287df.png)

- 애플리케이션을 구성할 때 각 모듈이 DB 커넥션을 필요로 하는 수가 다르다.
- 때문에 하드하게 DB를 사용하는 모듈에는 커넥션을 많이 주고, 아닌 쪽엔 적게 주는 형태를 취하고 싶음
- 그러나 common 모듈에 설정을 몰아둔 경우 모든 모듈이 그 설정을 따라감
- DB는 커넥션을 더 이상 제공해줄 수 없어서 장애가 발생

<br>

##  무엇이 문제였을까?
- 모듈에 대한 정의가 모호했다.
- 모듈화란 무엇일까? 무엇을 중심으로 정의를 내려야할까?
    - 역할과 책임이 명확하다면 좋은 모듈이 될 수 있지 않을까?
    - 역할과 책임에 대한 범위는 어떻게 잡아야하지?

### 잘 되어있는 오픈소스 라이브러리를 염탐하자!
![image](https://user-images.githubusercontent.com/37354145/150038259-2706fe2b-d832-4f6f-8b2d-47cbfadab552.png)
![image](https://user-images.githubusercontent.com/37354145/150038337-d10cf640-36e2-4900-bfec-5a2b7c0e453a.png)
![image](https://user-images.githubusercontent.com/37354145/150038365-1ee741d2-2a73-4617-bd51-48347350d392.png)

- 뭔가 계층이 보인다.
- 계층 내부에는 모듈이 하나씩 구성이 될 것이고, 계층간 의존적 흐름이 구성된다.
- 그러나 개발자를 위한 라이브러리와 사용자를 위한 애플리케이션은 분명한 차이가 존재했다.
    - 라이브러리(프레임워크): 사용성, 기능 제공
    - 애플리케이션: 서비스 제공
- "내가 직접 계층, 역할을 나누어 봐야겠다!"

<br>

## 멀티 모듈 구성하기
### 주의사항
- "시스템"
    - 독립적으로 실행 가능한 애플리케이션을 "서비스"라고 부른다.
    - 1개 이상의 "서비스"와 "공유 인프라"가 모여 하나의 "시스템"을 구성한다.
- "애플리케이션 비즈니스", "도메인 비즈니스"
    > **주문요청 API**
    > 1. 요청값 검증 (애플리케이션)
    > 2. 주문 요청 (애플리케이션)
    >   - 주문 데이터 생성 (도메인)
    >   - 주문 데이터 검증 (도메인)
    >   - 주문 데이터 저장 (도메인)
    > 3. 주문 결과 처리 (애플리케이션)
    > 4. 응답 (애플리케이션)
    - 서비스에 대한 플로우나 흐름을 제어하면 애플리케이션 비즈니스
    - 도메인 단위에서 생성/변경/소멸의 라이프 사이클을 가지면 도메인 비즈니스
- "새로운 시각"
![image](https://user-images.githubusercontent.com/37354145/150038938-dfa013f9-c94e-450d-be51-d90bd3edcbcc.png)
![image](https://user-images.githubusercontent.com/37354145/150038959-a7c90b8b-af8d-444f-92e6-33d03617d6eb.png)

<br>

## 레이어 구상
![image](https://user-images.githubusercontent.com/37354145/150075433-6edda5de-1f62-4ad6-afd4-d7332a0c23ea.png)
![image](https://user-images.githubusercontent.com/37354145/150077511-07b1324e-d7e7-4732-8f60-c640d4c75067.png)

모듈은 역할이 분명해야 나올 수 있으므로 역할을 착실히 나누려고 노력했다.

![image](https://user-images.githubusercontent.com/37354145/150077643-90b3ee88-fe75-43d4-b1e4-8b7d510f8ff3.png)
![image](https://user-images.githubusercontent.com/37354145/150077820-f9f259c6-13af-4387-a516-f9af80248405.png)
![image](https://user-images.githubusercontent.com/37354145/150077872-1b7edc1a-ef1c-4797-b00a-fcfe831cdd3f.png)
![image](https://user-images.githubusercontent.com/37354145/150077920-d753fa95-6dbb-4d4a-a224-982644736193.png)

<br>

## 내부모듈 계층
- 시스템 안에서 의미를 갖는다.
- 애플리케이션, 도메인의 비즈니스를 모른다.
- 시스템과 주고 받는 스펙은 굉장히 명시적이다.
- 문제는 그 명시적인 스펙을 여러 애플리케이션에 걸쳐서 수행하게 된다면
    불필요한 중복과, 스펙을 동기화 시키기 위한 사람 의존적인 일을 반복해야 한다.
- 외부 시스템 변경에 대한 영향이 어디까지 퍼지는지 파악이 어렵다.
- 그래서 떠올린 방법 '외부통신을 담당하는 모듈을 만들어보자'

![image](https://user-images.githubusercontent.com/37354145/150234185-0b2cf740-6702-4a4a-957d-3e3512c787f3.png)
![image](https://user-images.githubusercontent.com/37354145/150234251-2c1d2b61-6cc7-4197-bc74-dfb0578751df.png)
![image](https://user-images.githubusercontent.com/37354145/150234422-cfa21d14-c09c-4392-a3af-608341f00beb.png)


- 환경별 시스템의 호스트와 헤더 관리
- 요청, 응답에 대한 스펙 관리
- 예외 처리 추상화 수준 통일
    - 기존에는 어디서는 예외처리를 해서 넘기고, 어디서는 하지 않고 넘기고
    - 예외를 어디서 처리했는지에 대한 파악이 어려움
    - 무조건 예외처리를 한다/안한다 명확한 룰을 만들어서 모듈 작성
- 내부 모듈로 분리
    - 역할이 분명함
    - 외부에 요청하는 비즈니스 로직만 품고 있음
    - 도메인과 거리가 멀다
    - 시스템에서 사용할 추상화 모듈을 정의하고 있다 (우리 시스템을 벗어나면 의미X)

![image](https://user-images.githubusercontent.com/37354145/150234706-e8fb6d4b-d790-432f-b0ac-5e397c43af8b.png)
![image](https://user-images.githubusercontent.com/37354145/150234764-0e649237-9a17-4f34-8053-7189dfc50a9e.png)

- build.gradle
    - 외부 요청만 처리하기 때문에 정말 필요한 의존성만 추가

![image](https://user-images.githubusercontent.com/37354145/150234886-71497a91-09b2-4be0-bf64-55c4a472af2c.png)
![image](https://user-images.githubusercontent.com/37354145/150234979-f7f813e6-a542-4157-a35b-ed43ca06a5b0.png)

- application-cuduck.yml, CoduckClientProperties를 통해서 빈주입 등 설정
- 요청에 대한 스펙과 응답에 대한 스펙을 CoduckClient에서 모두 정의
    - 클라이언트들의 공통 응답구조를 제네릭 형태(`ClientResponse`)로 만들어서 사용
    - 예외가 모듈 밖으로 절대 빠져나가지 않고 안에서 처리하도록 구성
    - 외부에서 이 모듈을 사용할 때는 예외를 핸들링하지 않고 '성공/실패'여부만 판단해서 비즈니스를 수행하도록

![image](https://user-images.githubusercontent.com/37354145/150235262-afbad496-729d-4bf9-aa61-fd59051c72c0.png)

- CoduckClientConfig
    - Coduck이라고 만들었던 클래스를 Bean으로 등록하는 과정
    - `@ConditionalOnMissingBean` 을 사용함
    - 사용하는 측에 변경의 여지를 주고 싶었다.
    - 나보다 상위 모듈에서 저 Bean을 정의하지 않았더라면 이 Bean을 띄운다
    - 하위에서 작성되는 대부분의 서비스나 클라이언트 모듈에서는 대부분이 상위에서 충분히 자기 마음껏 커스텀마이징을 할 수 있도록 여지를 줘야한다 생각함 (?)

### 내부 모듈을 구성해서 무엇을 얻었나?
![image](https://user-images.githubusercontent.com/37354145/150235456-f6aae2a3-2d21-4d09-878b-29215a619571.png)

- 스펙 변경에 대한 단일 변경 포인트
- 사용 추적이 용이하다

<br>

## 도메인 모듈 계층
![image](https://user-images.githubusercontent.com/37354145/150235704-5e6396a6-b2c6-42bd-a95a-a27c518f5c9c.png)
![image](https://user-images.githubusercontent.com/37354145/150235811-aff6eac2-4f8b-4b03-b1e9-8058547617f0.png)
![image](https://user-images.githubusercontent.com/37354145/150235842-159b6228-c907-4f2f-a531-2d7927915b5e.png)
![image](https://user-images.githubusercontent.com/37354145/150235922-d6c6203b-e5b6-42a4-ae07-15a93ffbd1aa.png)

- 도메인 모듈이니까 도메인은 당연히 알고 있다.
- 애플리케이션 비즈니스를 모른다.
- 하나의 모듈은 최대 하나의 인프라스트럭처에 대한 책임만 갖는다.
- 도메인 모듈을 조합한 더 큰 단위의 도메인 모듈이 있을 수 있다.
- 도메인 모듈을 구성할 떄는 객체지향 설계를 전제로 하고 있다.

![image](https://user-images.githubusercontent.com/37354145/150235969-564f2382-e110-496f-a712-5305a171fe46.png)
![image](https://user-images.githubusercontent.com/37354145/150236008-fa907e39-b935-4c67-8584-c7a6f4e1f8e5.png)

- '도메인이 인프라스트럭쳐를 알아야하나?'
    - '아름다운 도메인 구성을 위해선 도메인이 인프라를 몰라야 한다'
    - '언제든지 인프라 구현체를 바꿀 수 있어야하지 않을까?'
- 그러나 객체지향을 무조건 100% 다 지켜야하는가?를 고민해보아야 한다.
    - **순수성을 위해서 실용성을 포기하는건 어리석은 일이다.**
    - 실제로 인프라스트럭쳐에 대한 탈바꿈이 일어날 일이 거의 없다.
    - 인프라와 도메인이 서로 모르는 상태로 존재하면 도메인 계층에 수많은 인터페이스가 생성된다.
    - 그러나 그 인터페이스들은 있으나 마나한 존재들이 된다.
    - 어중간한 모듈화가 중복코드보다 나쁘다.

![image](https://user-images.githubusercontent.com/37354145/150236291-44f5434c-f4ae-4049-9a76-58c845c1497e.png)
![image](https://user-images.githubusercontent.com/37354145/150236371-b3716073-8ced-4153-a6c8-c132fcf11990.png)

- build.gradle
- spring.yml

### 다중 도메인
![image](https://user-images.githubusercontent.com/37354145/150236514-3db76d8a-d50e-41c7-b99f-264cc8afd94d.png)
![image](https://user-images.githubusercontent.com/37354145/150236550-ea07dc8b-e9ec-4a9e-b562-d19d712e199d.png)
![image](https://user-images.githubusercontent.com/37354145/150236576-cdbcd252-790e-472c-b101-69c5568d3163.png)
![image](https://user-images.githubusercontent.com/37354145/150236721-0f3504d1-883f-4f0a-8c91-bc0dc78a6e11.png)
![image](https://user-images.githubusercontent.com/37354145/150236732-41cd2c19-3382-4d70-96c9-4d21dd2ea7d1.png)

- 각각의 도메인을 모듈로 나눈다면, 인프라 구성만 하나 더 모듈로 나눈다
    - 각 도메인들 모듈들이 인프라 모듈을 사용하도록 구성
- 한 단계 더 분리해서 '시스템'으로 분리하는 것 또한, 모듈이 잘 나누어져있기 때문에 어렵지 않게 가능

### 다중 인프라스트럭처
![image](https://user-images.githubusercontent.com/37354145/150237050-4f6e0a4c-d83f-4ff1-a8f0-9ff9e11a7b0e.png)
![image](https://user-images.githubusercontent.com/37354145/150237090-91450245-cbd0-46a5-9fc0-f191bc7a6827.png)
![image](https://user-images.githubusercontent.com/37354145/150237104-fe869fe5-4fba-4834-b99c-c5b910b0c84d.png)
![image](https://user-images.githubusercontent.com/37354145/150237113-c7ad5d31-dc12-4b95-9fe3-8c70bc4b0063.png)
![image](https://user-images.githubusercontent.com/37354145/150237123-92315114-eb8e-42bd-8129-2a9de734c38d.png)

- DynamoDB는 프로비저닝을 사전에 대응하기 어렵다.
- 각 시스템의 원자성 데이터를 raw하게 들고 있었다.
- 그것을 Redis를 통해 한 계층 더 추상화를 진행한다
    - Key/Value로서 굉장히 쉽게 DB 조회가 가능해진다.
    - Redis에 없으면 Fallback으로 Dynamo에서 가져오고
    - Dynamo에서 가져올 땐 Redis에 Cache 하도록
- 이것을 모듈로 구성할 떈 어떻게 할까?
    - 간단하게 하나의 모듈에 Redis와 Dynamo를 모두 넣을 수 있겠다.
- 그러나 시스템에 따라 Redis를 꼭 필요로 하지 않을 수도 있다.
    - 그럴 때 Redis를 억지로 갖게 한다면 Common 모듈과 다를게 없다.
    - 그래서 생각한게 '인프라스트럭처를 분리하자!'

![image](https://user-images.githubusercontent.com/37354145/150237394-de7740ff-ad0f-49aa-ace2-c7dc1a0d6dd2.png)
![image](https://user-images.githubusercontent.com/37354145/150237493-ba0e7aef-2509-4385-b735-c7c9f274dae1.png)
![image](https://user-images.githubusercontent.com/37354145/150237527-e1035a48-f415-480f-b2f4-78fd64423607.png)
![image](https://user-images.githubusercontent.com/37354145/150237549-b6b2f33e-e09a-4416-a9f8-e437af8a77a5.png)
![image](https://user-images.githubusercontent.com/37354145/150237688-24f1ca4a-e95d-4fe5-8977-67a6c31a8924.png)
![image](https://user-images.githubusercontent.com/37354145/150237696-5f626c69-9b0e-4b67-b69d-4ac79468492e.png)

- 두 모듈간의 의존성을 완전히 분리하고 생각
- Fallback, Cache는 어떻게 동작시키는가?
    - Domain Service Module이 의존해서 동작하도록
- 이상적인 형태라면 각각에 인터페이스를 두어서 의존성을 살짝 분리하도록
    - 그렇게되면 인프라스트럭쳐 계층을 교체하기도 용이해진다.
    - 실용성이 떨어지는 부분이지만 말 그대로 '이상적'이니까

<br>

## 독립 모듈 계층
![image](https://user-images.githubusercontent.com/37354145/150239098-09bbe747-32ab-420f-9bd0-57c7286cdb76.png)
![image](https://user-images.githubusercontent.com/37354145/150239195-cd3244d5-729a-44e6-a54e-ea48c7ecbe69.png)
![image](https://user-images.githubusercontent.com/37354145/150239210-4bf2140c-bd6a-4047-94ca-bd5942600568.png)
![image](https://user-images.githubusercontent.com/37354145/150239220-a855cc98-b7d5-40cd-98b6-053352bd94f6.png)

- 시스템과 관련 없이 자체로서 독립적인 역할을 갖는다.
- 떼어내서 오픈 소스로 업로드 해도 될 정도의 스펙을 가진 모듈
- 'Dynamo를 사용할 때 어떻게 JPA처럼 사용할 수 있지?'를 해결하기 위해 만듦
    - 인프라스트럭쳐 연동 사용성 제공
    - Object Mapping 기능 지원
    - Reactor 사용성 제공

![image](https://user-images.githubusercontent.com/37354145/150239260-0e38447e-bcf9-4d08-8da4-9a8ca830e5c1.png)
![image](https://user-images.githubusercontent.com/37354145/150239271-f00a1fec-cf05-4e12-8cd4-594a62c6a55c.png)

- build.gradle
    - 시스템에 들어가는 어떤 것도 의존하지 않는다.
- DynmoReactiveRepository.java
    - JPA 같은 사용성을 위해서 구현체를 만듦
- 똑 떼어내서 오픈소스화를 해도 괜찮은 정도의 모듈이 완성된다.
    - 그러나 우리 시스템에는 필요한!

<br>

## 공통 모듈 계층
![image](https://user-images.githubusercontent.com/37354145/150239649-906f6812-1a69-45b4-99d6-cf0e93d65f08.png)

- 공통 모듈을 최대한 지양하고 싶었는데, 어쩔 수 없이 만들어짐
- Type, Util 등을 정의한다.
    - 공용으로 사용되는 Enum 등
- 대신에 공통 모듈에는 엄청 큰 제약을 걸어둔다.
    - 의존성을 절대 가져가지 않는다.
    - 자바를 순수하게 사용할 수 있는거 외에는 불가능하다.
    - (이것도 꼭 순수성을 지키기보단.. 실용성을 잘 따져보면서..)

![image](https://user-images.githubusercontent.com/37354145/150239863-657df932-9d31-4979-b574-a8ba058951a2.png)
![image](https://user-images.githubusercontent.com/37354145/150240003-3a71c20c-7d6c-45d2-8895-3ba6a79f0d86.png)
![image](https://user-images.githubusercontent.com/37354145/150240013-f5230fcc-e664-41a2-b855-3727cc7bda28.png)

- 전체 시스템에 물려있어야해서, 자기 자신을 제외하고 의존을 문다.
- 의존을 물어도 무거워지지 않을 수 있는 이유는 자바를 제외한 다른 의존성은 아예 없기 때문에

<br>

## 애플리케이션 모듈 계층
![image](https://user-images.githubusercontent.com/37354145/150240109-75f8a6e7-f257-46fb-8635-f4552a8e3d37.png)

- 모든 모듈들을 조합하는 계층

![image](https://user-images.githubusercontent.com/37354145/150240118-e34a6512-48b7-432f-8a73-d2a5628aad62.png)
![image](https://user-images.githubusercontent.com/37354145/150240187-e16bfee5-fd8e-412b-9938-de2e6ea0c798.png)
![image](https://user-images.githubusercontent.com/37354145/150240211-4030fcbb-50bb-49df-b0e4-9d6cc8c2cce6.png)
![image](https://user-images.githubusercontent.com/37354145/150240264-f387dc2e-1386-4984-b799-d6b5256db035.png)
![image](https://user-images.githubusercontent.com/37354145/150240276-cc295653-3186-451e-9fc7-d2787cce6706.png)
![image](https://user-images.githubusercontent.com/37354145/150240283-de62a1e4-92c5-40e8-a532-b1e7e1352e6b.png)

- 각 모듈들을 필요에 맞게 의존하면서 비즈니스 로직을 수행하게 된다.
- 최종 그림은 아래와 같아진다.

![image](https://user-images.githubusercontent.com/37354145/150240330-15661ad4-215d-4f09-899d-8f89f1a6a7bc.png)

<br>

## 효과
![image](https://user-images.githubusercontent.com/37354145/150240478-bf37959a-b899-4c80-8c50-9de96d739088.png)
![image](https://user-images.githubusercontent.com/37354145/150240511-ec3ea9ec-4bb5-42a7-862f-a7c1ce6e97be.png)
![image](https://user-images.githubusercontent.com/37354145/150240519-3b642edf-61f7-4d02-ae84-283c7f0b5448.png)

- 명확한 추상화 경계
    - '얘는 ~까지 알고 있어야한다' 등의 선을 그어서 추상화를 진행한 상태였고
    - 각 모듈에게 엄격한 정의를 내려서 모듈을 설계, 그 이후 역할과 책임의 선이 명확히 그려졌다.
    - 각 모듈이 갖는 책임과 역할이 명확해서 리팩토링, 기능 변경의 영향 범위 파악 용이
    - 경계가 명확해짐으로써 기능의 제공 정도를 예측 가능하여 스파게티 코드 발생 가능성 저하
    - 역할과 책임에 대한 애매함이 없어짐으로써 어떤 모듈에서 어느정도까지를 개발되야 할지 명확

![image](https://user-images.githubusercontent.com/37354145/150240741-33aba75f-631f-4c12-869e-768e668a2fbc.png)
![image](https://user-images.githubusercontent.com/37354145/150240750-a92c781a-7773-40cd-b074-31b2f0820dd0.png)

- 최소 의존성
    - 애플리케이션도 자기가 필요한 것만 구성을 하게 됨
    - 불필요한 의존성은 최소한으로 가져가게 된다.
    - 변경포인트를 쉽게 알 수 있게 된다.

<br>

## 멀티 모듈을 구성하는 꿀팁
### Application.java 패키지 위치 변경
![image](https://user-images.githubusercontent.com/37354145/150241022-e282f7d9-abd6-40f6-ac64-163d1276edfd.png)

- 보통 `baemin.com`의 프로젝트를 개설하게 된다면
    - com / baemin / projectName / moduleName ... 순서로 패키지를 구성하게 될 것이다
    - 이 때 Component Scan의 Base Package는 모듈 패키지가 선택된다.
    - 멀티모듈을 구성하게 되면 괴리감이 생길 수 있다.

![image](https://user-images.githubusercontent.com/37354145/150241278-e07d1238-90cc-4859-9bca-93b1cd5f648a.png)
![image](https://user-images.githubusercontent.com/37354145/150241288-ee646a3f-2da0-4d8d-be2f-5f3e3b72374a.png)

- 으악!
    - 주요 문제점은 `Application.java` 파일의 위치
    - Component Scan의 Base package가 여러 곳에서 잡히기 때문이다.
    - 해결책은 간단하게도 `Application.java`의 위치를 한 단계 높이는 것!

![image](https://user-images.githubusercontent.com/37354145/150241387-f67a3824-5180-482f-8134-95addf8a5209.png)
![image](https://user-images.githubusercontent.com/37354145/150241421-7a2d008c-16b7-4890-b1ad-23e701bde853.png)

- 특히나 이 방법은 의존성, Bean 관련 실수를 없애주기 때문에 매우 추천한다.

### 각 모듈의 Property
![image](https://user-images.githubusercontent.com/37354145/150241632-0db474e4-bd85-4e14-892e-9848f97b0ae1.png)
![image](https://user-images.githubusercontent.com/37354145/150241643-40e28cab-966d-4893-96fd-e0064d003c9a.png)
![image](https://user-images.githubusercontent.com/37354145/150241662-77a72b29-439c-4717-b949-ec0b44a44188.png)

- 스프링 Property 관리의 불편함
    - 모듈마다의 Spring property yml 파일 설정
    - 애플리케이션에서 include에 일일히 추가를 해주어야함

![image](https://user-images.githubusercontent.com/37354145/150241819-b4937ec9-cd2d-4720-b3f7-2a7446cbe5ce.png)

- spring-boot-custom-ymal-importer
    - 권용근님이 만드신 "독립 모듈" (ㅋㅋ)

### 의존성 숨기기
![image](https://user-images.githubusercontent.com/37354145/150242035-b7f5bdd6-c40b-4524-8806-328886fa144f.png)
![image](https://user-images.githubusercontent.com/37354145/150242043-83db114a-b652-4670-80e1-44f9ccddea18.png)

- 의존을 숨겨서 보호 받아야할 계층(모듈)을 숨길 수 있도록 Gradle이 지원한다.

<br>

## QnA
Q. 멀티 모듈을 분리하면 어쨌거나 공용으로 사용되는 모듈이 생기기 마련인데,
공용으로 사용되는 모듈을 변경 후 배포할 때 이펙트가 어디까지 미치는지 확인할 
수 있는 자동화를 해본 경험이 있는지? 혹은 방법이 있는지?

A. 없음. 모듈을 나누게 될 땐 엄청 많이 나누게 될거 같지만 실제로는 많이 안나눔.
프로젝트 내부에서 의존성들이 어떻게 흘러가고 있는지는 파악이 잘 될거임.
그래서 일일히 확인하고 있음. 특별히 자동화 해둔 경험은 없음.
