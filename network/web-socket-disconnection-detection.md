# Web Socket 연결 끊김 감지

## Summary
클라이언트A가 서버와 소켓 연결을 끊을 경우(Disconnect) 서버에 해당 소식을 전달하고, 소식을 접한 서버가 
나머지 클라이언트들에게 클라이언트A의 연결이 끊겼음을(퇴장했음을) 알리는 형태를 구성하고 싶었다.

사용자가 Disconnect 버튼을 직접 눌러서 서버와 소켓 연결을 끊을 경우 연결이 끊기기 직전 
서버 쪽으로 알림 요청을 먼저 보내는 방법이 채택 가능하겠으나, 사용자들은 브라우저의 탭을 닫는 등 
(개발자 기준)비정상적인 방법을 많이 이용한다.

그래서 우리는 Disconnect 버튼과 관계없이, 클라이언트와 연결이 끊기면 서버에서 이를 감지해내는 방법을 찾아야 했다.

<br>

## @EventListener + SessionDisconnectEvent
```java
@Configuration
public class SomeConfigClass {

    @EventListener
    public void onDisconnectEvent(SessionDisconnectEvent event) {
        ...
    }
}
```

`@EventListener` 어노테이션과 `SessionDisconnectEvent` 파라미터를 조합한 메서드를 준비하면 
웹 소켓 연결이 종료될 경우 `SessionDisconnectEvent` 파라미터에 연결종료 관련 정보가 담겨 핸들링이 가능했다. 수 차례 테스트를 반복해보면서 소켓 연결이 끊길 경우 어떤 값들이 넘어오는지 조사했다. 그 중 유일하게 `sessionId`가 어떤 클라이언트인지를 식별 할 수 있었다.

`sessionId`는 최초 서버와 클라이언트가 소켓으로 연결이 되는 시점에 발급되는 식별자다. 
`sessionId`를 통해 연결 종료시 클라이언트를 식별 할 수 있게 되었지만, 클라이언트의 데이터(데이터베이스에 저장 될)와 매핑되어 있진 않았기 때문에 연결 시점에 클라이언트 데이터와 `sessionId`를 함께 알아내지 못하면 말짱 도루묵이었다. 

<br>

## 최초 연결 때 - 클라이언트 데이터를 포함한 연결요청
앞서 `SessionDisconnectEvent`를 이용했기 떄문에, 반대로 `SessionConnectEvent`를 사용하면 될 것이라고 간단하게 생각했다. 웹소켓은 소켓 연결을 위해 최초 HTTP 프로토콜을 이용해 서버에 프로토콜 전환 요청을 보낸다. 이걸 `SessionConnectEvent`로 캐치해서 핸들링하고자 했다.

```javascript
stompClient = Stomp.over(socket);
stompClient.connect({}, function (frame) {
    frame.headers["clientData"] = "현구막은 대단해!"; 
}
```
```java
@Configuration
public class SomeConfigClass {

    @EventListener
    public void connectEvent(SessionConnectEvent event){
        ...
    }
}
```

`SessionDisconnectEvent`와 동일한 형태로 연결되는 순간을 잡아낼 수 있었으나, 말 그대로 
연결에 관련된 Session정보만 얻을 수 있었지, 클라이언트의 데이터를 함께 포함시킬 수 없었다.

javascript쪽 코드에서 `stompClient.connect`는 헤더에 클라이언트의 데이터를 포함시킨 후 연결이 진행되는 것이 아닌, 이미 연결이 진행된 후 추가적으로 헤더에 데이터를 기입하는 방식으로 동작했다.
헤더 외에도 여러가지 방법을 시도해보았으나, `SessionConnectEvent`로는 만족스러운 결과를 얻을 수 없었다.

<br>

## 최초 연결 때 - ChannelInterceptor를 써보자

```java
@Configuration
public class StompHandler implements ChannelInterceptor {

    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor accessor = StompHeaderAccessor.wrap(message);
        if (StompCommand.SUBSCRIBE == accessor.getCommand()) {
            ...
        }

        return message;
    }
}
```

고심을 하던 중 `ChannelInterceptor` 를 발견했다. 

> Spring Integration. 스프링 기반 어플리케이션 내에 메시징 기반 서비스를 제공하고 선언적 어댑터를 사용해 외부 시스템과의 통합을 쉽게 해주는 프레임워크다. 리모팅, 메시징, 스케줄링과 같이 스프링이 제공하는 기능들을 추상화하고 있다. 그것 중 하나가 `ChannelInterceptor`.

`ChannelInterceptor.preSend()` 메서드를 구현해주니 클라이언트에서 보낸 정보를 중간에 가로챌 수 있었다.

![image](https://user-images.githubusercontent.com/43930419/125774991-276ee14c-77e0-4d78-b5e1-49d28a9618c8.png)

현재 요청이 `CONNECT` 때 온 것인지, `SUBSCRIBE` 때 온 것인지 등등 
`SessionConnectEvent`를 사용하는 것보다 훨씬 많은 정보를 얻어낼 수 있었다.

![image](https://user-images.githubusercontent.com/43930419/125775258-34db6fac-ef67-44fa-acb7-fa53da9a62a1.png)

`SUBSCRIBE` 시점에는 구독주소에 대한 정보도 추가적으로 얻을 수 있었다! 그러나 
여전히 클라이언트의 데이터를 포함시키진 못했다.

<br>

## 최초 연결 말고, 그 직후에 매핑하기
최초 연결시 클라이언트 데이터를 포함시키는 것을 포기하고, 연결 직후 세션정보와 클라이언트 데이터를 1회 전송하는 방식으로 관점을 변경했다.

```js
stompClient = Stomp.over(socket);
stompClient.connect({}, function (frame) {

    subscription = stompClient.subscribe('/topic/', function (chat) {
        ...
    });

    const sessionId = // 무언가의 방법
    const clientData = $("#clientData").val();
    stompClient.send('/ws', {}, JSON.stringify({sessionId, clientData}));
});
```

일반적인 메시지를 서버에 송신하는 것처럼, 연결이 완료 된 직후 `send()`를 통해서 클라이언트 데이터와 세션 정보를 담아서 보낸다면 서버에서 2가지 정보를 동시에 받아 매핑이 가능했다. 매핑이 완료될 경우 
소켓 연결이 강제로 종료 되었을 때도 `sessionId`를 통해 클라이언트의 데이터를 찾아 후처리가 가능하다!

남은 문제는 '클라이언트 측에서 `sessionId`를 어떻게 찾아내는가?' 다.

<br>

## 클라이언트에서 발급된 sessionId가 무엇인지 확인하기
```java
var socket = new SockJS('/connection');
stompClient = Stomp.over(socket);

stompClient.connect({}, function(frame) {
        console.log(socket._transport.url); 
        // ws://localhost:8080/connection/039/{sessionId}/websocket
    });
```
stackoverflow 에서 위 코드와 같은 해법을 찾을 수 있었다. `socket._transport.url` 을 통해 
긴 주소를 받을 수 있었고, 거기서 `{sessionId}`에 해당하는 영역에 `sessionId`가 포함되어 있는 것이다. `sessionId` 파싱작업이 추가로 필요해졌지만, 이 외에 클라이언트 단에서 `sessionId`를 찾아낼 방법을 강구하기 어려웠다.

```js
stompClient = Stomp.over(socket);
stompClient.connect({}, function (frame) {

    subscription = stompClient.subscribe('/topic/', function (chat) {
        ...
    });

    const sessionId = 파싱_메서드(socket._transport.url);
    const clientData = $("#clientData").val();
    stompClient.send('/ws', {}, JSON.stringify({sessionId, clientData}));
});
```

결국 이와 같은 형태로 클라이언트 데이터와 `sessionId`를 동시에 보낼 수 있게 되었다!

<br>

## 결론

- `@EventListener + SessionDisconnectEvent`
- `ChannelInterceptor`
- `socket._transport.url`

위 기술들을 사용해서 아래와 같은 흐름이 그려진다.

### 연결
1. 클라이언트와 서버 간 연결을 진행한다. 
3. 연결에 성공하면, 클라이언트는 `session Id` 와 데이터를 서버에게 일반적인 메세지를 보내듯 **send()**로 전송한다.
4. 서버는 `sessionId`와 클라이언트 데이터를 매핑해서 DB에 저장한다.

### 연결 종료
1. 연결이 끊어질 경우 서버가 이를 감지하고, `sessionId`를 잡아낸다.
2. 서버는 `sessionId`를 통해 후처리를 진행한다.

<br>

## References
- [소켓통신할 경우 session 관리 문의 - OKKY](https://okky.kr/article/283384)
- [[Java] WebSocket의 Session 사용 방법(Broadcast)과 웹 채팅 소스 예제 - 명월 일지](https://nowonbun.tistory.com/286)
- [Spring Boot + STOMP + JWT Socket 인증하기 - velog](https://velog.io/@tlatldms/Spring-Boot-STOMP-JWT-Socket-%EC%9D%B8%EC%A6%9D%ED%95%98%EA%B8%B0#handshakeinterceptor-vs-channelinterceptor)
- [How to get session id on the client side? (WebSocket)](https://stackoverflow.com/questions/28009764/how-to-get-session-id-on-the-client-side-websocket)
- [Spring Integration 이란? - Rednics Blog](https://springsource.tistory.com/47)
