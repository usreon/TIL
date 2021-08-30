### req.params
An object containing parameter values parsed from the URL path.

For example if you have the route /user/:name, then the "name" from the URL path wil be available as req.params.name. This object defaults to {}.



### req.params vs req.query
서버에서 req.params로 받을 수 있는 것은, 이미 예약된 값이라고 생각 할 수 있다. 예를 들어 서버의 routing코드가 아래와 같다고 하자.

post.get("/:id/:name", function1);
그리고 데이터를 요청하는 클라이언트 측의 axios가 아래와 같다고 하자.


```js
await axios({
  method: "get",
  url: `www.example.com/post/1/jun`,
  params: { title: 'hello!' },
})
```

이 경우 전송되는 url은 ‘www.example.com/post/1/jun?title=hello!’ 이다. <br>

이럴 경우 서버에서 req.params와 req.query를 출력하면 결과값은 어떻게 나올까? 위에서 조금 애매하게 말하긴 했지만, req.params는 예약된 값이라고 했다. routing을 보면 id와 name이 예약되어 있음을 알고있다. 즉 서버에서 id라는 변수로 어떤 값이 들어올 것을 알고, name이라는 변수에 어떤 값이 들어올 것임을 알고 대기하고 있다. req.params는 url을 분석하여 id와 name자리에 있는 값을 낚아챈다. 따라서 req.params를 출력해보면 아래와 같다.

```js
console.log(req.params); // { id: '1', name: 'jun' }
```
한편 req.query를 출력하면 아래와 같다.

```js
console.log(req.query); // { title : 'hello!' }
```

#### req.query
즉, url에서 ?뒤에 입력되는 query문을 req.query로 받아오는 것이다.


## 스프린트를 통해 이해하기
### A. 주문하기

#### Request
`POST /users/:userId/orders/new`
[요청] 사용자가 새로운 주문을 추가합니다. <br>

클라이언트의 요청 본문엔 다음 내용이 포함되어 있습니다.

+ 요청 형식: JSON
  - MIME 타입: application/json

|:parameter:|:형식:|:설명:|:필수 포함 여부:|
|:orders:|:배열:|:주문 아이템:|:필수:|
|:totalPrice:|:숫자:|:주문 합계:|:필수:|
[표] 파라미터 정보

```js
{
  "orders": [
    {
      "quantity": 1,
      "itemId": 2
    }
    // ...여러 개의 주문 아이템
  ],
  "totalPrice": 16900
}
```
[데이터] Request에 따른 Response 예시




**Controller Code**
```js
    post: (req, res) => {
      console.log(req.params, req.query) // {userId:'1'} {}
      const userId = req.params.userId;
      const { orders, totalPrice } = req.body;

      if (orders.length === 0) {
        return res.status(400).send('Bad request.');
      } else {
        models.orders.post(Number(userId), orders, totalPrice, (error, result) => {
          if (error) {
            res.status(404).send('Not found');
          } else {
            res.status(201).send('Order has been placed.');
          }
        });
      }
    }
  },
```


### 정리 
위에서 봤듯 `POST /users/:userId/orders/new` 니까 req.params.userId 로 값을 받아올 수 있었다.