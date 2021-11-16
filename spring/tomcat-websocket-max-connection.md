# 톰캣 최대 커넥션 개수 테스트

```yml
server:
  tomcat:
    max-connections: 8192 # default 8192
    threads:
      max: 1            # default 200
```

`threads max` 값을 1으로 두었을 경우 웹소켓 연결 자체는 진행되었으나, 연결 과정과 채팅에서 간헐적으로 병목현상이 발견됨. (채팅방 입장이 1초 지연, 채팅이 몰려서 전송된다던지)

```yml
server:
  tomcat:
    max-connections: 3 # default 8192
    threads:
      max: 200            # default 200
```

`threads max` 값이 웹소켓 연결에 별다른 영향을 끼치지않음을 알게 되었으니, `max-connections`값을 3으로 줄여서 테스트를 진행해보았다. 기대하는대로라면 3번째 유저까진 정상적으로 채팅방에 접속할 수 있고, 4번째 유저는 채팅방에 접속할 수 없어야 한다.

그러나 정상적으로 접속이 되었다. 왜지?

[아파치 톰캣9 웹소켓 관련 문서](http://tomcat.apache.org/tomcat-9.0-doc/web-socket-howto.html)를 
확인해보아도 마땅한 해결책을 강구하지 못했다.

같은 방식으로 테스트를 진행한 나봄은 4번째 유저가 채팅방에 접속할 수 없었다고 한다.
나는 어떤 부분에서 차이가 발생한걸까?
로드 어펜터 설정을 바꾸고 스레드를 체크해보아야겠다.
