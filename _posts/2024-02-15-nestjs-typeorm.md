---
title: NestJs TypeORM
author: kjb4494
date: 2024-02-15 16:00:00 +0900
categories: [Backend, NestJs]
tags: [nestjs, backend, typeorm]
---

## ORM이란?

객체와 관계형 데이터베이스의 데이터를 자동으로 변형 및 연결하는 작업입니다. ORM을 이용한 개발은 객체와 데이터베이스의 변형에 유연하게 사용할 수 있습니다.

## TypeORM이란?

TypeORM은 node.js에서 실행되고 typescript로 작성된 객체 관계형 매퍼 라이브러리입니다. TypeORM은 MySQL, PostgreSQL, MariaDB, SQLite, MS SQL Server, Oracle, SAP Hana 및 WebSQL과 같은 여러 데이터베이스를 지원합니다.

### Pure Javascript와의 코드 비교

- TypeORM

  ```typescript
  const boards = Board.find({ title: "Hello", status: "PUBLIC" });
  ```

- Pure Javascript
  ```javascript
  db.query(
    'SELECT * FROM boards WHERE title = "Hello" AND status = "PUBLIC"',
    (err, result) => {
      if (err) {
        throw new Error("Error");
      }
      boards = result.rows;
    }
  );
  ```

### 특징과 이점

- 모델을 기반으로 데이터베이스 테이블 체계를 자동으로 생성합니다.
- 데이터베이스에서 개체를 쉽게 삽입, 업데이트 및 삭제할 수 있습니다.
- 테이블 간의 매핑을 만듭니다.
- 간단한 CLI 명령을 제공합니다.
- 간단한 코딩으로 ORM 프레임워크를 사용하기 쉽습니다.
- 다른 모듈과 쉽게 통합됩니다.

## 설치 모듈

- **@nestjs/typeorm**: NestJs에서 TypeORM을 사용하기 위해 연동시켜주는 모듈
- **typeorm**: TypeORM 모듈
- **pg**: Postgres 모듈

```shell
yarn add pg typeorm @nestjs/typeorm
```

[_공식문서_](https://docs.nestjs.com/techniques/database){:target="\_blank"}

## 설정

```typescript
// configs/typeorm.config.ts
import { TypeOrmModuleOptions } from "@nestjs/typeorm";

export const typeORMConfig: TypeOrmModuleOptions = {
  type: "postgres",
  host: "localhost",
  port: 5432,
  username: "postgres",
  password: "P@ssw0rd",
  database: "board-app",
  entities: [__dirname + "/../**/*.entity.{js,ts}"],
  synchronize: true
};

// app.module.ts
import { Module } from "@nestjs/common";
import { BoardsModule } from "./boards/boards.module";
import { TypeOrmModule } from "@nestjs/typeorm";
import { typeORMConfig } from "./configs/typeorm.config";

@Module({
  imports: [TypeOrmModule.forRoot(typeORMConfig), BoardsModule],
  controllers: [],
  providers: []
})
export class AppModule {}
```

설정파일을 작성해주고 애플리케이션 모듈에 import 시켜줍니다.

## Entity 생성하기

### 엔티티 생성 소스 코드

```typescript
import { BaseEntity, Column, Entity, PrimaryGeneratedColumn } from "typeorm";
import { BoardStatus } from "./board.model";

@Entity()
export class Board extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  description: string;

  @Column()
  status: BoardStatus;
}
```

- **@Entity()**: Board 클래스가 엔티티임을 나타내는 데 사용됩니다.
- **@PrimaryGeneratedColumn()**: id 열이 Board 엔티티의 기본 키 열임을 나타내는 데 사용됩니다.
- **Column()**: Board 엔티티의 title 및 description과 같은 다른 열을 나타내는 데 사용됩니다.

## Repository란?

리포지토리는 엔티티 개체와 함께 작동하며 엔티티 찾기, 삽입, 삭제 등을 처리합니다.

[_공식 문서_](https://typeorm.delightful.studio/classes/_repository_repository_.repository)

## CRUD 예제 코드

### Module

```typescript
import { Module } from "@nestjs/common";
import { BoardsController } from "./boards.controller";
import { BoardsService } from "./boards.service";
import { TypeOrmModule } from "@nestjs/typeorm";
import { Board } from "./board.entity";

@Module({
  imports: [TypeOrmModule.forFeature([Board])],
  controllers: [BoardsController],
  providers: [BoardsService]
})
export class BoardsModule {}
```

### Service

```typescript
import { Injectable, NotFoundException } from "@nestjs/common";
import { CreateBoardDto } from "./dto/create-board.dto";
import { Board } from "./board.entity";
import { InjectRepository } from "@nestjs/typeorm";
import { Repository } from "typeorm";
import { BoardStatus } from "./board-status.enum";

@Injectable()
export class BoardsService {
  constructor(
    @InjectRepository(Board)
    private boardRepository: Repository<Board>
  ) {}

  async getAllBoards(): Promise<Board[]> {
    return this.boardRepository.find();
  }

  async createBoard(createBoardDto: CreateBoardDto): Promise<Board> {
    const { title, description } = createBoardDto;
    const newBoard = this.boardRepository.create({
      title,
      description,
      status: BoardStatus.PUBLIC
    });
    await this.boardRepository.save(newBoard);
    return newBoard;
  }

  async getBoardById(id: number): Promise<Board> {
    const board = await this.boardRepository.findOneBy({ id });
    if (!board) {
      throw new NotFoundException(`Can't found Board with id ${id}`);
    }
    return board;
  }

  async deleteBoard(id: number): Promise<void> {
    await this.getBoardById(id);
    await this.boardRepository.delete(id);
  }

  async updateBoardStatus(id: number, status: BoardStatus): Promise<Board> {
    const board = await this.getBoardById(id);
    board.status = status;
    await this.boardRepository.save(board);
    return board;
  }
}
```

### Controller

```typescript
import {
  Body,
  Controller,
  Delete,
  Get,
  Param,
  Patch,
  Post,
  UsePipes,
  ValidationPipe
} from "@nestjs/common";
import { BoardsService } from "./boards.service";
import { CreateBoardDto } from "./dto/create-board.dto";
import { BoardStatusValidationPipe } from "./pipes/board-status-validation.pipe";
import { Board } from "./board.entity";
import { BoardStatus } from "./board-status.enum";

@Controller("boards")
export class BoardsController {
  constructor(private boardsService: BoardsService) {}

  @Get()
  async getAllBoards(): Promise<Board[]> {
    return this.boardsService.getAllBoards();
  }

  @Post()
  @UsePipes(ValidationPipe)
  async createBoard(@Body() createBoardDto: CreateBoardDto): Promise<Board> {
    return this.boardsService.createBoard(createBoardDto);
  }

  @Get("/:id")
  async getBoardById(@Param("id") id: number): Promise<Board> {
    return this.boardsService.getBoardById(id);
  }

  @Delete("/:id")
  async deleteBoard(@Param("id") id: number): Promise<void> {
    await this.boardsService.deleteBoard(id);
  }

  @Patch("/:id/status")
  async updateBoardStatus(
    @Param("id") id: number,
    @Body("status", BoardStatusValidationPipe) status: BoardStatus
  ): Promise<Board> {
    return this.boardsService.updateBoardStatus(id, status);
  }
}
```
