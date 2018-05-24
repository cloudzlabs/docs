---
date: "2018-05-11T13:55:54+09:00"
title: "[Kubernetes 활용(7/8)] Secret"
authors: ["blingeeeee"]
series: []
categories:
  - posts
tags:
  - kubernetes
  - container
  - container orchestration
  - k8s
  - secret
description: ""
draft: false
---
지난 챕터의 ConfigMap에 이어서 이번에는 Secret Object를 보도록 하겠습니다.

## Secret 이란?
---
Secret은 비밀번호나 OAuth 토큰값 또는 ssh key 등의 민감한 정보를
유지하기 위해 사용됩니다.<br/>
이러한 정보를 Docker 이미지나 Pod에 그대로 정의하기 보다 Secret을
활용하면 더욱 안전하고 유동적으로 사용할 수 있습니다.

## Secret 적용하기
---
### Secret 생성

#### 명령어를 통해 생성하기

아래와 같이 `kubectl create secret` 명령어를 통해 Secret을
생성합니다.

Window OS의 경우 아래 yaml 파일로 Secret을 직접 생성하는 방식으로
사용하세요.

``` bash
$ kubectl create secret generic db-user-pass --from-literal=user=admin --from-literal=password=1f2d1e2e67df
secret "db-user-pass" created
```

  

  

아래와 같이 `kubectl get` 명령어와 `kuberctl describe`
명령어를 통해 생성된 Secret을 확인합니다.

이 명령어들로는 Secret 데이터를 볼 수 없으며 이는 Secret이 노출되지
않도록 보호하기 위함입니다.

데이터를 보려면 아래 **Sercret 디코딩** 을 참고하세요.

``` bash
$ kubectl get secrets
NAME                  TYPE                                  DATA      AGE
db-user-pass          Opaque                                2         25s

$ kubectl describe secrets/db-user-pass
Name:         db-user-pass
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  12 bytes
user:      5 bytes
```

  

#### yaml 파일을 통해 생성하기

yaml 형식의 파일을 만들고 `kubectl create` 명령어를 통해 Secret 을
생성합니다. 각 항목은 base64로 인코딩 되어야 합니다.

``` bash
$ echo -n "admin" | base64
YWRtaW4=

$ echo -n "1f2d1e2e67df" | base64
MWYyZDFlMmU2N2Rm
```

**secret.yaml**

``` yml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

``` bash
$ kubectl apply -f secret.yaml
secret "mysecret" created
```

####  Secret 디코딩

Secret에 작성된 실제 데이터를 확인하려면 `kubectl get secret`
명령어를 통해 인코딩 된 데이터를 확인합니다.

``` bash
$ kubectl get secret mysecret -o yaml
apiVersion: v1
data:
  password: MWYyZDFlMmU2N2Rm
  username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: 2017-12-11T06:31:37Z
  name: mysecret
  namespace: default
  resourceVersion: "76497"
  selfLink: /api/v1/namespaces/default/secrets/mysecret
  uid: f0f1b4f5-de3c-11e7-a37c-08002780475f
type: Opaque
```

인코딩 되어 있는 password 값을 디코드하여 실제 데이터를 확인합니다.

``` bash
$ echo "MWYyZDFlMmU2N2Rm" | base64 --decode
1f2d1e2e67df
```

###  Secret 사용

#### Secret을 Pod에서 환경변수로 사용

아래와 같이 Secret을 사용할 Pod의 spec에  env를 개별적으로 정의합니다.

**env[].valueFrom.secretKeyRef** 에 Secret의 이름과 key를 채워 환경 변수를 정의합니다.

**gs-spring-boot-docker-deployment.yaml**

``` yml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: gs-spring-boot-docker-deployment
  labels:
    app: gs-spring-boot-docker
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gs-spring-boot-docker
  template:
    metadata:
      labels:
        app: gs-spring-boot-docker
    spec:
      containers:
      - name: gs-spring-boot-docker
        image: dtlabs/gs-spring-boot-docker:1.0
        ports:
        - containerPort: 8080
        env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: username
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: password
      restartPolicy: Always
```

이렇게 설정한 뒤 Deployment를 배포합니다.

``` bash
kubectl apply -f gs-spring-boot-docker-deployment.yaml
```

  

Pod의 컨테이너 내부에서는 Secret key 값이 base64 디코드된 값의 정상
환경변수로 나타납니다.

 배포된 Pod에 접속하여  환경변수가 잘 적용되었는지 확인합니다.

아래와 같이 컨테이너 접속 후 env 명령어를 통해 Secret 정보인 환경
변수(SECRET\_USERNAME, SECRET\_PASSWORD)가 제대로 적용되었음을
확인합니다.

``` bash
$ kubectl get pod
NAME                                                READY     STATUS    RESTARTS   AGE
gs-spring-boot-docker-deployment-5d7db89759-xvm5t   1/1       Running   0          1m

$ kubectl exec -it gs-spring-boot-docker-deployment-5d7db89759-xvm5t sh
/ # env
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
JAVA_ALPINE_VERSION=8.131.11-r2
HOSTNAME=gs-spring-boot-docker-deployment-5d7db89759-xvm5t
SHLVL=1
HOME=/root
SECRET_PASSWORD=1f2d1e2e67df
JAVA_VERSION=8u131
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin
KUBERNETES_PORT_443_TCP_PORT=443
JAVA_OPTS=
KUBERNETES_PORT_443_TCP_PROTO=tcp
LANG=C.UTF-8
SECRET_USERNAME=admin
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
PWD=/
JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
KUBERNETES_SERVICE_HOST=10.96.0.1
```

###  Secret 관리

#### Secret  생성

`kubectl create secret` 명령어 또는 yaml 파일을 통해 생성합니다.

자세한 내용은 위의 활용 부분을 참고하세요.

#### Secret 수정

##### Secret 수정

`kubectl edit secret` 명령어를 통해 수정합니다.

수정할 데이터는 base64로 인코딩 된 값이어야 하며 예제에서는 user 값을
변경해 봅시다.

Window OS의 경우 해당 명령어로 수정이 불가합니다. 아래 파일을 직접
수정하는 방식으로 사용하세요.

``` bash
$ echo -n "test" | base64
dGVzdA==
$ kubectl edit secret db-user-pass 
secret "db-user-pass" edited
$ kubectl get secret db-user-pass -o yaml
apiVersion: v1
data:
  password: MWYyZDFlMmU2N2Rm
  user: dGVzdA==
kind: Secret
metadata:
  creationTimestamp: 2017-12-11T06:16:25Z
  name: db-user-pass
  namespace: default
  resourceVersion: "85279"
  selfLink: /api/v1/namespaces/default/secrets/db-user-pass
  uid: d0eb0495-de3a-11e7-a37c-08002780475f
type: Opaque
```

  

또는 Secret yaml 파일을 직접 수정한 뒤 적용합니다. 예제에서는 mysecret의
username 값을 변경한 뒤 

**secret.yaml**

``` yml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: dGVzdA==
  password: MWYyZDFlMmU2N2Rm
```

  

아래 명령어 `kubectl apply`를 통해 적용하고 확인합니다.

``` bash
$ kubectl apply -f secret.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
secret "mysecret" configured


$ kubectl get secret mysecret -o yaml
apiVersion: v1
data:
  password: MWYyZDFlMmU2N2Rm
  username: dGVzdA==
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"password":"MWYyZDFlMmU2N2Rm","username":"dGVzdA=="},"kind":"Secret","metadata":{"annotations":{},"name":"mysecret","namespace":"default"},"type":"Opaque"}
  creationTimestamp: 2017-12-11T06:31:37Z
  name: mysecret
  namespace: default
  resourceVersion: "86747"
  selfLink: /api/v1/namespaces/default/secrets/mysecret
  uid: f0f1b4f5-de3c-11e7-a37c-08002780475f
type: Opaque
```

#####  수정된 Secret을 Pod에 적용

Secret을 수정한 뒤에 Pod에 적용하기 위해서 Pod를 재생성해야 합니다.

`kubectl replace --force -f` 명령어를 통해 Pod를
재생성합니다. ***-l*** 옵션은 스케일링된 Pod일 경우 동일한 라벨을 가지고
스케일링 되므로 해당 라벨을 가진 모든 Pod에 적용하기 위한 설정입니다.

``` bash
$ kubectl get pod -l app=gs-spring-boot-docker -o yaml | kubectl replace --force -f -
pod "gs-spring-boot-docker-deployment-5d7db89759-xvm5t" deleted
pod "gs-spring-boot-docker-deployment-5d7db89759-xvm5t" replaced
```

  

새로 생성된 Pod에 변경된 Secret이 제대로 적용되었는지 확인합니다. 
컨테이너에 접속하여 env 명령어를 통해 환경변수(SECRET\_USERNAME) 값이
test로 제대로 변경되었는지 확인합니다.

``` bash
$ kubectl get pod
NAME                                                READY     STATUS    RESTARTS   AGE
gs-spring-boot-docker-deployment-5d7db89759-hn5q8   1/1       Running   0          1m
$ kubectl exec -it gs-spring-boot-docker-deployment-5d7db89759-hn5q8 sh
/ # env
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
JAVA_ALPINE_VERSION=8.131.11-r2
HOSTNAME=gs-spring-boot-docker-deployment-5d7db89759-hn5q8
SHLVL=1
HOME=/root
SECRET_PASSWORD=1f2d1e2e67df
JAVA_VERSION=8u131
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin
KUBERNETES_PORT_443_TCP_PORT=443
JAVA_OPTS=
KUBERNETES_PORT_443_TCP_PROTO=tcp
LANG=C.UTF-8
SECRET_USERNAME=test
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
PWD=/
JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
KUBERNETES_SERVICE_HOST=10.96.0.1
```

####  Secret 삭제

`kubectl delete` 명령어를 통해 yaml 파일을 통해 생성했던
mysecret을 삭제합니다.

``` bash
$  kubectl delete -f secret.yaml
secret "mysecret" deleted
```

  

`kubectl get secret` 명령어를 통해 secret 목록을 확인합니다.
mysecret이 제대로 삭제되었는지 확인해봅니다.

``` bash
$ kubectl get secret
NAME                  TYPE                                  DATA      AGE
db-user-pass          Opaque                                2         4h
```
