> [Dependency injection in Angular](https://angular.io/guide/dependency-injection) 문서를 기반으로 의존성 주입 concept에 대해 이해합니다.

Nest is built around the strong design pattern commonly known as Dependency injection in which a class requests dependencies from external sources rather than creating them.

### Dependency injection
+ Dependency : 어떤 함수, 클래스 등이 내부에 다른 함수, 클래스를 사용함
+ Injection : 어떤 함수, 클래스 등이 내부에 사용하는 다른 함수, 클래스를 내부에서 생성하는 것이 아니라 외부에서 생성하여 넣어주는 것
+ How? 의존하고 있는 코드를 클래스에서 생성자로 받아오면 된다

서비스(provider)를 컴포넌트/서비스에 주입하려고 한다면 필요로 하는 프로바이더를 생성자에 `타입 정의`로 작성할 수 있다.


#### src/app/heroes/hero.service.ts
```ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root', // means that the HeroService is visible throughout the application
})
export class HeroService {
  constructor() { }
}
```

The `@Injectable()` decorator specifies that Angular can use this class in the DI system.

다음으로, hero 데이터를 얻기 위해서 `getHeroes()` 메소드를 추가한다. 
```ts
import { Injectable } from '@angular/core';
import { HEROES } from './mock-heroes';

@Injectable({
  // declares that this service should be created
  // by the root application injector.
  providedIn: 'root',
})
export class HeroService {
  getHeroes() { return HEROES; }
}
```

### Injecting services
`To inject a dependency in a component's constructor(), supply a constructor argument with the dependency type. `

**종속성 주입을 하기 위해선, constructor 안에 타입을 정의해야한다.**

The following example specifies the HeroService in the HeroListComponent constructor. The type of heroService is HeroService.

#### src/app/heroes/hero-list.component
```ts
constructor(heroService: HeroService)
```

### Using services in other services
When a service depends on another service, follow the same pattern as injecting into a component. In the following example `HeroService` depends on a `Logger` service to report its activities.

First, import the `Logger` service. Next, inject the `Logger` service in the `HeroService` `constructor()` by specifying `private logger: Logger` within the parentheses.

When you create a class whose `constructor()` has parameters, specify the type and metadata about those parameters so that Angular can inject the correct service.

Here, the `constructor()` specifies a type of `Logger` and stores the instance of Logger in a private field called logger.

The following code tabs feature the `Logger` service and two versions of `HeroService`. The first version of 

#### src/app/heroes/hero.service (v2) 
`HeroService` does depend on the `Logger` service.

```ts
import { Injectable } from '@angular/core';
import { HEROES } from './mock-heroes';
import { Logger } from '../logger.service';

@Injectable({
  providedIn: 'root',
})
export class HeroService {
  // `HeroService`가 `Logger`서비스에 의존하고 있다면, 
  // `HeroService` 클래스의 `constructor`에 `Logger`를 타입 정의해준다. 
  constructor(private logger: Logger) {  }

  getHeroes() {
    this.logger.log('Getting heroes ...');
    return HEROES;
  }
}
```

#### src/app/heroes/hero.service (v1)
`HeroService` does **not** depend on the `Logger` service.
```ts
import { Injectable } from '@angular/core';
import { HEROES } from './mock-heroes';

@Injectable({
  providedIn: 'root',
})
export class HeroService {
  getHeroes() { return HEROES; }
}
```

#### src/app/logger.service
```ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class Logger {
  logs: string[] = []; // capture logs for testing

  log(message: string) {
    this.logs.push(message);
    console.log(message);
  }
}
```
In this example, the `getHeroes()` method uses the `Logger` service by logging a message when fetching heroes.

### Dependency Inversion Principle(DIP)
HeroService는 Logger에 대해 의존성을 띄고 있다.
Logger 클래스를 생성자 안에서 쓰고있기 때문이다.

여기서, Logger가 아니라 의존성을 가지는 HeroService에서 Logger를 설정하는 관계가 되는 의존 관계의 역전을 볼 수 있다.

#### 의존 역전 원칙(Dependency Inversion Principle) 정의
고수준 모듈은 저수준 모듈의 구현에 의존해서는 안 된다. 저수준 모듈이 고수준 모듈에서 정의한 추상 타입에 의존해야 한다. 즉, "자신보다 변하기 쉬운 것에 의존하지 마라"라고 이해하면 편하다.

#### 고수준 모듈과 저수준 모듈
DIP의 대표적인 예제인 자동차와 스노우타이어가 있다. 현재는 겨울이기 때문에 스노우 타이어를 구매하여 자동차에 끼도록 설계하였다. 즉, 고수준 모듈인 자동차가 저수준 모듈인 스노우 타이어에 의존하는 상태이다.

하지만, 날씨가 따뜻해지면서 더이상 스노우 타이어를 사용할 필요가 없어졌다. 그래서 일반 타이어로 교체하기로 결정했다. 그런데, 단순히 스노우 타이어를 일반 타이어로 바꾼다고 코드가 끝나는 것이 아니라 이것에 의존하고 있던 자동차의 코드도 연쇄적으로 영향을 끼치게 된다.

이것은 [개방-폐쇄 원칙](https://steady-coding.tistory.com/378)을 위반하는 것이므로 추상화나 다형성을 통해 문제를 해결해야 한다. 의존 역전 원칙은 추상화를 이용한다. 스노우 타이어나 일반 타이어를 '타이어' 자체로 추상화하는 것이다.

**추상화한 모습**
![](https://images.velog.io/images/usreon/post/6dc74b85-0213-4357-9604-74527ef73421/%E1%84%83%E1%85%A1%E1%84%8B%E1%85%AE%E1%86%AB%E1%84%85%E1%85%A9%E1%84%83%E1%85%B3.png)

여기서 타이어는 저수준 모듈보다는 고수준 모듈인 자동차 입장에서 만들어지는데, 이것은 고수준 모듈이 저수준 모듈에 의존했던 상황이 역전되어 저수준 모듈이 고수준 모듈에 의존하게 된다는 것을 의미한다. 이런 맥락에서 이 원칙의 이름은 의존 역전 원칙이다.

다만, 소스 코드의 의존은 자동차가 '타이어'를 의존하지만, 런타임에서의 객체 의존은 타이어가 아니라 하위 타이어 중 하나를 의존한다. 따라서, 의존 역전 원칙은 런타임에서의 의존을 역전시키는 것이 아니라 소스 코드 단계에서의 의존을 역전시킨다는 것을 유의해야 한다.


## 결합성을 낮춰주는 의존성 주입
### 의존성 주입이 없을 때
```js
import { Controller, Get, Post } from '@nestjs/common';
import { AppService } from './app.service';

@Controller('abc') 
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get('hello') 
  getHello(): string {
    return new AppService().getHello();
```

이미 코드에 `return` 값이 `new AppService().getHello()`로 고정되어있기 때문에, 테스트를 하려면 `return`값을 `new TestAppService().getHello();`로 변경해야 되는 번거로움이 있다.

또한 객체를 내부에서 만들어서 부모 객체에 의존하기 때문에 결합성이 강해져 단일 책임분리에 어긋난다. 따라서 `DI`를 활용해 객체를 밖에서 만들어 넘겨줌으로써 결합성을 낮춰줄 수 있다.

### DI 예시

하지만, DI를 적용해서 `this.appService.getHello()`를 쓴다면
```js
import { Controller, Get, Post } from '@nestjs/common';
import { AppService } from './app.service';
import { TestAppService } from './app.service';

@Controller('abc') 
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get('hello') 
  getHello(): string {
    return this.appService.getHello();
  }

new AppController(new AppService()); // 실제 환경
new AppController(new TestAppService()); // 테스트 환경
```
이런 식으로 코드를 작성하여 환경에 따라 테스트 할 수 있다. 코드의 재사용성과 활용성이 높아지는 장점이 있다.


#### 더보기
```js
function a (b,c) {
  return b + c;
}
```
조금 더 쉬운 예를 들자면, DI를 적용하지 않은 코드는 
`b + c`가 아닌 `5 + c`로 고정된 값을 받는 것과 같다.
`this.appService...`는 매개변수와 같은 역할을 해준다. 
`DI`를 통해서 인자가 고정되지 않고 매개변수를 활용하여 결합성을 낮춘다.

