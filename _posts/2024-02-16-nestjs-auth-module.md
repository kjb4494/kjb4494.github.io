---
title: NestJs 인증 기능 구현하기
author: kjb4494
date: 2024-02-16 16:00:00 +0900
categories: [Backend, NestJs]
tags: [nestjs, backend]
---

## 모듈, 컨트롤러, 서비스 생성

```shell
# 모듈 생성
nest g module auth

# 컨트롤러 생성
nest g controller auth --no-spec

# 서비스 생성
nest g service auth --no-spec
```

## 엔티티 생성

### 유저 이름에 유니크한 값 주기

유저 이름을 유니크로 만드는 데는 두 가지 방법이 있습니다.

1. 서비스 레벨에서 검색 후 에러 처리하기
1. 데이터베이스 레벨에서 유니크 키를 설정하여 에러 처리하기

```typescript
import {
  BaseEntity,
  Column,
  Entity,
  PrimaryGeneratedColumn,
  Unique
} from "typeorm";

@Entity()
@Unique(["username"])
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  password: string;
}
```

두번째 방법을 이용하여 엔티티에서 `username` 칼럼에 유니크 키를 설정하도록 했습니다.

```typescript
async signup(authCredentialsDto: AuthCredentialsDto): Promise<void> {
	const { username, password } = authCredentialsDto;
	const user = this.userRepository.create({ username, password });
	// Todo: Needs improvement
	try {
		await this.userRepository.save(user);
	} catch (error) {
		if (error.code === '23505') {
			throw new ConflictException('Existing username');
		} else {
			throw new InternalServerErrorException();
		}
	}
}
```

만약 유니크 키가 있는 칼럼에 이미 존재하는 값을 입력하려고 하면 23505 에러 코드가 발생합니다. 별도의 처리가 없을 경우 500 에러가 발생하므로 예외 처리를 해줘야합니다.

## 비밀번호 암호화 하기

### bcryptjs

비밀번호 암호화 기능을 구현하기 위해서 bcryptjs 라는 모듈을 설치하여 사용하겠습니다.

```shell
yarn add bcryptjs @types/bcryptjs
```

```typescript
import * as bcrypt from "bcryptjs";

async signup(authCredentialsDto: AuthCredentialsDto): Promise<void> {
    const { username, password } = authCredentialsDto;

    const salt = await bcrypt.genSalt();
    const hashedPassword = await bcrypt.hash(password, salt);
    const user = this.userRepository.create({
      username,
      password: hashedPassword,
    });
  }
```

랜덤 솔트를 생성한 다음, 솔트와 함께 해싱한 문자열을 비밀번호 값으로 저장해주면 됩니다.

## JWT를 이용해서 토큰 생성하기

```shell
yarn add @nestjs/jwt @nestjs/passport passport passport-jwt @types/passport-jwt
```

```typescript
import { Module } from "@nestjs/common";
import { AuthController } from "./auth.controller";
import { AuthService } from "./auth.service";
import { TypeOrmModule } from "@nestjs/typeorm";
import { User } from "./user.entity";
import { JwtModule } from "@nestjs/jwt";
import { PassportModule } from "@nestjs/passport";

@Module({
  imports: [
    PassportModule.register({ defaultStrategy: "jwt" }),
    JwtModule.register({
      secret: "secretkey1234",
      signOptions: {
        expiresIn: 60 * 60
      }
    }),
    TypeOrmModule.forFeature([User])
  ],
  controllers: [AuthController],
  providers: [AuthService]
})
export class AuthModule {}
```

필요한 모듈을 설치해주고 모듈 설정을 해줍니다.

- **Secret**: 토큰을 만들 때 이용하는 시크릿 키 입니다. 아무 텍스트나 넣어줘도 되지만 너무 짧을 경우 보안에 취약할 수 있습니다.
- **ExpiresIn**: 정해진 시간 이후에는 토큰이 유효하지 않게 됩니다. 60 \* 60은 한시간 이후에는 이 토큰이 더 이상 유효하지 않게 됩니다.

```typescript
async login(
	authCredentialsDto: AuthCredentialsDto,
): Promise<{ accessToken: string }> {
	const { username, password } = authCredentialsDto;
	const user = await this.userRepository.findOneBy({ username });

	if (user && (await bcrypt.compare(password, user.password))) {
		// Generate user token
		const payload = { id: user.id, username };
		const accessToken = this.jwtService.sign(payload);
		return { accessToken };
	} else {
		throw new UnauthorizedException('login failed');
	}
}
```

인증 서비스 레이어에서 JwtService 의존성을 주입 받은 후, 로그인 로직에서 엑세스 토큰을 생성해 클라이언트에게 전달해주면 됩니다.

## Guards

NestJs에는 여러가지 미들웨어가 있습니다.

- **Pipes**: 파이프는 요청 유효성 검사 및 페이로드 변환을 위해 만들어집니다.
- **Filters**: 필터는 오류 처리 미들웨어입니다. 특정 오류 처리기를 사용할 경로와 각 경로 주변의 복잡성을 관리하는 방법을 알 수 있습니다.
- **Guards**: 가드는 인증 미들웨어입니다. 지정된 경로로 통과할 수 있는 사람과 허용되지 않는 사람을 서버에 알려줍니다.
- **Interceptors**: 인터셉터는 응답 매핑 및 캐시 관리와 함께 요청 로깅과 같은 전후 미들웨어입니다. 각 요청 전후에 이를 실행하는 기능은 매우 강력하고 유용합니다.

각각의 미들웨어가 불러지는 순서는 다음과 같습니다.

```text
middleware -> gaurd -> interceptor (before) -> pipe -> controller -> service -> controller -> interceptor (after) -> filter (if apllicable) -> client
```

인증 미들웨어를 구현하기 위해 Guards 사용법을 알아보겠습니다.

### PassportStrategy 구현

```typescript
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { ExtractJwt, Strategy } from "passport-jwt";
import { Repository } from "typeorm";
import { User } from "./user.entity";
import { InjectRepository } from "@nestjs/typeorm";

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {
    super({
      secretOrKey: "secretkey1234",
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken()
    });
  }

  async validate(payload) {
    const { id, username } = payload;
    const user: User = await this.userRepository.findOneBy({ id });

    if (!user) {
      throw new UnauthorizedException();
    }

    return user;
  }
}
```

super()에서 `secretOrKey` 값은 `JwtModule.register()`에서 설정했던 시크릿 키 값을 넣어줍니다. Authorization 헤더의 Bearer 토큰으로 전달 받기 위해 `jwtFromRequest`는 `ExtjwtFromRequestractJwt.fromAuthHeaderAsBearerToken()`로 설정해줍니다.

### 모듈 설정

```typescript
@Module({
	...
  providers: [..., JwtStrategy],
  exports: [PassportModule],
})
export class AuthModule {}

```

모듈에서 JwtStrategy를 프로바이더로 등록해주고 다른 모듈에서도 사용 가능하도록 PassportModule을 exports에 등록해줍니다.

### 테스트

```typescript
@Get('/test')
@UseGuards(AuthGuard())
test(@Req() req) {
	console.log(req.user);
}
```

컨트롤러에서 `@UseGuards(AuthGuard())`를 사용하면 PassportStrategy에서 인증 로직을 수행해 validate 메소드에서 return한 User 객체가 req 속성에 추가됩니다.

```shell
curl -X GET -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwidXNlcm5hbWUiOiJ0ZXN0ZXI0IiwiaWF0IjoxNzA4MDY0ODY0LCJleHAiOjE3MDgwNjg0NjR9.Y0TrjpRdxIoyoCPDL7JkVVTPScIVqJCsMII_a4QZLpU" http://localhost:3000/auth/test
```

```text
User {
  id: 5,
  username: 'tester4',
  password: '$2a$10$g4cJ4VtgBRAhzfri73XgZu/UVOBoL5283Th1NDxW8qrF3kTu71/rS'
}
```

### 커스텀 데코레이터 생성하기

클라이언트의 사용자 인증 정보를 얻을 때마다 `req.user`를 사용하는 것은 귀찮기 때문에 커스텀 데코레이터를 사용하여 바로 유저 정보를 가져올 수 있습니다.

```typescript
import { ExecutionContext, createParamDecorator } from "@nestjs/common";
import { User } from "./user.entity";

export const GetUser = createParamDecorator(
  (data, ctx: ExecutionContext): User => {
    const req = ctx.switchToHttp().getRequest();
    return req.user;
  }
);
```

```typescript
@Get('/test')
@UseGuards(AuthGuard())
test(@GetUser() user: User) {
	console.log(user);
}
```

단, 커스텀 데코레이터로 만든 GetUser는 유저 정보가 있다는 가정하에 동작하고 있기 때문에 Guards가 없는 컨트롤러에서는 사용하면 안 됩니다.

### 다른 모듈에서 인증 모듈 사용하기

1. 인증 모듈을 imports에 등록해줍니다.

   ```typescript
   import { Module } from "@nestjs/common";
   import { BoardsController } from "./boards.controller";
   import { BoardsService } from "./boards.service";
   import { TypeOrmModule } from "@nestjs/typeorm";
   import { Board } from "./board.entity";
   import { AuthModule } from "src/auth/auth.module";

   @Module({
     imports: [TypeOrmModule.forFeature([Board]), AuthModule],
     controllers: [BoardsController],
     providers: [BoardsService]
   })
   export class BoardsModule {}
   ```

1. 컨트롤러에서 `@UseGuards` 데코레이터를 사용합니다.

   ```typescript
   @Controller('boards')
   @UseGuards(AuthGuard())
   export class BoardsController {
   	constructor(private boardsService: BoardsService) {}

   	...
   }
   ```
