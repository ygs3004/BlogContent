
https://docs.aws.amazon.com/ko_kr/serverless-application-model/latest/developerguide/install-docker.html

```shell

# 패키지 ㅈ최신화
sudo yum update -y

# 도커 설치 시작
sudo yum install -y docker

# Docker 서비스 시작
sudo service docker start

# ec2-user가 docker 명령어 실행할 수 있도록 그룹에 추가
# 추가 이후 로그아웃 이후 로그인 해야 docker 명령어 사용가능
sudo usermod -a -G docker ec2-user

# 확인
docker ps 

```