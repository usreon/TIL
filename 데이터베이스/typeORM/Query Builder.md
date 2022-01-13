# Select using Query Builder

## QueryBuiler란?
typeORM의 가장 큰 특징으로 SQL 쿼리를 편리하게 작성할 수 있도록 해준다.

Simple example of QueryBuilder:
```ts
const firstUser = await connection
    .getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .getOne();

```
It builds the following SQL query:
```ts
SELECT
    user.id as userId,
    user.firstName as userFirstName,
    user.lastName as userLastName
FROM users user
WHERE user.id = 1
```

and returns you an instance of User:
```ts
User {
    id: 1,
    firstName: "Timber",
    lastName: "Saw"
}
```

### unique parameters in WHERE pexpressions
시퀄라이즈와 마찬가지로 WHERE 조건문을 작성해준다.

This will not work:
```ts
const result = await getConnection()
    .createQueryBuilder('user')
    .leftJoinAndSelect('user.linkedSheep', 'linkedSheep')
    .leftJoinAndSelect('user.linkedCow', 'linkedCow')
    .where('user.linkedSheep = :id', { id: sheepId })
    .andWhere('user.linkedCow = :id', { id: cowId });
```

we uniquely named :sheepId and :cowId instead of using :id twice for different parameters.
```ts
const result = await getConnection()
    .createQueryBuilder('user')
    .leftJoinAndSelect('user.linkedSheep', 'linkedSheep')
    .leftJoinAndSelect('user.linkedCow', 'linkedCow')
    .where('user.linkedSheep = :sheepId', { sheepId }) // 구조 분해 할당과 같이 :id 대신 직접 불러오고 싶은 객체 키이름으로 받아올 수 있다.
    .andWhere('user.linkedCow = :cowId', { cowId });
```

### How to create and use a QueryBuilder

#### Query Builder를 create하는 방법
+ Using connection
```ts
import {getConnection} from "typeorm";

const user = await getConnection()
    .createQueryBuilder()
    .select("user")
    .from(User, "user")
    .where("user.id = :id", { id: 1})
    .getOne();
```

+ Using entity manager
```ts
import {getManager} from "typeorm";

const user = await getManager()
    .createQueryBuilder(User, "user")
    .where("user.id = :id", { id: 1})
    .getOne();
```

+ Using repository
```ts
import {getRepository} from "typeorm";

const user = await getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1})
    .getOne();
```

entity manager 보다는 repository를 이용하자.
Now let's refactor our code and use Repository instead of EntityManager. Each entity has its own repository which handles all operations with its entity.

### QueryBuilder types
+ SelectQueryBuilder 
```ts
import {getRepository} from "typeorm";

const user = await getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .getOne();
```

+ InsertQueryBuilder 
관계가 있다면, InsertQueryBuilder 보다는 cascade를 이용하자.
```ts
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .insert()
    .into(User)
    .values([
        { firstName: "Timber", lastName: "Saw" },
        { firstName: "Phantom", lastName: "Lancer" }
     ])
    .execute();
```

+ UpdateQueryBuilder 
마찬가지로 UpdateQueryBuilder 보다는 cascade: ['update']를 쓰자
```ts
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .update(User)
    .set({ firstName: "Timber", lastName: "Saw" })
    .where("id = :id", { id: 1 })
    .execute();
```

+ DeleteQueryBuilder 
```ts
import {getRepository} from "typeorm";

await getRepository(User)
    .createQueryBuilder()
    .delete()
    .where("id = :id", { id: 1 })
    .execute();
```


### QueryBuilder를 사용하여 데이터 얻기

+ #### getOne():

```ts
const timber = await getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id OR user.name = :name", { id: 1, name: "Timber"})
    .getOne();
```

+ #### getOneOrFail
DB에서 single result를 찾고 값이 없다면 `EntitynotFoundError`를 보낸다

```ts
const timber = await getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id, user.name = :name", {id: 1, name: "Timber"})
    .getOneOrFail();
```

> entity :  
raw data :
+ #### getMany

