---
title: Spring Http Client는 RestTemplate 대신 WebClient
author: kjb4494
date: 2024-01-06 00:10:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend, webclient]
---

## WebClient란?

오늘날 웹 애플리케이션 개발에서 HTTP 클라이언트는 필수적인 역할을 담당합니다. 특히, 마이크로서비스 아키텍처(MSA)가 주류가 되면서, 이러한 클라이언트는 서비스 간의 통신에서 핵심적인 역할을 하게 되었습니다. 마이크로서비스는 작고 독립적인 서비스들로 구성되어 있으며, 이 서비스들은 서로 네트워크를 통해 통신합니다. 이 과정에서 HTTP 클라이언트는 서비스 간의 데이터 교환, API 호출, 외부 서비스와의 통합 등 다양한 목적으로 활용됩니다. 이런 맥락에서 Spring Framework에서는 전통적인 `RestTemplate`을 넘어서, `WebClient`가 새로운 표준으로 자리잡고 있습니다. `WebClient`는 비동기, 논블로킹 방식으로 작동하며, 리액티브 프로그래밍을 지원하여 현대적인 웹 애플리케이션 개발에 적합한 성능과 유연성을 제공합니다.

## RestTemplate vs WebClient: 동기 vs 비동기

### RestTemplate의 한계

![사진](/assets/img/posts/2024-01-06/RestTemplate.png)

`RestTemplate`은 Spring에서 오랫동안 사용되어 온 HTTP 클라이언트로, 동기적이고 블로킹 방식으로 작동합니다. 이는 HTTP 요청을 보내고 서버의 응답을 기다리는 동안 현재 스레드가 차단된다는 것을 의미합니다. 이러한 방식은 간단하고 직관적이며, 작은 규모의 애플리케이션에서는 충분히 효과적일 수 있습니다. 그러나 마이크로서비스 아키텍처와 같이 복잡하고 대규모 트래픽을 처리해야 하는 환경에서는 몇 가지 문제점이 드러납니다:

1. **성능 저하**: 서버의 응답을 기다리는 동안 스레드가 차단되기 때문에, 리소스 사용이 비효율적이며, 처리량이 제한됩니다.
2. **스케일링 문제**: 높은 트래픽 상황에서 더 많은 스레드와 리소스가 필요하며, 이는 서버의 부담을 증가시킵니다.
3. **응답성 저하**: 다수의 요청 처리 시 응답 시간이 길어지고, 사용자 경험이 저하될 수 있습니다.

### WebClient의 강점

![사진](/assets/img/posts/2024-01-06/WebClient.png)

반면에 `WebClient`는 비동기, 논블로킹 방식을 채택하며, 리액티브 프로그래밍 모델을 지원합니다. 이러한 특성은 `RestTemplate`의 한계를 극복하고, 현대적인 웹 애플리케이션 개발의 요구사항을 충족시킵니다:

1. **비동기적 처리**: `WebClient`는 요청을 보낸 후 응답을 기다리는 동안 다른 작업을 수행할 수 있습니다. 이는 I/O 작업에 의한 스레드 차단을 방지하고, 시스템의 처리량을 극대화합니다.
2. **논블로킹 I/O**: 논블로킹 I/O 모델을 사용하여 리소스를 효율적으로 활용합니다. 이는 더 적은 스레드로 더 많은 요청을 처리할 수 있게 해주며, 서버의 부하를 줄여줍니다.
3. **리액티브 프로그래밍**: 데이터 스트림과 이벤트 기반의 프로그래밍을 통해, 더 효율적인 데이터 처리와 더 빠른 응답 시간을 달성할 수 있습니다.

## WebClient 사용 예시

### 의존성 추가

`WebClient`를 사용하기 위해서는 의존성을 추가해줘야합니다.

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-webflux'
}
```

### 간단한 예제 코드

아래는 `WebClient`를 사용하여 REST API에 GET 요청을 보내고 응답을 처리하는 간단한 예제 코드입니다.

```java
WebClient webClient = WebClient.create("http://example.com");
Mono<String> response = webClient.get()
    .uri("/resource")
    .retrieve()
    .bodyToMono(String.class);
```

- **WebClient.create()**: WebClient 인스턴스를 초기화합니다.
- **get()**: GET 요청을 생성합니다.
- **uri()**: 요청할 리소스의 URI를 설정합니다.
- **retrieve()**: 응답을 검색합니다.
- **bodyToMono(String.class)**: 응답 본문을 String 타입으로 변환합니다.

```java
response.subscribe(content -> {
    // 여기에서 응답을 처리합니다.
});
```

이 코드의 `subscribe` 부분은 `WebClient`가 서버로부터 응답을 받았을 때 실행되는 콜백 함수를 정의합니다. 리액티브 프로그래밍에서 `Mono<T>`는 0개 또는 1개의 결과를 비동기적으로 반환하는 객체입니다. `subscribe` 메소드는 `Mono<T>`가 실제 데이터를 받을 준비가 되었을 때 호출되며, 이를 통해 결과 데이터를 처리할 수 있습니다.

이 코드의 `content` 변수는 서버에서 반환된 HTTP 응답 본문을 담고 있으며, 이를 통해 원하는 작업을 수행할 수 있습니다. 예를 들어, 응답 데이터를 로깅하거나, 추가적인 데이터 변환을 수행하거나, 다른 서비스와의 통신을 이어갈 수 있습니다.

`subscribe` 메소드는 비동기적으로 작동하기 때문에, 이 메소드가 호출되는 시점에는 이미 `WebClient`의 HTTP 요청이 완료되었거나, 진행 중일 수 있습니다. 이는 `WebClient`가 논블로킹 방식으로 작동함을 의미하며, 애플리케이션은 다른 작업을 계속 진행할 수 있습니다.

이러한 비동기적 처리 방식은 높은 동시성과 효율적인 리소스 사용을 가능하게 하며, 이것이 `WebClient`를 사용하는 주된 이유 중 하나입니다.

### 고급 설정 & Bean 등록 예제 코드

```java
@Configuration
public class WebClientConfig {
    @Bean
    public WebClient webClientBuilder() {
        ConnectionProvider connectionProvider = ConnectionProvider.builder("http-pool")
                .maxConnections(100)
                .pendingAcquireTimeout(Duration.ofMillis(0))
                .pendingAcquireMaxCount(-1)
                .maxIdleTime(Duration.ofMillis(1000L))
                .build();

        HttpClient httpClient = HttpClient.create(connectionProvider)
                .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 10000);

        return WebClient.builder()
                .baseUrl("https://example.com")
                .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                .codecs(clientCodecConfigurer -> clientCodecConfigurer.defaultCodecs().maxInMemorySize(2 * 1024 * 1024)) // 2MB
                .clientConnector(new ReactorClientHttpConnector(httpClient))
                .build();
    }
}
```

- **ConnectionProvider**:

  - **maxConnections**: 커넥션 풀 내에서 유지할 수 있는 최대 커넥션의 수입니다. 이 값은 서버로의 동시 연결 수를 제한하는 데 사용됩니다.
  - **pendingAcquireTimeout**: 커넥션 풀에서 사용 가능한 커넥션을 얻기 위해 대기하는 최대 시간입니다. 이 시간이 초과되면 요청은 실패합니다. `Duration.ofMillis(0)`은 즉시 사용 가능한 커넥션이 없을 경우 대기하지 않고 실패를 반환하도록 설정합니다.
  - **pendingAcquireMaxCount**: 커넥션 풀에서 커넥션을 얻기 위해 시도할 수 있는 최대 요청 수입니다. `-1`은 제한이 없음을 의미하며, 이는 커넥션 풀이 가득 차 있을 때 새로운 커넥션 요청이 계속 대기할 수 있음을 나타냅니다.
  - **maxIdleTime**: 커넥션 풀에서 유휴 상태(idle status)인 커넥션이 유지되는 최대 시간입니다. 이 시간이 지나면, 해당 커넥션은 풀에서 제거되고 종료됩니다. 이 설정은 리소스 사용 최적화에 도움이 됩니다.

- **HttpClient 연결 시간 초과 설정**:

  ```java
  HttpClient httpClient = HttpClient.create(connectionProvider)
      .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 10000);
  ```

  `HttpClient`의 `CONNECT_TIMEOUT_MILLIS` 옵션은 서버에 연결을 시도할 때 적용되는 최대 대기 시간을 설정합니다. 이 시간 동안 연결이 성공적으로 이루어지지 않으면, `HttpClient`는 연결 시도를 중단하고 오류를 반환합니다. 이는 네트워크 지연이나 서버 문제로 인해 연결이 지연되는 경우를 방지하고, 애플리케이션의 응답성을 유지하는 데 중요합니다.

  위 코드의 `.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 10000)`는 연결 시간 초과를 10초(10000밀리초)로 설정합니다.

- **WebClient.Builder**:
  - **baseUrl**: 모든 요청에 대한 기본 URL을 설정합니다.
  - **defaultHeader**: 모든 요청에 기본적으로 포함될 헤더를 지정합니다.
  - **codecs maxInMemorySize**: 메모리에 데이터를 버퍼링하는 데 사용할 수 있는 최대 크기를 설정합니다. Spring WebFlux에서 기본 in-memory buffer 크기는 256KB로 설정되어 있습니다. 이는 작은 HTTP 메시지 처리에는 적합하지만, 256KB를 초과하는 크기의 메시지를 처리할 때 DataBufferLimitException 오류를 발생시킬 수 있습니다. 이 설정을 통해 최대 메모리 크기를 증가시켜, 더 큰 HTTP 응답을 처리할 수 있도록 할 수 있습니다.
  - **clientConnector**: `HttpClient`를 `WebClient`와 연결합니다. 커넥션 풀, 타임아웃, 기타 네트워크 관련 설정을 적용할 수 있습니다.

### ParameterizedTypeReference를 이용한 데이터 추출

![사진](/assets/img/posts/2024-01-06/스크린샷%202024-01-06%20011324.png)

`WebClient`를 사용하여 API에서 데이터를 가져올 때, 위와 같이 응답이 특정한 공통 포맷을 따르는 경우가 많습니다. 이러한 상황에서 응답 본문의 특정 부분만을 추출하려면 `ParameterizedTypeReference`를 사용하는 것이 효과적입니다. 이 클래스를 사용하면 복잡한 제네릭 타입을 가진 객체도 쉽게 처리할 수 있습니다.

다음은 `ParameterizedTypeReference`를 사용하여 API 응답에서 데이터 부분만 추출하는 예제입니다:

```java
public Mono<UserClientRes.MyDto> getUserData() {
    return webClient.get()
            .uri("/api/v1/user/my")
            .retrieve()
            .bodyToMono(new ParameterizedTypeReference<ComResponseDto<UserClientRes.MyDto>>() {})
            .map(ComResponseDto::getBody);
}
```

이 코드에서 `bodyToMono` 메소드는 `ParameterizedTypeReference`를 사용하여 복잡한 제네릭 타입의 응답을 처리합니다. 여기서 `ComResponseDto<>`는 API 응답의 공통 포맷을 나타냅니다. 이 포맷은 일반적으로 상태 코드, 메시지와 함께 실제 데이터를 `body` 필드에 포함합니다.

`.map(ComResponseDto::getBody)` 부분은 `Mono<ComResponseDto<UserClientRes.MyDto>>`에서 `body` 필드만을 추출하여 `Mono<UserClientRes.MyDto>`로 변환합니다. 이렇게 하면 응답의 다른 부분은 무시하고 실제 필요한 데이터 부분만을 얻을 수 있습니다.

이 방법을 사용하면 API 응답의 공통 포맷을 처리하고, 필요한 데이터 부분만을 효과적으로 추출할 수 있으며 복잡한 응답 구조에서도 원하는 데이터를 쉽고 깔끔하게 접근할 수 있습니다.

## 결론

`WebClient`는 비동기, 논블로킹 특성과 리액티브 프로그래밍을 지원하여 효율적인 통신 환경을 제공해줍니다. 하지만 `webflux` 의존성을 추가해야한다는 것과 함수형 코드로 되어있어 진입 장벽이 있을 수도 있겠습니다.

다행히도 Spring 6.1이 릴리즈 되면서 RestTemplate를 대체할 현대적인 동기 HttpClient인 [_RestClient_](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestClient.html){:target="\_blank"}가 나왔다고 합니다.([_참고_](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html){:target="\_blank"}) 나중에 저도 한 번 써봐야겠습니다. 😄
