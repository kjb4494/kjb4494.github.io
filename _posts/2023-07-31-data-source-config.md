---
title: Spring Boot에서 Master-Slave 구조의 멀티 데이터베이스 설정하기
author: kjb4494
date: 2023-07-31 16:00:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend]
---

프로젝트의 기능이 많아지고 사용자가 늘어날수록 트래픽의 증가로 인한 시스템 부하 문제에 직면하게 됩니다. 특히, 데이터베이스는 동시에 많은 요청을 처리해야 하는 중추적인 역할을 하므로 그 부하가 특히 심합니다. 이러한 문제에 대처하기 위해 여러 가지 솔루션이 존재하지만, 그 중 가장 효과적인 방법 중 하나는 Master-Slave 구조를 도입하는 것입니다.

Master-Slave 구조는 이름에서 알 수 있듯이 데이터베이스를 'Master'와 'Slave' 두 개로 나누어 각각 쓰기 작업과 읽기 작업을 담당하게 합니다. 이렇게 함으로써 쓰기와 읽기의 데이터베이스 트래픽을 분산시켜 시스템의 부하를 줄일 수 있습니다.

저도 역시 이런 문제에 직면하여, 서비스의 효율적인 운영을 위해 Master-Slave 구조를 도입하게 되었습니다. 이번 포스트에서는 Spring Boot 환경에서 Master-Slave 구조의 멀티 데이터베이스를 어떻게 설정하는지에 대해 자세히 알아보도록 하겠습니다.

## ReplicationRoutingDataSource 클래스

`ReplicationRoutingDataSource`는 `AbstractRoutingDataSource`를 상속받아 `determineCurrentLookupKey()` 메서드를 오버라이드하는 클래스입니다. 현재 트랜잭션이 읽기 전용인지 확인하여 읽기 전용이면 Slave를, 그렇지 않으면 Master를 리턴합니다.

다음은 `ReplicationRoutingDataSource` 클래스의 예시 코드입니다.

```java
package com.petdiary.domain.rdscore;

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
import org.springframework.transaction.support.TransactionSynchronizationManager;

import javax.sql.DataSource;
import java.util.HashMap;

public class ReplicationRoutingDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return TransactionSynchronizationManager.isCurrentTransactionReadOnly() ? "slave" : "master";
    }

    public void setMasterSlave(DataSource masterDataSource, DataSource slaveDataSource) {
        HashMap<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put("master", masterDataSource);
        dataSourceMap.put("slave", slaveDataSource);
        this.setTargetDataSources(dataSourceMap);
        this.setDefaultTargetDataSource(masterDataSource);
    }
}
```

## DataSourceConfig 클래스

`DataSourceConfig` 클래스에서는 `ReplicationRoutingDataSource`를 설정하고, Master와 Slave의 DataSource를 정의합니다.

아래 코드는 `DataSourceConfig` 클래스의 예시입니다.

```java
package com.petdiary.domain.rdspetdiarymembershipdb.config;

import com.petdiary.domain.rdscore.ReplicationRoutingDataSource;
import com.petdiary.domain.rdspetdiarymembershipdb.properties.PetDiaryMembershipMasterDataSourceProperties;
import com.petdiary.domain.rdspetdiarymembershipdb.properties.PetDiaryMembershipSlaveDataSourceProperties;
import com.zaxxer.hikari.HikariDataSource;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration("petDiaryMembershipDataSourceConfig")
@RequiredArgsConstructor
public class DataSourceConfig {
    private final PetDiaryMembershipMasterDataSourceProperties masterDataSourceProperties;
    private final PetDiaryMembershipSlaveDataSourceProperties slaveDataSourceProperties;

    @Bean("petDiaryMembershipMasterDataSource")
    public DataSource masterDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(masterDataSourceProperties.getJdbcUrl());
        dataSource.setUsername(masterDataSourceProperties.getUsername());
        dataSource.setPassword(masterDataSourceProperties.getPassword());
        dataSource.setDriverClassName(masterDataSourceProperties.getDriverClassName());
        dataSource.setMaximumPoolSize(masterDataSourceProperties.getMaximumPoolSize());
        return dataSource;
    }

    @Bean("petDiaryMembershipSlaveDataSource")
    public DataSource slaveDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(slaveDataSourceProperties.getJdbcUrl());
        dataSource.setUsername(slaveDataSourceProperties.getUsername());
        dataSource.setPassword(slaveDataSourceProperties.getPassword());
        dataSource.setDriverClassName(slaveDataSourceProperties.getDriverClassName());
        dataSource.setMaximumPoolSize(slaveDataSourceProperties.getMaximumPoolSize());
        return dataSource;
    }

    @Bean("petDiaryMembershipRoutingDataSource")
    public DataSource routingDataSource(
            @Qualifier("petDiaryMembershipMasterDataSource") DataSource masterDataSource,
            @Qualifier("petDiaryMembershipSlaveDataSource") DataSource slaveDataSource
    ) {
        ReplicationRoutingDataSource routingDataSource = new ReplicationRoutingDataSource();
        routingDataSource.setMasterSlave(masterDataSource, slaveDataSource);
        return routingDataSource;
    }
}
```

## DataSource Properties 클래스

DataSource 설정을 위한 프로퍼티들을 저장하는 `PetDiaryMembershipMasterDataSourceProperties`와 `PetDiaryMembershipSlaveDataSourceProperties` 클래스입니다.

`@ConfigurationProperties` 어노테이션을 사용하여 application.yml 또는 application.properties 파일의 특정 프로퍼티들을 매핑합니다. 이 예제에서는 "pet-diary-membership.master.datasource"와 "pet-diary-membership.slave.datasource"라는 prefix를 갖는 프로퍼티들을 매핑하였습니다.

다음은 `PetDiaryMembershipMasterDataSourceProperties` 클래스의 예제 코드입니다.

```java
package com.petdiary.domain.rdspetdiarymembershipdb.properties;

import lombok.Getter;
import lombok.Setter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties("pet-diary-membership.master.datasource")
@Getter @Setter
public class PetDiaryMembershipMasterDataSourceProperties {
    private String jdbcUrl;
    private String username;
    private String password;
    private String driverClassName;
    private int maximumPoolSize;
}
```

그리고 다음은 `PetDiaryMembershipSlaveDataSourceProperties` 클래스의 예제 코드입니다.

```java
package com.petdiary.domain.rdspetdiarymembershipdb.properties;

import lombok.Getter;
import lombok.Setter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties("pet-diary-membership.slave.datasource")
@Getter @Setter
public class PetDiaryMembershipSlaveDataSourceProperties {
    private String jdbcUrl;
    private String username;
    private String password;
    private String driverClassName;
    private int maximumPoolSize;
}
```

이렇게 설정된 프로퍼티들은 앞서 설명한 `DataSourceConfig` 클래스에서 각 Master와 Slave DataSource를 설정하는데 사용됩니다.

## Application 모듈 DataSourceConfig 설정

마지막으로 각 애플리케이션 모듈에서 `DataSourceConfig`를 설정합니다.

다음은 애플리케이션 모듈에서의 `DataSourceConfig` 설정 예제 코드입니다.

```java
package com.petdiary.config;

import jakarta.persistence.EntityManagerFactory;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.jdbc.datasource.LazyConnectionDataSourceProxy;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;

@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef = "petDiaryMembershipEntityManagerFactory",
        transactionManagerRef = "petDiaryMembershipTransactionManager",
        basePackages = {"com.petdiary.domain.rdspetdiarymembershipdb.repository"}
)
public class DataSourceConfig {
    @Primary
    @Bean("petDiaryMembershipDataSource")
    public DataSource petdiaryDataSource(@Qualifier("petDiaryMembershipRoutingDataSource") DataSource routingDataSource) {
        return new LazyConnectionDataSourceProxy(routingDataSource);
    }

    @Primary
    @Bean("petDiaryMembershipEntityManagerFactory")
    public LocalContainerEntityManagerFactoryBean EntityManagerFactory(
            EntityManagerFactoryBuilder builder,
            @Qualifier("petDiaryMembershipDataSource") DataSource dataSource
    ) {
        return builder
                .dataSource(dataSource)
                .packages("com.petdiary.domain.rdspetdiarymembershipdb.domain")
                .persistenceUnit("petDiaryMembershipEntityManager")
                .build();
    }

    @Primary
    @Bean("petDiaryMembershipTransactionManager")
    public PlatformTransactionManager transactionManager(
            @Qualifier("petDiaryMembershipEntityManagerFactory") EntityManagerFactory entityManagerFactory
    ) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}
```

## Transaction 설정에 따른 DataSource 선택

Master-Slave 구조를 적용하게 되면, 데이터의 읽기와 쓰기를 어디에서 처리할지 결정해야 합니다. 일반적으로 쓰기 작업은 Master에서, 읽기 작업은 Slave에서 처리하도록 설정합니다.

그렇다면 Spring에서는 어떻게 이를 구현할까요?

바로 `@Transactional` 어노테이션을 사용하면 됩니다. 이 어노테이션은 메소드 또는 클래스에 적용할 수 있으며, 해당 범위에서의 데이터베이스 작업이 하나의 트랜잭션으로 처리되도록 합니다.

`@Transactional`에는 `readOnly`라는 속성이 있는데, 이는 해당 트랜잭션이 읽기 전용인지 아닌지를 지정할 수 있습니다. `readOnly`가 `true`로 설정된 트랜잭션은 읽기 작업만을 수행하며, `false`로 설정된 트랜잭션은 읽기와 쓰기 작업 모두를 수행할 수 있습니다.

이 `readOnly` 설정에 따라서 `ReplicationRoutingDataSource`에서 어떤 DataSource를 사용할지 결정하게 됩니다. `readOnly`가 `true`로 설정된 경우, `ReplicationRoutingDataSource`는 Slave DataSource를 사용하게 됩니다. 반대로, `readOnly`가 `false`로 설정되거나 `@Transactional` 어노테이션이 적용되지 않은 경우, Master DataSource를 사용하게 됩니다.

이렇게 해서 서비스 레이어에서 `@Transactional`의 `readOnly` 설정에 따라 Master나 Slave에서 데이터를 읽거나 쓸지를 결정하게 됩니다.

다음은 `@Transactional`의 `readOnly` 설정을 이용하여 읽기 전용 트랜잭션을 수행하는 예제입니다.

```java
@Service
public class SomeService {

    @Transactional(readOnly = true)
    public SomeData getSomeData() {
        // 데이터 읽기 작업 수행
    }

    @Transactional
    public void updateSomeData(SomeData someData) {
        // 데이터 쓰기 작업 수행
    }
}
```

이상으로, Spring Boot에서 `@Transactional`의 `readOnly` 설정을 이용하여 Master-Slave 구조의 멀티 데이터베이스에서 데이터를 읽거나 쓰는 방법에 대해 알아보았습니다.

Master-Slave 구조의 도입은 트래픽 부하를 분산시켜 서비스의 성능 향상에 크게 기여하며, 읽기와 쓰기 작업의 분리를 통해 시스템의 견고성 또한 높일 수 있습니다. 물론, 이러한 아키텍처를 구성하고 유지하는 데는 추가적인 노력이 필요하며, 잘못 구성된 경우 데이터의 일관성 유지에 문제가 생길 수 있습니다.

따라서 Master-Slave 구조를 도입할 때에는 구조의 장점과 함께 주의해야 할 사항들에 대해 충분히 이해하고 적용하는 것이 중요합니다. 이를 바탕으로 트래픽 분산과 서비스의 안정성 향상 등의 이점을 최대한 활용하면서도, 시스템의 복잡성과 데이터 일관성 문제 등의 부작용을 최소화하는 방향으로 구성하시기 바랍니다.
