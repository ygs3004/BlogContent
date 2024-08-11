
개인 포트폴리오용 서버를 Docker + Nginx + Vue + SpringBoot 환경으로 배포를 시도하고 있는 중이다. 
수많은 문제점들과 SSL 인증서까지 다 적용에 간신히 성공했다. 수많은 트러블 중 만났던 하나의 트러블에 대한 해결방법을 남긴다.

SSL 인증서를 발급 받은 이후 docker nginx 에서 도메인으로 들어오는 http(80포트) 에 대해 https(443포트)로 리다이렉트 하도록 설정해 두었다. Frontend -> Backend API 요청은 문제없이 리다이렉트 되어서 해결되는 듯 하였지만 스웨거 페이지가 문제였다.

어째서인지 Swagger UI 에서의 요청은 Spring 에 닿지조차 못하였다. 그러던 중 요청시 https 페이지임에도 불구하고 curl 명령어가 http로 뜨는것을 확인했다. Swagger UI 에서는 curl 명령어로 실행해서 요청을 호출하는 것 같았다. 해당 명령어를 실행한 결과 Spring이 아니라 nginx 에서 Redirect 했다는 내용이 나오는 것을 확인했다.

아마 Swagger에서의 요청은 리다이렉트된 Spring 까지 도달하지 못하였던듯 하다. 관련해서 찾아보다 설정으로 처리 하는 방법을 발견하여 적용하였고 해결하였다.

```properties
server.forward-headers-strategy=framework
```

위와같은 설정을 application.properties로 추가하니 요청이 정상적으로 도달하였다.
해당 설정을 넣기전에는 스웨거의 curl 명령어가 https 라는 헤더정보가 누락된었던 듯 하며, 해당 설정 이후 정보들이 제대로 전달되어, swagger 의 curl 요청이 정상적으로 변경된듯 하다