
3.1.0 version 기준 example

```bash
# jdk 설치
sudo apt install openjdk-11jdk
java -version
javac -version

#kafka 링크 이용하여 다운로드
wget https://archive.apache.org/dist/kafka/3.1.0/kafka_2.13-3.1.0.tgz

# kafka bin 경로 체크
cd kafka_2.13-3.1.0/bin
pwd

# PATH 등록
nano .bashrc

# .bashrc 최하단 환경변수 추가
PATH="$PATH:/home/ygs/kafka_2.13-3.1.0/bin"
# 환경변수 추가 후 linux 재로그인, PATH 체크
echo $PATH 

# 주키퍼 실행
zookeeper-server-start.sh ~/kafka_2.13-3.1.0/config/zookeeper.properties

# kafka 실행[]()
kafka-server-start.sh ~/kafka_2.13-3.1.0/config/server.properties

# kafka, zookeper 설정, kafka 로그 및 zookeper 데이터 디렉토리 설정 가능
nano kafka_2.13-3.1.0/config/zookeeper.properties
nano kafka_2.13-3.1.0/config/server.properties

```
