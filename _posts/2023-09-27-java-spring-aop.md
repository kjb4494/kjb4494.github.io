---
title: Spring AOP(Aspect Oriented Programming)
author: kjb4494
date: 2023-09-27 12:00:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend, aop]
---

## 1. AOP란?

AOP는 특정 관점에서 애플리케이션의 코드를 모듈화하는 방법으로, 컴퓨터 프로그래밍의 한 패러다임입니다. 이를 통해 코드의 중복을 줄이고, 각 모듈의 독립성을 높일 수 있습니다. 예를 들어, 로깅이나 트랜잭션 관리와 같은 공통적인 기능은 여러 클래스와 메소드에 걸쳐 사용될 수 있습니다. 이러한 기능을 Aspect로 정의하고 프로그램 코드에서 분리할 수 있습니다.

## 2. AOP 사용 예제

### Transactional Aspect

아래의 Java 코드는 `@Transactional` 어노테이션이 붙은 메서드를 추적하여 로깅하는 Aspect의 예제입니다.

```java
@Aspect
@Component
@RequiredArgsConstructor
@Slf4j
@Profile("!prod")
public class TransactionalAspect {
    private final ApplicationContext applicationContext;

    @Before("@annotation(org.springframework.transaction.annotation.Transactional)")
    public void beforeTransactional(JoinPoint joinPoint) {
        String className = joinPoint.getTarget().getClass().getSimpleName();
        String methodName = joinPoint.getSignature().getName();
        String transactionManagerName = determineTransactionManagerName(joinPoint);
        boolean isReadOnly = determineIsReadOnly(joinPoint);
        Object bean = applicationContext.getBean(transactionManagerName);
        if(bean instanceof PlatformTransactionManager) {
            log.debug("[Transactional Info] Class: " + className + " | Method: " + methodName
                    + " | Using TransactionManager: " + transactionManagerName
                    + ", readOnly: " + isReadOnly);
        }
    }

    private boolean determineIsReadOnly(JoinPoint joinPoint) {
        // @Transactional 어노테이션에서 readOnly 속성을 가져옴
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        Transactional transactional = method.getAnnotation(Transactional.class);

        if (transactional == null) {
            return false;
        }

        return transactional.readOnly();
    }

    private String determineTransactionManagerName(JoinPoint joinPoint) {
        // @Transactional 어노테이션에서 transactionManager 속성을 가져옴
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        Transactional transactional = method.getAnnotation(Transactional.class);

        if (transactional == null) {
            log.debug("[Transactional Info] Transactional is null.");
            return DomainCoreConstants.DEFAULT_TRANSACTION_MANAGER;  // 기본값 반환
        }

        String transactionManagerName = transactional.transactionManager();

        // transactionManager 속성이 빈 문자열인 경우 value 속성을 확인
        if (transactionManagerName.isEmpty()) {
            transactionManagerName = transactional.value();
        }

        // 그래도 transactionManagerName가 빈 문자열인 경우 기본값을 반환
        if (transactionManagerName.isEmpty()) {
            log.debug("[Transactional Info] Transactional name is empty.");
            return DomainCoreConstants.DEFAULT_TRANSACTION_MANAGER;
        }
        else {
            return transactionManagerName;
        }
    }
}

```

이 `TransactionalAspect` 클래스는 `@Before` 어노테이션을 사용하여 `@Transactional` 어노테이션이 붙은 메서드를 실행하기 전에 호출됩니다. 이를 통해 해당 메서드의 정보를 로깅할 수 있습니다.

### Redis PetDiary Aspect

다음은 Redis Repository 메서드 호출을 로깅하는 Aspect의 예제입니다.

```java
@Aspect
@Component
@RequiredArgsConstructor
@Slf4j
@Profile("!prod")
public class RedisPetDiaryAspect {
    @Around("execution(* com.petdiary.domain.redispetdiary.repository..*(..))")
    public Object logPetDiaryRedisRepositoryMethods(ProceedingJoinPoint joinPoint) throws Throwable {
        String arguments = Arrays.stream(joinPoint.getArgs())
                .map(arg -> Optional.ofNullable(arg).map(Object::toString).orElse("null"))
                .collect(Collectors.joining(", "));
        log.debug("[Redis Repository Call] Entered {} with argument[s] = {}", joinPoint.getSignature().toShortString(), arguments);
        try {
            Object result = joinPoint.proceed();
            log.debug("[Redis Repository Call] Exited {} with result = {}", joinPoint.getSignature().toShortString(), result.toString());
            return result;
        } catch (IllegalArgumentException e) {
            log.debug("[Redis Repository Call] Illegal argument: {} in {}", arguments, joinPoint.getSignature().toShortString());
            throw e;
        }
    }
}
```

`RedisPetDiaryAspect` 클래스는 `@Around` 어노테이션을 사용하여 Redis Repository의 메서드가 호출될 때 실행됩니다. 이를 통해 메서드의 호출 정보를 로깅할 수 있습니다.

## 3. 정리

AOP는 코드의 모듈성을 높이는 강력한 도구입니다. AOP를 사용하면 중복 코드를 제거하고 코드의 관리를 더 효과적으로 할 수 있습니다. 이는 grafana와 prometheus를 이용한 모니터링에 대해 다룰 때 한 번 더 등장할 예정입니다. 😅
