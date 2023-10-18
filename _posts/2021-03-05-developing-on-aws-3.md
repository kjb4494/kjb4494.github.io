---
title: Developing on AWS 3일차
author: kjb4494
date: 2021-03-05 15:00:00 +0900
categories: [Cloud Platform, AWS]
tags: [cloud, aws]
---

### Amazon SQS

Simple Queue Service

- 특징
  - HTTP 프로토콜로 통신
  - 메시지 크기는 최대 256KB (meta data까지 모두 포함해서)
- Standard
  - 중복 데이터가 있더라도 유실 X
  - 트랜잭션 무한
  - 메시지 순서가 보장되지 않음
- FIFO
  - 메시지 순서가 유지됨
  - 메시지는 한 번만 수신됨
  - 트랜잭션 최대 300

### Amazon SNS

Simple Notification Service

- SQS와의 차이점

### Amazon ElastiCache

- 데이터 캐싱을 고려해야 하는 경우

  - 느린데도 비싼 쿼리를 통해서만 얻을 수 있는 데이터
  - 비교적 정적이며 자주 액세스하는 데이터, 예: 소셜 미디어 웹 사이트의 프로필
  - 날씨 데이터와 같이 일정 시간이 지나면 효력이 상실될 수 있는 정보

- Memcached vs Redis
  - Memcached
    - 멀티스레딩
    - 유지관리용이
    - Auto Discovery를 통해 손쉽게 수평 확장
    - 단일 AZ
  - Redis
    - 데이터 구조 지원
    - 지속성
    - 원자성 작업
    - pub/sub 메시징
    - 읽기 전용 복제본 및 장애 조치
    - 클러스트 모드 또는 샤딩된 클러스터
    - 다중 AZ
