![](https://images.velog.io/images/usreon/post/a2970ad5-2449-47dd-bc6b-3ca00b040f89/image%20(3).png)

nodeJS에서 비동기 처리를 위한 하나의 패턴으로 콜백 함수를 사용한다. 자바스크립트에서 함수는 1급 객체이다. 즉 함수를 다른 곳으로 전달할 수 있다. 그래서 콜백 개념이 JS에서 중요한 것이다. 콜백은 가독성을 위해 항상 마지막 인자에 넣고, 오류 우선 처리를 하기 위해 반드시 첫번째 인자로 err를 넣는다. 그리고 함수의 첫번째 줄에서 에러 핸들링을 해준다. 

그러나 순차적인 실행을 구현하기 위해 콜백 안에서 콜백을 호출함으로써 콜백 헬이 일어나 가독성이 나쁘고 비동기 처리 중 발생한 에러의 처리가 곤란하며 여러 개의 비동기 처리를 한번에 처리하는 데도 한계가 있다.

`ES6`에서는 비동기 처리를 위한 또 다른 패턴으로 프로미스(Promise)를 도입했다. 프로미스는 전통적인 콜백 패턴이 가진 단점을 보완하며 비동기 처리 시점을 명확하게 표현할 수 있다는 장점이 있다.



## Promise의 탄생

Promise는 비동기 작업의 최종적인 결과(또는 에러)를 담고 있는 객체이다. 프라미스의 용어로, 비동기 작업이 아직 완료되지 않았을 때는 대기중(pending)이라 하며, 작업이 성공적으로 끝났을 대를 이행됨(fullfilled)라고 표현하고, 작업이 에러와 함께 종료되었을 때 거부됨(Reject)라고 한다. 프라미스가 이행되거나 거부되면 결정된(settled)라고 간주합니다.

이행(fullfillment)값 이나 거부(rejection)와 관련된 에러(원인)를 받기위해 프라미스 인스턴스의 `.then()` 함수를 사용할 수 있다.

```js
promise.then(onFulfilled, onRejected);
```
Promise 생성자 함수는 비동기 작업을 수행할 콜백 함수를 인자로 전달받는데 이 콜백 함수는 `resolve`와 `reject` 함수를 인자로 전달받는다. 첫번째 인자는 프라미스가 성공할 때 실행할 콜백함수, 두번째 인자는 거부됐을 때 실행할 콜백함수이다.

위의 형식에서 `onFulfilled`는 최종적으로 프라미스의 이행값(Fulfillment value)을 받는 콜백이며, `onRejection`은 (이유가 있을 때) 거부 이유를 받는 콜백이다. 두 콜백 모두 선택 사항(option)이다. 

```js
asyncOperation(arg, (err, result) => {
	if(err) {
      // 에러 처리
    }
  // 결과 처리
})
```



다음과 같이 프라미스는 전형적인 CPS 방식의 코드들 보다 더 체계적이고 멋진 코드를 만들 수 있다.

## 후속 처리 메소드

Promise 객체의 후속 처리 메소드(then, catch)를 통해 비동기 처리 결과 또는 에러 메시지를 전달받아 처리한다. Promise 객체는 상태를 갖는다고 하였다. 이 상태에 따라 후속 처리 메소드를 체이닝 방식으로 호출한다.


> `.then`
두 개의 콜백 함수를 인자로 전달 받는다. 첫 번째 콜백 함수는 성공(fulfilled, resolve 함수가 호출된 상태) 시 호출되고 두 번째 함수는 실패(rejected, reject 함수가 호출된 상태) 시 호출된다. then 메소드는 Promise를 반환한다.


```js
asyncOperationPromise(arg)
  .then((result) => {
  // 결과 처리
}, (err) => {
  // 에러 처리
})
```

하지만 then 메서드에 두 번째 콜백 함수를 전달하는 것보다 catch 메서드를 사용하는 것이 가독성이 좋고 명확하다. then에서 두 번째 콜백 함수는 첫 번째 콜백 함수에서 발생한 에러를 캐치하지 못하고 코드가 복잡해져서 가독성이 좋지 않기 때문이다. 따라서 에러 처리는 then 메서드에서 하지 말고 catch 메서드를 사용하는 것을 권장한다.

## 에러 처리
> `catch`
예외(비동기 처리에서 발생한 에러와 then 메소드에서 발생한 에러)가 발생하면 호출된다. catch 메소드는 Promise를 반환한다.


```js
asyncOperationPromise(arg)
  .then((result) => {
  // 결과 처리
  }) 
  .catch((err) => {
  // 에러 처리
})
```

### try ...catch 
`catch` 문은,`try ...catch` 문의 `catch`이다. 프라미스에 속해있는 구문이 아니다.

catch 문은 try 문 내부에서 반환되는 Error 인스턴스(객체)를 잡아낼 수 있는 문이다. 그래서 catch로 에러를 잡아낼 수 있다. 

```js
try {
  // err가 throw 되면
 } 
catch(err) {
  // 여기서 잡아줌
 }
```

## 프라미스의 흐름 제어
`then()`함수의 특성은 **또 다른 프라미스를 동기적으로 반환한다는 것이다**. 
프라미스는 resolve 혹은 reject 둘 중 하나를 호출하도록 스케쥴링이 되고, 이들이 호출되면 `.then`에 등록된 콜백들이 비동기 기회가 있을 때 순서대로 실행되기 때문에 다른 콜백이 또 다른 콜백에 영향을 줄 수 없다.

`onFulfilled`나 `onRejected` 함수가 `x`라는 값을 반환한다면 `then()` 메소드에 의해 반환된 프라미스는 다음과 같이 동작한다.

+ x가 값이면(프라미스가 아닌) x를 가지고 이행(fulfilled) 한다. 
+ x가 프라미스라면 프라미스 x의 이행값으로 이행(fulfilled)한다.
+ x가 프라미스라면 프라미스 x의 거부 사유를 최종적인 거부 사유로(rejected) 하다.

## 프라미스 체이닝
이러한 동작들 덕분에 여러가지 환경에서 비동기 작업들을 손쉽게 통합하고 배치할 수 있게 해주는 프라미스 체인을 구성할 수 있다. 또한 `onFulfilled` 또는 `onRejected` 핸들러를 명시하지 않는다면, 이행값 또는 거부 사유는 자동으로 프라미스 체인 내의 다음 프라미스로 전달된다. 예를 들어, `onRejected` 핸들러에 의해 에러가 `catch` 될 때까지 에러는 전체 체인을 통과하면서 전파된다. 프로미스는 후속 처리 메소드를 체이닝(chainning)하여 여러 개의 프로미스를 연결하여 사용할 수 있다. 이는 비동기 작업들의 순차 실행을 손쉽게 만들어준다. 이로써 콜백 헬을 해결한다.


```js
asyncOperationPromise(arg)
  .then((result1) => {
  // 다른 프라미스를 반환
    return asyncOperationPromise(arg2);
   }) 
  .then((result2) => {
  // 값을 반환
    return 'done';
   })
  .then(undefined, (err) => {
    // 체인 내의 에러를 여기서 catch 함.
   })
```

## 프라미스 주요 특성
프라미스의 중요 특성은 비록 값을 가지고 프라미스를 동기적으로 해결(resolve) 한다 할지라도 적어도 한번은 `onFulfilled()` 와 `onRejected()` 콜백이 비동기적으로 호출된다는 보장을 한다는 것이다. 프라미스는 귀결이 되면 resolve, reject 둘 중 하나는 반드시 호출하기 때문에 콜백이 호출되지 않을 일은 없다. 비록 프라미스 객체가 `then`이 호출되는 순간 결정되어(settled) 있어도 `onFulfilled()`와 `onRejected()` 콜백은 비동기적으로 호출된다. 

이제 가장 중요한 부분이다. `onFulfilled()` 또는 `onRejected()` 핸들러에서 예외를 발생시키면 (throw 구문을 사용하여), `then()`메소드에서 반환된 프라미스는 발생된 예외를 거부 사유로 삼아 자동으로 거부된다. 이 부분이 CPS 에 비해 매우 큰 장점인데, 프라미스와 함께 예외(에러)가 체인 전체에 자동으로 전파되고 최종적으로 throw 문을 사용할 수 있기 때문이다. 따라서 에러 처리 핸들러를 따로 만들어주면, 에러 잡는 걸 잊어버려도 알아서 에러를 처리해준다.

## 더 생각해보기
아래 예시에서 `.catch` 는 트리거 될까? 
```js
new Promise(function(resolve, reject) {
  setTimeout(() => {
    throw new Error("에러 발생!");
  }, 1000);
}).catch(alert);
```


## 참고
+ https://poiemaweb.com/es6-promise
+ https://soldonii.tistory.com/130
+ https://ko.javascript.info/promise-error-handling