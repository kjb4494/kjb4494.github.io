---
title: AWS Lambda에 Node.js Express 애플리케이션 배포하기
author: kjb4494
date: 2023-08-01 14:00:00 +0900
categories: [Backend, Nodejs Express]
tags: [nodejs, backend, aws lambda]
---

안녕하세요! 오늘은 클라우드 컴퓨팅에서 가장 핵심적인 서비스 중 하나인 AWS Lambda에 Node.js 기반의 Express 애플리케이션을 배포하는 방법에 대해 알아보려고 합니다.

많은 개발자들이 이미 알고 있듯이, AWS Lambda는 이벤트 기반 컴퓨팅을 가능하게 하는 서버리스 서비스입니다. 서버리스 아키텍처는 애플리케이션의 인프라 관리에 대한 걱정을 덜어주고, 개발자들이 코드 작성에 좀 더 집중하게 해줍니다. 또한, AWS Lambda는 자동 확장, 높은 가용성, 이벤트 기반 실행 등과 같은 여러 장점을 제공합니다.

이 포스트에서는 Node.js와 Express 프레임워크를 사용하여 간단한 웹 애플리케이션을 만든 후, 이를 AWS Lambda에 배포하는 과정을 단계별로 설명할 예정입니다. AWS Lambda를 이용하면 웹 애플리케이션을 서버리스 환경에서 손쉽게 실행할 수 있습니다. AWS Lambda와 같은 서버리스 플랫폼은 요구에 따라 자동으로 확장되며, 미사용 시간에는 비용이 발생하지 않는 등의 효율성 때문에 많은 개발자들이 선호하는 선택입니다.

이 튜토리얼은 Node.js와 Express, 그리고 AWS에 대한 기본적인 지식을 가진 개발자를 대상으로 하며, 아래와 같은 주제들을 다룰 예정입니다:

1. Node.js와 Express를 사용하여 간단한 웹 애플리케이션 만들기
2. AWS Lambda와 함께 사용할 수 있도록 애플리케이션 패키징하기
3. AWS Lambda에 애플리케이션 배포하기
4. 배포한 애플리케이션 테스트하기

그럼 이제 본격적으로 시작해볼까요?

## Node.js와 Express를 사용하여 간단한 웹 애플리케이션 만들기

우리의 첫 번째 단계는 Node.js와 Express를 사용하여 간단한 웹 애플리케이션을 만드는 것입니다. 이를 위해, 우리는 Express의 공식 웹사이트에 제공되는 가이드를 참조할 것입니다. ([Express 애플리케이션 생성기](https://expressjs.com/ko/starter/generator.html))

먼저, 우리는 프로젝트를 진행하기 위한 최신 stable 버전의 Node.js를 설치해야 합니다. 이 포스트를 작성하는 2023년 8월 1일 현재, Node.js의 최신 stable 버전은 v18.17.0입니다.

Node.js를 설치한 후, Express 애플리케이션 생성기를 설치해야 합니다. 이 생성기는 새로운 Express 애플리케이션의 기본 구조를 쉽게 만들 수 있게 도와줍니다. 이를 설치하려면, 터미널에서 다음과 같은 명령어를 실행하면 됩니다:

```bash
npm install express-generator -g
```

위 명령어는 `express-generator`를 전역적(global)으로 설치하며, 이를 통해 어느 위치에서든지 `express` 명령어를 사용할 수 있게 해줍니다.

그런 다음, `express` 명령어를 사용하여 새로운 Express 애플리케이션을 생성합니다. 여기서는 'oauth2-manager'라는 이름의 프로젝트를 생성해 보겠습니다. 또한, `--view=pug` 옵션을 추가하여 Pug를 기본 뷰 템플릿 엔진으로 설정하겠습니다.

```bash
express --view=pug oauth2-manager
```

이제 Express 애플리케이션이 생성되었으니, 해당 디렉토리로 이동한 후에 필요한 종속 항목들을 설치해야 합니다. 이를 위해 다음의 명령어들을 실행합니다:

```bash
cd oauth2-manager
npm install
```

`npm install` 명령어는 프로젝트의 `package.json` 파일에 명시된 모든 종속 항목들을 설치해줍니다.

그 다음으로, 애플리케이션을 로컬에서 실행해보겠습니다. MacOS나 Linux에서는 다음과 같은 명령어를 사용하여 애플리케이션을 실행할 수 있습니다:

```bash
DEBUG=oauth2-manager:* npm start
```

Windows에서는 다음과 같은 명령어를 사용하면 됩니다:

```powershell
set DEBUG=oauth2-manager:* & npm start
```

이제 애플리케이션을 실행하고 브라우저에서 `http://localhost:3000/`를 방문하면 우리가 만든 웹 애플리케이션에 액세스할 수 있습니다.

지금까지 생성된 애플리케이션은 다음과 같은 디렉토리 구조를 가지고 있습니다:

```
.
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.pug
    ├── index.pug
    └── layout.pug
```

### Express 애플리케이션에 RESTful API 추가하기

우리의 Express 애플리케이션이 AWS Lambda에서 원활히 작동하려면, 애플리케이션에 RESTful API를 추가해야 합니다. 이번 섹션에서는 기존의 `routes/index.js` 파일을 수정하여 간단한 RESTful API를 만드는 방법에 대해 설명하겠습니다.

기존의 `routes/index.js` 파일은 다음과 같이 생겼습니다:

```javascript
var express = require("express");
var router = express.Router();

/* GET home page. */
router.get("/", function (req, res, next) {
  res.render("index", { title: "Express" });
});

module.exports = router;
```

이 파일은 Express 애플리케이션의 루트 경로(`/`)에 대한 GET 요청을 처리합니다. 요청이 들어오면, Express 애플리케이션은 `index` 뷰를 렌더링하고, 그 결과를 클라이언트에게 전송합니다.

하지만, 우리는 이 파일을 수정하여 루트 경로와 `/api/v1/status` 경로에 대한 GET 요청을 각각 다르게 처리하도록 만들겠습니다. 수정된 `routes/index.js` 파일은 다음과 같이 생겨야 합니다:

```javascript
var express = require("express");
var payload = require("../utils/payload-util");
var router = express.Router();

/* GET home page. */
router.get("/", function (req, res, next) {
  res.json(payload.success("home", {}));
});

router.get("/api/v1/status", function (req, res, next) {
  res.json(payload.success("fine", {}));
});

module.exports = router;
```

또한, `utils/payload-util.js` 파일은 다음과 같이 작성됩니다:

```javascript
module.exports = {
  success: function (message, obj) {
    return {
      timestamp: Date.now(),
      data: {
        message: message,
        detail: obj
      }
    };
  },
  error: function (status, code, message) {
    return {
      timestamp: Date.now(),
      error: {
        status: status,
        code: code,
        message: message
      }
    };
  }
};
```

위의 `payload-util.js` 파일은 성공 응답과 실패 응답을 위한 페이로드를 생성하는 두 개의 함수를 제공합니다. 이러한 함수들은 API 응답을 생성할 때 사용됩니다.

이제 `http://localhost:3000` 또는 `http://localhost:3000/api/v1/status` 주소로 요청을 보내면, 우리가 작성한 라우터에 따라 각각 다른 응답을 받을 수 있습니다.

### Express 애플리케이션의 에러 핸들러 수정하기

Express 애플리케이션에서 에러 핸들러는 애플리케이션에서 발생하는 모든 에러를 처리합니다. 기본적으로 Express는 에러를 HTML 페이지로 렌더링하여 보여줍니다. 하지만 RESTful API를 제공하는 서비스에서는 JSON 형식의 에러 메시지를 반환하는 것이 일반적입니다. 그러므로 우리는 기존의 에러 핸들러를 수정하여 JSON 형식의 에러 메시지를 반환하도록 만들어보겠습니다.

먼저, `app.js` 파일의 상단에 `utils/payload-util.js` 파일을 `require`로 불러오는 코드를 추가합니다.

```javascript
var createError = require('http-errors');
var express = require('express');
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');
var payload = require('./utils/payload-util'); // Add this line

var indexRouter = require('./routes/index');
var usersRouter = require('./routes/users');

var app = express();
...
```

그 다음, `app.js` 파일의 에러 핸들러 부분을 아래와 같이 수정합니다:

```javascript
// error handler
app.use(function (err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get("env") === "development" ? err : {};

  // render the error page
  const status = err.status || 500;
  const code = err.code || -1;
  res.status(status);
  res.json(payload.error(status, code, err.message));
});
```

수정된 에러 핸들러는 에러가 발생했을 때 에러 상태 코드와 에러 메시지를 JSON 형식으로 반환합니다. 이렇게 함으로써 클라이언트는 애플리케이션에서 발생하는 모든 에러에 대해 일관된 형식의 응답을 받을 수 있게 됩니다.

이로써, 우리의 Express 애플리케이션은 RESTful API를 제공하며, 이를 위한 라우터와 에러 핸들러를 가지게 되었습니다. 다음 단계에서는 이 애플리케이션을 AWS Lambda와 함께 사용할 수 있도록 패키징하는 방법을 알아보겠습니다.

## AWS Lambda와 함께 사용할 수 있도록 애플리케이션 패키징하기

이제 우리의 Express 애플리케이션을 AWS Lambda에서 실행할 수 있도록 패키징해야 합니다. 이 과정은 크게 두 단계로 나눠볼 수 있습니다.

첫 번째 단계는 Serverless 프레임워크를 설치하는 것입니다. Serverless 프레임워크는 AWS Lambda와 같은 서버리스 플랫폼에 애플리케이션을 배포하고 관리하는 데 매우 유용한 도구입니다. Serverless 프레임워크를 설치하려면, 다음과 같은 명령어를 실행합니다:

```bash
npm install -g serverless
```

이 명령어는 Serverless 프레임워크를 전역적으로 설치하므로, 어느 위치에서든지 `serverless` 명령어를 사용할 수 있습니다.

두 번째 단계는 Serverless 프레임워크를 사용하여 새로운 서버리스 서비스를 생성하는 것입니다. 이를 위해, 다음과 같은 명령어를 실행합니다:

```bash
serverless create --template aws-nodejs
```

이 명령어는 `aws-nodejs` 템플릿을 사용하여 새로운 서버리스 서비스를 생성합니다. 생성되는 서비스에는 `handler.js`라는 파일이 포함되어 있으며, 이 파일은 AWS Lambda 함수의 핸들러 역할을 합니다.

생성된 `handler.js` 파일은 다음과 같이 생겼습니다:

```javascript
"use strict";

module.exports.hello = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify(
      {
        message: "Go Serverless v1.0! Your function executed successfully!",
        input: event
      },
      null,
      2
    )
  };
};
```

### AWS Lambda를 위한 패키징: Express 애플리케이션 수정하기

그런데, 우리가 작성한 Express 애플리케이션이 AWS Lambda에서 잘 작동하도록 이 핸들러 함수를 수정해야 합니다. 우리의 목표는 `handler.js` 파일을 수정하여 AWS Lambda에서 Express 애플리케이션을 실행할 수 있게 하는 것입니다. 여기서는 `serverless-http` 라이브러리를 사용하여 우리의 Express 애플리케이션을 AWS Lambda 환경에서 실행 가능하도록 변환해 보겠습니다.

```bash
$ npm install serverless-http
```

먼저, `serverless-http` 라이브러리를 설치해주시고 `app.js` 파일의 내용을 `handler.js`에 그대로 복사합니다. 그리고 상단에 `serverless-http` 모듈을 불러오는 코드를 추가합니다.

```javascript
var serverless = require("serverless-http"); // Add this line

var createError = require('http-errors');
var express = require('express');
...
```

이 `serverless-http` 라이브러리는 Express 애플리케이션이 AWS Lambda에서 잘 동작하도록 돕는 미들웨어 역할을 합니다.

그 다음, 아래와 같이 `module.exports` 라인을 수정하여 `app`을 AWS Lambda 핸들러 함수로 내보내도록 합니다.

```javascript
// Serverless Lambda handler function export
module.exports.app = serverless(app);
```

`serverless(app)` 호출은 Express 애플리케이션을 래핑하여 AWS Lambda와 호환되는 함수를 반환합니다. 이렇게 하면, 우리의 애플리케이션이 AWS Lambda 환경에서도 똑같이 동작하게 됩니다.

이제 우리의 애플리케이션은 AWS Lambda 환경에서도 동작할 수 있도록 준비되었습니다.

### 로컬에서 Serverless 코드 테스트하기

하지만! AWS Lambda에 배포하기 전에, 로컬 환경에서 우리의 코드가 잘 동작하는지 확인하려면 `serverless-offline` 패키지를 설치해야 합니다. 이 패키지를 설치하면 Serverless 프레임워크를 사용하여 로컬 환경에서 Lambda 함수를 시뮬레이션할 수 있습니다.

```bash
$ npm install serverless-offline --save-dev
```

다음으로 `serverless.yml` 파일을 열고 아래와 같이 내용을 수정합니다.

```yaml
service: oauth2-manager
frameworkVersion: "3"

provider:
  name: aws
  runtime: nodejs18.x

plugins:
  - serverless-offline

functions:
  app:
    handler: handler.app
    events:
      - httpApi:
          path: /{proxy+}
          method: any
      - httpApi:
          path: /
          method: any
```

위의 `serverless.yml` 파일 설정은 Serverless 프레임워크에게 우리가 AWS의 Node.js 18.x 런타임을 사용하고 있으며, `handler.app`이라는 함수를 AWS Lambda 함수로 사용할 것이라고 알려줍니다. 또한 `serverless-offline` 플러그인을 사용하도록 설정했습니다.

이제, 아래와 같이 `serverless offline` 명령어를 실행하여 로컬 환경에서 애플리케이션을 테스트할 수 있습니다.

```bash
$ npx serverless offline
```

이제 브라우저를 열고 `http://localhost:3000/api/v1/status`에 접속해보세요. 이전에 로컬 서버에서 실행했던 것과 동일한 결과를 볼 수 있을 것입니다. 이는 우리의 Express 애플리케이션이 `serverless-http`을 통해 Lambda 환경에서도 잘 동작할 수 있음을 보여줍니다.

## AWS Lambda에 애플리케이션 배포하기

이제 준비된 모든 것이 잘 동작하는 것을 확인했으니, AWS Lambda에 배포해볼 차례입니다!

### AWS에서 Lambda 함수 생성하기

우리의 Express.js 애플리케이션을 AWS Lambda에 배포하기 위해 먼저 Lambda 함수를 생성해야 합니다. 그러나 이 글에서는 Serverless Framework를 사용하여 자동으로 Lambda 함수를 관리하는 방법에 대해 자세히 다루지 않겠습니다. 이 주제는 보안적으로 민감한 요소도 다뤄야하고 복잡하기 때문에, 추후 별도의 글에서 자세히 다루겠습니다. ;)

따라서, 이번에는 AWS Management Console에서 수동으로 Lambda 함수를 생성하겠습니다. 아래의 순서대로 진행하시면 됩니다.

1. AWS Management Console에 로그인합니다.
2. 서비스 목록에서 'Lambda'를 찾아서 클릭합니다.
3. '함수 생성' 버튼을 클릭합니다.
4. '함수 이름'에 원하는 이름을 입력합니다.
5. '런타임'에서 'Node.js 18.x'를 선택합니다.
6. '함수 생성' 버튼을 클릭하여 함수를 생성합니다.

### AWS에서 HTTP API 생성 및 라우팅 설정하기

Lambda 함수를 생성한 후에는 API Gateway를 통해 이 Lambda 함수에 접근할 수 있도록 설정해야 합니다. HTTP API를 생성하고 라우팅 설정하는 과정은 다음과 같습니다:

1. AWS Management Console에서 'API Gateway'를 검색하고 클릭합니다.
2. 'API 생성'을 선택하고 'HTTP API'를 구축해줍니다.
3. 생성된 API Gateway Routes 설정에서 `ANY /`와 `ANY /{proxy+}` 경로를 추가해줍니다.
4. Integrations 설정에서 전 단계에서 생성한 Lambda를 통합해줍니다.

이렇게 하면 HTTP API가 생성되고, 생성된 API의 URL로 모든 요청이 Lambda 함수로 전달됩니다.

### AWS Lambda에 Express.js 애플리케이션 배포하기

AWS Lambda에 애플리케이션을 배포하는 과정은 몇 단계로 나누어집니다. 이 섹션에서는 AWS Lambda에 Express.js 애플리케이션을 배포하는 과정을 설명하겠습니다.

#### 1. 패키지링

AWS Lambda에서는 애플리케이션과 함께 모든 필요한 패키지와 모듈을 함께 제공해야 합니다. Express.js 애플리케이션을 패키지링하는 방법은 다음과 같습니다:

1. 애플리케이션의 루트 디렉토리에서 `npm install --production` 명령을 실행하여 필요한 종속성을 설치합니다. `--production` 플래그는 개발 종속성을 제외하고 오직 프로덕션에서 필요한 종속성만을 설치하도록 합니다.

2. 애플리케이션의 모든 파일과 폴더(`node_modules` 포함)를 `.zip` 파일로 압축합니다.

#### 2. 배포

이제 Lambda 함수로 애플리케이션 패키지를 업로드할 수 있습니다. AWS Management Console에서 Lambda 함수 페이지로 이동하여 다음 단계를 수행합니다:

1. 전 단계에서 생성한 Lambda 함수를 선택합니다.
2. '코드' 섹션으로 이동합니다.
3. '.zip 파일 업로드'를 선택합니다.
4. `.zip` 파일을 업로드하고 '저장' 버튼을 클릭합니다.

이제 AWS Lambda에서 Express.js 애플리케이션이 실행됩니다. 이 과정은 애플리케이션의 업데이트마다 수행해야 하지만, 이를 자동화하는 방법들이 존재합니다. 예를 들어, 서버리스 프레임워크를 이용하면 이러한 과정을 자동화할 수 있습니다. 이에 대한 내용 역시 Serverless Framework를 주제로 추후에 다뤄보도록 하겠습니다. ;)

## 배포한 애플리케이션 테스트하기

애플리케이션을 AWS Lambda에 배포한 후, 실제로 제대로 동작하는지 확인해야 합니다. 애플리케이션을 테스트하는 방법은 아래와 같습니다:

1. API Gateway 콘솔에서 해당 API의 엔드포인트를 찾습니다. 이는 "API 엔드포인트 URL"이라고 표시되어 있을 것입니다.
2. 웹 브라우저를 열고, 이 API 엔드포인트 URL을 주소창에 입력해 접속합니다.
3. 또한, API 엔드포인트 URL 뒤에 `/api/v1/status`를 추가하여 해당 경로로의 라우팅이 잘 이루어지는지도 확인해봅니다.

예를 들어, `https://xxxxxx.execute-api.region.amazonaws.com/default`가 API Gateway의 엔드포인트라면, `/api/v1/status`를 추가한 URL은 `https://xxxxxx.execute-api.region.amazonaws.com/default/api/v1/status`가 됩니다.

이 과정을 통해 우리가 원래 작성했던 응답이 나오는지 확인할 수 있습니다. 만약 응답이 정상적으로 나온다면, AWS Lambda에 배포한 애플리케이션이 제대로 동작하는 것입니다. 이렇게 하면 우리는 서버 관리 없이 Express.js 애플리케이션을 AWS Lambda에서 운영할 수 있게 됩니다.

지금까지, 우리는 AWS Lambda와 API Gateway를 이용하여 Express.js 애플리케이션을 서버리스 환경에 배포하는 과정에 대해 상세히 살펴보았습니다. 이를 통해, 기존에 서버 기반으로 동작하던 애플리케이션을 서버리스 아키텍처로 전환하는 방법에 대한 이해가 되셨길 바랍니다. 긴 글 읽어주셔서 감사합니다! :)
