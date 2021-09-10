현재 서버에서는 세션을 이용한 인증방식을 사용하고 있습니다.
하지만 서버의 반응이 느려지는 등 서버에 가해지는 부하가 굉장한 상태입니다.
회사에서는 당장 서버의 확장은 어렵다는 입장입니다.
이제 책임자로서 여러분은 이제 조금이라도 부하를 줄이기 위해 인증방식을 토큰인증 방식으로 다시 구현해야 합니다.

> 토큰의 일종인 JSON Web Token 을 이용하여 토큰방식 인증을 구현합니다.


## 스프린트 진행 흐름

### Server
1. `index.js` https 서버 구현
2. `package.json` 위치에 인증서 파일 가져오기(key.pem,cert.pem)
3. `.env` 에 데이터베이스에 환경변수 설정(데이터베이스 이름 및 비밀번호 등)
4. `controllers/users/login.js` 에서 POST / login 구현
req.body에 아이디와 비밀번호가 들어오니까 DB를 확인해서
데이터가 없는경우, 응답작성
데이터가 있는경우, access token, refresh token 생성
access token은 응답에 넣어주고, refresh token은 쿠키에 넣어주기(클라이언트에서 그렇게 다루기때문)


**jsonwebtoken 라이브러리를 사용해 토큰을 생성하는 방법**
```js
const jwt = require('jsonwebtoken');
const token = jwt.sign(토큰에_담을_값, ACCESS_SECRET, { 옵션1: 값, 옵션2: 값, ... });
```

**POST 로그인 구현**
```js
module.exports = async (req, res) => {
    const bodyValues = req.body
    const userInfo = await Users.findOne({ where : { userId : bodyValues.userId , password : bodyValues.password}})

    if (!userInfo) {
      res.status(400).send({ data: null, message: "not authorized" })
    }

    try {     
      const data = userInfo.dataValues;
      const payload = { id: data.id, userId: data.userId, email: data.email, createdAt :data.createdAt, updatedAt: data.updatedAt, iat : Math.floor(Date.now()/1000), exp : Math.floor(Date.now()/1000) + (60*60) }
      const payload2 = { userId: data.userId, password : data.password, iat : Math.floor(Date.now()/1000), exp : Math.floor(Date.now()/1000) + (60*60) }

      const accessToken = jwt.sign(payload, process.env.ACCESS_SECRET);
      const refreshToken = jwt.sign(payload2, process.env.REFRESH_SECRET);

      res.setHeader("Set-Cookie", 'refreshToken', [sameSite='none', secure=true, httpOnly=true]); 
      res.json({ data: { accessToken: accessToken }, message: "ok" })
      // accessToken = req.headers, refreshToken = req.cookies
    } catch (err) {
      res.sendStatus(404)
    }
};


// 로그인이 되는 순간 토큰이 생성
// 생성된 refreshToken은 자동으로 로그인을 유지시켜준다(새로고침하거나 accessToken이 만료되어도)
```

5. accessTokenRequest
헤더에 발급한 토큰이 들었는지 확인하고, 있으면 정보조회 후 해당유저정보 보내주기

```js
module.exports = async (req, res) => {
  const authorization = req.headers.authorization // 토큰이 있으면 headers의 authorization에 값이 들어있음
  if (!authorization) res.json({ data: null, message: "invalid access token" })

  try {
    const token = authorization.split(' ')[1]; //토큰을 확인했을 때 1번째 인덱스 값을 불러와야 했다.
    
    // 토큰 해독하는 법 verify(해독, 검증)
    const data = jwt.verify(token, process.env.ACCESS_SECRET)
    delete data.iat
    const userInfo = await Users.findOne({ where : { userId : data.userId }})
    
    // 담겨있는 정보를 DB와 조회 후 존재하면 정보 넘기기
    if (userInfo) {
      res.json({ "data" : { "userInfo" : data}, "message" : "ok" })
    }
    res.json({ "data": null, "message": "access token has been tempered" })
  } catch (err) {
    res.sendStatus(404)
  }
};
```

6. refreshTokenRequest
쿠키에 발급한 토큰이 들었는지 확인하고,
있으면 accessToken 새로 발급해주고 유저정보 내보내기

```js
module.exports = async (req, res) => {
  const token = req.cookies.refreshToken
  if (!token) res.json({ "data": null, "message": "refresh token not provided" })

  try {
    const data = jwt.verify(token, process.env.REFRESH_SECRET) // 쿠키에 담긴 정보를 해독한 값
    const userInfo = await Users.findOne({ where : { userId : data.userId }})

    if(!data) { // 유효하지 않거나, 해독이 불가한 토큰인 경우
      res.json({ "data": null, "message": "invalid refresh token, please log in again" })
    }    
    if (userInfo) { // 일치하는 유저가 있을 경우
      delete data.iat
      const payload = { id: data.id, userId: data.userId, email: data.email, createdAt :data.createdAt, updatedAt: data.updatedAt, iat : Math.floor(Date.now()/1000), exp : Math.floor(Date.now()/1000) + (60*60) }
      const accessToken = jwt.sign(payload, process.env.ACCESS_SECRET);
      res.json({ "data" : { 
        "accessToken": accessToken,
        "userInfo" : data}, 
        "message" : "ok" })
    }
    res.json({ "data": null, "message": "refresh token has been tempered" })
  } catch (err) { // 유효하지 않거나, 해독이 불가한 토큰인 경우
    res.json({ "data": null, "message": "invalid refresh token, please log in again" })
  } 
  
// jwt.verify 할 때 네 번째 인자로 콜백 함수를 받는데 err인 경우와 decoded 된 결과값을 파라미터로 갖는다
// 때문에 에러 핸들링을 콜백 함수 안에서 할 수 있지만 거기서 하지 않으면 에러가 catch문으로 들어와서 그 안에서 에러 핸들링 해줄 수 있다.
};
```

> 제일 중요한건 정보를 내보낼 때, 비밀번호를 내보내면 절대 안된다.
