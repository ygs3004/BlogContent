도커 환경에서 배포 + 폰트가 필요한 라이브러리 사용 시 "Error while loading available fonts" 오류가 발생할 때가 있다.

도커 jdk 컨테이너에는 폰트 파일이 없어서 그런 경우이다. 
프로젝트에서 캡차 라이브러리, 제스퍼리포트 라이브러리를 사용할 때 폰트 문제 발생을 경험하였다.

docker compose 파일에 아래와 같이 entrypoint 를 추가하여 컨테이너 시작시 폰트 파일을 설치하도록 하여 해결하였다.

```yml
entrypoint: [ "apk add --no-cache fontconfig ttf-freefont && 다른 커맨드" ]
```