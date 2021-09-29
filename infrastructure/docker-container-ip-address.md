# 도커 컨테이너 IP 주소 트러블 슈팅

## 첫 번째 이슈 - Summary
babble 프로젝트를 진행하면서, 상용 서버에서 릴리즈 테스트를 진행하지 말고 
별도의 테스팅 서버를 만들어서 릴리즈 테스트를 진행하고자 했다.

상용 서버와 온전히 같은 환경을 갖추기 위해 EC2 인스턴스를 추가로 3~4개 개설하면서 
테스팅 서버를 개설할까 고민해봤지만, 도커를 사용해서 각 계층을 독립시키면 
EC2 인스턴스 1개만으로도 충분히 테스트가 가능할 것 같았다. (추후 인스턴스를 제거하기도 편하고)

![image](https://user-images.githubusercontent.com/37354145/126854249-a664eed8-3eb2-42a5-a551-0d0149449637.png)

결국 위 그림과 같은 형태로 도커 위에 3개의 컨테이너를 올렸다. 
web-server는 nginx로 구성하여 외부(public IP)로 부터 전달되는 요청을 
바로 옆 WAS 컨테이너로 전달한다. 요청을 전달 받은 WAS는 로직을 수행하고, 
영속성이 부여된 데이터가 필요하다면 바로 옆 Database 컨테이너를 찌른다.

이 때 컨테이너끼리 통신 시킬 방법이 필요했는데, 상용 서버와 최대한 비슷한 환경 구성을 위해 
도커에서 제공하는 방법을 사용하지 않고 IP로만 요청을 주고 받아야 했다. 
그러기 위해서 1차적으로 알아야 할 것은 도커 컨테이너의 IP 주소다.

```
$ sudo docker inspect [컨테이너 ID] | grep IPAddress
```
```
"SecondaryIPAddresses": null,
"IPAddress": "172.17.0.4",
    "IPAddress": "172.17.0.4",
```

inspect 명령을 통해 컨테이너 내부에서 사용되는 IP 주소를 알아낼 수 있다.

> 도커 컨테이너의 기본 IP 주소 범위는 `172.17.0.*` 이다. 
> 결국 가장 마지막 번호를 알아내기 위함이다.

이렇게 알아낸 IP주소로 `Web-server - WAS - DB` 컨테이너를 서로 연결했고, 성공적으로 테스트가 가능했다.

<br>

## 첫 번째 이슈 - 원인 파악
그러나 며칠 후 CI/CD를 통해 WAS가 재배포 되면서 문제가 발생했다. 
WAS가 빌드 된 후 컨테이너로 도커에 오르자마자 종료되는 것이었다. 처음에는 단순히 WAS 빌드가 잘못 되었다고 판단했으나, WAS를 로컬에 띄워보니 아무런 문제가 없었다.

결국 한참을 헤매고 나서야 원인을 찾을 수 있었는데, 바로 데이터베이스의 유저계정을 생성할 때 할당했던 IP주소가 문제였다.

```sql
CREATE USER [유저 이름]@'[IP주소]' IDENTIFIED BY '[비밀번호]';
```

데이터베이스를 구축할 때 보안을 위해 `[IP주소]`에 정확하게 WAS 컨테이너의 IP주소를 기입했는데, WAS가 재빌드 된 후 기존 컨테이너를 버리고 새로운 컨테이너에 다시 올라오면서 IP주소가 변경된 것이다.

![image](https://user-images.githubusercontent.com/37354145/126854744-bc7e1ec0-ea57-4242-a4ba-dfd1df81587a.png)

그러거나 말거나 데이터베이스는 기존 IP주소만 허용하고 있으니, WAS가 실행되는 단계에서 에러를 
발생시키고, 컨테이너가 바로 다운 되었던 것이다.

<br>

## 첫 번째 이슈 - 해결 방법
현재는 데이터베이스에서 모든 IP 주소에서 들어오는 요청을 허용하는 것으로 해결하였으나, 
도커 컨테이너를 올릴 때 원하는 IP를 직접 기입하는 방법으로도 해결 할 수 있다.

```
# docker run 명령에 함께할 수 있는 옵션들

--add-host : 컨테이너 내부의 /etc/hosts에 호스트 이름과 IP 주소를 설정해준다. (없으면 임의의 ip adress 할당)

--dns : DNS 서버의 IP 주소를 설정해준다.

--expose : 포트 번호 할당

--mac-address : 컨테이너 MAC 주소 설정

--net : 컨테이너 네트워크 설정 (bridge, none, container:<name or d> , host)

-h, --hostname : 컨테이너 호스트 이름 설정

-P, --publish-all : 임의의 포트를 컨테이너에 할당

-p [호스트포트]:[컨테이너포트] : 호스트와 컨테이너 포트를 매핑시켜준다.

--link : 다른 컨테이너에서 접근시 이름을 설정
```

<br>

## 두 번째 이슈 - summary
어느 날 Babble 팀 프로젝트 테스트용 EC2 인스턴스의 WAS 프로그램이 비정상적으로 종료되어 있음을 확인했다. 
종료된 시점은 Babble 팀 프로젝트에서 Spring 프로필 파일(yml)을 변경한 후였는데, 
기존 사용중이던 `application-local.yml` 프로필에선 같은 서브넷 내에 있는 DB 도커 컨테이너를 조회하고 있었고, 
새로이 바뀐 `application-local.yml` 프로필에선 H2 인메모리 DB를 조회하고 있었다.

이 때문에 새롭게 추가한 `application-dev` 프로필을 사용하도록 설정을 교체했는데, 
테스트용 EC2 인스턴스 내부에서 DB 도커 컨테이너를 조작할 경우 DB 도커 컨테이너의 IP가 변경될 가능성이 다분했다.
도커 컨테이너의 IP가 변경될 때마다 `application-dev.yml` 파일에서 IP를 변경하는 것은 공수가 크므로,
`application-dev.yml` 파일이 아닌 테스트용 EC2 인스턴스 내부에서 스크립트 파일로 데이터베이스 IP를 관리하고자 했다.

```yml
# application-dev.yml

spring:
  datasource:
#   url: jdbc:mariadb://{IP}:3306/babble
    driver-class-name: org.mariadb.jdbc.Driver
    username: babble
```
```
# EC2 내부 Dockerfile

FROM openjdk:8
COPY babble-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java", "-jar", "-Dspring.profiles.active=dev","-Dspring.datasource.url=jdbc:mariadb://{DB 컨테이너 IP}:3306/babble" ...(기타설정)]
```

EC2 내부 `Dockerfile`에 DB 컨테이너 IP를 기입하기 위해 DB 컨테이너 IP를 조회했다.

![image](https://user-images.githubusercontent.com/37354145/135214449-bf4952e5-a123-4fa4-8e37-4eeb52e04576.png)

`docker inspect {DB 컨테이너} | grep IPAddress` 명령을 통해 IP를 조회했을 때 `172.21.0.2` IP를 확인할 수 있었다. '도커 컨테이너 IP 서브넷은 `172.17.0.*` 가 아니었나? 뭔가 달라졌나...' 하고 대수롭지 않게 넘기고 `172.21.0.2` IP를 `Dockerfile`에 등록했다. 그러나 계속해서 Spring 애플리케이션 서버 빌드가 실패했다.

<br>

## 두 번째 이슈 - 해결 방법

이어서 [루트가 바톤을 넘겨받아 트러블 슈팅을 진행](https://iodized-capri-aa0.notion.site/Docker-network-dbc4610f6f33444f81d66046309b6d5a)했다. 
도커를 사용하면 `docker0` 라는 네트워크 인터페이스가 생성되고, 도커에서는 이 네트워크 인터페이스에 `bridge`라는 네트워크 드라이버를 이용해서 접근한다.

각각의 컨테이너는 여러 개의 네트워크(서브넷)에 속할 수 있고, 각 서브넷에 맞는 IP를 따로 할당 받는다. 이 때 우리는 `bridge` 네트워크의 IP 주소를 확인했어야했는데, `db_default`라는 네트워크의 IP주소를 확인하고 있었다.

```
"brdige": {
    ...(생략)
    "IPAddress": "172.17.0.2",
    ...(생략)
},
"db_default": {
    ...(생략)
    "IpAddress": "172.21.0.2",
    ...(생략)
}
```

<br> 

## 두 번째 이슈 - 느낀 점
얼마전에 스프링 프로필(yml)파일을 리팩토링하면서 재미본 부분이 많다보니 생각이 거기에만 갇혀있었던거 같다. 도커 네트워크를 얄팍하게 알고 있어서 '도커 IP 범위는 `172.17.0.*` 아닌가? 조금 다를수도 있나보다' 하고 넘어가버렸다. 얄팍하게 아는게 이렇게 위험하구나 느낀다.

모의면접 때 CU께서 "도커 네트워크를 공부해보세요." 하셨었는데, 그 때 도커 네트워크를 공부했다면 조금 더 수월하게 해결하지 않았을까 아쉬움이 생긴다.

<br>

## 두 번째 이슈 - 남는 의문

루트 덕분에 문제를 해결하고 "`docker inspect {DB 컨테이너} | grep IPAddress` 명령은 단편적인 정보만 제공하기 때문에 `docker inspect {DB 컨테이너}`로 전체를 확인해야하는구나." 라고 결론을 지었다.

그런데 테스트용 EC2 인스턴스에 다시 접속해서 `docker inspect {DB 컨테이너} | grep IPAddress` 명령을 사용하니 이전에 사용할 때와 달리 더 많은 정보를 포함하고 있었다.

![image](https://user-images.githubusercontent.com/37354145/135214460-5fbb3d99-0610-4cee-ad66-055fe1b15f1a.png)

분명히 `bridge` 네트워크의 IP 주소를 포함하지 않고 있었는데, 이번에는 포함되어 있었다.
루트가 별도로 설정을 만진게 있나 물었으나, 루트는 테스트를 위해서 커스텀 네트워크(서브넷)를 생성한거 뿐이라고 말했다. 

즉, 커스텀 네트워크를 생성하기 전엔 `bridge` 네트워크의 IP 주소가 함께 보이지 않았으나, 커스텀 네트워크를 생성하고 나니 `bridge` 네트워크와 커스텀 네트워크의 IP 주소가 함께 포함되어 출력되고 있었다...

원인 파악을 위해선 도커 네트워크에 대한 공부가 더 필요할 것 같다.

<br>

## References
- [docker run 과 docker 네트워크 설정, 컨테이너 라이프사이클](https://www.leafcats.com/191)
