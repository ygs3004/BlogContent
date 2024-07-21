
### Docker Image / container

```shell
# 실행중인 도커 컨테이너 정보
docker ps 

# 이미지 목록보기
docker images

# 실행중인 도커 컨테이너 멈추기 ${name}은 ps 에서 확인가능
docker stop ${name}

# 현재 경로 도커 이미지 빌드(완료시 image id 획득)
docker build .
	# 이미지는 name:tag 로 식별자 부여
	# -t ${name}:${tag}

# 이미지 REPOSITORY(name), TAG 재설정
docker tag ${old-name}:${old-tag} ${name}:${tag}

# 컨테이너 생성 및 시작
docker run -p ${docker-port}:${local-port} ${image-id}
	# -d : detached mode
	# -it 
		# -i : interactive(컨테이너 내부로 입력 가능하게 함)
		# -t : tty (터미널 생성)
	# -rm : 컨테이너 종료시 자동 제거
	# --name ${custom-continaer-name}: 원하는 컨테이너 이름 식별자 부여
	
# 컨테이너 백그라운드 시작
docker start ${container-id}
	# -a : attached mode

# 특정 컨테이너 내부 명령어 실행
docker exec ${container-name} ${command}

# 컨테이너 제거
docker rm ${...container-name}

# 이미지 제거 (컨테이너에서 사용되지 않는 것만 삭제 가능)
docker rmi ${...image-id}

# 사용되지 않는 이미지 전부 제거
docker image prune

# 로그 보기
docker logs
	# -f : follow mode 

```

### Docker Network

```shell
# 네트워크 목록보기
docker network ls

# 네트워크 만들기
docker network create ${network-name}
```

### In Code

```shell
# docker host machine ip in code
host.docker.internal
```


