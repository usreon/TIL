# Learn About HTTPS

## HTTPS 프로토콜
`Hyper Text Transfer Protocol Secure Socket layer`
: HTTP 요청을 SSL 혹은 TLS라는 알고리즘을 이용해, HTTP 통신을 하는 과정에서 내용을 암호화하여 데이터를 전송하는 방법이다.

+ SSL, TLS : 네트워크를 통해 작동하는 서버, 시스템 및 응용 프로그램간 인증 및 데이터 암호화를 제공하는 암호화 프로토콜


### 데이터 제공자의 신원을 확인하고 보장받는 게 중요한 이유
인증에서 HTTPS 프로토콜을 사용해야만 하는 이유는 HTTP보다 상대적으로 안전한 방법이고, 데이터 제공자의 신원을 보장받을 수 있기 때문이다. 클라이언트는 데이터 제공자가 제공해준 데이터를 사용한다. 그렇기 때문에 요청 및 응답을 중간에서 가로채는 중간자 공격(클라이언트와 서버 사이에서 공격자가 서로의 요청, 응답의 데이터를 탈취 및 변조하여 다시 전송하는 것)에 취약하다. 따라서 해당 데이터를 암호화시키는 작업이 필요하다.



### HTTPS 특징
#### 1) 암호화 
HTTPS 프로토콜의 특징 중 하나는 암호화된 데이터를 주고받기 때문에, 중간에 인터넷 요청이 탈취되더라도 그 내용을 알아볼 수 없다. 정확한 키로 복호화 하기전까지는 어떤 내용인지 알 수 없다.


#### 2) 인증서
브라우저가 응답과 함께 전달된 인증서 정보를 확인할 수 있다. 브라우저는 인증서에서 해당 인증서를 발급한 CA 정보를 확인하고 인증된 CA가 발급한 인증서가 아니라면 화면에 경고창을 띄워 서버와 연결이 안전하지 않다는 화면을 보여준다.

+ CA(certificate authority)는 다른 곳에서 사용하기 위한 디지털 인증서를 발급하는 하나의 단위이다. 
+ CA는 공개 키 인증서를 발급하여, 이 인증서는 해당 공개 키가 특정 개인이나 단체, 서버에 속해 있음을 증명한다.

# 사설 인증서 발급 및 HTTPS 서버 구현

### macOS
macOS 사용자의 경우, Homebrew를 통해 mkcert를 설치할 수 있습니다.
```js
$ brew install mkcert
```


### 인증서 생성
먼저 다음 명령어를 통해 로컬을 인증된 발급기관으로 추가해야 한다.
```js
$ mkcert -install
```

다음은 로컬 환경에 대한 인증서를 만들어야 한다. localhost로 대표되는 로컬 환경에 대한 인증서를 만들려면 다음 명령어를 입력해야 한다.
```js
$ mkcert -key-file key.pem -cert-file cert.pem localhost 127.0.0.1 ::1
```



### HTTPS 서버 작성
**node.js https 모듈 이용**
```js
const https = require('https');
const fs = require('fs');

https
  .createServer(
    {
      key: fs.readFileSync(__dirname + '/key.pem', 'utf-8'),
      cert: fs.readFileSync(__dirname + '/cert.pem', 'utf-8'),
    },
    function (req, res) {
      res.write('Congrats! You made https server now :)');
      res.end();
    }
  )
  .listen(3001);
  ```
이제 서버를 실행한 후 https://localhost:3001로 접속하시면 브라우저의 url 창 왼쪽에 자물쇠가 잠겨있는 HTTPS 프로토콜을 이용한다는 것을 알 수 있다.

**express.js 이용**
만약 express.js 를 사용하는 경우, 다음과 같이 https.createServer의 두번째 파라미터에 들어갈 callback 함수를 Express 미들웨어로 교체하면 된다.

```js
const https = require('https');
const fs = require('fs');
const express = require('express');

const app = express();

https
  .createServer(
    {
      key: fs.readFileSync(__dirname + '/key.pem', 'utf-8'),
      cert: fs.readFileSync(__dirname + '/cert.pem', 'utf-8'),
    },
    app.use('/', (req, res) => {
      res.send('Congrats! You made https server now :)');
    })
  )
  .listen(3001);
  ```