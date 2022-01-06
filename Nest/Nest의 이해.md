> [Nest 공식 문서](https://docs.nestjs.com)

## FUNDAMENTALS
### 1. Custom providers
In earlier chapters, we touched on various aspects of Dependency Injection (DI) and how it is used in Nest. One example of this is the constructor based **dependency injection used to inject instances (often service providers) into classes.**

인스턴스를 클래스에 inject한다는 것은, Nest에서 service를 만드는 순간 자동으로 인스턴스화를 시킨다. 왜냐면 service에 포함된 메소드들은 클래스가 아닌 인스턴스에 저장되기 때문이다. 따라서 inject를 할 때에도 실제 역할을 하는 인스턴스를 받아야하기 때문에 inject instances (often service providers) into classes라는 말을 쓴 것이다. 더 나아가서 `@Inject('커스텀 토큰')` 에 사용되는 문자열(커스텀 토큰)은 생략 가능하지만 인터페이스를 inject하는 경우에는 반드시 써줘야한다. 커스텀 토큰을 무조건 사용하기 때문이다. 문자열의 값이 없으면 기본적으로 클래스의 네임으로 찾고, 커스텀 토큰이 있으면 그걸로 값을 찾는다. 커스텀 토큰은 우리가 직접 이름 붙인 것이고, 그 이름으로 호출 가능하다. 

+ class : cat (cat이 하는 동작을 수행 e.g.뛰기, 밥먹기)
+ instance : 먼지 (new cat) new로 호출하여 만든 것



#### DI fundamentals
종속성 주입은 [IoC : Inversion of Control](https://en.wikipedia.org/wiki/Inversion_of_control) 기술이다. 자신의 코드에서 명령적으로 수행하는 대신 종속성 인스턴스화를 IoC 컨테이너(이 경우에는 NestJS 런타임 시스템)에 위임하는 것이다.

1. 먼저 공급자를 정의한다. `@Injectable()` 데코레이터에는 `CatsService`가 공급자로서의 클래스라는 걸 알려준다.
**cats.service.ts**
```ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

// cat을 찾아주는 함수
@Injectable() // 다른 provider에 주입이 가능하다 => 즉, 공급자로서의 역할을 한다.
export class CatsService {
  private readonly cats: Cat[] = [];

  findAll(): Cat[] {
    return this.cats;
  }
}
```

2. 그 다음, Nest가 provider를 controller class에 주입하도록 시킨다.

**cats.controller.ts**
```ts
import { Controller, Get } from '@nestjs/common';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  // cat을 찾아주는 함수인 service를 생성자로 가져와서 연결해주고
  constructor(private catsService: CatsService) {}
  
  // http:GET 요청을 해준다.
  @Get()
  async findAll(): Promise<Cat[]> {

      CatsService
    return this.catsService.findAll();
  }
}
```

3. 마지막으로 Nest IoC container에 공급자(provider)를 등록한다.

**app.module.ts**
```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```


#### 그래서 Nest는 어떻게 Ioc를 자동으로 해주는 걸까?
1. `cats.service.ts`에서, `@Injectable()` decorator는 `CatsService` class가 Nest IoC container에 의해 관리될 수 있다는 걸 정의한다.
2. `cats.controller.ts`에서, `CatsController`는 생성자에 주입됨으로써 `CatService token`에 의존하고 있다는 걸 보여준다.
```ts
constructor(private catsService: CatsService) 
```
3. `app.module.ts`에서, `CatsService` 토큰을 `cats.service.ts file`의 `CatsService class`와 연결합니다. 


### 2. Asynchronous providers
Nest에서 비동기 작업은 `useFactory` 구문과 함께 작업할 수 있다. 팩토리는 `Promise`를 반환하고 팩토리 함수는 비동기 작업을 `await` 할 수 있다. Nest는 프로바이더에 의존하는(주입하는) 클래스를 인스턴스화하기 전에 promise의 해결을 기다린다.

```ts
{
  provide: 'ASYNC_CONNECTION',
  useFactory: async () => {
    const connection = await createConnection(options);
    return connection;
  },
}
```

#### Injection
비동기 providers는 다른 프로바이더와 마찬가지로 토큰에 의해 다른 컴포넌트에 주입될 수 있다. 위의 예시에서 보자면, 이런 구문을 사용한다.
`@Inject('ASYNC_CONNECTION')`

**e.g.**
`abc.service.ts`
```ts
@Injectable()
export class ABCService {
  constructor(
    @Inject('BCDService') private bcdService: BCDService, 
    @Inject('DEFService')
    private defService: DEFService,
  ) {}
```

즉, controller - service 뿐만 아니라 service - service 도 서로 의존성 주입이 가능하다. 
대신 비동기 함수라면 `@Inject()` 를 사용하여 `constructor` 에 넣어준다.

### Interface
비동기 함수 뿐만 아니라 `Interface`를 주입할 때도 `@Inject()`를 쓴다. 대신 인터페이스의 경우 위에 기술했던대로 무조건 커스텀 토큰을 사용해야 된다.

> extends : 클래스의 확장, implement : 인터페이스 구현

