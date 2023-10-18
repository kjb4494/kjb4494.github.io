---
title: 도커를 이용한 로컬 데이터베이스 구축
author: kjb4494
date: 2022-09-28 02:30:00 +0900
categories: [기타, 개발환경]
tags: [docker, mysql]
---

## 로컬 데이터베이스가 필요했던 이유

이전에는 AWS 에서 개발용 데이터베이스만 만들어서 사용했었습니다. 하지만 모델을 설계할 때 다수의 개발자가 붙어서 테이블 구조를 수정하거나 테스트용 데이터를 다른 사람이 건드리는 등 개발에 방해되는 상황이 종종 발생했었습니다.

과거 Mybatis 를 사용할 때는 로컬 데이터베이스 환경을 구축하기 위해선 클라우드의 데이터베이스와 똑같은 구조를 갖추게 하기 위해 DDL 을 떠와 스크립트를 실행해야했고 테이블 구조에 수정사항이 있을 때마다 수정하는데 다소 불편함을 느꼈었습니다.

하지만 JPA 에는 ddl-auto 기능이 있어 엔티티를 설계하는 것만으로도 테이블을 자동으로 삭제 & 생성할 수 있기 때문에 로컬 데이터베이스를 간단하게 구축할 수 있게됐습니다.

## 도커를 사용하는 이유

로컬 환경에 데이터베이스를 구축하려면 일반적인 방법으로 호스트에 직접 데이터베이스 서버를 설치해도 됩니다. 하지만 이 방법은 도커를 이용해 설치하는 것보다 관리하기 매우 귀찮았습니다. 도커를 이용하면 서버의 생성, 삭제를 명령어 한 줄로 수행할 수 있으니까요. 게다가 마스터-슬레이브 구조 역시 docker-compose 를 이용하면 간단하게 구축할 수 있습니다.

## Mysql 서버 구축하기

### Mysql 도커 컨테이너 생성

```console
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=P@ssw0rd --name test-mysql-db mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

- -d

  컨테이너가 detached 모드로 실행됩니다. 즉, 백그라운드로 실행됩니다.

- -p 3306:3306

  `호스트의 포트:도커 컨테이너의 포트`입니다.

- -e MYSQL_ROOT_PASSWORD

  환경변수로 관리자 비밀번호를 설정합니다.

- --name

  도커 컨테이너의 이름을 명시합니다.

겨우 한 줄의 명령어로 간단하게 Mysql 서버가 생성되었습니다!

### 유저, 데이터베이스 생성 및 권한 부여

이 이후로는 필요한 유저, 데이터베이스 및 권한 설정을 해주면됩니다.

```sql
use mysql;
create user 'testuser'@'%' identified by '1q2w3e4r5t@#';
create database testdb;
grant all privileges on testdb.* to 'testuser'@'%';
flush privileges;
```
