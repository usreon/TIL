## OAuth 2.0
---

웹이나 앱에서 흔히 찾아볼 수 있는 소셜 로그인 인증 방식은 OAuth 2라는 기술을 바탕으로 구현된다. <br>

직접 작성한 서버에서 인증을 처리해주는 것과는 달리, OAuth는 인증을 중개해주는 메커니즘이다. 보안된 리소스에 액세스하기 위해 클라이언트에게 권한을 제공하는 프로세스를 단순화하는 프로토콜이다.
즉, 이미 사용자 정보를 가지고 있는 웹 서비스(GitHub, google, facebook 등)에서 사용자의 인증을 대신해주고, 접근 권한에 대한 토큰을 발급한 후, 이를 이용해 내 서버에서 인증이 가능해진다.


### OAuth란?
> 인증을 위한 표준 프로토콜의 한 종류. 보안된 리소스에 액세스하기 위해 클라이언트에게 권한을 제공(Authorization)하는 프로세스를 단순화하는 프로토콜 중 한 방법이다.


### OAuth는 언제, 왜 쓸까?
유저 입장에서 생각해보자면, 우리는 웹상에서 굉장히 많은 서비스를 이용하고 있고 각각의 서비스들을 이용하기 위해서는 회원가입 절차가 필요하다. 그 서비스별로 ID와 Password를 다 기억하는 것은 매우 귀찮은 일이다. OAuth 를 활용한다면 자주 사용하고 중요한 서비스들(예를 들어 google, github, facebook) 의 ID와 Password만 기억해 놓고 해당 서비스들을 통해서 소셜 로그인을 할 수 있다. <br>

뿐만 아니라 OAuth는 보안상의 이점도 있다. 검증되지 않은 App에서 OAuth를 사용하여 로그인한다면, 직접 유저의 민감한 정보가 App에 노출될 일이 없고 인증 권한에 대한 허가를 미리 유저에게 구해야 하기 때문에 더 안전하게 사용할 수 있다.

### OAuth에서 꼭 알아야 할 용어
+ Resource Owner : 액세스 중인 리소스의 유저 (사용자)
+ Client : Resource owner를 대신하여 보호된 리소스에 액세스하는 응용프로그램 (우리가 만들 서비스). 클라이언트는 서버, 데스크탑, 모바일 또는 기타 장치에서 호스팅할 수 있다.
+ Resource server : client의 요청을 수락하고 응답할 수 있는 서버 (google, github, facebook 등)
+ Authorization server : Resource server가 액세스 토큰을 발급받는 서버. 즉, Client 및 Resource Owner 를 성공적으로 인증한 후 액세스 토큰을 발급하는 서버
+ Authorization grant : 클라이언트가 액세스 토큰을 얻을 때 사용하는 자격 증명의 유형
  - Grant type이란?
    클라이언트가 액세스 토큰을 얻는 방법. 대표적으로 Authorization Code Grant type, Refresh Token Grant type이 있다.
    - Authorization Code Grant type
    액세스 토큰을 받아오기 위해서 먼저 Authorization Code를 받아 액세스 토큰과 교환하는 방법.
    이렇게 하는 이유는 보안을 강화하기 위해서이다. Client에서 Client-secret을 공유해 액세스 토큰을 가지고 오는 것은 탈취 위험이 있기 때문에 ( Client 단에 있으므로) Client에서는 Authorization Code만 받아오고 Server에서 Access token 요청을 진행한다.

    - Refresh Token Grant type
    일정 기간 유효 시간이 지나서 만료된 액세스 토큰을 편리하게 다시 받아오기 위해 사용하는 방법.
    Access token보다 Refresh token의 유효 시간이 대체로 조금 더 길게 설정하기 때문에 가능한 방법이다. 서버마다 refresh token에 대한 정책이 다 다르기 때문에 Refresh token을 사용하기 위해서는 사용하고자 하는 서버의 정책을 살펴볼 필요가 있다.

+ Authorization code
access token을 발급받기 위해 필요한 임시 비밀번호(client id와 client secret이 포함되어 있음). client ID로 이 code를 받아온 후, client secret과 code를 이용해 Access token 을 받아온다.
+ Access token
Authorization code와 client secret을 이용해 받아온 이 Access token으로 이제 resource server의 보호된 리소스에 접근할 수 있다.
+ Scope
scope는 토큰의 권한을 정의한다. 주어진 액세스 토큰을 사용하여 액세스할 수 있는 리소스의 범위이다. (리소스 서버에서 우리가 사용하고 싶은 기능)


### 0Auth logic flow
![0auth1]('../img/0auth2.png')
![0auth]('../img/0auth.jpeg')
