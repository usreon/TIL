> 로그인을 통해 인증 정보가 저장되고, 인증된 사용자가 어떤 식으로 웹사이트를 이용하는지 간단하게 알아보자.

### 세션기반 인증 (Session-based Authentication)

#### 로그인
사용자가 로그인 시도를 할 때 정확한 아이디와 비밀번호를 입력했다면, 서버는 인증(Authentication)에 성공했다고 판단할 것이다. 그렇다면, 다음번에 인증을 필요로 하는 작업(장바구니에 물품 추가 등)을 요청할 경우, 또 로그인 과정을 거쳐야 할까? 서버는 아이디 및 비밀번호의 해시를 이미 알고 있기 때문에, "인증에 성공했음"을 서버가 알고 있다면, 매번 로그인할 필요가 없을 것이다.

이때 서버와 클라이언트에 각각 필요한 것이 있다.

1. 서버는 사용자가 인증에 성공했음을 알고 있어야 한다.
사용자가 인증에 성공한 상태는 세션이라고 부른다. 서버는 일종의 저장소에 세션을 저장한다. 주로 in-memory(자바스크립트 객체) 또는 세션 스토어(redis 등과 같은 트랜잭션이 빠른 DB)에 저장한다. 세션이 만들어지면, 각 세션을 구분할 수 있는 세션 아이디도 만들어지는데 보통 클라이언트에 세션 성공을 증명할 수단으로써 세션 아이디를 전달한다.

2. 클라이언트는 인증 성공을 증명할 수단을 갖고 있어야 한다.
이때 웹사이트에서 로그인을 유지하기 위한 수단으로 쿠키를 사용한다. 쿠키에는 서버에서 발급한 세션 아이디를 저장한다. 쿠키를 통해 유효한 세션 아이디가 서버에 전달되고, 세션 스토어에 해당 세션이 존재한다면 서버는 해당 요청에 접근 가능하다고 판단한다. 하지만 쿠키에 세션 아이디 정보가 없는 경우, 서버는 해당 요청이 인증되지 않았음을 알려준다.


#### 로그아웃
세션 아이디가 담긴 쿠키는 클라이언트에 저장되어 있으며, 서버는 세션을 저장하고 있다. 서버는 그저 세션 아이디로만 요청을 판단한다.

로그아웃은 다음 두 가지 작업을 해야 한다.

1. 서버의 세션 정보를 삭제해야 한다.
2. 클라이언트의 쿠키를 갱신해야 한다.

서버가 클라이언트의 쿠키를 임의로 삭제할 수는 없다. 대신, set-cookie로 세션 아이디의 키값을 무효한 값으로 갱신해야 한다.

#### express-session
이런 세션을 대신 관리해주는 'express-session' 이라는 모듈이 존재한다.

'express-session'은 세션을 위한 미들웨어로, 'Express'에서 세션을 다룰 수 있는 공간을 보다 쉽게 만들어준다. 또한 필요한 경우 세션 아이디를 쿠키에 저장하고, 해당 세션 아이디에 종속되는 고유한 세션 객체를 서버 메모리에 저장한다.

+ [express-req.session](https://github.com/expressjs/session#reqsession)이 바로 세션 객체이며 req.session은 세션 객체에 세션 데이터를 저장하거나 불러오기 위해 사용한다.


> ## Goal 
1)쿠키와 세션은 서로 어떤 관계이며, 각각이 인증에 있어서 어떤 목적으로 존재하는지 이해할 수 있다. 
2)로그인과 로그아웃을 구현한다. 


쿠키는 장시간 유지되고, JS를 이용해 쿠키에 접근할 수 있어서 인증정보가 유출되기 쉬워서 중요정보는 담지 않는다. 중요 인증 정보 및 접속 상태는 세션객체에 저장하고, 해당 세션객체를 확인할 수 있는 키를 쿠키에 넘긴다.


### 서버 구현
1. 세션 옵션 설정
express-session 라이브러리를 이용해 쿠키 옵션 설정

```js
const session = require('express-session'); // 세션관리용 미들웨어

app.use(
  session({
    secret: '@codestates', //암호화하는 데 쓰일 키
    resave: false, 
    saveUninitialized: true, //세션이 저장되기 전 uninitialized 상태로 미리 만들어 저장
    cookie: {
      domain: 'localhost', // 클라는 서버도메인이 설정과 같아야 쿠키전송가능
      path: '/', // 해당 path를 만족하면 쿠키 보내줌
      maxAge: 24 * 6 * 60 * 10000,
      sameSite: 'None',
      httpOnly: true, // JS로 쿠키를 사용할 수 없도록 함
      secure: true, // https 쿠키 주고받게 함
    },
  })
);
```
2. login 구현 : 세션에 데이터 저장
+ request로부터 받은 userId, password와 일치하는 유저가 DB에 존재하는지 확인한다.
+ 일치하는 유저가 없을 경우: 로그인 요청을 거절한다.
+ 일치하는 유저가 있을 경우: 세션에 userId를 저장한다.

```js
const { Users } = require('../../models');
const session = require('express-session')

module.exports = {
  post: async (req, res) => {
    const userInfo = await Users.findOne({
      where: { userId: req.body.userId, password: req.body.password },
    });

    if (!userInfo) {
      res.json({message : 'not authorized'})
    } 
    
    try {
      let cookie = await req.session
      cookie.Id = userInfo.userId // 객체에 키와 값을 넣는 방식으로 추가해주었다.
      res.status(200).json({cookie, message : 'ok'})
    } catch (err) {
      res.sendStatus(500)
    }
  }
}
```



3. logout 구현 : session 삭제
+ 세션 객체에 저장한 값이 존재하면 세션을 삭제한다. (자동으로 클라이언트 쿠키는 갱신된다.)

```js
req.session.destroy();
res.json({ data: null, message: 'ok' });

// res.session = null; 공식문서에 있는 메소드
```

4. userinfo 구현 : sessionId 이용하기
+ 세션 객체에 저장한 값이 존재하면 사용자 정보를 데이터베이스에서 조회한 후 응답으로 전달한다.
+ 세션 객체에 저장한 값이 존재하지 않으면 요청을 거절한다.
```js
if (!req.session.Id) {
      res.status(400).json({message : 'not authorized'})
    } else {
      res.status(200).json({message : 'ok'})
```


