# TypeScript Interfaces
TypeScript interfaces are one of the most powerful construct of the language. Interfaces help in shaping "models" across your application so that any developer can pick that shape and conform to it when writing code.


## TypeScript in action
```js
function filterByTerm(input, searchTerm) {
  if (!searchTerm) throw Error("searchTerm cannot be empty");
  if (!input.length) throw Error("inputArr cannot be empty");
  const regex = new RegExp(searchTerm, "i");
  return input.filter(function(arrayElement) {
    return arrayElement.url.match(regex);
  });
}

filterByTerm("input string", "java");
```

이 자바스크립트 코드를 타입스크립트로 하나씩 변경해보자.


```ts
function filterByTerm(input: string, searchTerm: string) {
  if (!searchTerm) throw Error("searchTerm cannot be empty");
  if (!input.length) throw Error("inputArr cannot be empty");
  const regex = new RegExp(searchTerm, "i");
  return input.filter(function(arrayElement) {
    return arrayElement.url.match(regex);
  });
}

filterByTerm("input string", "java");
```

`filterByTerm` 함수 파라미터의 타입을 지정해주었다.
이 코드를 run하게되면 어떤 결과가 나올까?

```
error TS2339: Property 'filter' does not exist on type 'string'.
```

인자의 값을 보고 string으로 타입 지정을 해주었지만 filter는 배열 메소드기 때문에 error가 난 모습이다.
그렇다면 코드를 이렇게 수정할 수도 있을 것이다.
```ts
Option 1 with string[] :
function filterByTerm(input: string[], searchTerm: string) {
    // omitted
}

or if you like this syntax, option 2 with Array<string> :
function filterByTerm(input: Array<string>, searchTerm: string) {
    // omitted

}
```

이후 다시 run을 하면..
```
error TS2345: Argument of type '"input string"' is not assignable to parameter of type 'string[]'.
filterByTerm("input string", "java");
```

타입을 ["string"] 로 지정했기 때문에 인자를 string으로 받아선 안된다. 
따라서 인자를 filterByTerm(["string1", "string2", "string3"], "java");로 수정해준다.

전체 코드는 다음과 같다.
```ts
function filterByTerm(input: Array<string>, searchTerm: string) {
  if (!searchTerm) throw Error("searchTerm cannot be empty");
  if (!input.length) throw Error("input cannot be empty");
  const regex = new RegExp(searchTerm, "i");
  return input.filter(function(arrayElement) {
    return arrayElement.url.match(regex);
  });
}

filterByTerm(["string1", "string2", "string3"], "java");
```

하지만 또 error가 난 모습이다.
```
error TS2339: Property 'url' does not exist on type 'string'.
```

우리는 문자열 배열을 전달하지만, 코드에서 url이라는 속성에 access하려고 한다. 
문자열 속성에는 url이 없다. 즉, 타입스크립트는 객체 배열을 원하고 있다.
다음 섹션에서 더 자세하게 알아보자.

## TypeScript objects and interfaces

우선 인자값을 
```ts
filterByTerm(
  [{ url: "string1" }, { url: "string2" }, { url: "string3" }],
  "java"
);
```
이렇게 바꿔주고 객체 배열을 사용할 수 있도록 타입 값을 업데이트해주자.

```ts
fucntion filterByTerm (input: Array<object>, searchTerm: string) {
    // omitted
}
```
코드를 수정 후 run을 돌리면..

```
error TS2339: Property 'url' does not exist on type 'object'.
```
왜냐하면 Javascript 객체에는 url이란 이름을 가진 property가 없기 때문이다.
여기서 중요한 점은, properties를 랜덤으로 object에 할당할 수 없다. 이는 TypeScript는 코드의 모든 엔터티가 특정 shape을 준수할 것을 예상하고 있기 때문이다. 그리고 타입스크립트에서 그 shape은 이름을 가지고 있는데 바로 **interface**이다.


