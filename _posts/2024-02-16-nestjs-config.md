---
title: NestJs 설정
author: kjb4494
date: 2024-02-16 17:00:00 +0900
categories: [Backend, NestJs]
tags: [nestjs, backend]
---

소스 코드 안에서 어떠한 코드들은 개발 환경이나 운영 환경 등 환경에 따라서 다르게 코드를 넣어줘야 할 때가 있으며, 남들에게 노출 되지 않아야 하는 코드들도 있습니다. 이러한 코드들을 위해서 설정 파일을 따로 만들어서 보관해줍니다.

설정 파일은 runtime 도중에 바뀌는 것이 아닌 애플리케이션이 시작할 때 로드가 되어서 그 값들을 정의하여 줍니다. 그리고 설정 파일은 여러가지 파일 형식을 사용할 수 있습니다. 예로는 XML, Json, Yaml, Environment Variables 같이 많은 형식을 이용할 수 있습니다.

## Codebase vs Environment Variables

XML, Json, Yaml 같은 경우는 Codebase에 해당하며, 다른 방법으로는 Environment Variables 즉, 환경 변수가 있습니다. 비밀번호와 API Key 같은 외부에 노출 되면 안 되는 정보들을 주로 환경 변수를 이용해서 처리해줍니다.

- **Codebase**: 일반적으로 Port 같이 노출되도 상관 없는 정보들
- **환경 변수**: 비밀번호나 API Key 같이 노출되면 안 되는 정보들

## 설정하기 위해서 필요한 모듈

윈도우에서는 기본적으로 Mac, Linux 같은 환경변수를 지원하지 않기 때문에 개발환경에서는 `cross-env` 모듈을 설치해줍니다.

```shell
yarn add --dev cross-env
```

그 다음, config 모듈을 설치해줍니다.

```shell
yarn add config
```

## Config 모듈을 이용한 설정 파일 생성

루트 디렉토리에 config라는 폴더를 만든 후에 그 폴더 안에 Json이나 Yaml 파일을 생성합니다: `config/default.yml`, `config/development.yml`, `config/production.yml`

- **default.yml**: 기본 설정으로, 개발 환경이나 운영 환경 설정에도 적용됩니다.
- **development.yml**: default.yml + 개발 환경에서 필요한 정보
- **production.yml**: default.yml + 운영 환경에서 필요한 정보

### default.yml

```yml
server:
  port: 3000

db:
  type: "postgres"
  port: 5432
  database: "board-db"

jwt:
  expiresIn: 3600
```

### development.yml

```yml
db:
  host: "localhost"
  username: "postgres"
  password: "P@ssw0rd"
  synchronize: true
  logging: true

jwt:
  secret: "secretkey1234dev"
```

### production.yml

```yml
db:
  host: "localhost"
  username: "postgres"
  password: "P@ssw0rd"
  synchronize: false
  logging: false

jwt:
  secret: "secretkey1234prod"
```

## 설정 파일 코드에 적용하기

### main.ts

```typescript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import * as config from "config";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const serverConfig = config.get("server");
  const port = serverConfig.port;

  await app.listen(port);
}
bootstrap();
```

### typeorm.config.ts

```typescript
import { TypeOrmModuleOptions } from "@nestjs/typeorm";
import * as config from "config";

const dbConfig = config.get("db");

export const typeORMConfig: TypeOrmModuleOptions = {
  type: dbConfig.type,
  host: process.env.RDS_HOSTNAME || dbConfig.host,
  port: process.env.RDS_PORT || dbConfig.port,
  username: process.env.RDS_USERNAME || dbConfig.username,
  password: process.env.RDS_PASSWORD || dbConfig.password,
  database: process.env.RDS_DB_NAME || dbConfig.database,
  entities: [__dirname + "/../**/*.entity.{js,ts}"],
  synchronize: dbConfig.synchronize,
  logging: dbConfig.logging
};
```

환경변수 설정이 있을 경우 환경변수를 사용하고, 없을 경우 config 파일을 참조합니다.

## package.json 스크립트 수정

```json
{
  "scripts": {
    "build": "cross-env NODE_ENV=production nest build",
    "start": "cross-env NODE_ENV=development nest start",
    "start:dev": "cross-env NODE_ENV=development nest start --watch",
    "start:debug": "cross-env NODE_ENV=development nest start --debug --watch",
    "start:prod": "cross-env NODE_ENV=production node dist/main"
  }
}
```
