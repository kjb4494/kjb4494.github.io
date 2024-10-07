---
title: Windows Terminal에 git-bash 추가하기
author: kjb4494
date: 2021-02-16 13:00:00 +0900
categories: [기타, 개발환경]
tags: [git, shell]
---

### Windows Terminal Profile 추가

```json
{
  "guid": "{3b309cd0-e48b-4361-a95a-c56d51c101f4}",
  "commandline": "%programfiles%/git/usr/bin/bash.exe -il",
  "name": "Git Bash",
  "hidden": false,
  "icon": "%programfiles%/git/mingw64/share/git/git-for-windows.ico",
  "tabTitle": "Git Bash",
  "suppressApplicationTitle": true
}
```

![사진](/assets/img/posts/legacy/git/add-git-bash-in-windows-terminal-001.png){: .normal}

성공적으로 추가됐다. 이외에 더 필요한 프로필 설정이 있다면 [_공식 문서_](https://docs.microsoft.com/ko-kr/windows/terminal/customize-settings/profile-general)를 참고하자.

guid는 웹에서 [_guid generator_](https://www.guidgenerator.com/) 아무거나 사용해서 박으면 된다. 이 방법이 싫다면 아래 파이썬으로 생성하는 방법을 쓰자.

### guid 생성하기

1. 파이썬 실행
   ```shell
   python
   ```
1. 간단 코딩
   ```python
   import uuid
   str(uuid.uuid4())
   ```
1. 출력 결과(그 때 그 때 다름)
   ```text
   '6545b13c-a2e2-4537-9fe1-26c23d561843'
   ```
