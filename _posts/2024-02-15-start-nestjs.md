---
title: NestJs 시작하기
author: kjb4494
date: 2024-02-15 12:00:00 +0900
categories: [Backend, NestJs]
tags: [nestjs, backend]
---

## NestJs란?

Nest는 효율적이고 확장 가능한 Node.js 서버 측 애플리케이션을 구축하기위한 프레임 워크입니다. 프로그레시브 Javascript를 사용하고 typescript로 빌드되고 완벽하게 지원하며 OOP, FP 및 FRP 요소를 사용할 수 있게 해줍니다.

[_공식문서_](https://docs.nestjs.com/){:target="\_blank"}

## NestJs 내부 구성

Nest는 내부적으로 Express와 같은 강력한 HTTP 서버 프레임 워크를 사용하며 선택적으로 Fastify를 사용하도록 구성 할 수도 있습니다.

## NestJs의 철학

Node를 위한 훌륭한 라이브러리, 도우미 및 도구가 많이 존재하지만 이들 중 어느 것도 아키텍처의 주요 문제를 효과적으로 해결하지 못합니다. Nest는 개발자와 팀이 고도로 테스트 가능하고 확장 가능하며 느슨하게 결합되고 유지 관리가 쉬운 애플리케이션을 만들 수 있는 즉시 사용 가능한 애클리케이션 아키텍처를 제공합니다. 이 아키텍처는 Angular에서 크게 영감을 받았습니다.

## 프로젝트 시작하기

1. 프로젝트를 시작 할 폴더 생성
1. 폴더 안에서 nest 기본 구조 생성
   ```shell
   npm i -g @nestjs/cli
   nest new ./
   ```
1. 앱실행
   ```shell
   npm run start:dev
   ```

## NestJs 기본 구조

- **eslintrc.js**: 개발자들이 특정한 규칙을 가지고 코드를 깔끔하게 짤수있게 도와주는 라이브러리. 타입스크립트를 쓰는 가이드 라인 제시, 문법에 오류가 나면 알려주는 역할 등등
- **prettierrc**: 주로 코드 형식을 맞추는데 사용. 작은따옴표(')를 사용할지 큰 따옴표(")를 사용할지, Indent 값을 2로 줄지 4로 줄기 등등, 에러 찾는 것이 아닌 코드 포맷터 역할
- **nest-cli.json**: Nest 프로젝트를 위해 특정한 설정을 할 수 있는 json 파일
- **tsconfig.json**: 어떻게 타입스크립트를 컴파일 할지 설정
- **tsconfig.build.json**: tsconfig.json의 연장선상 파일이며, build를 할 때 필요한 설정들. "excludes"에서는 빌드할 때 필요없는 파일들을 명시
- **package.json**: `build`는 운영환경을 위한 빌드. `format`은 린트에러가 났을지 수정. `start`는 앱 시작
- **src 폴더**: 대부분의 비즈니스 로직이 들어가는 곳. `main.ts`는 앱을 생성하고 실행. `app.module.ts`는 앱 모듈을 정의

## Providers란?

프로바이더는 Nest의 기본 개념입니다. 대부분의 기본 Nest 클래스는 서비스, 리포지토리, 팩토리, 헬퍼 등 프로바이더로 취급될 수 있습니다. 프로바이더의 주요 아이디어는 `종속성으로 주입`할 수 있다는 것입니다. 즉, 객체는 서로 다양한 관계를 만들 수 있으며 객체의 인스턴스를 `연결`하는 기능은 대부분 Nest 런타임 시스템에 위임될 수 있습니다.

Provider 등록은 module 파일의 @Module 데코레이터에서 providers에 할 수 있습니다.

## Service란?

서비스는 소프트웨어 개발내의 공통 개념이며, NestJs에서만 쓰이는 개념이 아닙니다. @Injectable 데코레이터로 감싸져서 모듈에 제공되며, 이 서비스 인스턴스는 애플리케이션 전체에서 사용될 수 있습니다. 서비스는 컨트롤러에서 데이터의 유효성 체크를 하거나 데이터베이스에 아이템을 생성하는 등의 작업을 하는 부분을 처리합니다.

- 의존성 주입(DI) 예시:

  ```typescript
  import { Controller } from "@nestjs/common";
  import { BoardsService } from "./boards.service";

  @Controller("boards")
  export class BoardsController {
    boardsService: BoardsService;

    constructor(boardsService: BoardsService) {
      this.boardsService = boardsService;
    }
  }
  ```

  또는, 아래와 같이 `private` 키워드로 사용할 수 있습니다.

  ```typescript
  import { Controller } from "@nestjs/common";
  import { BoardsService } from "./boards.service";

  @Controller("boards")
  export class BoardsController {
    constructor(private boardsService: BoardsService) {}
  }
  ```

## DTO란?

계층간 데이터 교환을 위한 객체입니다. DB에서 데이터를 얻어 Service나 Controller 등으로 보낼 때 사용하는 객체를 말합니다. DTO는 데이터가 네트워크를 통해 전송되는 방법을 정의하는 객체입니다. Interface나 Class를 이용해서 정의 될 수 있으나, NestJs에서는 Class를 사용하는 것을 추천하고 있습니다. Class는 Interface와 다르게 런타임에서 작동하기 때문에 파이프 같은 기능을 이용할 때 더 유용합니다.

DTO의 사용은 데이터 유효성을 체크하는데 효율적이며 더 안정적인 코드로 만들어 줍니다.

## 생성 명령어

- **Board 모듈 생성**: `nest g module boards`
- **Board 컨트롤러 생성**: `nest g controller boards --no-spec`: 기본적으로 테스트를 위한 spec을 생성해줍니다.
- **Board 서비스 생성**: `nest g service boards --no-spec`

## Pipe란?

파이프는 @Injectable() 데코레이터로 주석이 달린 클래스입니다. 파이프는 Data Transformation과 Data Validation을 위해서 사용 됩니다. 파이프는 컨트롤러 경로 처리기에 의해 처리되는 인수에 대해 작동합니다. Nest는 메소드가 호출되기 직전에 파이프를 삽입하고 파이프는 메소드로 향하는 인수를 수신하고 이에 대해 작동합니다.

### Data Transformation

입력 데이터를 원하는 형식으로 변환합니다. 만약 숫자를 받길 원하는데 문자열 형식으로 온다면 파이프에서 자동으로 숫자로 바꿔줍니다.

### Data Validation

입력 데이터를 평가하고 유효한 경우 변경되지 않은 상태로 전달하면 됩니다. 그렇지 않으면 데이터가 올바르지 않을 때 예외를 발생시킵니다. 만약 이름의 길이가 10자 이하여야 하는데 10자를 초과하면 에러를 발생시킵니다.

### 파이프 사용하는 법

- Handler-level Pipes

  ```typescript
  @Post()
  @UsePipes(pipe)
  createBoard(@Body('title') title, @Body('description') description) {}
  ```

  핸들러 레벨에서의 파이프는 모든 파라미터에 적용됩니다. (title, description)

- Parameter-level Pipes

  ```typescript
  @Post()
  createBoard(@Body('title', ParameterPipe) title, @Body('description') description) {}
  ```

  파라미터 레벨에서의 파이프는 특정한 파라미터에게만 적용이 되는 파이프 입니다. (title)

- Global Pipes

  ```typescript
  async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    app.useGlobalPipes(GlobalPipes);
    await app.listen(3000);
  }
  bootstrap();
  ```

  글로벌 레벨에서의 파이프는 애플리케이션 전체에 대한 파이프입니다. 클라이언트에서 들어오는 모든 요청에 적용이 됩니다. 가장 상단 영역인 main.ts에 넣어주시면 됩니다.

### Built-in Pipes

NestJs에 기본적으로 사용할 수 있게 만들어 놓은 6가지의 파이프가 있습니다.

- ValidationPipe
- ParseIntPipe
- ParseBoolPipe
- ParseArraryPipe
- ParseUUIDPipe
- DefaultValuePipe

```typescript
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {}
```

### 파이프 사용예시

```shell
yarn add class-validator class-transformer
```

class-validator, class-transformer 모듈을 설치합니다.

```typescript
// controller
@Post()
@UsePipes(ValidationPipe)
createBoard(@Body() createBoardDto: CreateBoardDto): Board {
   return this.boardsService.createBoard(createBoardDto);
}

// dto
import { IsNotEmpty } from 'class-validator';

export class CreateBoardDto {
  @IsNotEmpty()
  title: string;

  @IsNotEmpty()
  description: string;
}
```

컨트롤러에 ValidationPipe를 적용해주고, [_class-validator 문서_](https://github.com/typestack/class-validator?tab=readme-ov-file#validation-decorators){:target="\_blank"}를 보고 필요한 데코레이터를 DTO에 적용해줍니다.

### 커스텀 파이프 구현 방법

먼저 Pipe Transform이란 인터페이스를 새롭게 만들 커스텀 파이프에 구현해줘야 합니다. 이 Pipe Treansform 인터페이스는 모든 파이프에서 구현해줘야 하는 인터페이스입니다. 그리고 이것과 함께 모든 파이프는 transform() 메소드를 필요로 합니다. 이 메소드는 NestJs가 인자(arguments)를 처리하기 위해서 사용됩니다.

```typescript
// pipe
import { ArgumentMetadata, PipeTransform } from "@nestjs/common";

export class BoardStatusValidationPipe implements PipeTransform {
  transform(valude: any, metadata: ArgumentMetadata) {
    console.log("value", value);
    console.log("metadata", metadata);

    return value;
  }
}

// controller
@Patch('/:id/status')
updateBoardStatus(
   @Param('id') id: string,
   @Body('status', BoardStatusValidationPipe) status: BoardStatus,
): Board {
   return this.boardsService.updateBoardStatus(id, status);
}
```

```text
value PRIVATE
metadata { metatype: [Function: String], type: 'body', data: 'status' }
```

- **transform() 메소드**: 이 메소드는 두 개의 파라미터를 가집니다. 첫번째는 처리가 된 인자의 값이며, 두번째는 인자에 대한 메타 데이터를 포함한 객체입니다. transform() 메소드에서 return 된 값이 Route 핸들러로 전달됩니다. 만약 예외가 발생하면 클라이언트에 바로 전달됩니다.

```typescript
import {
  ArgumentMetadata,
  BadRequestException,
  PipeTransform
} from "@nestjs/common";
import { BoardStatus } from "../board.model";

export class BoardStatusValidationPipe implements PipeTransform {
  readonly StatusOptions = [BoardStatus.PRIVATE, BoardStatus.PUBLIC];

  transform(value: any, metadata: ArgumentMetadata) {
    value = value.toUpperCase();

    if (!this.isStatusValid(value)) {
      throw new BadRequestException(`${value} isn't in the status options`);
    }
    return value;
  }

  private isStatusValid(status: any) {
    const index = this.StatusOptions.indexOf(status);
    return index !== -1;
  }
}
```

위 코드는 status 값을 검증하는 커스텀 파이프 예시입니다.
