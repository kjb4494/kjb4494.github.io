---
title: Docker로 쥬피터 노트북 사용하기
author: kjb4494
date: 2021-03-10 13:00:00 +0900
categories: [Python]
tags: [python, jupyter-notebook]
---

### 쥬피터 서버 열기

- [_jupyter docker images_](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html)
  자신에게 맞는 docker image를 선택한다.

  ![사진](/assets/img/posts/legacy/ml/jupyter-notebook-001.png)

  내가 사용할 도커 이미지는 jupyter/tensorflow-notebook 이다.

  ```text
  jupyter/tensorflow-notebook includes popular Python deep learning libraries.

  - Everything in jupyter/scipy-notebook and its ancestor images
  - tensorflow and keras machine learning libraries
  ```

- 도커 이미지 가져오기
  ```shell
  docker pull jupyter/tensorflow-notebook
  ```
  ```text
  Using default tag: latest
  latest: Pulling from jupyter/tensorflow-notebook
  83ee3a23efb7: Pull complete
  db98fc6f11f0: Pull complete
  f611acd52c6c: Pull complete
  c4098b6705f2: Pull complete
  b3847c167dd3: Pull complete
  36d64a2901de: Pull complete
  4f4fb700ef54: Pull complete
  04bf1ef33fda: Pull complete
  ecb7e35b277c: Pull complete
  f8af53310761: Pull complete
  34f3c0672f1f: Pull complete
  d82d62fd6b02: Pull complete
  0d193fb595d1: Pull complete
  ddb5b9120f01: Pull complete
  69197629a5fe: Pull complete
  0f5006a719e0: Pull complete
  84b0f8e14606: Pull complete
  1c481b305315: Pull complete
  06e58f5baf79: Pull complete
  74f2ea015222: Pull complete
  dbee37cea197: Pull complete
  d7ff9ed42b78: Pull complete
  c003574f4cc3: Pull complete
  Digest: sha256:8a906d91d86ef91aef870cc1a117ed7f5ba7ef238d905d35abdc7f5dd19a07fb
  Status: Downloaded newer image for jupyter/tensorflow-notebook:latest
  docker.io/jupyter/tensorflow-notebook:latest
  ```
- 도커 컨테이너 실행하기

  ```shell
  docker run -p 8888:8888 --name jupyter-notebook jupyter/tensorflow-notebook
  ```

  ```text
  .
  .
  .
  [I 05:41:33.253 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
  [C 05:41:33.257 NotebookApp]

      To access the notebook, open this file in a browser:
          file:///home/jovyan/.local/share/jupyter/runtime/nbserver-8-open.html
      Or copy and paste one of these URLs:
          http://cdd2e246459e:8888/?token=3a2157024f244ee2a21f0f27bb0b1185f3d5a942dacbfad0
      or http://127.0.0.1:8888/?token=3a2157024f244ee2a21f0f27bb0b1185f3d5a942dacbfad0
  ```

  쥬피터 서버가 열리면 콘솔에 출력된 토큰값을 이용해서 로그인할 수 있다. 하지만 다음 로그인을 위해서 패스워드를 설정해주자.

  ![사진](/assets/img/posts/legacy/ml/jupyter-notebook-002.png)

  여기까지 했으면 콘솔창에서 ctrl+c를 눌러 빠져나와도 된다.

### 쥬피터 노트북 사용해보기

![사진](/assets/img/posts/legacy/ml/jupyter-notebook-003.png)

- 서버 닫기
  ```shell
  docker stop jupyter-notebook
  ```
  서버를 닫을 때는 위 명령을 사용해도 되고 브라우저로 접속해서 quit 버튼을 눌러도 된다.
- 컨테이너 상태 확인
  ```shell
  docker ps -a
  ```
  ```text
  CONTAINER ID   IMAGE                         COMMAND                  CREATED          STATUS                   PORTS                    NAMES
  cdd2e246459e   jupyter/tensorflow-notebook   "tini -g -- start-no…"   24 minutes ago   Up 3 minutes             0.0.0.0:8888->8888/tcp   jupyter-notebook
  ```
  STATUS가 Up이면 서버가 열려있는 상태이며 Exited면 닫혀있는 상태이다.
- 서버 다시 열기

  ```shell
  docker start jupyter-notebook
  ```

  ![사진](/assets/img/posts/legacy/ml/jupyter-notebook-004.png)

  서버를 다시 열었을 때 이전 단계에서 패스워드 설정을 했다면 패스워드로 로그인할 수 있다.
  만약 패스워드 설정을 하지 않았다면 아래 명령어를 이용해 토큰을 확인할 수 있다.

  ```shell
  docker logs jupyter-notebook
  ```

- 쥬피터 노트북 사용 예시
  ![사진](/assets/img/posts/legacy/ml/jupyter-notebook-005.png)
  ![사진](/assets/img/posts/legacy/ml/jupyter-notebook-006.png)
