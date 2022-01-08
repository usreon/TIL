# Learn About HTTPS


## HTTPS와 HTTP
### HTTP
+ 인터넷 상에서 정보를 주고 받기 위한 프로토콜
+ 클라이언트 <-> 서버
+ 암호화되지 않은 방법으로 데이터를 전송(악의적 감청, 데이터 변조 가능)
### HTTPS
+ 보안이 강화된 HTTP
+ Hyper Text Transfer Protocol Secure의 약자
+ 모든 HTTP 요청과 응답 데이터는 네트워크로 보내지기 전에 암호화된다.
+ HTTPS는 HTTP의 하부에 SSL과 같은 보안 계층을 제공함으로써 동작한다

: HTTP 요청을 SSL 혹은 TLS라는 알고리즘을 이용해, HTTP 통신을 하는 과정에서 내용을 암호화하여 데이터를 전송하는 방법이다.

![HTTPS 프로토콜](../img/HTTPS.png)

+ SSL, TLS : 네트워크를 통해 작동하는 서버, 시스템 및 응용 프로그램간 인증 및 데이터 암호화를 제공하는 암호화 프로토콜

### HTTPS 특징
#### 1) 암호화 
HTTPS 프로토콜의 특징 중 하나는 암호화된 데이터를 주고받기 때문에, 중간에 인터넷 요청이 탈취되더라도 그 내용을 알아볼 수 없다. 정확한 키로 복호화 하기전까지는 어떤 내용인지 알 수 없다.
+ HTTPS 암호화 방법에는 1. 대칭 키 암호화 와 2. 비대칭 키 암호화가 있다.

#### 2) 인증서
브라우저가 응답과 함께 전달된 인증서 정보를 확인할 수 있다. 브라우저는 인증서에서 해당 인증서를 발급한 CA 정보를 확인하고 인증된 CA가 발급한 인증서가 아니라면 화면에 경고창을 띄워 서버와 연결이 안전하지 않다는 화면을 보여준다.

+ CA(certificate authority)는 다른 곳에서 사용하기 위한 디지털 인증서를 발급하는 하나의 단위이다. 
+ CA는 공개 키 인증서를 발급하여, 이 인증서는 해당 공개 키가 특정 개인이나 단체, 서버에 속해 있음을 증명한다.


### 왜 인증서를 사용하는가?
오늘날 우리는 네트워크를 통해 패킷을 주고 받는다. 그러나 중간자 공격에 의해 누군가 패킷을 훔쳐볼 수 있다. 만약 데이터가 암호화된다면, 데이터 유출을 막을 수 있다. 가장 널리 쓰이고 있는 암호화 방식이 SSL/TLS 라는 것인데, 이 방식은 '인증서'라고 하는 일종의 서명을 사용한다. <br>

인증서 라는 것은 결국 '당신이 신뢰할 수 있는 사람이냐' 라는 것을 확인하기 위한 용도이며 이것이 [패킷 데이터](https://enlqn1010.tistory.com/9)를 암호화하기 위한 첫 단계라고 생각하면 된다. 인증서가 신뢰할 수 있다는 검증 작업을 거치고 나서야 비로소 데이터 암호화 작업(SSL)이 수행된다. 


## SSL 디지털 인증서
클라이언트와 서버간의 통신을 공인된 제 3자 업체(CA)가 보증해주는 전자화된 문서


### SSL과 TLS
SSL(Secure Socket Layer)와 TLS(Transport Layer Security)는 같은 것이라고 볼 수 있다. SSL은 TCP/IP 암호화 통신에 사용되는 규약으로써 넷스케이프에서 만들었으며, SSL 3.0 버전에서부터 IETF에서 표준으로 정해서 TLS 1.0이 되었다. 하지만 보통 SSL로 많이 불려진다. 

** IETF(Internet Engineering Task Force): 국제 인터넷 표준화 기구로 인터넷의 운영, 관리 개발에 대해 협의하고 프로토콜과 구조적인 사안들을 분석하는 표준화 작업 기구.

### SSL에서 사용하는 암호화의 종류
+ 암호: 텍스트를 아무나 읽지 못하도록 인코딩하는 알고리즘
+ 키: 암호의 동작을 변경하는 매개변수, 키에 따라서 암호화 결과가 달라지기 때문에 키를 모르면 복호화가 불가능하다.

### 대칭키 암호화 방식
+ 인코딩과 디코딩에 같은 키를 사용하는 알고리즘
+ 대칭키를 전달하는 과정에서 키가 유출되면 암호의 내용을 복호화할 수 있기 때문에 위험
+ 보통 사용하는 AES가 대칭키 방식의 암호화.
+ 단점으로는 발송자와 수신자가 서로 대화하려면 둘 다 공유키를 가져야 한다.
** AES(Advanced Encryption Standard): 고급 암호화 표준이라는 용어로 사용되며 높은 안정성과 속도로 많이 사용됨.

### 공개키 암호화 방식
+ 두개의 키로 암호화와 복호화를 하는 암호화 방식
+ A키로 암호화를 하면 B키로 복호화를 할 수 있고, B키로 암호화를 하면 A키로 복호화를 할 수 있다.
+ 보통 하나는 자신만 갖고 이를 개인키(private, secret Key : 서버에서 주는 키)라고 하며, 다른 하나는 상대방에게 공개하며 공개키(public key : 클라이언트가 가지고 있다 -> 브라우저에 저장)라고 얘기한다. 대표적인 공개키(비대칭키)암호화 방식은 RSA.
+ 공개키와 개인키의 분리는 메시지의 인코딩은 누구나 할 수 있도록 함과 동시에, 메시지의 디코딩은 개인키를 가지고 있는 사람만 가능하게 한다.
+ 단점으로는 처리 알고리즘이 대칭키보다 느리다.

#### HTTPS 통신에서 실제 전송되는 데이터의 암호화에는 대칭키 암호화 방식을 사용하고, 키 교환에는 공개키(비대칭키) 암호화를 사용하여 처리 속도가 느린 문제를 해결하고 있다.
![HTTPS 통신](../img/HTTPS통신.png)


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