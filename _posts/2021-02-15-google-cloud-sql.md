---
title: Google Cloud SQL
author: kjb4494
date: 2021-02-15 16:00:00 +0900
categories: [Cloud Platform, GCP]
tags: [cloud, gcp]
---

### 들어가기 전...

> - [Google Cloud SDK 설치하기]({% post_url 2021-02-15-google-cloud-sdk %})

Cloud SDK가 있으면 인증이 편하다.

### 프록시 서버 열기

> - [_공식 문서 참고_](https://cloud.google.com/sql/docs/mysql/connect-admin-proxy?hl=ko)

로컬에서 Cloud SQL의 데이터베이스에 접근하기 위해 프록시 서버를 열어준다.

- [_Cloud SQL 프록시 설치_](https://dl.google.com/cloudsql/cloud_sql_proxy_x64.exe?hl=ko)
  ![사진](/assets/img/posts/legacy/gcp/google-cloud-sql-001.png)
  파일명을 cloud_sql_proxy.exe로 바꾸고 Cloud SDK가 설치된 경로의 bin 폴더에 넣으면 따로 환경변수를 설정하지 않아도 터미널에서 사용할 수 있다. (귀찮)

- 프록시 서버 열기
  ```shell
  cloud_sql_proxy -instances=<INSTANCE_CONNECTION_NAME>=tcp:3306
  ```
  ![사진](/assets/img/posts/legacy/gcp/google-cloud-sql-002.png)
  INSTANCE_CONNECTION_NAME은 여기 적혀 있다.
