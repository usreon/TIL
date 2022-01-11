## 코드 예시

### 선언 변수명 : 타입 지정 = 값 지정
```ts
const arr : number[] = [1,2,3]
const strArr : string[] = ["Hellowww"]
```

```ts
function 함수 이름 (파라미터 : 파라미터 타입 지정) {
    return 리턴 값
}

function 함수 이름 () : 파라미터 타입 지정 {
    return 리턴 값
}

function 함수 이름 (파라미터 : { key1 : key1_type; key2 : key2_type})
```

```ts
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });

// [LOG]: "The coordinate's x value is 3" 
// [LOG]: "The coordinate's y value is 7" 
```

#### any
you can use whenever you don’t want a particular value to cause typechecking errors.

```ts
let obj: any = { x: 0 };
// None of the following lines of code will throw compiler errors.
// Using `any` disables all further type checking, and it is assumed 
// you know the environment better than TypeScript.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```



### Union Type
: two or more other types, representing values that may be any one of those types

```ts
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
Argument of type '{ myID: number; }' is not assignable to parameter of type 'string | number'.
  Type '{ myID: number; }' is not assignable to type 'number'.
```

유니온 타입을 사용했다면, 명시된 모든 타입에 적용이 되는 메소드만 사용 가능하다.
```ts
function printId(id: number | string) {
  console.log(id.toUpperCase());
}
  
// Property 'toUpperCase' does not exist on type 'string | number'.
// Property 'toUpperCase' does not exist on type 'number'.
```

#### The solution is to narrow the union with code
```ts
function printId(id: number | string) {
  if (typeof id === "string") {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase());
  } else {
    // Here, id is of type 'number'
    console.log(id);
  }
}

function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // Here: 'x' is 'string[]'
    console.log("Hello, " + x.join(" and "));
  } else {
    // Here: 'x' is 'string'
    console.log("Welcome lone traveler " + x);
  }
}
```

만약 모든 타입이 같은 메소드를 가지고 있다면..
```ts
// Return type is inferred as number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
```



## Functions
TypeScript allows you to specify the types of both the input and output values of functions.

### Parameter Type Annotations
```ts
// Parameter type annotation
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}

// Would be a runtime error if executed!
greet(42);
Argument of type 'number' is not assignable to parameter of type 'string'.
```

### Optional Properties
Object types can also specify that some or all of their properties are optional. To do this, add a ? after the property name:

```ts
function printName(obj: { first: string; last?: string }) {
  // ...
}
// Both OK
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
```