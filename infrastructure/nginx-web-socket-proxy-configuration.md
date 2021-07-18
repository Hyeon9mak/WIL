# Nginx 웹 소켓 프록시 설정
기존 구성해둔 `nginx.conf` 파일의 location 설정 부분에 웹 소켓 연결 요청을 위한 
몇 가지 헤더값 설정만 진행해주면 된다.

```
# Web-socket 관련 설정들

# 1. HTTP/1.1 버전에서 지원하는 프로토콜 전환 메커니즘을 사용한다
proxy_http_version 1.1;

# 2. hop-by-hop 헤더를 사용한다
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
# 3. -
proxy_set_header Host $host;
```

실제로 적용한 모습

```
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