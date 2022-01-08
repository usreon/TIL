> [Sequelize ORM 공식문서](https://sequelize.org/)와 [ORM : Sequelize in Node](https://www.duringthedrive.com/2017/05/06/models-migrations-sequelize-node/) 를 참고하며 작성했습니다.

# ORM(객체 관계형 관리자)은 어떻게 작동할까?
ORM은 데이터베이스 언어를 직접 작성하지 않고도 데이터베이스를 쿼리할 수 있는 코드 라이브러리이다. 따라서 Sequelize ORM을 사용하면 JavaScript로 데이터베이스 쿼리문을 작성할 수 있다. 이것은 코드를 극적으로 단순화하는 데 도움이 된다. ORM은 "모델"을 사용하여 테이블 행 중 하나에 "셀"을 저장한다. 그리고 이러한 모델은 마이그레이션을 위한 골격을 생성할 수 있다.

## 모델과 마이그레이션
마이그레이션은 "up" 기능과 "down" 기능이 있는 일련의 데이터베이스 작업이다. "up" 기능은 데이터베이스를 변경하고 "down" 기능은 "up" 기능이 실행되기 전의 상태로 데이터베이스를 복원한다. 마이그레이션을 사용하여 테이블, 열 및 기타 항목 추가와 같은 구조적 데이터베이스 작업을 할 수 있고 마이그레이션을 사용하여 테이블에 데이터(행)를 추가할 수도 있다. 그러나 대부분의 사람들이 마이그레이션에 대해 이야기할 때 의미하는 것은 전자의 구조적 데이터베이스 작업이다. <br>

이 부분은 번역 보다 원본으로 보는 게 더 이해하기 쉽다.
Models are used every time you attempt send data (table rows) through your app and into your database tables. Migrations are used to set up those tables’ columns or to bulk add rows. So while a model and a migration for the same database table may seem repetitive, remember that the model controls what data can enter the database, and migrations actually themselves make changes to the database.

즉, 모델은 데이터베이스에 입력할 수 있는 데이터를 제어하고 마이그레이션은 실제로 데이터베이스를 변경한다.

## 데이터 구조화
모델을 생성하기 전에 충분히 그리고 깊게 생각해봐야 한다. SQL 테이블은 데이터에 대한 형태를 결정하고 데이터가 포함된 후에는 조작하기가 상대적으로 번거롭기 때문이다. 우리는 어떤 테이블의 데이터가 어떤 다른 테이블의 데이터에 속하는지, 그리고 그 관계가 1:N인지 N:M 인지 잘 결정해야 한다.


# Practice
## Tools and Set-Up

### 1. sequelize 설치
```js
npm i sequelize 
```

### 2. sequelize-cli 설치
sequelize-cli는 마이그레이션을 할 수 있도록 돕는 툴로, CLI에서 모델을 생성해주거나, 스키마 적용을 할 수 있도록 돕는다.
```js
npm i —save-dev sequelize-cli
```

### 3. project bootstrapping
+ Bootstrapping (초기 단계 설정)
cli를 통해 ORM을 잘 사용할 수 있도록 bootstraping(프로젝트 초기 단계를 자동으로 설정할 수 있도록 도와주는 일)을 해줘야 한다.

빈 프로젝트를 만드는 명령어 : `npx sequelize-cli init` <br>

이 명령어를 사용하면 다음과 같은 폴더가 만들어진다. <br>

1. config : 데이터베이스에 연결하는 방법을 CLI에 알려주는 구성 파일이 포함되어 있습니다.
2. models : 프로젝트의 모든 모델을 포함합니다.
3. migrations : 모든 마이그레이션 파일 포함
4. seeders : 모든 시드 파일을 포함합니다.

## Sequelize CLI : 모델 생성과 데이터 베이스에 데이터 추가
### 모델 생성과 속성값 지정하기
```js
sequelize model:create --name (모델 이름) --attributes (속성1:속성값1, 속성2:속성값2...)
```

이때 중요한 점은 모델 명을 단수로 표기한다. 마이그레이션 후에는 알아서 복수로 바뀌기 때문이다.

### Running Migrations
이 단계까지는 데이터베이스에 아무 것도 추가되지 않았다. 모델과 마이그레이션 파일을 만들었다면, 이제 실제로 데이터베이스에 해당 테이블을 생성하기 위해 db:migrate명령을 실행해야 한다.
```js
npx sequelize-cli db:migrate
```

### 마이그레이션 실행 취소
```js
npx sequelize-cli db:migrate:undo
```
이 명령은 가장 최근의 마이그레이션을 취소시킨다.



## Sequelize methods 
Sequelize provides various methods to assist querying your database for data. <Br>
공식 문서에 의하면 sequelize에서는 쿼리를 지원해주는 메소드가 있다고 한다. SQL 사용 대신에 ORM을 사용하여 sequelize에서는 어떻게 메소드로 쿼리문을 작성하는지 확인해 보자.

### INSERT
#### findOrCreate
특정 요소를 검색해 값이 존재하면 해당 인스턴스를 반환하고 아니면 값을 생성한다.<br>
두 가지 값을 반환하는데, 첫번째는 객체를 포함한 배열이고 두번째는 `boolean`  값이다. 새 객체를 생성했으면 true, 원래 값이 있는 경우는 false를 나타낸다.
```js
try {
  const [result, created] = await models.url.findOrCreate({
    where : {url},
    default: {title}
    })

  if (created) { // 인스턴스를 생성했으면 
    return res.status(201).json(result)
  }
  res.status(201).json(result)
}
```
### SELECT
#### findAll() 
모든 데이터를 불러온다.
```js
get: async (req, res) => {
    const data = await models.url.findAll()
    res.status(200).json(data);
}
```
#### findOne({ where : { 조건 }})
조건에 맞는 데이터를 불러온다.
```js
const url = await models.url.findOne({ where : { id : id } })
```

### UPDATE
#### update()
데이터를 수정한다. <br>
첫 번째 인자로 수정할 데이터를, 두 번째 인자로 조건을 전달하면 된다.
```js
try {
    await url.update({ visits : url.visits +1}) 
    // 이미 조건으로 걸러진 url 변수를 받아온 것이라서 조건은 생략하였다.
    // 업데이트 된 값을 send 하는 게 아니고 그냥 업데이트만 시켜줘야 해서 변수에 담지 않았다.
    const redirectUrl = url.url;
    res.status(302).redirect(redirectUrl)
}
```