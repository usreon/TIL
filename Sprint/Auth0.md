

### redirect는 어떻게 이루어질까?
OAuth 2.0 API는 공격자가 인증 코드 또는 액세스 토큰을 가로챌 수 있는 리디렉션 공격을 방지하기 위해 사용자를 등록된 URL로만 리디렉션합니다. 일부 서비스에서는 웹 앱과 모바일 앱에 동일한 클라이언트 ID를 사용하거나 개발 및 프로덕션 서비스에 동일한 클라이언트 ID를 사용할 때 도움이 될 수 있는 여러 리디렉션 URL을 등록할 수 있습니다.

[URL 및 상태 리디렉션](https://www.oauth.com/oauth2-servers/getting-ready/)
**code**
```js
// TODO: GitHub로부터 사용자 인증을 위해 GitHub로 이동해야 합니다. 적절한 URL을 입력하세요.
this.GITHUB_LOGIN_URL = `https://github.com/login/oauth/authorize?client_id=${클라이언트 아이디 값}`
// https://example.com/auth?destination=account. 리디렉션 URL에 쿼리 문자열 매개변수를 사용하지 않고 경로만 포함하도록 하는 것이 가장 좋습니다.

socialLoginHandler() {
window.location.assign(this.GITHUB_LOGIN_URL)
}

<button
  onClick={this.socialLoginHandler}
  className='socialloginBtn'
>
  Github으로 로그인
</button>
```