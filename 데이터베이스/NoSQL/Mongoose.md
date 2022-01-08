Node.js(express)와 MongoDB 연동 RESTful API 
---

## Mongoose 란?
Mongoose는 Node.js와 MongoDB를 위한 ODM(Object Data Mapping) library이다. Java 기반의 Hibernate. iBatis 등의 ORM(Object Relational Mapping)과 유사한 개념이다.

ODM의 사용은 코드 구성이나 개발 편의성 측면에서 장점이 많다. 호환성이 없는 프로그래밍언어(JavaScript) Object와 MongoDB의 데이터를 Mapping하여 간편한 CRUD를 가능하게 한다.

필요에 따라 확장 및 변경이 가능한 자체 검증(Validation)과 타입 변환(Casting)이 가능하며 Express와 함께 사용하면 MVC Concept 구현이 용이하다.

## Mongoose의 RESTful API

### RESTful이란?
코딩을 하다 보면 레스트풀하게 API를 구현하라고 한다. 레스트풀이라는 게 뭘까? <br>
HTTP 프로토콜을 의도에 맞게 정확히 활용하여 디자인하도록 유도하고 있기 때문에 디자인 기준이 명확해지며, 의미적인 범용성을 지니므로 중간 계층의 컴포넌트들이 서비스를 최적화하는 데 도움이 된다. REST의 기본 원칙을 충실히 지킨 서비스 디자인을 “RESTful” 이라고 표현한다.

#### REST에서 가장 중요하며 기본적인 규칙은 아래 두 가지이다.

+ URI는 정보의 자원을 표현해야 한다.
+ 자원에 대한 행위는 HTTP Method로 표현한다.

HTTP Method 사용의 예는 아래와 같다.

| Verb | Action| Path| Used for| 
| ----| ---- | ---- | ---- | 
| GET | index | /books | 모든 서적 리스트 조회| 
| GET | retrieve | 	/books/:id | 특정 서적 조회| 
| POST | create | /books | 신규 서적 생성| 
| PUT | replace	| /books/:id | 특정 서적 갱신(존재하지 않으면 생성)| 
| PATCH | update | /books/:id |	특정 서적 갱신| 
| DELETE | 	delete | /books | 모든 서적 삭제| 
| DELETE | 	delete | /books/:id | 특정 서적 삭제| 

## Mongoose로 개발 환경 세팅하기
[환경 세팅하기](https://poiemaweb.com/mongoose)는 여기롤 참고하자.


## Schema & Model 

### Schema
RDBMS의 Schema는 데이터베이스를 구성하는 레코드의 크기, 키(key)의 정의, 레코드와 레코드의 관계, 검색 방법 등을 정의한 것이다.

Mongoose의 Schema는 MongoDB에 저장되는 document의 Data 구조 즉 필드 타입에 관한 정보를 JSON 형태로 정의한 것으로 RDBMS의 테이블 정의와 유사한 개념이다.

MongoDB는 Schema-less하다. 이는 RDMS처럼 고정 Schema가 존재하지 않는다는 뜻으로 같은 Collection 내에 있더라도 document level의 다른 Schema를 가질 수 있다는 의미이다.

이는 자유도가 높아서 유연한 사용이 가능하다는 장점이 있지만 명시적인 구조가 없기 때문에 어떤 필드가 어떤 데이터 타입인지 알기 어려운 단점이 있다. 이러한 문제를 보완하기 위해서 Mongoose는 Schema를 사용한다.

**models/todo.js에 아래의 코드를 작성한다.**

```js
const mongoose = require('mongoose');

// Define Schemes
const todoSchema = new mongoose.Schema({
  todoid: { type: Number, required: true, unique: true },
  content: { type: String, required: true },
  completed: { type: String, default: false }
},
{
  timestamps: true
});

// Create Model & Export
module.exports = mongoose.model('Todo', todoSchema);
```

primary-key인 _id는 document 내에서 유일함이 보장되는 12 byte binary data이다. 명시적으로 정의하지 않아도 insert()나 save() 메소드 호출시 자동으로 추가된다.

Schema는 Model 생성 시 인자로 전달한 후, 더이상 사용되지 않는다.


+ [Mongoose Schema Types](https://mongoosejs.com/docs/schematypes.html)
+ [option:timestamps](https://mongoosejs.com/docs/guide.html#timestamps)


### Model
model() 메소드에 문자열과 schema를 전달하여 model을 생성한다. model은 보통 대문자로 시작한다.

```js
const Todo = mongoose.model('Todo', todoSchema);
```

model은 생성자이므로 instance를 생성할 수 있는데 이것은 개별 document를 나타낸다. instance 생성시 생성자에 초기값을 전달하거나 instance 생성 후 속성과 값을 추가하여 document를 생성한다.

```js
const todo = new Todo({
  todoid: 1,
  content: 'MongoDB',
  completed: false,
  ...
});

// or

// instance형
const todo = new Todo();
todo.todoid = 1;
todo.content = 'MongoDB';
todo.completed = false;
...


//models/todo.js의 마지막 라인에서 model을 생성하고 export한다.
module.exports = mongoose.model('Todo', todoSchema);
```

### Schema와 Model 그리고 document의 관계
!(몽구스 스키마와 모델 관계)[../../img/mongoose-scheme-model.png]


## CRUD
코드는 [이 곳](https://poiemaweb.com/mongoose#8-crud) 을 참고한다.

아래 예시들은 Create, Read, Update, Delete (CRUD) 쿼리들을 Validator와 미들웨어를 통과하도록 사용하는 적절한 방법을 제시한다.
### Create
아래 두 예제는 정확하게 같은 일을 한다. Model.create(doc) 는 new Model(doc).save() 과 일치한다.
```js
new User({
    name: 'Brian'
}).save()

User.create({
    name: 'Brain'
})
```

### Read
어떻게 사용하든 상관없다. 다만 주의해야 할 점은 findOne()와 find()는 서로 다른 미들웨어(각각 findOne과 find)를 호출한다. 또한 findById(id)는 findOne({ _id: id })와 같다.
```js
await User.findOne({}).exec()
await User.findById({}).exec()
```

### Delete
삭제에서는 데이터 검증이 중요하지 않다. 하지만 미들웨어는 호출해야 할 수도 있다. 예를 들면, 어떤 도큐먼트가 삭제될 때 함께 정리되어야 하는 의존성이 존재하는 경우가 있을 수 있다. 따라서 미들웨어 호출을 보장하기 위해선 아래와 같이 먼저 도큐먼트를 불러온 뒤 삭제해야 한다.
```js
const user = await User.findById(req.params.id).exec()
await user.remove()
res.send({ data: user })
```

아래 중 remove()의 경우 remove 훅을 발생시키지 않는다. 그러나 findByIdAndRemove() 와 findOneAndRemove() 을 호출하면 findOneAndRemove 훅이 발생한다.
```js
// 주의하세요
User.remove()
User.findByIdAndRemove()
User.findOneAndRemove()
```

## Reference
+ [Mongoosejs](https://mongoosejs.com/)
+ [poiemaweb.mongoose](https://poiemaweb.com/mongoose)
+ [Mongoose의 올바른 사용 가이드 1 - Validator, Middleware 그리고 Query](https://a1p4ca.netlify.app/2018/09/10/mongoose%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-%EC%82%AC%EC%9A%A9-%EA%B0%80%EC%9D%B4%EB%93%9C-1-validator-middleware-%EA%B7%B8%EB%A6%AC%EA%B3%A0-query/)