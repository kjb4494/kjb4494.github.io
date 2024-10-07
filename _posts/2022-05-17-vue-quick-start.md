---
title: Vue 3 프로젝트 생성 및 개발환경 세팅하기
author: kjb4494
date: 2022-05-17 00:00:00 +0900
categories: [Frontend, VueJs]
tags: [vue, typescript, pinia, vite]
---

### Vue 프로젝트 생성하기

vue 프로젝트를 생성하는 작업은 매우 쉽다.

현재 글을 작성하고 있는 2022-05-17 을 기준으로 [_Vue 공식 홈페이지의 Quick Start_](https://vuejs.org/guide/quick-start.html#with-build-tools) 에서 보면 아래 명령어로 프로젝트를 생성하면 다음과 같은 초기 세팅을 선택할 수 있다.

```shell
> npm init vue@latest

Vue.js - The Progressive JavaScript Framework

√ Project name: ... test-project
√ Add TypeScript? ... Yes
√ Add JSX Support? ... No
√ Add Vue Router for Single Page Application development? ... Yes
√ Add Pinia for state management? ... Yes
√ Add Vitest for Unit Testing? ... Yes
√ Add Cypress for End-to-End testing? ... Yes
√ Add ESLint for code quality? ... Yes
√ Add Prettier for code formatting? ... Yes
```

![프로젝트 초기 구조](/assets/img/posts/2022-05-17/%ED%99%94%EB%A9%B4%20%EC%BA%A1%EC%B2%98%202022-05-17%20011851.png){: style="max-height: 350px" .left}
확장 스크립트를 제외하고 모든 선택지에 Yes 를 했을 경우의 프로젝트 구조다.

가장 눈에 띄는 건 번들러로 `vite` 가 쓰였다는 점이다. 기존 프로젝트에서 쓰인 vue-cli 에 비해 미리보기 빌드 속도가 무려 10배 이상 빨라졌다. 이는 vite 가 어떤 모듈이 수정될 경우 수정된 모듈과 관련된 부분만을 교체하기 때문이다. 그래서 앱 사이즈가 커져도 갱신 시간에 영향을 끼치지 않는다. 이는 프로젝트 크기가 커질수록 빌드 시간이 느려지는 vue-cli 와 비교하면 큰 이점이다.

두번째로는 store 로 vuex 대신 `pinia` 가 쓰였다. vuex 와 비교했을 때 큰 차이점은 mutations 가 없다는 것이었다! 이는 추후에 상세히 다뤄볼 예정이다.

그 밖에 Unit Test 를 할 수 있는 Vitest 와 End-to-End Test 를 할 수 있는 cypress 가 추가되었다.

<br>

### 개발환경 세팅하기

ide 는 당연하지만 vscode 를 사용했다. 타입스크립트를 사용함으로써 타입 추론과 문법 검증이 가능해지면서 ide 의 자동완성 기능이 고도화되어 개발 생산성을 상당히 높힐 수 있었다. 하지만 프로젝트 초기 세팅만으로는 거슬리는 부분이 일부 있었는데, 다음과 같이 해결했다.

- ts 파일에서 vue 파일 import 시 red line 이 발생하는 현상

  ![에러1](/assets/img/posts/2022-05-17/%ED%99%94%EB%A9%B4%20%EC%BA%A1%EC%B2%98%202022-05-17%20015228.png){: style="min-width: 100%"}

  ```typescript
  /* eslint-disable */
  declare module "*.vue" {
    import type { DefineComponent } from "vue";
    const component: DefineComponent<{}, {}, any>;
    export default component;
  }
  ```

  {: file='/src/shims-vue.d.ts'}

  위와 같이 shims-vue.d.ts 파일을 추가해주는 것으로 해결했다.

- Delete `␍`eslint(prettier/prettier) 경고가 발생하는 현상

  ![에러2](/assets/img/posts/2022-05-17/%ED%99%94%EB%A9%B4%20%EC%BA%A1%EC%B2%98%202022-05-17%20015753.png){: style="min-width: 100%"}

  해당 경고는 prettier 2 버전부터 endOfLine 의 기본값이 lf 로 변경돼서 윈도우 개발환경에서 발생한다. `.gitattributes` 나 별도 git config 수정이 없을 경우 윈도우 환경에서는 원격 저장소에서 받아올 때 개행문자를 crlf 로 바꿔서 가져온다.

  그렇다고 git 설정을 바꾸기는 귀찮으니 `.eslintrc.cjs` 에서 rules 을 추가해 prettier 의 `endOfLine` 설정을 `auto` 로 바꿔주자.

  ```json
  {
    "rules": {
      "prettier/prettier": [
        "error",
        {
          "endOfLine": "auto"
        }
      ]
    }
  }
  ```

  {: file='.eslintrc.cjs'}

- vue 파일에서 import 시 모듈을 찾지 못 하는 현상

  ```json
  {
    "compilerOptions": {
      "baseUrl": ".",
      "paths": {
        "@/*": ["src/*"]
      }
    }
  }
  ```

  {: file='tsconfig.json'}

  compilerOptions 설정을 추가해준다.

### 편리한 Vscode 확장 프로그램

- Auto Close Tag
  HTML 태그를 선언할 때 자동으로 닫아준다.
- Auto Rename Tag
  HTML 태그를 수정할 때 닫는 태그도 함께 자동으로 수정해준다.
- `ESLint` & `Prettier - Code formatter`
  이 두 개는 필수!! `.eslintrc.js` 파일을 읽어 코드 스타일을 강제해준다. 특히 협업할 때 중요하다. macOS: `⌘ + ,`, Windows: `Ctrl + ,`, Linux: `Ctrl + ,` 로 vscode `settings.json` 을 열어서 `"editor.formatOnSave": true` 를 추가하면 저장할 때 자동으로 포맷팅된다.
- Markdown Preview Github Styling
  md 파일을 프리뷰 해준다. README.md 등을 작성할 때 편리하다.
- Vetur
  vue 파일의 코드를 하이라이팅해준다. 거의 필수(?)
- Vue 3 Snippets
  vue 의 `template`, `script`, `style` 등을 한 방에 생성해준다. 특히 타입스크립트 사용할 경우 거의 필수!
