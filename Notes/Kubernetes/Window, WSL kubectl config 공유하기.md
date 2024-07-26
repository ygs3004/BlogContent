Window 에서 kubectl 을 설치 후 WSL 에서 클러스터 사용을 공유하기 위해 config 파일을 공유하기

```bash

# wsl Linux 에 kubectl 설치
# https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# kubectl 실행 권한 설정
chmod +x /kubectl

# 설정파일 저장 디렉토리 생성
mkdir -p ~/.kube

# 윈도우 설정파일 링크
ln -sf "/mnt/c/users/$windowsUser/.kube/config" ~/.kube/config
```