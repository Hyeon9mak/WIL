# Reactor Netty Metrics 고카디널리티로 인한 메모리 릭

## 발생 현상

힙 덤프 분석 결과 Micrometer `StatsdMeterRegistry` 내부에 비정상적 메모리 점유가 확인되었다.

- `preFilterIdToMeterMap`: 799,129개 엔트리 (~40MB)
- `meterMap`: 2M capacity 테이블 (64MB)
- `MicrometerHttpClientMetricsRecorder` 캐시: 84,884개 엔트리 (~37MB)

서비스가 운영될수록 메모리 사용량이 선형적으로 증가하는 전형적인 메모리 누수 패턴이었다.

## 원인

### 고카디널리티 URI 태그

Reactor Netty의 `HttpClient.metrics(true, uriTagValueFunction)`과 `HttpServer.metrics(true, uriTagValueFunction)`에서 URI를 그대로 메트릭 태그로 사용하고 있었다.

```kotlin
// 클라이언트 (ClientConfig.kt)
HttpClient.create()
    .metrics(true, Function.identity()) // URI 정규화 없음

// 서버 (ServerConfig.kt)
httpServer.metrics(true) { uri -> uri } // URI 정규화 없음
```

API 경로에 UUID path variable이 포함되어 있어서, 요청마다 고유한 URI가 메트릭 태그로 기록되었다.

```
/threads/550e8400-e29b-41d4-a716-446655440000/runs/stream
/threads/6ba7b810-9dad-11d1-80b4-00c04fd430c8/runs/stream
/threads/f47ac10b-58cc-4372-a567-0e02b2c3d479/runs/stream
...
```

Micrometer는 태그 조합마다 별도의 `Meter` 인스턴스를 생성하므로, 유저 요청이 누적될수록 Meter 수가 무한히 증가했다.

### 발생 지점

**클라이언트 사이드 (outbound 호출)**

WebClient로 외부 API를 호출할 때 문자열 보간으로 UUID를 URL에 직접 삽입하고 있었다.

| 호출 패턴                                             |
| ------------------------------------------------- |
| `"$baseUrl/threads/$threadId/runs/stream"`        |
| `"$baseUrl/threads/$threadId/runs/$runId/stream"` |
| `"$baseUrl/threads/$threadId/runs/$runId"`        |
| `"$baseUrl/threads/$threadId/runs/$runId/cancel"` |

`Function.identity()`가 URI를 그대로 통과시키므로, 확장된 URI가 메트릭 태그로 기록되었다.

**서버 사이드 (inbound 요청)**

`ServerConfig`에서 `{ uri -> uri }`로 identity 함수를 사용하고 있었다. 서버로 들어오는 요청의 URI에도 UUID path variable이 포함되어 동일한 고카디널리티 문제가 발생했다.

## 해결

### Reactor Netty의 `uriTagValueFunction`에서 UUID 정규화

`HttpClient.metrics(true, uriTagValueFunction)`과 `HttpServer.metrics(true, uriTagValueFunction)`의 두 번째 파라미터는 Reactor Netty가 공식 제공하는 URI 태그 정규화 확장 포인트다. 이 함수는 확장된 URI path를 받아서 릭 태그로 사용할 문자열을 반환한다.

UUID를 `{id}`로 치환하는 정규화 함수를 적용했다.

```kotlin
// ClientConfig
HttpClient.create()
    .metrics(true) { uri -> uri.replace(UUID_PATTERN, "{id}") }

// ServerConfig
httpServer.metrics(true) { uri -> uri.replace(UUID_PATTERN, "{id}") }

// UUID 정규식
private val UUID_PATTERN = Regex("[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}")
```

정규화 결과:

```
/threads/550e8400-e29b-41d4-a716-446655440000/runs/stream
→ /threads/{id}/runs/stream

/threads/550e8400-.../runs/f47ac10b-.../cancel
→ /threads/{id}/runs/{id}/cancel
```

새 엔드포인트가 추가되더라도 UUID 형식이면 자동으로 정규화되므로 Config 수정이 필요 없다.

## 주의사항

### 1. WebClient `.uri(template, vars)` 방식은 Reactor Netty 메트릭을 해결하지 못한다

Spring WebClient의 URI 템플릿 방식(`.uri("/threads/{threadId}/runs/stream", threadId)`)을 사용하면, Spring이 `URI_TEMPLATE_ATTRIBUTE`에 템플릿을 저장한다. 그러나 이 값은 **Spring Observation 레이어(`http.client.requests` 메트릭)에서만 사용**되며, Reactor Netty에는 확장된 URI만 전달된다.

```
WebClient.uri("/threads/{threadId}/runs/stream", threadId)
  ↓ URI_TEMPLATE_ATTRIBUTE = "/threads/{threadId}/runs/stream"  ← Spring Observation용
  ↓ 변수 확장 → concrete URI 생성
  ↓ ReactorClientHttpConnector → Reactor Netty에 확장된 URI 전달
  ↓ Reactor Netty metrics: uriTagValue.apply("/threads/abc-123/runs/stream")  ← 확장된 URI
```

따라서 `reactor.netty.http.client.*` 메트릭의 카디널리티를 제어하려면 반드시 `uriTagValueFunction`을 사용해야 한다.

참고: [spring-framework#30027](https://github.com/spring-projects/spring-framework/issues/30027), [spring-framework#29885](https://github.com/spring-projects/spring-framework/issues/29885)

### 2. `max-uri-tags`는 Reactor Netty 레벨 메트릭에 적용되지 않는다

`management.metrics.web.server.max-uri-tags` 속성은 Spring Observation 기반 `http.server.requests` 메트릭에만 적용된다. `reactor.netty.http.client.*`이나 `reactor.netty.http.server.*` 메트릭에는 효과가 없다.

### 3. Spring Boot WebFlux 와 Reactor Netty 메트릭은 별개 레이어다.

Spring Boot WebFlux 자동설정은 `http.server.requests` 메트릭을 `@RequestMapping` 패턴 기반으로 자동 정규화한다. `ServerConfig`에서 `httpServer.metrics(true)`를 호출하면 이와 별개로 `reactor.netty.http.server.*` 메트릭이 추가된다. 이 두 레이어는 독립적으로 동작하며, 각각 별도로 카디널리티를 관리해야 한다.

| 레이어                | 메트릭 접두사                                                       | URI 정규화 방식                                      |
| ------------------ | ------------------------------------------------------------- | ----------------------------------------------- |
| Spring Observation | `http.server.requests` / `http.client.requests`               | `@RequestMapping` 패턴 / `URI_TEMPLATE_ATTRIBUTE` |
| Reactor Netty      | `reactor.netty.http.server.*` / `reactor.netty.http.client.*` | `uriTagValueFunction` 파라미터                      |

### 4. 새로운 WebClient 빈을 추가할 때

새로운 `HttpClient.create().metrics(true, ...)` 설정을 만들 때 `Function.identity()`를 사용하지 말 것. 반드시 URI 정규화 함수를 적용해야 한다. 동적 path variable(UUID, 숫자 ID 등)이 포함된 URL을 호출하면 동일한 메모리 릭이 재발한다.
