---
title: Google Cloud SDK 설치하기
author: kjb4494
date: 2021-02-15 15:00:00 +0900
categories: [Cloud Platform, GCP]
tags: [cloud, gcp]
---

### 설치 방법

> - [_공식 문서 참고_](https://cloud.google.com/sdk/docs/downloads-interactive?hl=ko)

공식 문서만 잘 따라가도 된다.
Windows 10 환경 기준으로 정리

1. 파워쉘을 이용해서 설치 프로그램 실행
   ```shell
   (New-Object Net.WebClient).DownloadFile("https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe", "$env:Temp/GoogleCloudSDKInstaller.exe")
   & $env:Temp/GoogleCloudSDKInstaller.exe
   ```
1. 설치 프로그램으로 설치 진행
1. 설치 완료 후 터미널에서 초기 설정
   ```shell
   gcloud init
   ```

---

### 제거 방법

> - [_공식 문서 참고_](https://cloud.google.com/sdk/docs/uninstall-cloud-sdk?hl=ko)

Windows 10의 경우 Cloud SDK 디렉토리에 있는 uninstaller.exe를 실행한다.
