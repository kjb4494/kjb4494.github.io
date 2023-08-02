---
title: JPA 이야기 04 편 - Count 쿼리 없이 Pageable 사용하기
author: kjb4494
date: 2023-08-01 18:00:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend, jpa]
---

## 서론

JPA를 사용하여 데이터베이스에서 데이터를 가져올 때, Pageable 인터페이스를 사용하면 편리하게 페이지네이션을 구현할 수 있습니다. 하지만, 기본적인 방식을 사용하면 각 페이지 요청마다 count 쿼리가 실행되어 성능 저하가 발생할 수 있습니다.

특히 대용량 데이터를 처리할 때, count 쿼리는 전체 데이터의 수를 계산하기 위해 전체 데이터를 스캔해야 하므로 성능에 큰 영향을 미칠 수 있습니다.

이러한 문제를 해결하기 위해선 limit 문법을 사용할 수 있지만, JPA에서는 기본적으로 제공하지 않습니다. limit를 사용하려면 native SQL을 사용해야 하지만, 이는 데이터베이스 종류에 따라 SQL 문법이 다르기 때문에 데이터베이스 종속적인 코드가 될 수 있습니다. 이러한 문제를 해결하기 위해, 우리는 count 쿼리 없이 limit 문법을 사용하는 방법을 살펴볼 것입니다.

## 핵심 코드

먼저, JpaRepository를 확장한 ExtendedRepository 인터페이스를 생성합니다.

```java
package com.petdiary.domain.rdscore.repository;

import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.data.jpa.repository.JpaRepository;

import java.io.Serializable;
import java.util.List;

public interface ExtendedRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
    /**
     * count 쿼리없이 limit 문법이 필요할 때 사용하기 위한 함수
     */
    List<T> findAllWithoutCount(Specification<T> spec, Pageable pageable);
}
```

다음으로, ExtendedRepository 인터페이스를 구현하는 클래스를 작성합니다.

```java
package com.petdiary.domain.rdscore.repository;

import jakarta.persistence.EntityManager;
import jakarta.persistence.TypedQuery;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.data.jpa.repository.support.JpaEntityInformation;
import org.springframework.data.jpa.repository.support.SimpleJpaRepository;

import java.io.Serializable;
import java.util.List;

public class ExtendedRepositoryImpl<T, ID extends Serializable> extends SimpleJpaRepository<T, ID> implements ExtendedRepository<T, ID> {
    public ExtendedRepositoryImpl(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
        super(entityInformation, entityManager);
    }

    @Override
    public List<T> findAllWithoutCount(Specification<T> spec, Pageable pageable) {
        TypedQuery<T> query = getQuery(spec, pageable);
        query.setFirstResult((int) pageable.getOffset());
        query.setMaxResults(pageable.getPageSize());
        return query.getResultList();
    }
}
```

## ExtendedRepository 설정하기

`ExtendedRepository`를 사용하려면, 애플리케이션 모듈의 `DataSourceConfig`에 `repositoryBaseClass` 설정이 필요합니다. 이 설정은 `ExtendedRepositoryImpl`를 사용하도록 지시하는 역할을 합니다. 아래 코드에서는 이를 적용한 예시를 확인할 수 있습니다.

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        repositoryBaseClass = ExtendedRepositoryImpl.class,
        entityManagerFactoryRef = "petDiaryMembershipEntityManagerFactory",
        transactionManagerRef = "petDiaryMembershipTransactionManager",
        basePackages = {"com.petdiary.domain.rdspetdiarymembershipdb.repository"}
)
public class DataSourceConfig {
    // 기타 설정은 여기에 추가...
}
```

## ExtendedRepository 사용하기

이제 도메인 모듈의 Repository에서 `ExtendedRepository`를 상속받아 사용할 수 있습니다. 다음은 `MemberRepository`에서 사용한 예제 코드입니다.

```java
package com.petdiary.domain.rdspetdiarymembershipdb.repository;

import com.petdiary.domain.rdscore.repository.ExtendedRepository;
import com.petdiary.domain.rdspetdiarymembershipdb.domain.Member;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;

public interface MemberRepository extends ExtendedRepository<Member, Long>, JpaSpecificationExecutor<Member> {
}
```

이제 서비스 레이어에서 `findAllWithoutCount` 메서드를 사용하여 count 쿼리 없이 Pageable를 사용하는 예를 보겠습니다.

```java
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    // Constructor injection
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    public List<Member> getMembers(Specification<Member> spec, Pageable pageable) {
        return memberRepository.findAllWithoutCount(spec, pageable);
    }
}
```

## 결론

이 글을 통해 JPA에서 count 쿼리 없이 Pageable을 사용하는 방법을 배웠습니다. 기존에는 JPA에서 count 쿼리를 제외하고 limit를 사용하기 위해선 native SQL이 필요했지만, 우리는 ExtendedRepository를 통해 이를 해결했습니다. 이 방법을 통해, 데이터베이스에 종속적이지 않은 코드를 유지하면서도 limit를 사용할 수 있게 되어, 성능 최적화에 큰 도움이 됩니다.
