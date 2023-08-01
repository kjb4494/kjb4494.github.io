---
title: JPA 이야기 02 편 - 다양한 연관관계 매핑
author: kjb4494
date: 2022-10-14 16:00:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend, jpa]
---

JPA에서 연관관계를 매핑할 때는 3가지를 고려해야합니다.

첫번째로는 `다중성`입니다. 즉, 연관관계가 다대일(ManyToOne)인지, 일대다(OneToMany)인지, 일대일(OneToOne)인지, 다대다(ManyToMany)인지를 정하는 것입니다. 이와 관련해서 일대일(OneToOne)에서 연관관계의 주인이 아닌 엔티티를 가져올 때 Lazy Loading이 동작하지않는 문제, 다대다(ManyToMany)의 구조적 문제까지 다뤄볼 예정입니다.

두번째로는 `단방향으로 할 것인지, 양방향으로 할 것인지`입니다. 관계형 데이터베이스의 테이블은 원래부터 양방향으로 참조가 가능한 구조입니다. 하지만 JAVA는 객체지향언어이기 때문에 결국 JPA에서 다루는 것은 `객체`입니다. 객체간의 관계는 일방통행이기 때문에 양방향 참조를 하고싶다면 단방향 매핑 2개를 만들어줘야합니다. 하지만 단방향으로만 맺어도 테이블 매핑은 완료됩니다. 양방향으로 참조하는 엔티티 연관관계가 많아질수록 프로젝트가 복잡해질 수 있기 때문에 저는 엔티티 설계시엔 단방향으로 맺고, 추후에 필요해질 경우 양방향으로 맺어주고 있습니다.

세번째로는 `연관관계의 주인이 누구인지`입니다. 관계형 데이터베이스의 테이블에서는 연관관계를 외래 키가 관리합니다. 하지만 엔티티를 양방향으로 매핑하면 양쪽 모두의 객체에서 서로를 참조하게 됩니다. 따라서 둘 중 누가 연관관계를 관리하는 주인이 될 것인지 정해야합니다. 보통은 `외래 키를 가진 테이블과 매핑한 엔티티`가 외래키를 관리하는 게 효율적이기 때문에 해당 엔티티를 주인으로 선택합니다. 예를 들어 `일대다(OneToMany) 관계에서는 다(Many)쪽이 외래키를 가지기 때문에 다(Many)쪽이 연관관계의 주인이 됩니다.` JPA에서 연관관계의 주인이 아니면 `mappedBy` 속성을 사용하고 연관관계의 주인 필드 이름을 값으로 입력해야합니다.

## 다대일(ManyToOne) & 일대다(OneToMany)

두 관계는 항상 함께 존재하며 위에서 언급했듯이 연관관계의 주인은 항상 다(Many)쪽입니다. 예를 들어 `책장`과 `책`이라는 두 개의 엔티티 객체가 있고 두 객체간의 연관관계는 책장(One):책(Many) 입니다. 여기서 주의할 것은 주인을 정할 때 비지니스적 관계는 고려하지말아야한다는 점입니다. 언뜻보기엔 책장이 주인으로 보일 수 있기 때문이죠. 일대다(OneToMany) 관계에서는 다(Many)쪽이 외래키를 가지고 있고, 외래키를 가진 엔티티가 주인이기 때문에 해당 예시에서는 `책이 주인`이 됩니다.

아래는 토이프로젝트의 게시글(Post)과 댓글(Comment) 엔티티입니다.

{: file='Post.java'}

```java
@Entity
@Table(name = "post")
@Getter @Setter @ToString
@NoArgsConstructor @AllArgsConstructor @Builder
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "idx")
    private Long idx;

    @Column(name = "status_code", nullable = false)
    @Convert(converter = StatusCodeConverter.class)
    private StatusCode statusCode;

    @Column(name = "reg_date", nullable = false)
    private LocalDateTime regDate;

    @Column(name = "updated_date")
    private LocalDateTime updatedDate;

    @Column(name = "title")
    private String title;

    @Column(name = "comment_count", nullable = false)
    @ColumnDefault("0")
    private Integer commentCount;

    @OneToMany(mappedBy = "post")
    private List<BoardComment> commentList;

    @Column(name = "user_idx")
    private Long userIdx;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_idx", insertable = false, updatable = false)
    private BoardUser user;

    @OneToOne(mappedBy = "post", cascade = CascadeType.ALL, optional = false, fetch = FetchType.LAZY)
    private PostDetail postDetail;
}
```

{: file='Comment.java'}

```java
@Entity
@Table(name = "comment")
@Getter @Setter @ToString
@NoArgsConstructor @AllArgsConstructor @Builder
public class BoardComment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "idx")
    private Long idx;

    @Column(name = "status_code", nullable = false)
    @Convert(converter = StatusCodeConverter.class)
    private StatusCode statusCode;

    @Column(name = "reg_date")
    private LocalDateTime regDate;

    @Column(name = "updated_date")
    private LocalDateTime updatedDate;

    @Column(name = "contents")
    private String contents;

    @Column(name = "post_idx")
    private Long postIdx;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_idx", insertable = false, updatable = false)
    private Post post;

    @Column(name = "user_idx")
    private Long userIdx;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_idx", insertable = false, updatable = false)
    private BoardUser user;
}
```

### mappedBy

Post와 Comment는 일대다(ManyToOne) 연관관계를 가지고있습니다. Comment 엔티티 객체가 주인이기 때문에 주인이 아닌 Post에는 `mappedBy` 속성을 사용하고 Comment 객체에서 Post의 필드이름인 `post`를 값으로 줬습니다. (`@OneToMany(mappedBy = "post")`)

### 지연로딩(Lazy Loading)

`@ManyToOne(fetch = FetchType.LAZY)`는 Comment를 데이터베이스에서 가져올 때 Post를 동시 참조하지않고 나중에 Post를 사용할 때 참조해서 가져오도록하는 설정입니다. 만약 사용하지도않는데 Comment를 가져올때마다 Post까지 검색하면 데이터베이스의 부담도 그만큼 커질 것입니다. X(

### @JoinColumn

`@JoinColumn(name = "post_idx", insertable = false, updatable = false)`는 post_idx 컬럼값으로 Post를 참조하되 해당 필드를 이용해 insert 하거나 update 하지는 않겠다는 뜻입니다. 즉 read only 필드로 사용하겠다는 뜻이죠.

### @Id

`@Id`는 해당 필드가 기본키라는 뜻입니다. `JPA에서 기본키는 무조건 존재해야합니다.`

### @GeneratedValue

`@GeneratedValue`는 기본키를 자동으로 생성할 때 어떤 전략을 사용할 것인지를 명시합니다. `GenerationType.IDENTITY`는 기본키 생성을 데이터베이스에 위임하는 방식입니다. 따라서 insert 할 때 필드값을 따로 명시하지않아도 데이터베이스가 자동으로 `AUTO_INCREMENT`로 기본키를 생성해줍니다.

`기본키를 직접 할당해 개발자가 관리할 경우 이 어노테이션은 사용하지않습니다.` 자동 생성 전략은 위에서 사용한 `IDENTITY` 외에 `SEQUENCE`, `TABLE` 전략이 있습니다. 자동 생성 전략이 이처럼 다양한 이유는 데이터베이스 벤더마다 지원하는 방식이 다르기 때문입니다. 예를 들어 오라클 데이터베이스는 시퀀스를 제공하지만 MySQL은 시퀀스를 제공하지 않죠. 따라서 MySQL에서 SEQUENCE를 사용하고싶다면 별도의 테이블을 생성해 시퀀스를 흉내내야합니다. 이 경우 오라클 데이터베이스는 SEQUENCE 전략을, MySQL는 TABLE 전략을 사용하면 됩니다. 전략을 `AUTO`로 설정할 경우 선택한 데이터베이스 방언에 따라 자동으로 설정해줍니다. 예를 들어 오라클 데이터베이스를 사용하면 SEQUENCE를, MySQL을 선택하면 IDENTITY를 사용합니다. `AUTO`의 장점은 데이터베이스를 변경해도 코드를 수정할 필요가 없다는 것입니다. 하지만 되도록이면 `AUTO`로 설정하지않고 사용할 전략을 제대로 명시해주는게 좋습니다.

JPA는 엔티티가 영속 상태로 만들기 위해선 식별자가 반드시 필요합니다. 하지만 `IDENTITY` 전략을 사용하면 인스턴스가 데이터베이스에 저장된 후에야 식별자를 구할 수 있기 때문에 em.persist()를 호출하는 즉시 insert sql문이 전달됩니다. 이러한 메커니즘 때문에 `IDENTITY 전략을 사용하는 엔티티는 쓰기 지연이 동작하지 않습니다`.

### @Convert

`@Convert`는 엔티티의 필드값을 데이터베이스에 저장할 때 또는 데이터베이스의 값을 엔티티의 필드값으로 가져올 때 어떻게 변환할지를 명시합니다.

아래는 statusCode 필드의 converter로 사용된 StatusCodeConverter.class 입니다.

{: file='StatusCodeConverter.java'}

```java
public class StatusCodeConverter implements AttributeConverter<StatusCode, Byte> {
    @Override
    public Byte convertToDatabaseColumn(StatusCode attribute) {
        if (attribute == null) return null;
        return attribute.getCode();
    }

    @Override
    public StatusCode convertToEntityAttribute(Byte dbData) {
        if (dbData == null) return null;
        return StatusCode.getByCode(dbData);
    }
}
```

{: file='StatusCode.java'}

```java
public enum StatusCode {
    Private((byte)0),
    Public((byte)1),
    Deleted((byte)2);

    final private byte code;

    StatusCode(byte code) {
        this.code = code;
    }

    public byte getCode() {
        return code;
    }

    public static StatusCode getByCode(byte code) {
        for (StatusCode statusCode: StatusCode.values()) {
            if (statusCode.code == code) return statusCode;
        }
        return null;
    }
}
```

## 일대일(OneToOne)

일대일 관계는 양쪽이 서로 하나의 관계만을 가집니다. 이와 관련해서 구글링할 때 일대일 관계는 Bad Practice인가? 라는 우려의 글도 많이 보였습니다. 하지만 JPA를 사용하면 대부분 엔티티 객체 단위로 움직이게되기 때문에 MEDIUMTEXT 타입같은 비교적 큰 용량의 데이터를 가지는 칼럼은 따로 빼는게 좋다고 생각했습니다. 그 밖에도 기획상 추가해야할 기능이 필요할 경우 불가피하게 일대일 관계의 테이블을 만들어야하는 경우도 생기고말이죠. 물론 하나의 엔티티에 우겨넣은 후 DTO를 사용하는 방법도 생각해봤습니다만 분기점도 많고 가독성도 떨어지기에 유지보수가 어려울거라고 판단했습니다.

아래는 위의 게시글(Post) 엔티티와 일대일 관계를 가지고있는 엔티티입니다.

{: file='PostDetail.java'}

```java
@Entity
@Table(name = "post_detail")
@Getter @Setter @ToString
@NoArgsConstructor @AllArgsConstructor @Builder
public class PostDetail {
    @Id
    @Column(name = "post_idx")
    private Long idx;

    @MapsId
    @OneToOne
    @JoinColumn(name = "idx")
    private Post post;

    @Column(name = "contents", columnDefinition = "MEDIUMTEXT")
    private String contents;
}
```

### MapsId

JPA에서는 테이블간 연관 관계를 맺을 때 기본적으로 두 테이블 중 하나에 외래키를 생성합니다. `MapsId`는 `기본키를 외래키로 사용할거니까 외래키 따로 만들지마!`라고 명시함으로써 불필요한 컬럼 생성을 막아줍니다.

### Cascade

`@OneToOne(mappedBy = "post", cascade = CascadeType.ALL, optional = false, fetch = FetchType.LAZY)` Post와 PostDetail 엔티티는 같은 기본키 값을 사용하며 insert와 delete가 함께 일어나기 때문에 `cascade` 옵션을 사용했습니다. `cascade`는 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화시켜주는 `영속성 전이(transitive persistence)`를 제공해줍니다. 즉, `부모 엔티티를 저장할 때 자식 엔티티도 함께 저장`할 수 있게됩니다.

아래는 Post의 CUD 서비스레이어입니다.

{: file='PostService.java'}

```java
@Service
@RequiredArgsConstructor
public class PostService {
    final private PostRepository postRepository;

    @Transactional
    public void create(PostDto dto) {
        Post entity = PostMapper.MAPPER.toEntity(dto);
        entity.setRegDate(LocalDateTime.now());
        entity.setCommentCount(0);
        entity.setPostDetail(PostDetail.builder()
                .post(entity)
                .contents(dto.getContents())
                .build());
        postRepository.save(entity);
    }

    @Transactional
    public void update(Long idx, PostDto dto) {
        Post entity = getEntity(idx, dto.getUserIdx(), true);
        entity.setUpdatedDate(LocalDateTime.now());
        entity.setStatusCode(dto.getStatusCode());
        entity.setTitle(dto.getTitle());
        entity.getPostDetail().setContents(dto.getContents());
        postRepository.save(entity);
    }

    @Transactional
    public void delete(Long idx, Long userIdx) {
        Post entity = getEntity(idx, userIdx, true);
        entity.setStatusCode(StatusCode.Deleted);
        postRepository.save(entity);
    }
}
```

cascade를 설정해줬기 때문에 PostRepository를 save해준 것만으로도 PostDetail까지 변경사항을 적용할 수 있습니다. cascade는 위에서 사용한 `CascadeType.ALL` 이외에도 `PERSIST`, `MERGE`, `REMOVE`, `REFRESH`, `DETACH`로 원하는 동작만 선택하여 설정하는 것도 가능합니다.

### 일대일 관계에서의 Lazy 로딩 문제

JPA에서 제가 처음 일대일 관계를 구성했을 때 가장 당황스러웠던 것은 Lazy 로딩 전략이 적용되지않는 것이었습니다. 정확히는, 이 현상은 `주인이 아닌 엔티티(외래키가 없는 테이블)를 가져올 때 발생`합니다. 그 이유는 JPA의 객체 참조가 `프록시` 기반으로 동작하기 때문입니다. 연관 관계가 있는 객체는 참조시 NULL이 아닌 객체를 반환합니다. 일대다나 다대일 연관 관계에서 이런 현상이 발생하지 않았던 이유는 `일대다는 데이터가 없더라도 비어있는 배열 객체로 표현`할 수 있으며 `다대일은 참조객체의 존재가 보장`되기 때문입니다. Lazy 로딩이 안 된다는 건 불필요한 쿼리가 발생하여 성능 저하를 일으킬 수 있기 때문에 이를 해결해야했습니다.

자주 참조하는 엔티티에 외래키를 두어 엔티티간 주종관계를 바꾸는 방법도 고려해봤습니다만 막상 적용해보니 비지니스 모델과 너무 동떨어지게되고 확장성이 좋지않았습니다. 고민하던 중 `프록시의 메커니즘 때문에 발생하는 문제니까 프록시를 속여볼까?`라는 생각을 해봤습니다. `optional = false` 옵션을 추가해 '참조하는 객체는 nullable하지않으니까 Lazy 로딩해도 돼~'라고 프록시에게 알려주는 것이었습니다. 이는 정말로 참조객체가 nullable하지않다면 정석적인 사용방법이지만... 문제는 그렇지 않을 때 발생합니다. X(

nullable하더라도 참조객체를 사용하지만않는다면 문제없습니다. 하지만 참조객체를 조회하려는 순간 프록시가 비명을 지르며 런타임 에러가 발생하게 됩니다. 'nullable 참조객체라며 왜 데이터가 없는거야!!'하면서 말이죠... 그래서 만약 참조객체의 조회까지 필요한 경우라면 `Fetch Join 한 엔티티를 조회`한 후 `참조객체 사용시 null check`를 해줘야합니다.

## 다대다(ManyToMany)

### @ManyToMany의 구조적 문제

JPA로 다대다 관계를 구성하는건 추천하지않습니다. 우리는 관계형 데이터베이스에서 다대다 관계를 형성할 때 3개의 테이블을 사용합니다. 만약 `@ManyToMany`를 사용하여 2개의 엔티티 객체로 표현하면 연결 테이블에는 접근할 수 없게됩니다. 이것이 위에서 언급한 `다대다 관계에서의 구조적 문제`입니다. 따라서 객체로 표현할 때 역시 3개로 표현하는 것이 좋습니다. 이는 테이블 접근성이 원활해지고 확장성도 용이하게 해주기 때문이죠. 따라서 `@ManyToMany`를 사용하기보단 `연결 테이블을 엔티티로 만들고 일대다&다대일 관계로 연결하는 것`을 추천드립니다.

아래는 토이프로젝트의 유저(User), 역할(Role)과 연결 테이블(UserRole)에 대한 엔티티입니다.

{: file='BoardUser.java'}

```java
@Entity
@Table(name = "user")
@Getter @Setter @ToString
@NoArgsConstructor @AllArgsConstructor @Builder
public class BoardUser {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "idx")
    private Long idx;

    @Column(name = "email", unique = true, nullable = false)
    private String email;

    @Column(name = "password", nullable = false)
    private String password;

    @Column(name = "name", nullable = false)
    private String name;

    @Column(name = "latest_login_date")
    private LocalDateTime latestLoginDate;

    @Column(name = "reg_date")
    private LocalDateTime regDate;

    @OneToMany(mappedBy = "user")
    private Set<RefreshToken> refreshTokens;

    @Column(name = "status_code", nullable = false)
    @Convert(converter = UserStatusConverter.class)
    @Comment("1 = 정상, 2 = 정지, 3 = 비인증, 4 = 탈퇴, 5= 휴면계정, 9 = 삭제된 계정")
    private UserStatus statusCode;

    @OneToMany(mappedBy = "boardUser", cascade = CascadeType.ALL)
    private List<UserRole> userRoles;

    public void addUserRole(UserRoleType role) {
        this.userRoles.add(UserRole.builder()
                .userIdx(this.idx)
                .role(role)
                .build());
    }

    public void clearUserRole() {
        this.userRoles.clear();
    }
}
```

{: file='BoardRole.java'}

```java
@Entity
@Table(name = "role")
@Getter @Setter @ToString
@NoArgsConstructor @AllArgsConstructor @Builder
public class BoardRole {
    @Id
    @Column(name = "idx")
    @Convert(converter = UserRoleTypeConverter.class)
    private UserRoleType role;

    @Column(name = "description")
    private String description;
}
```

{: file='UserRole.java'}

```java
@Entity
@Table(name = "user_role")
@Getter @Setter @ToString
@NoArgsConstructor @AllArgsConstructor @Builder
@IdClass(UserRoleKey.class)
public class UserRole implements Serializable, Persistable<UserRoleKey> {
    @Id
    @Column(name = "user_idx")
    private Long userIdx;

    @Id
    @Column(name = "role_idx")
    @Convert(converter = UserRoleTypeConverter.class)
    private UserRoleType role;

    @ManyToOne
    @MapsId(value = "userIdx")
    @JoinColumn(name = "user_idx", insertable = false, updatable = false)
    private BoardUser boardUser;

    @ManyToOne
    @MapsId(value = "role")
    @JoinColumn(name = "role_idx", insertable = false, updatable = false)
    private BoardRole boardRole;

    @Override
    public UserRoleKey getId() {
        return UserRoleKey.builder()
                .userIdx(this.userIdx)
                .role(this.role)
                .build();
    }

    @Override
    public boolean isNew() {
        return true;
    }
}
```

### 복합키 매핑하기

연결 테이블에 대한 엔티티를 구성하려면 복합키를 사용해야합니다. JPA에서 복합키를 구성하는 방법은 크게 `@IdClass`와 `@EmbeddedId` 두 가지가 있습니다. 어떤 것을 사용하든 상관없기 때문에 취향에 맞는걸로 일관성 있게 사용하시면 됩니다. 저는 `@IdClass`를 사용했습니다. :q

복합 키에서 `@GenerateValue`는 사용할 수 없습니다. 복합 키를 가진 엔티티를 구성하려면 `식별자 클래스`가 필요한데 위의 `UserRoleKey.class`가 바로 식별자 클래스입니다!

{: file='UserRoleKey.java'}

```java
@Getter @Setter @ToString
@NoArgsConstructor @AllArgsConstructor @Builder
public class UserRoleKey implements Serializable {
    private Long userIdx;
    private UserRoleType role;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        UserRoleKey that = (UserRoleKey) o;
        return userIdx.equals(that.userIdx) && role == that.role;
    }

    @Override
    public int hashCode() {
        return Objects.hash(userIdx, role);
    }
}
```

보시다싶이 식별자 클래스는 `equals`, `hashCode`를 구현해야합니다. 이건 Intellij 등 IDE에서 자동으로 생성할 수 있습니다.

---

여기까지가 JPA를 만져보기 위한 최소한의 엔티티 설정 방법들입니다. :D

앞으로 일대다 관계의 참조객체를 Fetch해서 페이징할 때 주의할 점과 n+1 문제 등을 다뤄보겠습니다~
