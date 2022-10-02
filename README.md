# NEST.JS

## Installation

```
npm i -g @nestjs/cli
nest new project-name
```

## Modules

A module is a class annotated with a @Module() decorator. The @Module() decorator provides metadata that Nest makes use of to organize the application structure.

### App Module

**app.module.ts** is the root module of the application. That will import other modules.

### Creating a New Module

#### Manually

```typescript
// auth/auth.module.ts

import { Module } from '@nestjs/common'

@Module({})
export class AuthModule {}
```

#### Using the CLI

```
nest g module auth
```

### Importing a Module

```typescript
// app.module.ts

import { Module } from '@nestjs/common'
import { AuthModule } from './auth/auth.module'

@Module({
  imports: [AuthModule],
})
export class AppModule {}
```

## Providers

Providers are responsible for the business logic of the application.

### Creating a Provider

#### Manually

```typescript
// auth/auth.service.ts

import { Injectable } from '@nestjs/common'

@Injectable()
export class AuthService {}
```

#### Using the CLI

```
nest g service auth
nest g service auth --no-spec       -> without test files
```

## Controllers

Controllers are responsible for handling incoming requests and returning responses to the client.

### Creating a Controller

#### Manually

```typescript
// auth/auth.controller.ts

import { Controller } from '@nestjs/common'

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}
}
```

#### Using the CLI

```
nest g controller auth
```

## Using Controller & Providers

```typescript
// auth/auth.module.ts

import { Module } from '@nestjs/common'
import { AuthController } from './auth.controller'
import { AuthService } from './auth.service'

@Module({
  controllers: [AuthController],
  providers: [AuthService],
})
export class AuthModule {}
```

## Creating a API

```typescript
// auth/auth.controller.ts

import { Controller, Post } from '@nestjs/common'
import { AuthService } from './auth.service'

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('signup')
  signup() {
    return 'Signup'
  }
  @Post('signin')
  signin() {
    return { msg: 'Signin' }
  }
}
```

## Handling Business Logic

```typescript
// auth/auth.service.ts

import { Injectable } from '@nestjs/common'

@Injectable()
export class AuthService {
  signup() {
    return 'Signup'
  }

  signin() {
    return { msg: 'Signin' }
  }
}
```

```typescript
// auth/auth.controller.ts

import { Controller, Post } from '@nestjs/common'
import { AuthService } from './auth.service'

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('signup')
  signup() {
    return this.authService.signup()
  }
  @Post('signin')
  signin() {
    return this.authService.signin()
  }
}
```

## Setting Up Database In Docker

Docker allows us to run database directly on our computer without installing it.

```yml
// docker-compose.yml

version: '3.8'
services:
  dev-db:
    image: postgres:13
    ports:
      - 5434:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 123
      POSTGRES_DB: test
    networks:
      - mynetwork
networks:
  mynetwork:
```

```
docker compose up dev-db -d
```

## Setting Up Prisma

```
npm -D prisma
npm i @prisma/client
npx prisma init
```

After that change the env

### Creating a Prisma Model

```prisma
model User {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  email String @unique
  hash  String
  firstName String?
  lastName  String?
}
```

### Runing Prisma Migrations

```
npx prisma migrate dev
```

With migrate npx primsa generate is automatically run, which create typescript interfaces for models.<br>
npx primsa studio can be used to inspect the database.

### Creating Prisma Module

```typescript
// prisma/prisma.module.ts

import { Injectable } from '@nestjs/common'
import { PrismaClient } from '@prisma/client'

@Injectable()
export class PrismaService extends PrismaClient {
  constructor(config: ConfigService) {
    super({
      datasources: {
        db: {
          url: dbURL,
        },
      },
    })
  }
}
```

```typescript
// prisma/prisma.module.ts

import { Module } from '@nestjs/common'
import { PrismaService } from './prisma.service'

@Module({
  providers: [PrismaService],
})
export class PrismaModule {}
```

### Using the Prisma Service

#### Using Imports

```typescript
// prisma/prisma.module.ts

import { Module } from '@nestjs/common'
import { PrismaService } from './prisma.service'

@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

```typescript
// auth/auth.module.ts

import { Module } from '@nestjs/common'
import { PrismaModule } from '../prisma/prisma.module.ts'
import { AuthController } from './auth.controller'
import { AuthService } from './auth.service'

@Module({
  imports: [PrismaModule],
  controllers: [AuthController],
  providers: [AuthService],
})
export class AuthModule {}
```

#### Using Global

```typescript
// prisma/prisma.module.ts

import { Global, Module } from '@nestjs/common'
import { PrismaService } from './prisma.service'

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

```typescript
// auth/auth.module.ts

import { Module } from '@nestjs/common'
import { AuthController } from './auth.controller'
import { AuthService } from './auth.service'

@Module({
  controllers: [AuthController],
  providers: [AuthService],
})
export class AuthModule {}
```

## Getting Req, Res & Body

### Using The Express Way

```typescript
// auth/auth.controller.ts

@Post('signup')
signup(@Req() req: Request) {
 console.log(req.body)
 return this.authService.signup()
}
```

### Using DTO (Data Transfer Object)

```typescript
// auth/dto/auth.dto.ts

export interface AuthDto {
  email: string
  password: string
}
```

```typescript
// auth/dto/index.ts

export * from './auth.dto'
```

```typescript
// auth/auth.controller.ts

@Post('signup')
signup(@Body() dto: AuthDto) {
 console.log(dto)
 return this.authService.signup()
}
```

### NestJs Pipes

Pipes can be used to transform or validate the input data

```typescript
@Post('signup')
signup(@Body('password', ParseIntPipe) password: string) {
 console.log(password)
}
```

### Applying Validations

```
npm i class-validator class-transformer
```

```typescript
// auth/dto/auth.dto.ts

import { IsEmail, IsNotEmpty, IsString } from 'class-validator'

export class AuthDto {
  @IsEmail()
  @IsNotEmpty()
  email: string

  @IsString()
  @IsNotEmpty()
  password: string
}
```

```typescript
// main.ts

import { ValidationPipe } from '@nestjs/common'
import { NestFactory } from '@nestjs/core'
import { AppModule } from './app.module'

async function bootstrap() {
  const app = await NestFactory.create(AppModule)
  app.useGlobalPipes(new ValidationPipe())
  await app.listen(5000)
}
bootstrap()
```

To block input data other any dto

```typescript
// main.ts

import { ValidationPipe } from '@nestjs/common'
import { NestFactory } from '@nestjs/core'
import { AppModule } from './app.module'

async function bootstrap() {
  const app = await NestFactory.create(AppModule)
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
    })
  )
  await app.listen(5000)
}
bootstrap()
```
