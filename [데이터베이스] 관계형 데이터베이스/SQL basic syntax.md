# 데이터베이스 관련 명령어

#### 데이터베이스 생성
```js
CREATE DATABASE 데이터베이스_이름;
```

#### 데이터베이스 사용
데이터베이스를 이용해 테이블을 만들거나 수정하거나 삭제하는 등의 작업을 하려면, 먼저 데이터베이스를 사용하겠다는 명령을 전달해야 한다.
```js
USE 데이터베이스_이름;
```

#### 테이블 생성
USE 를 이용해 데이터베이스를 선택했다면, 이제 테이블을 만들 수 있다.
```js
CREATE TABLE user (
  id int PRIMARY KEY AUTO_INCREMENT,
  name varchar(255),
  email varchar(255)
);
```

+ CHAR, VARCHAR 중 어떤 문자열 유형을 사용해야 하는가? 
  - 주민등록번호나 사번과 같이 고정적인 길이를 갖고있는 경우는 CHAR를 사용
  - 고정적인 길이를 갖지 못한다면 VARCHAR를 사용



#### 테이블 정보 확인
```js
DESCRIBE user;

mysql> describe user;
+-------+--------------+------+-----+---------+----------------+
| Field | Type         | Null | Key | Default | Extra          |
+-------+--------------+------+-----+---------+----------------+
| id    | int          | NO   | PRI | NULL    | auto_increment |
| name  | varchar(255) | YES  |     | NULL    |                |
| email | varchar(255) | YES  |     | NULL    |                |
+-------+--------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)
```



## SQL 명령어 간략하게 살펴보기


#### SELECT & FROM
+ 테이블과 관련한 작업을 할 경우 반드시 입력해야 한다. FROM 뒤에는 결과를 도출해낼 데이터베이스 테이블을 명시한다.
```js
SELECT 특성_1, 특성_2
FROM 테이블_이름
```
> * 는 와일드카드 (wildcard) 로 전부 선택할 때에 사용된다. 



#### WHERE
+ 필터 역할을 하는 쿼리문이다. WHERE은 선택적으로 사용할 수 있다.

**특정 값과 동일한 데이터 찾기**
```js
SELECT 특성_1, 특성_2
FROM 테이블_이름
WHERE 특성_1 = "특정 값"
```


#### ORDER BY
+ 돌려받는 데이터 결과를 어떤 기준으로 정렬하여 출력할지 결정한다. 
+ 기본 정렬은 오름차순이나, DESC를 써서 내림차순으로도 정렬할 수 있다.
```js
SELECT *
FROM 테이블_이름
ORDER BY 특성_1 DESC
```


## JOIN
![JoinTable](../img/JOIN_table.jpeg)

#### INNER JOIN
+ `INNER JOIN` 이나 `JOIN` 으로 실행할 수 있다.
  - 둘의 교집합을 찾는 거니까 FROM과 JOIN 뒤에 나오는 테이블 순서가 바뀌어도 상관없다.
```js
SELECT *
FROM 테이블_1
JOIN 테이블_2 ON 테이블_1.특성_A = 테이블_2.특성_B
```


### OUTER JOIN
> LEFT, RIGHT 등의 OUTER JOIN은 기준이 되는 테이블이 무엇이냐에 따라 다르다. 결과가 왼쪽 테이블 전체 데이터 대상이라면 LEFT를 ,오른쪽 테이블의 전체 데이터가 대상이라면 RIGHT를 사용한다.


#### 1. LEFT JOIN
+ 왼쪽 테이블을 기준으로 오른쪽의 데이터를 붙여서 볼때 사용한다. 그래서 B에 데이터가 없는것에 대해서도 결과가 보인다. 

```js
SELECT *
FROM 테이블_1 // 기준이 되는 테이블 
LEFT OUTER JOIN 테이블_2 ON 테이블_1.특성_A = 테이블_2.특성_B
```



#### 2. RIGHT JOIN
+ 오른쪽 테이블을 기준으로 왼쪽의 데이터를 붙여서 볼때 사용한다. 그래서 A에 데이터가 없는것에 대해서도 결과가 보인다. 
```js
SELECT *
FROM 테이블_1
RIGHT OUTER JOIN 테이블_2 // 기준이 되는 테이블
ON 테이블_1.특성_A = 테이블_2.특성_B
```


#### LEFT ? RIGHT?
아직은 Right Join의 필요성을 느끼지 못하겠다. LEFT JOIN을 이용해서 기준 테이블을 바꾸면 right join의 결과값도 얻을 수 있다.

## 블로깅
+ [내가 공부한 SQL 문법](https://velog.io/@usreon/Mysql-%EC%BD%94%EB%93%9C-%EB%B6%84%EC%84%9D)