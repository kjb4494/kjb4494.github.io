---
title: JPA 이야기 05 편 - n+1 문제
author: kjb4494
date: 2023-08-09 14:00:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend, jpa]
---

JPA는 데이터베이스 작업을 추상화하고 간단하게 만들어주는 강력한 도구입니다. 그러나 때로는 이러한 추상화로 인해 예상치 못한 성능 문제가 발생할 수 있습니다. 이 글에서는 그 중에서도 n+1 문제에 대해 다루고, 이를 해결하는 방법에 대해 알아보겠습니다.

## n+1 문제란?

n+1 문제는 JPA에서 연관된 엔터티를 로딩할 때 발생하는 흔한 성능 문제입니다.
이 문제의 이름에서 "n"은 연관된 엔터티의 수를 나타내며, "1"은 처음 엔터티를 조회하는 쿼리를 나타냅니다. 따라서, n+1 문제는 하나의 쿼리로 부모 엔터티를 조회한 후, 각 부모 엔터티에 대해 연관된 자식 엔터티를 조회하는 추가적인 쿼리가 발생하는 현상을 의미합니다.

## n+1 문제의 해결 방법

### 1. **Fetch Join**

JPQL의 JOIN FETCH를 사용하여 연관된 엔터티를 즉시 로딩할 수 있습니다.

```java
@Entity
public class Post {
    @OneToMany(fetch = FetchType.LAZY)
    private List<Comment> comments;
}

List<Post> posts = em.createQuery("SELECT p FROM Post p JOIN FETCH p.comments", Post.class).getResultList();
```

### 2. **Entity Graph**

`@EntityGraph`를 사용하여 필요한 속성만 선택적으로 즉시 로딩할 수 있습니다.

```java
@EntityGraph(attributePaths = "comments")
@Query("SELECT p FROM Post p")
List<Post> findAllWithComments();
```

### 3. **Batch Size**

Batch Size 설정은 Lazy Loading의 성능 이슈를 해결하기 위한 방법 중 하나입니다. 특히 JPA에서 엔터티나 컬렉션을 Lazy Loading할 때, 실제로 해당 데이터에 접근하려 할 때 로딩이 발생합니다. 이때 한 번에 하나의 엔터티나 컬렉션만 로딩하는 대신, Batch Size를 설정하여 여러 엔터티나 컬렉션을 한 번의 쿼리로 효율적으로 로딩할 수 있습니다. 이러한 방식으로 n+1 문제를 줄일 수 있습니다. Batch Size 설정 방식은 크게 두 가지가 있습니다.

#### @BatchSize 어노테이션 사용

`@BatchSize` 어노테이션은 특정 엔터티나 컬렉션에 대한 배치 크기를 지정합니다. 예를 들어, 아래와 같이 설정하면 `Comment` 리스트를 로딩할 때 한 번에 10개의 `Comment`를 가져옵니다.

```java
@OneToMany
@BatchSize(size = 10)
private List<Comment> comments;
```

#### hibernate.default_batch_fetch_size 설정 사용

전역적으로 배치 가져오기 크기를 설정하고 싶다면, `hibernate.default_batch_fetch_size` 속성을 사용하면 됩니다. 이 설정은 `application.properties`나 `application.yml` 파일에 지정할 수 있습니다.

```properties
hibernate.default_batch_fetch_size = 20
```

만약 `@BatchSize` 어노테이션과 `hibernate.default_batch_fetch_size` 설정이 동시에 사용된다면, `@BatchSize` 어노테이션이 설정된 엔터티나 컬렉션에 대해서는 해당 어노테이션의 값이 우선적으로 적용됩니다. 다시 말해, `@BatchSize`의 우선순위가 `hibernate.default_batch_fetch_size` 설정보다 높습니다.

#### Batch Fetch Style

Hibernate는 Batch fetch style을 통해 연관된 엔터티나 컬렉션을 어떻게 가져올지 결정할 수 있습니다. 이 전략은 `hibernate.batch_fetch_style` 프로퍼티를 통해 설정할 수 있습니다.

```properties
hibernate.batch_fetch_style = padded
```

아래는 각 전략에 대한 설명입니다:

- `legacy`: Hibernate 3.x의 fetch style입니다. 기본적으로 사용되는 전략으로, 이전 버전의 Hibernate에서 사용되던 방식입니다.
- `padded`: 항상 고정된 수의 파라미터로 쿼리를 실행합니다. 이로 인해 데이터베이스는 쿼리 계획을 일관되게 생성하고 캐싱할 수 있습니다. 이 전략은 일반적으로 좋은 성능을 제공하며, 쿼리 계획의 캐싱으로 인한 성능 향상을 원하는 경우에 유용합니다.
- `dynamic`: 필요한 만큼의 파라미터로 쿼리를 동적으로 생성합니다. 이 전략은 쿼리 파라미터의 수가 고정되지 않아도 되는 데이터베이스에 유용합니다. 하지만 데이터베이스에서 쿼리 계획을 캐싱하기 어려워, 성능에 영향을 줄 수 있습니다.

각 배치 가져오기 전략은 애플리케이션의 특성과 데이터베이스 환경을 고려하여 최적의 전략을 선택하는 것이 중요합니다. 전략 선택시 성능 테스트와 분석을 통해 실제 환경에서 어떤 전략이 더 효과적인지 확인하시는게 좋을거같습니다.

### 4. **Projection (DTO 사용)**

DTO를 사용하여 필요한 데이터만 선택적으로 가져올 수 있습니다. 이 방법은 필요한 데이터만 로딩하므로 성능을 크게 향상시킬 수 있습니다.

먼저, 전통적인 방식으로 DTO 객체를 생성하고, JPQL을 사용하여 필요한 데이터를 가져오는 예제를 보겠습니다:

```java
// DTO 객체
public class PostCommentDto {
    private String postTitle;
    private String commentContent;

    public PostCommentDto(String postTitle, String commentContent) {
        this.postTitle = postTitle;
        this.commentContent = commentContent;
    }

    // getters, setters...
}

// Repository
public interface PostRepository extends JpaRepository<Post, Long> {
    @Query("SELECT new com.example.PostCommentDto(p.title, c.content) FROM Post p JOIN p.comments c")
    List<PostCommentDto> findAllPostComments();
}
```

위와 같이 직접 DTO 객체를 정의하고 JPQL을 이용하면 원하는 데이터만 선택적으로 가져올 수 있습니다.

하지만 보시다싶이 쿼리문이 너무 복잡해지게 됩니다. 이 경우, 인터페이스 기반의 프로젝션을 제공해 코드를 간결하게 만들 수 있습니다:

```java
public interface PostCommentProjection {
    String getPostTitle();
    String getCommentContent();
}

public interface PostRepository extends JpaRepository<Post, Long> {
    @Query("SELECT p.title as postTitle, c.content as commentContent FROM Post p JOIN p.comments c")
    List<PostCommentProjection> findAllPostComments();
}
```

이렇게 인터페이스를 사용하면 메서드의 반환 타입만으로 간단하게 프로젝션을 정의할 수 있습니다. 주의할 점은 여기서 `PostCommentProjection` 인터페이스의 메서드명과 JPQL 쿼리의 alias 이름 (`postTitle`, `commentContent`)이 일치해야한다는 것입니다.

### 5. **Open Session in View 설정 변경**

`Open Session In View`는 `Hibernate` 세션을 HTTP 요청의 끝까지 열어두는 설정입니다. 이로 인해 뷰 렌더링 중에도 LAZY 로딩이 가능하게 됩니다. Spring Boot에서는 `spring.jpa.open-in-view` 프로퍼티로 이 설정을 제공하며, 기본적으로는 `true`로 설정되어 있습니다.

그러나, 이 설정을 `true`로 두면 뷰 렌더링 시점에 예기치 않은 쿼리가 발생할 수 있어 성능 문제가 발생할 위험이 있습니다. 또한, API의 응답을 생성하는 도중에 추가적인 쿼리가 발생하는 것은 바람직하지 않습니다.

따라서 권장하는 방식은 `spring.jpa.open-in-view`를 `false`로 설정하고 필요한 데이터를 컨트롤러나 서비스 레이어에서 미리 로딩하는 것입니다. 이렇게 하면 뷰 렌더링 중 LAZY 로딩을 시도할 때 `LazyInitializationException`이 발생하므로, 개발자는 문제의 원인을 쉽게 파악할 수 있습니다.

```properties
# application.properties
spring.jpa.open-in-view=false
```

특히나 Restful API 서버에서는 이 설정을 `false`로 하는 것이 바람직합니다.

## 결론

JPA는 Spring의 ORM으로서, 데이터베이스 작업을 단순화하면서 개발자에게 편리한 수단을 제공합니다. 하지만 막상 도입하려고하면 n+1 같은 성능상 문제들을 직면하게 됩니다. 이 글이 문제를 해결하는데 조금이나마 도움이 되었으면 좋겠습니다. 😊
