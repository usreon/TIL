## 계속 까먹는 model, controller, migration, DB 정리하기


### MVC를 통해 이해하기
1. 사용자가 웹사이트에 접속한다. (User)
2. Controller는 사용자가 요청한 웹페이지를 제공하기 위해서 Model에서 데이터를 호출한다.
3. Model은 데이터베이스나 파일과 같은 데이터 소스를 제어한 후에 그 결과를 리턴한다.
4. Controller는 Model이 리턴한 결과를 View에 반영한다. (Updates)
5. 데이터가 반영된 VIew는 사용자에게 보여진다. (Sees)


> 브라우저에서 이벤트 발생 -> 서버에 요청(라우터) -> 라우팅 된 요청들이 컨트롤러로 감 -> 이 컨트롤러는 데이터베이스랑 연결이 되어야 하는데 직접적으로 소통이 불가능해서 데이터 베이스랑 관련된 로직을 처리하는 곳인 모델과 소통을 한다. -> 모델에서 데이터를 가져와 뷰를 사용자에게 보여준다.

사용자가 네이버에 접속을 한다. 화면에 보이는 네이버 창은 View로 인해 다루어진다. 사용자가 웹툰을 보고 싶어 화요일 웹툰 버튼을 클릭하였다. 여기서 목요일 웹툰이 아닌 알맞는 데이터를 불러와야 한다. 이게 알아서 데이터베이스에서 그냥 불려와지는 것일까? 아니다. 웹툰 버튼을 클릭하면 웹툰과 관련된 많은 양의 데이터를 Model로부터 불러와야 한다. 이렇게 View와 Model 사이에서 사용자의 요청을 분석하고 이에 맞는 데이터를 불러오는 작용(목요일이 아닌 화요일 웹툰을 불러온 것) 중간에서 매개하는 요소가 바로 Controller인 셈이다. 

이 컨트롤러는 이제 데이터 베이스와 연결이 되어야하기 때문에, 데이터 베이스랑 관련된 로직을 처리하는 곳인 모델(DB와 소통 = sequelize ORM)과 소통을 한다.

#### ORM : 관계형 db랑 object를 연결해주는 매개체
`즉 orm은 모델을 기술하는 도구이다.`

### 모델 vs 컨트롤러
CRUD는 대부분의 컴퓨터 소프트웨어가 가지는 기본적인 데이터 처리 기능인 Create, Read, Update, Delete를 묶어서 일컫는 말이다. 즉, 데이터 베이스는 CRUD가 가능해야 한다.

자바스크립트 안에서 데이터베이스에서 정보를 가져오거나 추가 또는 삭제를 하는 로직을 작성해야 하는데 그걸 사용할 수 있도록 모델이 매개체가 되는 거다.

#### Model code

```js
const db = require("../db");

module.exports = {
  orders: {
    get: (userId, callback) => {
      // 해당 유저가 작성한 모든 주문을 가져오는 함수
      let sql = `SELECT * FROM ...`

      db.query(sql, (err, result) => {
       ...
    },
    post: (userId, orders, totalPrice, callback) => {
      // 해당 유저의 주문 요청을 데이터베이스에 생성하는 함수
      let sql1 = `INSERT INTO orders (user_id, total_price) VALUES ?`;
      let sql2 = `INSERT INTO order_items (order_id,item_id, order_quantity) VALUES ?`;


      db.query(sql1, [params1], (err, result) => {
        ...

  items: {
    get: (callback) => {
      // 모든 상품을 가져오는 함수
      const queryString = `SELECT * FROM items`;

      db.query(queryString, (err, result) => {
       ...
    },
  },
};
```



#### Controller code


```js
const models = require('../models');

module.exports = {
  orders: {
    get: (req, res) => {
      const userId = req.params.userId;

      if (!userId) {
        return res.status(401).send('Unauthorized user.');
      } else {
        models.orders.get(Number(userId), (error, result) => {
          if (error) {
            res.status(404).send('No orders found.');
          } else {
            res.status(200).json(result);
          }
        });
      }
    },
    post: (req, res) => {
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
  items: {
    get: (req, res) => {
      models.items.get((error, result) => {
        if (error) {
          res.status(404).send('Not found');
        } else {
          res.status(200).json(result);
        }
      });
    }
  }
};
```



### 모델 vs 마이그레이션
시퀄라이즈로 모델을 생성하면 마이그레이션이 생긴다. 마이그레이션은 데이터베이스의 설계도(스키마 디자인같은)라고 생각하면 된다. 그럼 마이그레이션만 필요하지 않을까? 왜 모델까지 만드는 걸까?

자바스크립트 문법에서 DB에서 정보를 가져오거나 추가하거나 삭제하는 로직을 작성해야 되는데(DB를 컨트롤할 수 있는 문법) 그걸 사용할 수 있게 모델이 매개체가 되는 것이다. 따라서 모델은 데이터 베이스랑 관련된 로직을 자바스크립트 언어로 처리하는 곳이라고 볼 수 있다. 데이터 베이스의 변화가 생기면 두 파일 다 수정해줘야 한다.

처음에는 DB가 모델에 존재하는지 알았는데 아니었다.. 그냥 데이터 베이스는 우리 컴퓨터 어딘가 이를 테면 하드 디스크 같은 곳에 있다고 한다.

