---
title: Python Anaconda 설치하기
author: kjb4494
date: 2021-02-11 15:00:00 +0900
categories: [Python]
tags: [python, anaconda]
---

### Windows10에 설치하기

- [_Miniconda 설치하기_](https://docs.conda.io/en/latest/miniconda.html)
  ![사진](/assets/img/posts/legacy/python/install-conda-001.png){: style="min-width: 100%"}
  환경 변수에 추가하기 체크

- 설치 후 파워쉘에서 사용할 수 있도록 초기화
  ```shell
  conda init powershell
  ```
  만약 내 문서 경로에 한글이 있을 경우,
  ![사진](/assets/img/posts/legacy/python/install-conda-002.png){: style="min-width: 100%"}
  ![사진](/assets/img/posts/legacy/python/install-conda-003.png){: style="min-width: 100%"}
  이런식으로 이상한 경로로 폴더를 만들어 프로필을 생성할 때가 있다.
  이럴 경우엔 저 이상한 경로로 되어 있는 폴더 안의 내용물을 직접 내문서로 옮겨준다.
  가장 좋은 방법은 그냥 윈도우 설치할 때 마이크로소프트 계정 연동하지 말고 로컬 계정으로 만들었다가 나중에 연결하는 식으로 계정 만드는게 좋은듯...(위 사진은 옛날에 찍어놓은 스크린샷 ㅎㅎ...)

### Mac에 설치하기

- [_miniconda 설치하기_](https://docs.conda.io/en/latest/miniconda.html)
  간편하게 [_Miniconda3 MacOSX 64-bit pkg_](https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.pkg)로 설치했다. 전부 기본값으로 설치하면 '/opt/miniconda3' 경로에 설치된다.
- init
  ```shell
  /opt/miniconda3/bin/conda init zsh
  ```
  자신이 사용하고 있는 쉘로 init 한 뒤, 터미널을 껏다 켜준다.

### 많이 쓰는 명령어 모음

- 가상환경 조회
  ```shell
  conda env list
  ```
- 가상환경 만들기
  ```shell
  conda create -n <new_env_name> python=<python_version>
  ```
- 가상환경 실행
  ```shell
  activate <env_name>
  ```
- 가상환경 종료
  ```shell
  deactivate
  ```
- conda 가상환경 내 package 목록/버전 확인
  ```shell
  conda list -n <env_name>
  ```
- 특정 conda 가상환경을 복제해 새로운 가상환경을 생성
  ```shell
  conda create --name <new_env_name> --clone <old_env_name>
  ```
- 가상환경 제거
  ```shell
  conda remove --name <env_name> --all
  ```

---

### 잡담

wsl2 ubuntu 환경에서 gpu 리소스를 사용해보려고 windows10 preview 버전을 설치했다가 몇 달 지나지않아 잦은 버그에 이기지 못하고 밀어버리고 말았다 ㅠㅠ...

stable 버전 os로 재설치할 겸, 완전 초기화하고 개발환경을 다시 구성하려니 귀찮음이 몰려온다.

그래도 컴퓨터가 깨끗해졌다는 생각에 만족 :) ㅋ
