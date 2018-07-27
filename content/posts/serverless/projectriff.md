---
date: "2018-07-17T11:04:50+09:00"
title: "Kubernetes와 Serverless의 : Riff Project"
authors: ["hunkee1017"]
series: []
categories:
  - posts
tags:
  - serverless
  - riff
  - kubernetes
  - container
  - spring framework
cover:
  image: "../projectriff/projectriff.png"
  caption: ""
description: ""
draft: true
---

### 서버리스?


### Riff Project

2017.12 샌프란시스코에서 열린 피보탈 주최 SpringOne Platform의 메인스테이지에서 Riff프로젝트가 처음으로 대중에게 공개되었습니다.    
발표자가 라이브코딩으로 Function을 생성하고 이를 Minikube 환경에서 구동시켰을 때 Pod이 생성되었다가 부하가 심해지면 Pod의 갯수를 늘리고 함수의 처리가 끝나면 사라지는 데모를 시현했던것이 기억납니다.    

그 자리에는 저도 참석하고 있었는데 Serverless를 Kubernetes 환경에서 손쉽게 사용할 수 있다는 장점이 크게 다가왔고 이에 Riff를 다뤄보고자 합니다.




Riff를 처음 접하고 나서 0.0.5 버전에서부터 가이드 문서를 보고 설치를 시도했었지만 원인을 알 수 없는 에러로 인해 계속 실패를 반복하다가 0.0.7 버전이 나와서 다시 시도해서 성공한 내용을 포스팅 하려고 합니다.

현재 제 PC에 설치된 동작하는 버전은 다음과 같습니다.

``` 
minikube version: v0.26.0
riff CLI version: v0.0.7
helm version: v2.8.2
```

### Riff 설치에서 구동까지

#### Riff 설치

Riff 프로젝트를 설치하기 위해서는 선행으로 설치되어야할 몇가지가 있습니다.

1. **Docker** : Riff에서 Docker CLI를 사용하기 위해서 설치해야 합니다.    
<https://www.docker.com/community-edition>

2. **Kubectl** : Kubernetes CLI를 활용하기 위해서 설치해야 합니다.    
<https://kubernetes.io/docs/tasks/tools/install-kubectl/>

3. **Minikube** : Minikube는 1 master, 1 node 클러스터 환경으로 로컬에서 Kubernetes를 활용하기 위한 가장 쉬운 접근 방법입니다.     
<https://kubernetes.io/docs/tasks/tools/install-minikube/>

4. **Helm** : Helm은 Kubernetes에 패키징된 응용프로그램을 설치하고 lifecycle을 관리해주는 도구입니다.    
<https://docs.helm.sh/using_helm/#installing-helm>

5. **Riff CLI** : Riff 프로젝트에서 사용할 Riff CLI는 주어진 파일을 통해서 Function을 빌드하거나 배포, 삭제할 수 있습니다.    
<https://github.com/projectriff/riff/releases>

Minikube 실행
``` bash
minikube start --memory=4096 --bootstrapper=kubeadm
```

``` bash
eval $(minikube docker-env)
```

``` bash
helm repo add projectriff https://riff-charts.storage.googleapis.com
helm repo update
```

``` bash
kubectl -n kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account=tiller
```

``` bash
helm install projectriff/riff \
  --name projectriff \
  --namespace riff-system \
  --set kafka.create=true \
  --set httpGateway.service.type=NodePort
```

``` bash
NAME:   projectriff
LAST DEPLOYED: Mon Jul 16 16:55:33 2018
NAMESPACE: riff-system
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Deployment
NAME                                  DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
projectriff-kafka                     1        1        1           0          0s
projectriff-zookeeper                 1        1        1           0          0s
projectriff-riff-function-controller  1        1        1           0          0s
projectriff-riff-http-gateway         1        1        1           0          0s
projectriff-riff-topic-controller     1        1        1           0          0s

==> v1/Pod(related)
NAME                                                  READY  STATUS             RESTARTS  AGE
projectriff-kafka-855897b9b6-pr4fm                    0/1    ContainerCreating  0         0s
projectriff-zookeeper-5f898c6869-cwpb7                0/1    ContainerCreating  0         0s
projectriff-riff-function-controller-59f964dd4-nsd9z  0/1    ContainerCreating  0         0s
projectriff-riff-http-gateway-645dd59964-dlp8c        0/1    ContainerCreating  0         0s
projectriff-riff-topic-controller-76b8d67f6-bxlgq     0/1    ContainerCreating  0         0s

==> v1beta1/ClusterRoleBinding
NAME              AGE
projectriff-riff  0s

==> v1beta1/Role
NAME              AGE
projectriff-riff  0s

==> v1beta1/ClusterRole
projectriff-riff  0s

==> v1beta1/RoleBinding
NAME              AGE
projectriff-riff  0s

==> v1/Service
NAME                           TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)       AGE
projectriff-kafka              ClusterIP  10.97.178.71   <none>       9092/TCP      0s
projectriff-zookeeper          ClusterIP  10.98.194.31   <none>       2181/TCP      0s
projectriff-riff-http-gateway  NodePort   10.102.203.57  <none>       80:31223/TCP  0s

==> v1/ServiceAccount
NAME              SECRETS  AGE
projectriff-riff  1        0s

==> v1beta1/CustomResourceDefinition
NAME                      AGE
functions.projectriff.io  0s
invokers.projectriff.io   0s
topics.projectriff.io     0s


NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace riff-system -o jsonpath="{.spec.ports[0].nodePort}" services projectriff-riff-http-gateway)
  export NODE_IP=$(kubectl get nodes --namespace riff-system -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

참조 : **(<https://projectriff.io/docs/getting-started-on-minikube/>**

