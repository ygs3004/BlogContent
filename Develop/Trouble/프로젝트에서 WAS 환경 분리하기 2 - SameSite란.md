
[프로젝트에서 WAS 환경 분리하기 1 - Intellij 환경 변수 이용하기 :: 꿀잠 (tistory.com)](https://ygs3004.tistory.com/19)

WAS 환경을 분리하기 위해 axios 의 base url 을 분리하였고, 문제는 해결되어진 것으로 보았다. 그러나 그 이후 더 큰 문제가 발생하였다. 개발 WAS 에서 Session ID 가 요청마다 새로 생기는 문제였다. 우리의 로그인 방식이 세션 방식이었기에 이 문제는 크게 작용하였다. 

로컬 WAS 환경에선 문제 없었으나 개발 WAS 에선 로그인 이후에 바로 다음 요청시 세션이 초기화 되어 로그인 인증을 할 수 없는 것이었다.

이것은 CSRF 방지를 위한 브라우저의 보안 정책 때문이었다. 세션 ID를 담은 쿠키를 보내려면 적절한 SameSite 정책이 포함되어야 하는 것이었다. Dev WAS 와 Local WEB으로 분리된 프로젝트 환경이기에 기본적으로 같은 사이트(SameSite)가 아닌 상황이었고, 서버는 SameSite 가 아닌 WEB 프로젝트의 쿠키를 폐기하고 새로 생성하였던 것이다.

아래와 같이 Spring Boot 의 CookieSerializer Bean을 새로 등록하는 방식으로 SameSite 정책을 None으로 변경해주었다.

```java
@Bean
public CookieSerializer cookieSerializer() throws MalformedURLException {
    DefaultCookieSerializer serializer = new DefaultCookieSerializer();
    serializer.setSameSite("None");
	serializer.setUseSecureCookie(true); // None 일 경우 필수
    return serializer;
}
```

위와 같이 None 설정을 하여 진행하였지만 이후에 추가적인 문제가 생겼다. 바로 Secure 옵션을 반드시 주어야 한다는 점이다. 프로젝트에 HTTPS를 적용하여야 했다. Local 환경에 SSL 인증서를 설치하는 방법도 있지만 개발 환경에서 serve 명령어를 변경하여 간단히 해결하는 방법도 있다. package.json 에서 npm run serve 명령어를 아래와 같이 옵션을 주는 형태로 해결하였다.

``` json
 "serve": "vue-cli-service serve --https true",
 "serve-local": "vue-cli-service serve",
```

 Dev Server WAS 환경을 바라 볼 때는 npm run serve 명령어를 사용하고, Local WAS 를 바라볼 때는 npm run serve-local를 사용하면 된다. Local 에서는 SameSite 인 상황이기에 쿠키의 새로운 생성을 걱정하지 않아도 되었기 때문이다.

분리 방법을 요약 하자면 아래와 같다.
1. 환경 변수를 이용한 axios base url 분리
2. Local Web -> Server Was 환경에서 Session 이 초기화 되면 안되는 환경일 경우 SameSite None, SecureCookie 설정
3. Local Web Https 적용(로컬 SSL 인증서 설치 또는 실행 명령어로 해결)


이로써 로컬 및 서버 WAS 인지에 따른 개발환경 분리를 완료할 수 있었다.