---
title: Github에서 내 Commit에 서명하기
author: kjb4494
date: 2021-02-16 15:00:00 +0900
categories: [기타, 개발환경]
tags: [git, github, commit]
---

### GPG Key 등록하기

> - [_공식 문서_](https://docs.github.com/en/github/authenticating-to-github/managing-commit-signature-verification)

각 쉘 명령어는 모두 Git Bash에서 진행한다.

1. gpg key 생성하기
   ```shell
   gpg --full-generate-key
   ```
   > - RSA 알고리즘을 사용한다.
   > - key size는 4096 이상으로 한다.
   > - real name과 email은 commit할 username과 email로 한다.
1. gpg key 확인하기
   ```shell
   gpg --list-secret-keys --keyid-format LONG
   ```
   - 결과
     ```text
     /Users/hubot/.gnupg/secring.gpg
     ------------------------------------
     sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
     uid                          Hubot
     ssb   4096R/42B317FD4BA89E7A 2016-03-10
     ```
     여기서 3AA5C34371567BD2가 gpg key id 이다.
1. export
   ```shell
   gpg --armor --export 3AA5C34371567BD2
   ```
   3AA5C34371567BD2 이 부분은 위에서 확인한 gpg key id를 적어준다.
   - 결과
     ```text
     -----BEGIN PGP PUBLIC KEY BLOCK-----
     .
     .
     .
     -----END PGP PUBLIC KEY BLOCK-----
     ```
1. github에 gpg key 등록하기
   [_GPG keys Settings_](https://github.com/settings/keys)에서 export한 내용을 복사해 등록해준다.

### Mac에서...

```shell
brew install gnupg
brew install pinentry-mac
```

brew의 gnupg는 실제로 gnupg2로 링크되어 있기 때문에 github가 요구하는 gpg2 버전을 충족한다.
mac에 기본적으로 pinentry가 깔려있겠지만, 만약 commit 할 때 아래와 같은 에러가 발생한다면 pinentry-mac을 설치해준다.

```text
error: gpg failed to sign the data
fatal: failed to write commit object
```

pinentry-mac을 설치해도 이 같은 에러가 계속 난다면 아래 절차를 수행해주자.

```shell
echo "pinentry-program /usr/local/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf
gpgconf --kill gpg-agent
```

### commit 할 때 서명하기

- git config
  ```shell
  git config --global commit.gpgsign true
  ```
- commit 할 때
  ```shell
  git commit -S -m your commit message
  ```

### commit 테스트

```shell
git commit --allow-empty -S -m 'commit sv test'
```

![사진](/assets/img/posts/legacy/git/github-commit-sv-001.png)

성공! X)
