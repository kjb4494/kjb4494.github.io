---
title: JPA 이야기 03 편 - 읽기 전용 엔티티 만들기
author: kjb4494
date: 2023-08-01 12:00:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend, jpa]
---

이 글에서는 JPA(Jakarta Persistence API)에서 읽기 전용 엔티티를 만드는 방법에 대해 다뤄보겠습니다.

프로젝트를 진행하다 보면 가끔 특정 엔티티를 읽기 전용으로 설정해야 하는 상황이 발생합니다. 예를 들어, 일정 기간 동안 변경이 불가능한 보증서나 레코드 등의 엔티티를 구현해야 할 때, 혹은 엔티티가 수정되면 안 되는 비즈니스 로직을 강제하려 할 때 등이 있을 수 있습니다.

이런 상황에서 JPA의 엔티티 생명주기 이벤트를 활용하면 읽기 전용 엔티티를 쉽게 구현할 수 있습니다.

```java
package com.petdiary.domain.rdscore.listener;

import jakarta.persistence.PrePersist;
import jakarta.persistence.PreUpdate;
import jakarta.persistence.PreRemove;

public class PreventAnyUpdate {
    @PrePersist
    void onPrePersist(Object o) {
        throw new IllegalStateException("[PreventAnyUpdate]: JPA is trying to persist an entity of type " + (o == null ? "null" : o.getClass()));
    }

    @PreUpdate
    void onPreUpdate(Object o) {
        throw new IllegalStateException("[PreventAnyUpdate]: JPA is trying to update an entity of type " + (o == null ? "null" : o.getClass()));
    }

    @PreRemove
    void onPreRemove(Object o) {
        throw new IllegalStateException("[PreventAnyUpdate]: JPA is trying to remove an entity of type " + (o == null ? "null" : o.getClass()));
    }
}
```

위의 코드는 JPA의 엔티티 이벤트 리스너로, 엔티티가 저장, 업데이트, 삭제되기 전에 호출되는 메서드들을 정의합니다. 각 메서드에서는 `IllegalStateException`을 던져, 이 리스너가 부착된 엔티티에 대한 모든 쓰기 작업을 금지합니다.

그렇다면 이 리스너를 실제 엔티티에 어떻게 적용할 수 있을까요?

그 방법은 간단합니다.

엔티티 클래스에 `@EntityListeners` 어노테이션을 사용하여 해당 리스너 클래스를 지정하면 됩니다. 예를 들어, 아래와 같이 `User` 엔티티에 리스너를 적용해보겠습니다.

```java
import jakarta.persistence.Entity;
import jakarta.persistence.EntityListeners;

@Entity
@EntityListeners(PreventAnyUpdate.class)
public class User {
    // ... field declarations, getters, setters, etc.
}
```

이렇게 하면 `User` 엔티티는 이제 읽기 전용이 됩니다. `User` 엔티티에 대한 어떤 쓰기 작업도 `IllegalStateException`이 발생하면서 중단되게 되고, 엔티티의 변경을 방지할 수 있습니다.

이러한 접근법은 변경이 허용되지 않는 특정 상황에서 유용하게 사용될 수 있습니다. 하지만 이 기능을 너무 자주 사용한다면 데이터베이스의 데이터를 제대로 활용하지 못할 수 있으므로 주의해야 합니다. 따라서, 항상 프로젝트의 요구 사항과 상황을 고려하여 적절한 전략을 선택하는 것이 중요합니다.
