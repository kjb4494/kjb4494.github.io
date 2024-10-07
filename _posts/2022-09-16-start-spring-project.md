---
title: Java Spring Toy Project 01 편 - 프로젝트 시작하기
author: kjb4494
date: 2022-09-16 15:06:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend, toy-project]
---

## 토이 프로젝트 시작 계기

처음에는 작았던 프로젝트의 규모가 점점 커지기 시작하면서 프로젝트 구조에서 불편함을 느끼기 시작했습니다.

그 이유는 바로 공통(Common) 모듈... cms 와 web, mobile api 를 만들기 위해서 Entity, Repository, 서비스 레이어 등 공통으로 사용할만한 것들을 Common 모듈에 전부 때려박은 것이 불행의 시작이었습니다 ㅠㅠ...

무엇보다도 가장 불편했던 것은 데이터베이스 의존성이었습니다. 멀티데이터베이스를 사용하는 환경에서 원하는 데이터베이스만 선택적으로 등록하기 까다로웠거든요... 조건부 애노테이션을 달아서 해결했지만 해당 데이터베이스를 사용하는 모든 서비스 레이어에 애노테이션을 달아줘야하는게 코드 보기에도 지저분했습니다. 그 밖에도 Common 모듈이 요구하는 프로퍼티 값에 의한 의존성 스트레스는 나날이 커져갔습니다.

그러다가 우아한형제들 기술블로그에서 [_멀티모듈 설계 이야기_](https://techblog.woowahan.com/2637/)라는 글을 발견했습니다. `공통(Common) 모듈의 저주`... 지금 프로젝트가 딱 이 상황이더군요 ㅠㅠ... 저자분께서 멀티 모듈 설계 노하우를 정말 깔끔하게 설명해주신 덕분에 이 글을 참고해서 `잘 설계된 작은 프로젝트`를 만들어보자는 생각을 하게됐습니다. :)

## 프로젝트 시작하기

위 글을 참고하여 크게 5개 모듈 종류로 나누었습니다.

- servers (apllication)
- clients (in system available)
- domain (system domain)
- core (system core)
- modules (independently available)

차근차근 진행하기 위해서 일단 [_spring initializr_](https://start.spring.io/) 에서 프로젝트를 대충 생성한 후 servers 에 애플리케이션을 추가해보겠습니다.

개발 환경은 다음과 같습니다.

- java 8
- spring boot 2.7.2
- gradle 7.5

먼저 프로젝트 전체에서 사용할 의존성을 설정해줬습니다.

```gradle
buildscript {
	ext {
		springBootVersion = '2.7.2'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath "org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}"
		classpath "io.spring.gradle:dependency-management-plugin:1.0.13.RELEASE"
	}
}

subprojects {
	apply plugin: 'java'
	apply plugin: 'java-library'
	apply plugin: 'org.springframework.boot'
	apply plugin: 'io.spring.dependency-management'

	group = 'com.smctlab'
	version = '0.0.1-SNAPSHOT'
	sourceCompatibility = '1.8'
	targetCompatibility = '1.8'
	compileJava.options.encoding = 'UTF-8'

	repositories {
		mavenCentral()
	}

	dependencies {
		implementation 'org.springframework.boot:spring-boot-starter-web:2.7.2'
		compileOnly 'org.projectlombok:lombok'
		developmentOnly 'org.springframework.boot:spring-boot-devtools'
		annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
		annotationProcessor 'org.projectlombok:lombok'
		testImplementation 'org.springframework.boot:spring-boot-starter-test:2.7.2'
		testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.2'
		testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.2'
	}

	test {
		useJUnitPlatform()
	}
}
```

{: file='build.gradle'}

Intellij 모듈 추가 기능을 이용해 servers.api-server 모듈을 생성해주면 `settings.gradle` 에 다음과 같이 추가됩니다.

```gradle
rootProject.name = 'toyboard'
include 'servers:api-server'
findProject(':servers:api-server')?.name = 'api-server'
```

{: file='settings.gradle'}

그 다음엔 애플리케이션에 필요한 의존성을 추가해줬습니다.

```gradle
bootJar {
    enabled = true
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-security:2.7.2'
    // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-validation
    implementation 'org.springframework.boot:spring-boot-starter-validation:2.7.2'
    // https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt
    implementation 'io.jsonwebtoken:jjwt:0.9.1'
    // https://mvnrepository.com/artifact/io.springfox/springfox-boot-starter
    implementation 'io.springfox:springfox-boot-starter:3.0.0'
    testImplementation 'org.springframework.security:spring-security-test:5.7.2'
}
```

{: file='servers/api-server/build.gradle'}

위에서부터 순서대로 각각 기능은 이렇습니다.

- spring security
- validation
- jwt
- swagger

실행 후 `http://localhost:8080/` 에 접속했을 때 `Whitelabel Error Page` 가 나오면 준비는 끝났습니다~ XD

## Tomcat -> Undertow

이제 `Tomcat` 을 `Undertow` 로 교체해보겠습니다.

```gradle
// Use undertow instead tomcat.
// https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-undertow
implementation 'org.springframework.boot:spring-boot-starter-undertow:2.7.2'
implementation ('org.springframework.boot:spring-boot-starter-web:2.7.2') {
  exclude module: 'spring-boot-starter-tomcat'
}
```

{: file='build.gradle'}

교체는 매우 간단하게, `Tomcat` 을 exclude 시켜주고 `Undertow` 로 대체해주면 됩니다.

```text
Failed to start bean 'documentationPluginsBootstrapper';
```

만약 해당 에러가 발생한다면 swagger 때문에 생긴 에러입니다. 프로퍼티에 아래 설정을 추가해서 해결해줍니다.

```properties
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```

{: file='application.properties'}
