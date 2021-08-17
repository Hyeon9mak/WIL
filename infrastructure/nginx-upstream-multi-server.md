# NGINX 다중 서버 upstream 설정

## summary
기존 Web-server (Reverse-proxy) 역할로 사용자와 WAS(Web Application Server)간 통신을 
이어주던 NGINX에서 소나큐브 서버도 통신을 지원하도록 설정하고 싶었다. 그림으로 표현하자면 아래와 같다.

![image](https://user-images.githubusercontent.com/37354145/128661608-1459f365-769b-4ec4-abbb-ee2acab7a66c.png)

<br>

## nginx.conf 설정
```
# nginx.conf

events {}

http {

  # -------------------- spring-boot WAS -------------------- 
  upstream app {
    server 192.168.1.20:8080;
  }

  # Redirect all traffic to HTTPS
  server {
    listen 80;
    server_name api.babble.gg;
    return 308 https://$host$request_uri;
  }

  server {
    listen 443 ssl;
    server_name api.babble.gg;

    ssl_certificate /etc/letsencrypt/live/api.babble.gg/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.babble.gg/privkey.pem;

    ...(생략)

    location / {
      proxy_pass http://app;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
    }
  }

  # -------------------- sonarqube server -------------------- 
  upstream sonarqube {
    server 192.168.1.15:9000;
  }

  server {
    listen 80;
    server_name sonarqube.babble.gg;
    return 308 https://$host$request_uri;
  }

  server {
    listen 443 ssl;
    server_name sonarqube.babble.gg;

    ssl_certificate /etc/letsencrypt/live/sonarqube.babble.gg/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sonarqube.babble.gg/privkey.pem;

    ...(생략)

    location / {
      proxy_pass http://sonarqube;
    }
  }
}
```

서버 별 설정은 서버의 특성에 맞게 지정되므로 크게 중요하지 않아 생략했고,
`nginx.conf` 파일 쪽에 **명령어 구성**을 어떻게 하느냐가 핵심이다.

### upstream
우선 `upstream` 변수를 설정해준다. `upstream` 변수는 `server` 설정에서 
NGINX가 받아들인 요청을 어떤 서버로 흘려보내 줄 것인지 결정할 때 사용된다.
보통 아래와 같이 IP와 PORT를 지정해주는 것으로 설정이 끝난다.

```
upstream exampleServer {
  server 192.168.0.1:1234;
}
```

### server - https redirect
`server` 명령어를 통해 http 요청을 https 로 redirecting 시키는 용도로 사용했다.
내부의 `listen` 명령어를 통해 NGINX가 받아들이는 PORT 번호(`80`)를 지정하고, 
`server_name` 명령어를 통해 도메인 명을 지정해주는 것으로 어떤 서버 쪽의 요청에 반응하는 것인지 설정한다.

```
server {
    listen 80;
    server_name example.server.com;
    return 308 https://$host$request_uri;
  }
```

`return` 명령어를 통해 상태코드와 `https://$host$request_uri`를 넘겨주는데, 
상태코드를 308로 할 것을 추천한다.

> 301,302,303 Redirect HTTP Code는 POST, PUT 같은 요청 GET으로 바꿔서 Redirect 시킬 가능성이 있어서 `302->307`, `301->308`로 사용하는 것이 좋다고 한다.

### server - configure
`server` 명령을 통해 서버 설정을 진행한다.
내부의 `listen` 통해 https(`443`) 요청을 받아들이게 지정하고,
`server_name` 명령어를 통해 도메인 명을 지정해주는 것으로 어떤 서버 쪽의 요청에 반응하는 것인지 설정한다.
여러가지 서버 설정 이후 `location` 설정에 `upstream` 변수로 등록한 주소를 기입하여 최종적으로 요청을 흘려보낸다.

<br>

## letsencrypt pem 키 설정
`nginx.conf` 파일 내용을 살펴봤다면 예상하다 싶이 letsencrypt 인증을 사용하고 있다.
그런데 기존 `Dockerfile`을 이용해 `nginx.conf`를 다룰 때 아래와 같이 `pem` 키들도 다루고 있었다.

```
# Dockerfile

FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf
COPY fullchain.pem /etc/letsencrypt/live/api.babble.gg/fullchain.pem
COPY privkey.pem /etc/letsencrypt/live/api.babble.gg/privkey.pem
```

소나큐브와 스프링부트 WAS서버 각각의 인증서를 발급받았으므로 도커 컨테이너 상의 각자의 경로(`/etc/letsencrypt/live/[도메인]/`)에 복제를 해야하는데, `fullchain.pem`과 `privkey.pem` 이름이 겹치게 된다.

> 원래 로컬상의 pem키도 `/etc/letsencrypt/live/[도메인]/` 경로에 존재하지만, `Dockerfile`에서 `/etc/letsencrypt/live/[도메인주소]/`까지 접근할 권한이 없기 때문에(=접근할 수 없기 때문에) `sudo cp` 명령어를 통해 미리 `Dockerfile`과 같은 경로로 pem 키들을 복제해둔 상태다.

해결방법은 간단하게 각각의 도메인 이름에 맞춰 pem 키 이름을 바꿔서 복제해두고, `Dockerfile`에서 COPY를 수행할 때만 다시 기존의 이름으로 되돌려주면 된다.

```
# Dockerfile

FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf
COPY api-fullchain.pem /etc/letsencrypt/live/api.babble.gg/fullchain.pem
COPY api-privkey.pem /etc/letsencrypt/live/api.babble.gg/privkey.pem
COPY sonarqube-fullchain.pem /etc/letsencrypt/live/sonarqube.babble.gg/fullchain.pem
COPY sonarqube-privkey.pem /etc/letsencrypt/live/sonarqube.babble.gg/privkey.pem
```

<br>

## References
- [하나의 Nginx로 여러 upstream 처리하기 :: 공부 일기](https://bepoz-study-diary.tistory.com/386)