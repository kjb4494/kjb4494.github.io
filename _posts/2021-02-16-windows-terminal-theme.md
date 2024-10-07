---
title: Windows Termial 테마 적용하기
author: kjb4494
date: 2021-02-16 16:00:00 +0900
categories: [기타, 개발환경]
tags: [terminal, theme]
---

### 프로필 설정하기

- 설정 파일 Open

  ![사진](/assets/img/posts/legacy/룩덕질/windows-terminal-theme-001.png){: .normal}

- 테마 스키마 추가
  ```json
  "schemes": [
      {
          "name": "idea",
          "black": "#adadad",
          "red": "#fc5256",
          "green": "#98b61c",
          "yellow": "#ccb444",
          "blue": "#437ee7",
          "purple": "#9d74b0",
          "cyan": "#248887",
          "white": "#181818",
          "brightBlack": "#ffffff",
          "brightRed": "#fc7072",
          "brightGreen": "#98b61c",
          "brightYellow": "#ffff0b",
          "brightBlue": "#6c9ced",
          "brightPurple": "#fc7eff",
          "brightCyan": "#248887",
          "brightWhite": "#181818",
          "background": "#202020",
          "foreground": "#adadad"
        }
  ]
  ```
  [_이곳_](https://windowsterminalthemes.dev/)으로 가면 여러가지 테마가 있다. 마음에 드는 걸로 골라 복붙한다.
- 스키마 적용
  ```json
  "list":
      [
          {
              "colorScheme": "idea",
              "name": "Windows PowerShell",
          }
      ]
  ```
  위에서 추가한 스키마명을 원하는 쉘의 colorScheme 레이블에 적용시켜주면 된다.
