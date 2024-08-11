### Minikube

```shell
# minikube 상태확인  
minikube status  

# minikube 클러스터 삭제
minikube delete

# minikube 클러스터 생성
minikube start --driver=${가상머신}  

# minikube 서비스 접근
minikube service ${deployment-name}
```

### Kubectl

```shell
# deployment 생성, 가상머신이므로 로컬 image가 아닌 hub repository 이용
kubectl create deployment ${deployment-name} --image=${remote-image} 

# deployment 제거
kubectl delete deployment ${deployment-name}

# deployment check
kubectl get deployments

# pod check
kubectl get pods

# LoadBalancer 이용하여 port 노출 및 service 생성  
kubectl expose deployment ${deployment-name} --type=LoadBalancer  --port=8080

# service check
kubectl get services

# service 접근  
minikube service ${deployment-name}  
  
# 다중 컨테이너 pods 실행  
kubectl scale deployment/${deplyment-name} --replicas=3

# 기존 deployment 컨테이너 새로운 이미지 설정  
kubectl set images deployment/${deployment-name} ${before-image-name}=${new-image-docker-hub-repository}

# deployment update 확인  
kubectl rollout status deployment/${deployment-name}  
  
# deployment history  
kubectl rollout history deployment/${deployment-name}  
kubectl rollout history deployment/${deployment-name} --revision=${revision}  
  
# deployment rollback  
# 이전  
kubectl rollout undo deployment/${deployment-name}  
# 특정 revision
kubectl rollout undo deployment/${deployment-name} --to-revision=${revision}  
  
# service 제거  
# kubectl delete service ${deployment-name}

# deployment 제거  
kubectl delete deployment ${deployment-name}

```

### Kubectl(yaml 파일 이용)

```shell
# 선언형 리소스 적용
kubectl apply -f ${file-name}.yaml

# 선언형 리소스 제거  
kubectl delete -f=${file-name}.yaml
```