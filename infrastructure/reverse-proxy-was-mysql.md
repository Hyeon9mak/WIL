# EC2 인스턴스 3개로 ReverseProxy - WAS - MySQL 서버 만들기


## 1. docker + nginx를 통한 Reverse Proxy 관리
Reverse Proxy EC2 인스턴스 생성 및 접속 후 아래 명령을 입력한다.

```
$ sudo docker run -it --rm --name certbot \
  -v '/etc/letsencrypt:/etc/letsencrypt' \
  -v '/var/lib/letsencrypt:/var/lib/letsencrypt' \
  certbot/certbot certonly -d '[도메인명]' --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

`[도메인명]` 부분에 도메인 이름을 기입한다. 명령이 실행되면서 동의를 몇 가지 구하고, 

![img](https://techcourse-storage.s3.ap-northeast-2.amazonaws.com/ad712ce0a1b943b18cef2cb255c2baf5)

제공된 레코드 값을 DNS 사이트에서 지원하는 TXT(SPF) 레코드로 추가한다.

> **SPF(Sender Policy Framework)**  
> SPF는 DNS(Domain Name Service) 레코드의 한 유형으로, 스팸 발송자가 전자 메일 주소를 "스푸핑 (spoof)"하거나 도용하여 수백, 수천 또는 수백만 개의 전자 메일을 불법으로 보낼 수 있는 전자 메일 시스템의 허점을 방지하기 위해 2003년에 만들어졌다.  
> 
> SPF는 메일서버 정보를 사전에 DNS에 공개 등록함으로써 수신자로 하여금 이메일에 표시된 발송자 정보가 실제 메일서버의 정보와 일치하는지를 확인할 수 있도록 하는 메일 검증기술로 메일서버등록제 라고도 불리우며 국제표준인 RFC 7208에 정의되어 있다.
>
> 즉 내 도메인으로 스팸메일이 발송되지 않도록 방어한다.  
> 출처: http://www.codns.com/b/B05-189

그 후 현재 경로로 인증서를 복제해온다. 

> (아마도 원본 키가 잘못될 가능성을 염두해서 백업해놓는 용도인거 같다.)

```
$ sudo cp /etc/letsencrypt/live/[도메인주소]/fullchain.pem ./
$ sudo cp /etc/letsencrypt/live/[도메인주소]/privkey.pem ./
```

`Dockerfile` 파일을 아래와 같이 작성한다.

```
# Dockerfile

FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf 
COPY fullchain.pem /etc/letsencrypt/live/[도메인주소]/fullchain.pem
COPY privkey.pem /etc/letsencrypt/live/[도메인주소]/privkey.pem
```

`nginx.conf` 파일을 아래와 같이 작성한다.

```
# nginx.conf

events {}

http {       
  upstream app {
    server [WAS 인스턴스 public IP]:8080;
  }
  
  # Redirect all traffic to HTTPS
  server {
    listen 80;
    return 308 https://$host$request_uri;
  }

  server {
    listen 443 ssl;  
    ssl_certificate /etc/letsencrypt/live/[도메인주소]/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/[도메인주소]/privkey.pem;

    # Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # 통신과정에서 사용할 암호화 알고리즘
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    # Enable HSTS
    # client의 browser에게 http로 어떠한 것도 load 하지 말라고 규제합니다.
    # 이를 통해 http에서 https로 redirect 되는 request를 minimize 할 수 있습니다.
    add_header Strict-Transport-Security "max-age=31536000" always;

    # SSL sessions
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;      

    location / {
      proxy_pass http://app;    
    }
  }
}
```

> `return 301 https://$host$request_uri;`에서 `return 308 https://$host$request_uri;`로 반환 코드가 변경되었는데, 
> 301,302,303 Redirect HTTP Code는 POST, PUT 같은 요청 GET으로 바꿔서 Redirect 시킬 가능성이 있어서 `302->307`, `301->308`로 사용하는 것이 좋다고 한다.

도커 컨테이너를 다시 만들고 실행시킨다.

```
$ docker build -t nextstep/reverse-proxy:0.0.2 .
$ docker run -d -p 80:80 -p 443:443 --name proxy nextstep/reverse-proxy:0.0.2
```

<br>

## 2. 별도의 MySQL 인스턴스 관리

별도의 EC2 인스턴스 생성 및 접속 후 

`apt 업데이트` - `mysql-server 설치` - `mysql 접속계정 생성` - `mysql schema 생성` - `외부 접속용 계정 권한 부여` 

과정을 거친다.

```
$ sudo apt update
$ sudo apt install mysql-server

-- MySQL 최고 권한으로 접속 (기본설정 비밀번호 없음)
$ sudo mysql -u root -p

-- 외부 접속용 계정 생성
# create user [사용 계정명]@'[WAS 서버 public IP]' identified by '[비밀번호]';  

-- schema(database) 생성
# create database [DB 이름];

-- 외부 접속용 계정에 권한 부여
# grant all privileges on [DB 이름].* to [사용 계정명]@'[WAS 서버 public IP]';

```

아래 경로의 mysql 설정 파일을 관리자 권한으로 열기

```
$ sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

포트번호와 바인딩 주소 설정

```
# mysqld.cnf

-- 보안그룹에 허용되어있는 포트. 우테코 SG-DEFAULT 보안그룹 기준 8080과 80을 사용할 수 있음.
-- 기본 등록되어 있는 3306 포트는 SG-DEFAULT 보안그룹에 제공되어 있지 않아 사용불가.
port		= 8080
또는
port    = [보안그룹에 맞는 포트번호]

-- address를 0.0.0.0으로 열어둘 경우 접속을 위에서 설정한 접속용 계정을 통해서만 제어 가능
-- mysql 자체 설정의 접속 제어를 원한다면 WAS 서버 public IP 등록
bind-address		= 0.0.0.0
또는
port            = [WAS 서버 public IP]
```

mysql 재시작으로 설정 적용.

```
$ sudo service mysql restart
```

<br>

## 3. Web Application Server yml 파일 설정

WAS용 EC2 인스턴스에서 서버 프로젝트 파일을 `$ git clone` 해서 가져온 후, `src/main/resource/application-prod.yml` 파일을 아래와 같이 생성한다.

```
# application-prod.yml

spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://[Mysql EC2 인스턴스 public IP]:[Mysql config.d 설정 port 번호]/[database(schema) 이름]?serverTimezone=UTC&characterEncoding=UTF-8
    username: [mysql 외부 접속용 계정]
    password: [mysql 외부 접속용 계정 비밀번호]
handlebars:
  suffix: .html
  enabled: true
security:
  jwt:
    token:
      secret-key: my_secret_is_secret
      expire-length: 3600000
```

여기까지 끝나면 `Reverse Proxy` - `WAS` - `MySQL` 3개의 인스턴스가 연결된다.

빌드 후 실행시켜보자. 

> (별도의 설정을 진행하지 않았다면 MySQL에 테이블이 존재하지 않아 에러가 발생할 것이다.)

POSTMAN 으로 Reverse Proxy 등록한 도메인 네임에 대해 테스트를 진행하면 된다.

## References
- [(HTTP) 상태 코드 - 307 vs 308](https://perfectacle.github.io/2017/10/16/http-status-code-307-vs-308/)