# Spring web-socket 기본 구성
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // 구독주소. 주소를 구독한 클라이언트는 모든 브로드캐스팅 메세지를 수신한다.
        registry.enableSimpleBroker("/topic"); 
        // 송신주소. 클라이언트는 송신주소를 통해 메세지를 서버로 전송한다.
        registry.setApplicationDestinationPrefixes("/ws");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/connection") // 웹 소켓 생성 및 연결 값
            .setAllowedOriginPatterns("*") // Cors 설정
            .withSockJS(); // 웹소켓을 지원하지 않는 브라우저도 연결 가능하도록
    }
}
```

위와 같은 설정을 준비한 후, 컨트롤러에서 메세지를 전달받고 되돌려주는 작업을 진행하면 된다.

```java
@Controller
public class ChatController {
    private final SimpMessagingTemplate template;
    private final ChatService chatService;

    public ChatController(SimpMessagingTemplate template, ChatService chatService) {
        this.template = template;
        this.chatService = chatService;
    }

    @MessageMapping("/rooms/{roomId}/chat")
    public void chat(@DestinationVariable Long roomId, MessageRequest request) {
        template.convertAndSend(String.format("/topic/rooms/%s/chat", roomId),
                chatService.sendChatMessage(request));
    }
}
```

추상화가 잘 되어 있는 `SimpMessagingTemplate` 덕분에 쉽게 메세지를 주고 받을 수 있다.

<br>