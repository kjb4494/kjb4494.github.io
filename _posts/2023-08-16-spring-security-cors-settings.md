---
title: Spring Security에서 CORS 설정하기
author: kjb4494
date: 2023-08-16 15:00:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend, spring security]
---

Spring Boot와 Spring Security를 사용하여 CORS (Cross-Origin Resource Sharing)를 설정하는 방법을 알아보겠습니다.

## 목차

1. CORS란?
2. Spring Security와 CORS
3. CORS 설정 방법

## 1. CORS란?

CORS는 Cross-Origin Resource Sharing의 약자입니다. 웹 애플리케이션은 보안상의 이유로 동일 출처 정책(Same-Origin Policy)을 따릅니다. 이 정책은 브라우저가 다른 출처의 리소스를 기본적으로 차단하는 정책입니다. CORS는 이를 해결하기 위한 웹 표준입니다.

## 2. Spring Security와 CORS

Spring Security는 기본적으로 보안을 엄격하게 적용하기 때문에, 별도의 설정을 하지 않으면 CORS 요청이 차단됩니다. 따라서 Spring Security 설정에서 CORS를 명시적으로 허용해줘야 합니다.

## 3. CORS 설정 방법

### 3.1 Custom CORS Configuration Class: **CorsProperties**

먼저, `CorsProperties`라는 Configuration 클래스를 만들어서 프로퍼티 값을 가져옵니다. 이 클래스는 `application.properties`나 `application.yml` 파일에서 설정된 CORS 관련 값을 로딩합니다.

```java
@Configuration
@ConfigurationProperties(prefix = "cors")
@Getter @Setter
public class CorsProperties {
    private List<String> allowedOrigins;
    private String allowedHeader;
    private String allowedMethod;
    private boolean allowedCredentials;
}
```

### 3.2 **CorsConfigurationSource** Bean Configuration

`CorsConfigurationSource`를 Bean으로 등록하고, 이를 사용해서 CORS를 설정합니다. 이 Bean은 CORS 허용 설정을 정의하고 이를 Spring Security에 적용합니다.

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    configuration.addAllowedHeader(corsProperties.getAllowedHeader());
    configuration.addAllowedMethod(corsProperties.getAllowedMethod());
    configuration.setAllowCredentials(corsProperties.isAllowedCredentials());
    if (corsProperties.getAllowedOrigins() != null) {
        corsProperties.getAllowedOrigins().forEach(configuration::addAllowedOriginPattern);
    }

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", configuration);

    return source;
}
```

### 3.3 Integrate CORS Configuration in **SecurityFilterChain**

Spring Security 설정에서, 이전에 정의한 `CorsConfigurationSource` Bean을 사용하여 CORS 설정을 적용합니다.

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .cors(cors -> cors.configurationSource(corsConfigurationSource()))
        // ... other security configurations ...
        ;
    return http.build();
}
```

여기서 `.cors(cors -> cors.configurationSource(corsConfigurationSource()))`는 Spring Security에 `CorsConfigurationSource`를 적용하는 부분입니다. 이를 통해, 우리가 정의한 CORS 설정이 Spring Security에 정상적으로 적용됩니다.

### 3.4 Application Properties or YAML Configuration

마지막으로, CORS 관련 설정을 `application.properties` 또는 `application.yml` 파일에 추가해줍니다.

`application.properties` 예제:

```properties
# cors settings
cors.allowed-origins=https://example.com,https://another-example.com
cors.allowed-header=*
cors.allowed-method=*
cors.allowed-credentials=true
```

`application.yml` 예제:

```yml
cors:
  allowed-origins:
    - https://example.com
    - https://another-example.com
  allowed-header: "*"
  allowed-method: "*"
  allowed-credentials: true
```

이렇게 설정 파일에서 콤마로 구분하여 여러 도메인을 허용하는 설정을 할 수 있습니다.

## 마치며

Restful API 서버를 처음 구축하다 보면, CORS 문제로 난항을 겪는 경우가 꽤 있습니다. 이런 문제는 특히 프론트엔드와 백엔드가 서로 다른 도메인에서 작동하는 경우에 자주 발생하게 됩니다. 이런 상황에서 서버가 클라이언트의 요청을 올바르게 처리할 수 있도록 CORS 설정이 필요합니다.

이번 글에서는 Spring Security를 이용한 CORS 설정 방법을 상세히 다루어 보았습니다. 특히 `CorsConfigurationSource`의 사용법과 이를 Spring Security 설정에 통합하는 방법, 그리고 이에 필요한 프로퍼티 설정 방법까지 단계별로 설명하였습니다.

처음 CORS 개념을 접할 경우 복잡하고 까다롭게 느껴질 수 있는데, 그런 분들께 이 글이 도움을 줄 수 있다면 좋겠습니다. 😊
