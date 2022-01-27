# Custom route [decorators](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841)

## 사용자 지정 데코레이터 만들기

```js
const user = req.user;
```
node.js 세계에서는 이렇게 수동으로 추출했었다.

Nest에서는 코드를 더 읽기 쉽고 투명하게 만들기 위해 `@User()`데코레이터를 만들고 모든 컨트롤러에서 재사용할 수 있다. 
```ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator( // custom decorator를 만들기 위해선 createParamDecorator를 사용한.
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

커스텀 데코레이터를 만들었다면 요구 사항에 맞는 모든 곳에서 import하여 간단히 사용할 수 있다.
```ts
import { User } from '../helpers/decorators/custom-decorators';

@Get()
async findOne(@User() user: UserEntity) {
  console.log(user);
}


```
