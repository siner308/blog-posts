---
title:  "쿠버네티스 정리 3. 로컬에서 서비스 띄워보고 수정해보기"
subtitle: "minikube, kubectl 첫 경험"
tags:
    - kubernetes
    - container
date:   2022-03-05
image: https://user-images.githubusercontent.com/34048253/156888390-6c95f6c4-dd5c-49b1-b6bc-b6ec5ca5c624.png
---

# minikube 소개
k8s를 사용하기 위해서는 master 노드 2개와 worker 노드 3개가 필요하고 이들을 연결해야하는데, 로컬에서 테스트하는 환경에서는 이러한 작업이 매우 힘들기 때문에, minikube라는 친구를 사용해서 로컬에서 작업을 합니다.

minikube는 master process와 worker process 모두를 가지고 있는 one node cluster입니다.
아래의 두 링크중 하나를 선택하고, 아래의 과정을 이어서 진행하면 됩니다.

- [minikube start | minikube](https://minikube.sigs.k8s.io/docs/start/) minikube 설치
- [Hello Minikube | Kubernetes](https://kubernetes.io/ko/docs/tutorials/hello-minikube/) 웹 터미널에서 가볍게 해보기

# kubectl 소개

k8s의 api server와 인터렉션 할수 있는 방법은 UI대시보드, CLI툴, API 직접호출 이렇게 세가지가 있는데, 이중 CLI를 맡고있는 것이 kubectl입니다.

kubectl은 minikube뿐만 아니라, 모든 k8s 쿨러스터에서 사용됩니다.
테스팅환경에서는 minikube를 사용하고 운영환경에서는 k8s를 사용하지만, 이를 다루는 방법은 kubectl이라는 CLI 툴 하나로 통일되어있어서 실무 작업을 하는 입장에서 두번 공부할 필요가 없어서 참 좋습니다.

# minikube, kubectl 설치

아래의 명령어는 Mac OS에서의 설치 방법입니다.
```bash
brew install hyperkit
brew install minikube
```

웹 터미널을 사용하는 경우에도 해당 명령어를 입력해야 합니다.
```bash
minikube start
```

아래의 명령어들로 각각의 설치가 잘 되었는지 확인이 가능합니다.
```bash
kubectl get nodes
minikube status
```

- minikube cli: minikube를 시작하거나 종료시키기 위해 사용합니다.
- kubectl cli: minikube를 포함한 쿠버네티스의 상태를 변경시키기 위해 사용합니다. (deployment의 추가 등)

# basic kubectl commands

## create deployment
실제로 deployment를 생성해보면서 명령어를 익혀봅시다.

아래 명령어는 nginx 이미지를 통해 deployment를 생성해줍니다.
```bash
kubectl create deployment nginx-depl --image=nginx
```

### CRUD 명령어 예시
- `Create` deployment
  - kubectl create deployment [name]
- `Edit` deployment
  - kubectl edit deployment [name]
- `Delete` deployment
  - kubectl delete deployment [name]

### k8s 컴포넌트 상태 확인
```bash
kubectl get nodes
kubectl get pod
kubectl get services
kubectl get replicaset
kubectl get deployment
```

`-o wide` 옵션으로 더 많은 정보를 볼 수 있습니다.

```bash
kubectl get pod -o wide
```

### Debugging pods
- `로그` 확인
  - kubectl logs [pod name]
- `터미널` 접속
  - kubectl exec -it [pod name] -- bin/bash
 
## edit deployment

아래의 명령어를 입력하면 nginx-depl의 설정 yaml파일을 직접 수정할 수 있습니다.
```bash
kubectl edit deployment nginx-depl
```

수정 이후 새로운 deployment로 교체되는것을 확인할 수 있습니다.

```bash
$ kubectl get pod
NAME                          READY   STATUS              RESTARTS   AGE
nginx-depl-5ddc44dd46-knmnw   1/1     Running             0          5m11s
nginx-depl-7d459cf5c8-fd5rw   0/1     ContainerCreating   0          5s

$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5ddc44dd46   0         0         0       5m55s
nginx-depl-7d459cf5c8   1         1         1       49s
```

## delete deployment
```bash
kubectl delete deployment nginx-depl
```

# using configuration file
cli로 모든 인프라를 관리하는건 어렵고, 형상관리가 힘들다는 점이 있습니다.
yaml 파일로 관리하면 형상관리가 가능하고, 읽기 편합니다. ~들여쓰기는 안편합니다~

**nginx-deployment.yaml** 파일을 생성해서 다음의 내용을 넣습니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec: # for deployment
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec: # for pods
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 80
```

아래의 명령어로 적용합니다.
```bash
kubectl apply -f nginx-deployment.yaml
```

배포 내용을 변경하고싶다면 파일을 수정한 뒤, 위의 명령어를 다시 입력해주면 됩니다.

## 3 part of configuration file
yaml파일은 3가지 파트로 구분됩니다.

1. metadata
2. specification
3. status (k8s에서 자동으로 생성해주는 부분. 내가 작성하는것이 아님.)

k8s는 etcd를 통해 status data를 가져옵니다.
![image](https://user-images.githubusercontent.com/34048253/156887814-570c64ff-3c0d-4c34-9447-c4b9e3daa84c.png)

아래의 명령어를 통해 status를 직접 확인할 수 있습니다.
```bash
kubectl get deployment nginx-deployment -o yaml > nginx-deployment-result.yaml
```
![image](https://user-images.githubusercontent.com/34048253/156887977-30862692-dc90-499e-b406-69efc930986b.png)


label과 selector로 연결해줍니다.

![image](https://user-images.githubusercontent.com/34048253/156887868-d324adb7-c634-4280-ab19-a1408f42891f.png)

service에서 targtePort를 사용하는 경우, 다른 서비스에서 port로 호출시, targetPort로 보내줍니다. (아래 예시)

![image](https://user-images.githubusercontent.com/34048253/156887919-9acea45c-f5ea-49f7-8b31-6a24e484a101.png)
