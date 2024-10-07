---
title: Spring Boot에서 Lettuce를 사용한 Redis 설정
author: kjb4494
date: 2023-11-03 12:00:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend, redis]
---

## 의존성 추가하기

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
```

`org.springframework.boot:spring-boot-starter-data-redis` 의존성은 Spring Boot 프로젝트에서 Redis를 사용하기 위한 스타터 패키지입니다. 이 스타터는 다음과 같은 것들을 포함하고 있습니다:

- Spring Data Redis - Redis와의 통합을 위한 Spring Data의 일부분
- Lettuce - 기본 Redis 클라이언트 (비동기 이벤트 기반 Redis 클라이언트)
- Jedis - 선택적으로 사용할 수 있는 다른 Redis 클라이언트 라이브러리 (동기 블로킹 방식의 클라이언트)
- Commons Pool 2 - 커넥션 풀링 지원

`spring-boot-starter-data-redis` 스타터는 필요한 라이브러리와 자동 설정을 포함하므로, 대부분의 표준 사용 사례에서 추가적인 의존성 없이 Redis를 사용할 수 있게 해줍니다.

그러나, 특정 기능을 사용하기 위해 추가적인 설정이나 의존성이 필요한 경우도 있을 수 있습니다. 예를 들어, Redis 트랜잭션 관리나 JSON 직렬화, Spring Session 통합 등 말이죠. 이러한 기능들은 별도의 의존성 추가나 설정을 요구할 수 있습니다.

기본적인 Redis 캐싱과 간단한 데이터 접근에는 `spring-boot-starter-data-redis`만으로 충분하지만, 프로젝트의 요구 사항에 따라서 추가적인 설정이나 의존성을 고려해야 할 수 있습니다.

### Jedis vs Lettuce

Redis에 접근하기 위한 두 가지 주요 Java 클라이언트 라이브러리는 Jedis와 Lettuce입니다. 두 라이브러리는 각각 아래와 같은 장단점이 있습니다:

#### Jedis:

- **장점:**
  - 간단하고 직관적인 API를 제공합니다.
  - 오랜 시간에 걸쳐 사용되며 검증된 라이브러리입니다.
- **단점:**
  - 동기적인 블로킹 I/O를 사용합니다.
  - Jedis 인스턴스는 스레드에 안전하지 않으므로 멀티스레딩 환경에서는 Connection Pool이 필요합니다.

#### Lettuce:

- **장점:**
  - 비동기적이고 논블로킹 I/O 작업을 지원합니다.
  - 스레드 안전하며 멀티스레딩 환경에서 하나의 인스턴스를 공유할 수 있습니다.
  - 반응형 프로그래밍과 잘 어울립니다.
- **단점:**
  - 비동기 프로그래밍 모델이 복잡하게 느껴질 수 있으며, Jedis에 비해 API가 더 복잡할 수 있습니다.

현재는 다음과 같은 이유로 대다수가 Lettuce를 더 선호하고 있습니다.

- 멀티스레딩 환경에서 단일 인스턴스를 공유할 수 있어 연결 관리가 더 간편합니다.
- 비동기 처리 능력으로 인해 더 효율적인 리소스 활용과 빠른 응답 시간을 제공합니다.
- Redis의 reactive 프로그래밍 모델을 사용하고자 할 때 더 적합합니다.

Lettuce는 내부적으로 Netty 프레임워크를 사용하여 비동기 통신을 구현합니다. 이는 현대의 높은 동시성을 요구하는 애플리케이션에 적합합니다. Lettuce의 비동기 논블로킹 접근 방식은 고성능을 유지하면서도 리소스 사용을 최소화하는 데 도움이 됩니다.

## Redis 직렬화 설정

Spring Data Redis에서는 Redis와의 데이터 교환을 위해 직렬화가 필요합니다. 다음과 같이 직렬화 전략을 정의하여 데이터의 형식을 관리할 수 있습니다.

```java
// 문자열 직렬화 전략 설정
@Bean("coreStringRedisSerializer")
public StringRedisSerializer stringRedisSerializer() {
    return new StringRedisSerializer();
}

// JSON 직렬화 전략 설정
@Bean("coreJsonRedisSerializer")
public Jackson2JsonRedisSerializer<Object> jsonRedisSerializer() {
    return new Jackson2JsonRedisSerializer<>(Object.class);
}
```

## Lettuce 기반 Redis 연결 설정

```java
@Bean("petDiaryRedisConnectionFactory")
LettuceConnectionFactory redisConnectionFactory() {
    // RedisStandaloneConfiguration을 통해 단일 노드 Redis 서버의 호스트와 포트를 설정합니다.
    RedisStandaloneConfiguration configuration = new RedisStandaloneConfiguration(
            petDiaryRedisProperties.getHost(),
            petDiaryRedisProperties.getPort()
    );
    // Redis 비밀번호를 설정합니다.
    configuration.setPassword(petDiaryRedisProperties.getPassword());

    // LettuceClientConfigurationBuilder를 사용하여 Lettuce 클라이언트의 설정을 구성합니다.
    LettuceClientConfiguration.LettuceClientConfigurationBuilder clientConfigBuilder = LettuceClientConfiguration.builder();
    // Redis 서버가 TLS/SSL을 사용한다면, SSL을 사용하도록 설정하고, 피어 검증을 비활성화합니다.
    if (petDiaryRedisProperties.isUseTls()) clientConfigBuilder.useSsl().disablePeerVerification();
    // 설정 빌더를 통해 LettuceClientConfiguration 객체를 생성합니다.
    LettuceClientConfiguration clientConfig = clientConfigBuilder.build();

    // LettuceConnectionFactory를 생성하고, 위에서 정의한 Redis 서버 설정과 클라이언트 설정을 적용합니다.
    return new LettuceConnectionFactory(configuration, clientConfig);
}
```

## RedisTemplate 설정

RedisTemplate을 통해 Redis 연결과 데이터 조작을 수행합니다.

```java
@Bean("petDiaryRedisTemplate")
public RedisTemplate<String, Object> redisTemplate(@Qualifier("petDiaryRedisConnectionFactory") LettuceConnectionFactory redisConnectionFactory) {
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(redisConnectionFactory);
    return redisTemplate;
}
```

이제 Spring Boot 애플리케이션에서 Lettuce를 사용하여 Redis 연결을 관리하고, RedisTemplate를 통해 데이터 조작을 수행할 수 있습니다.

## Spring Data Redis

Java에서 Redis를 위와 같은 방식으로 사용하는 것을 "Spring Data Redis"라고 합니다. 여기서 사용된 방식은 JPA(Java Persistence API)의 어노테이션 스타일을 차용하여, 객체를 Redis에 저장하는 객체-해시 매핑(Object-Hash Mapping) 방식입니다. 이런 방식은 JPA의 객체-관계 매핑(Object-Relational Mapping, ORM)을 비슷하게 따르고 있어서, Redis에 대해 ORM과 유사한 개념인 OXM(Object-XML Mapping)이나 OHM(Object-Hash Mapping)이라고 볼 수 있습니다.

Spring Data Redis는 Spring 프레임워크의 일부로, 개발자가 Java 객체를 Redis 데이터 구조와 매핑할 수 있게 해주는 것이 특징입니다. 이를 통해 복잡한 Redis 명령어를 직접 사용하지 않고도, 익숙한 Spring Data 인터페이스와 어노테이션을 통해 Redis 작업을 수행할 수 있습니다.

### Entity

```java
@Getter @ToString
@Builder @AllArgsConstructor @NoArgsConstructor
@RedisHash(value = "MemberAccessToken")
public class RedisMemberAccessToken {
    @Id
    private String jwt;
    @Indexed
    private Long memberIdx;

    @TimeToLive
    private long expiredTime;
}
```

위에서 사용한 어노테이션은 다음과 같은 역할을 합니다.

- **@RedisHash**: 클래스가 Redis 해시에 저장될 엔티티임을 나타냄
- **@Id**: Redis 키를 구성하는 필드
- **@Indexed**: 쿼리 가능한 필드로 만들어줌
- **@TimeToLive**: Redis에서 해당 엔티티의 만료 시간을 설정함 (단위: 초)

### Repository

```java
public interface RedisMemberAccessTokenRepository extends CrudRepository<RedisMemberAccessToken, String> {
    List<RedisMemberAccessToken> findByMemberIdx(Long memberIdx);
}
```

Entity에서 memberIdx 필드에 @Indexed를 걸어줬기 때문에 MemberIdx로 검색할 수 있습니다.

### Service

```java
public void deleteAllAccessTokenByMemberIdx(Long memberIdx) {
    List<RedisMemberAccessToken> redisMemberAccessTokenList = redisMemberAccessTokenRepository.findByMemberIdx(memberIdx);
    redisMemberAccessTokenList.forEach(redisMemberAccessToken -> redisMemberAccessTokenRepository.deleteById(redisMemberAccessToken.getJwt()));
}
```

이제 서비스단에서 JPA처럼 사용할 수 있습니다.
