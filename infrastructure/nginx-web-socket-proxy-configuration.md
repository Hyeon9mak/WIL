# Nginx 웹 소켓 프록시 설정
Nginx는 버전 1.3부터 ​​WebSocket을 지원하며, WebSocket의 로드 밸런싱 을 수행 할 수 있다. 

HTTP에서 WebSocket으로 연결 전환시 HTTP의 Upgrade 및 Connection 헤더를 사용한다. 

WebSocket을 지원할 때 리버스 프록시 서버가 직면하는 몇 가지 문제가 있다. 
하나는 WebSocket이 hop-by-hop 프로토콜이므로 프록시 서버가 클라이언트의 Upgrade 요청을 가로챌 때 적절한 헤더를 포함하여 WAS 서버에 업그레이드 요청을 보내야 한다는 것이다. 
또한 HTTP의 단기 연결과 달리 WebSocket은 오래 지속되기 때문에, 리버스 프록시는 연결을 닫지 않고 열린 상태로 유지하는 것을 허용해야 한다.

Nginx는 클라이언트와 WAS 간 터널(소켓)을 설정할 수 있도록 WebSocket을 지원한다. NGINX가 클라이언트에서 WAS로 업그레이드 요청을 보내려면 Upgrade 및 Connection 헤더를 명시적으로 설정해야 한다.

```conf
# Web-socket 관련 설정들

# 1. HTTP/1.1 버전에서 지원하는 프로토콜 전환 메커니즘을 사용한다
proxy_http_version 1.1;

# 2. hop-by-hop 헤더를 사용한다
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
# 3. 받는 대상 서버(WAS)
proxy_set_header Host $host;
```

실제로 적용한 모습

기존 구성해둔 `nginx.conf` 파일의 location 설정 부분에 웹 소켓 연결 요청을 위한 
몇 가지 헤더값 설정만 진행해주면 된다.

```conf
# nginx.conf

events {}

http {
  upstream app {
    server [WAS 인스턴스 IP]:[WAS 프로그램 포트번호];
  }

  # Redirect all traffic to HTTPS
  # HTTP 요청을 HTTPS 로 강제전환
  server {
    listen 80;
    return 308 https://$host$request_uri;
  }

  # HTTPS 요청 처리
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
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
    }
  }
}
```

## References
- [Using NGINX as a WebSocket Proxy](https://www.nginx.com/blog/websocket-nginx/)
- [Nginx를 이용한 WebSocket reverse proxy - Joinc](https://www.joinc.co.kr/w/man/12/Nginx/wsproxy)
