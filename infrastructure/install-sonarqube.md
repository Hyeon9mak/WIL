# 소나큐브 설치하기
이 글에서는 현재 최신버전인 9.0버전이 아니라, LTS버전인 8.9버전을 사용했다!

![image](https://user-images.githubusercontent.com/37354145/128431747-91c1d05f-3c7d-4582-9c51-2a2c764a802e.png)

> 소나큐브는 프로그래밍 언어에서 버그, 코드 악취, 보안 취약점을 발견할 목적으로 정적 코드 분석으로 자동 리뷰를 수행하기 위한 지속적인 코드 품질 검사용 오픈 소스 플랫폼이다. 중복 코드, 코딩 표준, 유닛 테스트, 코드 커버리지, 코드 복잡도, 주석, 버그 및 보안 취약점의 보고서를 제공한다.

<br>

## Summary
> **백엔드**
> - 정적 분석 리포트를 공유한다.
> - CloudWatch logs 대시보드를 구성한다.

정적 분석 리포트를 공유한다는 요구사항이 등장했다. '정적 분석 리포트가 뭘까...' 궁금해하고 있을 때 나봄이 질문을 던져줬고, 구구께서 "소나큐브를 적용해보면 된다." 라는 답변을 주셨다. 그리하여 소나큐브 트러블 슈팅 대단원이 시작되었다.

<br>

## 소나큐브 동작구조
![image](https://user-images.githubusercontent.com/37354145/128432355-cdd5a0ab-cf09-4a16-a366-b4a47e2721a6.png)

소나큐브 공식문서에 따르면 소나큐브는 크게 **스캐너**, **서버**, **DB**로 구성되어 있다고 한다.
각각은 많은 리소스를 필요로 하기 때문에 별도의 호스트로 구성하길 추천하고 있으며, 그러는 와중에서도 서버와 DB간 
네트워크 거리는 최대한 가깝게 구성할 것을 요구하고 있다.

최초 나는 설명을 읽고 '소나큐브를 사용하려면 3개의 프로그램을 설치해야하네?' 라는 생각을 했다.
그러나 실습을 진행하면서 오해임을 알게 되었는데, 실질적으로 소나큐브 측에서 제공하는 설치 프로그램은 **서버**뿐이며, 
**스캐너**는 서버에서 제공한 token 값을 통해 빌드를 수행하는 '쉘 명령어'에 불과하고, **DB**는 그냥 일반적인 DBMS 프로그램을 요구하는 것이었다.

고로 **스캐너**는 이미 사용중이던 CI/CD 호스트에 포함시키면 되고, 소나큐브 서버와 DB만 구성하면 되었다. 
서버와 DB마저도 하나의 호스트에 올려버렸는데, 포츈이 로컬에 직접 소나큐브를 설치하고 테스트 해본 결과 우리 팀 프로젝트의 규모 정도에선 굉장히 적은 리소스를 필요로 했기 때문에 
(소나큐브 공식문서에서는 엔터프라이즈 급 프로젝트에 대해 리소스가 많이 필요함을 이야기한 듯 하다.) 
하나의 호스트에 도커를 통해 서버와 DB를 분리하여 구성했다.

![image](https://user-images.githubusercontent.com/37354145/128433708-858bf565-adb0-4758-a7b6-dd8222a42aab.png)

> 대략 위와 같은 구성이었던 것...

<br>

## 소나큐브 설치 - 서버, DB
[소나큐브 공식문서](https://docs.sonarqube.org/8.9/setup/install-server/)에 따르면 
소나큐브 서버를 설치하는 방법에는 압축(ZIP)파일을 해제하는 방법과 도커 이미지를 도커 허브로부터 받아와 
사용하는 방법이 있었다. 우리 팀의 경우 도커를 이용할 계획이었기 때문에 압축 파일을 해제하는 방법은 
간단한 로컬 테스트 정도로 지나쳤다.

아래는 소나큐브 공식문서에서 제공하는 `docker-compose.yml` 파일을 우리 팀 입맛대로 바꾼 내용이다.
주석을 통해 상세한 내용을 확인해보자.

```yml
# yml 문법 3.7버전을 사용하겠다.
version: "3.7"

# docker-compose.yml 파일을 통해 수행되길 원하는 작업들.
services:
  # sonarqube 변수명으로 수행될 작업.
  sonarqube:
    # 컨테이너 이름을 지정하고자 할 때 사용.
    # 사용하지 않을 시 도커가 임의의 이름을 지정한다.
    container_name: [컨테이너 이름]
    # 컨테이너에 올릴 이미지. 로컬 도커에 없을 경우 자동으로 도커 허브에서 pull.
    # 공식 문서에서는 sonarqube:community 로 입력되어 있다.
    # sonarqube:lts-community 로 바꿔서 lts 버전을 사용한다.
    image: sonarqube:lts-community
    # 하단의 db 변수명으로 수행될 작업과 의존관계를 맺는다.
    depends_on:
      - db
    # DB와 연결을 위한 설정.
    # 5432는 postgreSQL이 사용하는 포트번호.
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/[DB 테이블명]
      SONAR_JDBC_USERNAME: [DB 아이디]
      SONAR_JDBC_PASSWORD: [DB 패스워드]
    # 로컬 디렉토리와 컨테이너의 디렉토리를 매핑해서 데이터에 영속성을 부여한다.
    volumes:
      - ./sonarqube/data:/opt/sonarqube/data
      - ./sonarqube/extensions:/opt/sonarqube/extensions
      - ./sonarqube/logs:/opt/sonarqube/logs
    # 로컬 네트워크 요청 포트번호와 컨테이너가 사용하는 포트번호를 매핑한다.
    ports:
      - "9000:9000"
  
  # db 변수명으로 수행될 작업.
  db:
    # 컨테이너 이름을 지정하고자 할 때 사용.
    # 사용하지 않을 시 도커가 임의의 이름을 지정한다.
    container_name: [컨테이너 이름]
    # 컨테이너에 올릴 이미지. 로컬 도커에 없을 경우 자동으로 도커 허브에서 pull.
    image: postgres:12
    # DB 생성에 필요한 설정.
    environment:
      POSTGRES_USER: [DB 아이디]
      POSTGRES_PASSWORD: [DB 패스워드]
    # 로컬 디렉토리와 컨테이너의 디렉토리를 매핑해서 데이터에 영속성을 부여한다.
    volumes:
      - ./postgresql:/var/lib/postgresql
      - ./postgresql/data:/var/lib/postgresql/data
```
> 최초 9000포트 접근이 불가능했다. 보안그룹에 9000포트가 열려있지 않았기 때문이었다.8000:9000으로 매핑 후 사용해도 됐지만, 더 깔끔하게 매핑 하고 싶어 CU께 상황설명을 설명 드리니 포트를 개방해주셨다. 항상 감사합니다 CU! 🙇‍♂️

위와 같이 `docker-compose.yml` 파일을 작성 후 곧장 도커 컨테이너를 올리면 에러를 마주친다.

```
[1]: max virtual memory areas vm.max_map_count [65530] is too low,
increase to at least [262144]
```

`vm.max_map_count` 값이 최소 262144는 되어야 소나큐브 서버가 동작할 수 있다. 이를 위한 설정을 
조금 해주어야 한다.

```
# /etc/sysctl.conf 파일의 최하단
vm.max_map_count=262144
```

이 설정을 통해 `vm.max_map_count` 값이 영구적으로 증가하긴 하나, 시스템이 완전히 재부팅 된 후부터 적용된다. 
재부팅을 하지 않고 곧바로 적용되길 원할 경우 아래 1회성 명령을 입력해주면 된다.

```
$ sudo sysctl -w vm.max_map_count=262144
```

> `/etc/sysctl.conf` 파일에 설정을 추가하지 않고 `sysctl -w vm.max_map_count=262144` 명령만 사용할 경우 
재부팅 되면 설정이 날아간다.

설정 완료 후 도커에 올리면 잘 동작할 것이다. 

<br>

## 소나큐브 설치 - 스캐너
소나큐브 스캐너 준비를 위해선 우선 소나큐브 서버 웹 콘솔에 접근해야한다.
소나큐브 서버와 데이터베이스를 구동시킨 상태에서 웹 브라우저를 통해 9000번 포트로 접근해보자. 곧바로 로그인을 요구할 것이다.

![image](https://user-images.githubusercontent.com/37354145/128582331-6e84c2d6-d1af-4e57-b0a0-b72760970e4d.png)

기본 계정명/비밀번호는 `admin`/`admin`이다. 로그인에 성공하면 곧바로 비밀번호 변경을 요구하니, 비밀번호를 변경해준다.

![image](https://user-images.githubusercontent.com/37354145/128582644-7248a63d-ff3f-4b0a-aeef-673572402a2b.png)

화면 1시 방향의 `Add Project - Manually` 메뉴로 접근한다. (community 버전을 사용 중이기 때문에 manually 외엔 선택이 불가능하다.)

![image](https://user-images.githubusercontent.com/37354145/128582684-88787068-0ca0-4235-801f-c181956a8681.png)

프로젝트 생성을 시작하면 `Project key` 입력을 요구한다. 소나큐브의 `Project key`는 단순하게 **프로젝트를 식별하는 고유한 이름**이라고 
생각하면 되겠다. 

![image](https://user-images.githubusercontent.com/37354145/128582751-2815e5e1-92d3-430a-9de9-c36b0fdb572b.png)

그 후엔 token을 발급 받는다. token의 명칭을 정하고 `Generate` 버튼을 눌러 token을 발급 받으면 된다.

![image](https://user-images.githubusercontent.com/37354145/128582786-ba031852-c158-48c0-a367-3534cafb11d7.png)

발급된 token을 확인하고 `Continue` 버튼을 누른다.

![image](https://user-images.githubusercontent.com/37354145/128582905-c5124ec6-ef3a-4f21-baac-45901b65ade0.png)

빌드 옵션에 따라 선택을 진행한다. 우리 팀의 경우 Gradle을 사용 중이므로, `Gradle` 탭을 선택했다.
`Gradle`탭을 선택하면 위 그림과 같이 `build.gradle`에 추가할 의존성 코드와 
소나큐브 스캐너 동작을 위한 스크립트가 제공된다.

소나큐브 스캐너 동작을 위한 스크립트를 자세히 살펴보면 아래와 같다.

```yml
./gradlew sonarqube \ # build.gradle에 추가한 의존성을 통해 sonarqube 빌드 진행
  -Dsonar.projectKey=[키이이이이] \ # 앞서 설정한 projectKey
  -Dsonar.host.url=http://[유알엘엘엘엘엘]:9000 \ # 소나큐브 서버 URL
  -Dsonar.login=[토크으으으으으은] # 앞서 발급 받은 token
```

앞서 소나큐브 서버 웹 콘솔을 통해 진행한 일련의 과정들은 모두 이 스크립트를 만들어내기 위함이었으며,
소나큐브 스캐너는 고작 이 스크립트를 통해 동작을 수행하게 된다.

프로젝트의 `build.gradle` 파일에 의존성을 추가하고, CI/CD 툴에 스크립트를 추가하자.
우리 팀의 경우 Github actions를 사용중이기 때문에, workflow 파일에 스크립트를 추가했다.

```yml
name: Java CI with Gradle
...(생략)

jobs:
  sonarqube-build:
    runs-on: deploy

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'

    - name: gradlew 권한 변경
      working-directory: ./back/babble
      run: chmod +x gradlew

    - name: 소나큐브 빌드 진행
      working-directory: ./back/babble
      run: ./gradlew sonarqube -Dsonar.projectKey=babble-sonarqube -Dsonar.host.url=${{ secrets.SONARQUBE_SERVER_URL }} -Dsonar.login=${{ secrets.SONARQUBE_TOKEN }}

  build:
    runs-on: deploy

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 8
      uses: actions/setup-java@v2
      with:
        java-version: '8'
        distribution: 'adopt'

    - name: gradlew 권한 변경
      working-directory: ./back/babble
      run: chmod +x gradlew

    - name: 빌드진행
    ...(생략)
```

> URL과 token 값은 Github secrets를 통해 숨겼다.

눈 여겨 볼 점은 `jobs`를 `sonarqube-build`(소나큐브 빌드)와 `build`(스프링 부트 프로젝트 빌드)로 구분하여 2가지 작업이 병렬적으로 진행되게 만들었다는 점과 각각의 빌드가 사용하는 자바 버전을 다르게 설정했다는 것이다.
우리 팀 프로젝트의 경우 Java8 버전을 사용한다. 반면 소나큐브의 경우 Java11 버전을 이용해야 한다. 서로 다른 버전의 JVM을 이용하기 때문에 
작업을 나누어 진행하도록 설정했다.

설정이 완료되었다면 빌드를 진행해보자.

![image](https://user-images.githubusercontent.com/37354145/128584026-897f919d-d430-4821-8acf-70b1e3cbff31.png)

<br>

## projectKey, URL, token
소나큐브 서버 웹 콘솔을 통해 `projectKey`와 `login(token)`을 만들어내면서 'projectKey와 token을 조합해서 
소나큐브 프로젝트를 식별하는구나!' 하고 생각할 수 있다. 그러나 실제 식별에 사용되는 조합은 **URL과 token**이다. projectKey는 
그저 프로젝트를 구별시키는 이름일 뿐이다.

소나큐브 스캐너는 URL과 token을 조합하여 명령의 유효성을 판단한 다음, projectKey의 이름으로 프로젝트를 생성하여 소나큐브 서버로 
분석 결과를 전달한다. **이 때 우선권을 가지는 것은 소나큐브 서버가 아니라 소나큐브 스캐너다.** 표현이 모호한가? 예시를 보자.

```yml
./gradlew sonarqube \ 
  -Dsonar.projectKey=babble_AAA \ 
  -Dsonar.host.url=http://111.111.1.1:9000 \ 
  -Dsonar.login=abcd1234 

./gradlew sonarqube \ 
  -Dsonar.projectKey=babble_BBB \ 
  -Dsonar.host.url=http://111.111.1.1:9000 \ 
  -Dsonar.login=abcd1234
```

URL과 token이 완전히 일치하는, projectKey 값만 다른 2개의 빌드 스크립트다. `babble_AAA` 스크립트는 
소나큐브 서버 웹 콘솔을 통해 만들어진 스크립트이며(즉, 소나큐브 서버에서 존재를 알고 있음), `babble_BBB` 스크립트는
소나큐브 서버 웹 콘솔을 거치지 않고 곧장 작성한 스크립트다(소나큐브 서버에서 존재를 모르고 있음). 이 2가지 빌드 스크립트를 각각 동작시키면 어떤 결과가 일어날까?

우선 소나큐브 서버에서 존재를 인식하고 있던 `babble_AAA`는 이미 존재하는 프로젝트의 분석 결과가 업데이트 되는 형태를 가진다. 
그리고 소나큐브 서버에서 존재를 인식하지 못했던 `babble_BBB`는 소나큐브 서버에 프로젝트가 자동으로 등록되며 
분석 결과가 업데이트 되는 형태를 가진다. 즉, 소나큐브 서버는 전달 받은 분석결과들을 projectKey 이름으로 분리하고 정돈해서 보여줄 뿐 
프로젝트를 직접 생성하는 등의 우선권을 가지고 있지 않다. **우선권을 가지는 것은 소나큐브 서버가 아니라 스캐너다.**

<br>

## 후기
- 역시 공식문서가 정답이다. 공식 문서에서 제공하는 docker-compose 파일이 너무 잘 되어 있었다.
- 초반에 공식문서에 나온 성능 이야기를 보고 겁을 먹은 나머지 시간을 너무 낭비했다.
  - 일단 들이박아보자. 별거 아니다. 인프라는 특히 더 그런거 같다.

<br>

## References
- [SonarQube Documentation | SonarQube Docs](https://docs.sonarqube.org/8.9/)
- [Side effects when increasing vm.max_map_count](https://www.suse.com/support/kb/doc/?id=000016692)