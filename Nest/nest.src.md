# Nest.js
> 코드와 함께 Nest 개념에 대해 이해합니다.

## Controller
### 정적 path와 동적 path
```ts
@Controller('abc') 
```

```ts
@Controller({
  path: `${process.env.어쩌고}/prelabel`,
})
```
위와 아래의 차이는 정적인 path와 동적인 path의 차이가 있음과 동시에,
객체 형식으로 받기 때문에 다른 협업자가 코드를 봤을 때 충분히 확장 가능성 있다는 걸 알려주기 위함이다.
```ts
e.g. 
@Controller({
  path: `${process.env.어쩌고}/prelabel`,
  host: `${process.env.저쩌고}}/prelabel` // 확장 가능
})
```

## Constructor
프로젝트 코드를 보다가 private로 선언된 생성자들이 꽤 많다는 걸 알았다.
여러가지 이유가 있지만 그 중 하나는 생성자를 private로 선언하게되면, 외부에서 인스턴스 생성이 불가하기 때문에 협업자들 간의 인터페이스 규칙을 지켜주는 목적도 있다. 예를들어, 내가 서비스를 맡고 다른 개발자가 컨트롤러를 맡았다면 컨트롤러에서 내가 만든 서비스를 생성자로 가져올 때 그 규칙을 지켜주기 위함이다.

*생성자로 호출 후 `Factory`를 이용해서 작업을 진행한다.

외부에서 인스턴스를 또 생성하기 보다는 클래스의 method와 field를 static으로 선언하여 **변수로** 외부에서 접근할 수 있도록 한다.
라고 이해하긴 했지만, 좀 더 공부가 필요하겠다.

