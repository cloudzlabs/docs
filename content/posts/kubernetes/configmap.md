---
date: "2018-05-11T13:55:48+09:00"
title: "[Kubernetes 활용(6/8)] ConfigMap"
authors: ["blingeeeee"]
series: []
categories:
  - posts
tags:
  - Kubernetes
  - Container
  - Container Orchestration
  - k8s
  - ConfigMap
description: ""
draft: false
---
이번에는 Kubernetes에서 제공하는 ConfigMap이라는 Object를 보도록 하겠습니다.

## ConfigMap 이란?
---
ConfigMap은 컨테이너 이미지에서 사용하는 환경변수와 같은 세부 정보를 분리하고, 그 환경변수에 대한 값을 외부로 노출 시키지 않고 내부에 존재하는 스토리지에 저장해서 사용하는 방법입니다. <br/><br/>혹시 마이크로서비스 아키텍처에서 사용하는 Spring Cloud Config(Config Server)를 사용한 적이 있다면 동일한 역할을 하는 것인지 하는 생각이 들 수 있는데요. Spring Cloud Config 같은 경우에는 설정 파일 자체를 분리하고 파일에 대한 내용이 변경된다면 자동으로 Refresh 해주는 기능을 가지고 있습니다.<br/> <br/>반면에 ConfigMap 은 자동 Refresh 기능이 없고 변경 시 다시 Pod를 띄워줘야 합니다. 또한 파일로 ConfigMap을 생성한다면 설정 파일 자체를 분리할 수는 있지만 실제로 설정 파일의 값을 애플리케이션에서 사용하기 위해서는 추가적인 파싱 작업이 필요합니다. <br/><br/>그래서 동일한 역할을 한다고 볼 수는 없으며 Spring Cloud Config 가 좀 더 강력한 기능을 제공한다고 볼 수 있을 것 같습니다. <br/>하지만.. Spring Cloud Config 라이브러리는 Spring Boot로 작성되기 때문에 확실히 언어적인 종속성이 있는 것은 사실입니다. 뭐 찾아보면 다른 언어로 작성된 오픈소스가 있기야 하겠지만 아무래도 실제 시스템에서 사용되기엔 많은 검증이 필요하겠지요.<br/><br/> 그럼 ConfigMap을 일반 문자열 또는 파일로 생성하는 두 가지 방식에 대해 알아봅시다.


## ConfigMap 활용하기
----

### ConfigMap 생성 (literal values)

#### 문자열 값으로 ConfigMap 생성

`kubectl create configmap` 명령어의 **--from–literal** 옵션으로
ConfigMap에 리터럴 값을 제공합니다. 

아래와 같이 special-config 를 작성해 봅시다.

Window OS의 경우 아래 yaml 파일로 ConfigMap을 직접 생성하는 방식으로
사용하세요.

``` bash
$ kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm
```

  

다른 방법으로는 yaml 파일로 ConfigMap을 생성할 수 있으며 파일에 데이터를
key:value 형태로 등록합니다.

아래와 같이  env-config 를 작성해 봅시다.

**env-config.yaml**

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
data:
  log_level: INFO
```

``` bash
$ kubectl apply -f env-config.yaml
```

  

`kubectl describe` 또는 `kubectl get` 명령어를 통해 생성된
ConfigMap의 정보를 확인합니다.

``` bash
$ kubectl describe configmaps special-config 
Name:         special-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
special.how:
----
very
special.type:
----
charm
Events:  <none>
```

``` bash
$ kubectl get configmaps env-config -o yaml
apiVersion: v1
data:
  log_level: INFO
kind: ConfigMap
metadata:
  creationTimestamp: 2017-12-07T09:32:59Z
  name: env-config
  namespace: default
  resourceVersion: "10304"
  selfLink: /api/v1/namespaces/default/configmaps/env-config
  uid: 9cfd35ce-db31-11e7-a37c-08002780475f
```

####  Pod에서 ConfigMap 데이터 사용

아래와 같이 배포할 Pod spec에 ConfigMap의 데이터를 환경변수(***env***)로
개별적으로 정의하거나

***envFrom***을 사용하여 ConfigMap의 모든 데이터를 Pod 환경 변수로
정의할 수 있습니다. 이렇게 설정한 경우 ConfigMap의 key가 Pod 환경 변수
이름이 됩니다.

**gs-spring-boot-docker-deployment.yaml**

``` yaml
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
        - name : SPRING_PROFILES_ACTIVE
          value : k8s
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special.how
        - name: SPECIAL_TYPE_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special.type
        envFrom:
          - configMapRef:
              name: env-config
      restartPolicy: Always
```

``` bash
$ kubectl apply -f gs-spring-boot-docker-deployment.yaml
```

  
배포된 Pod에 접속하여 환경변수가 잘 적용되었는지 확인합니다.

``` bash
$ kubectl get pod
NAME                                                READY     STATUS        RESTARTS   AGE
gs-spring-boot-docker-deployment-764579896b-zm69w   1/1       Running       0          3s
$ kubectl exec -ti gs-spring-boot-docker-deployment-764579896b-zm69w sh
/ # env
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
JAVA_ALPINE_VERSION=8.131.11-r2
HOSTNAME=gs-spring-boot-docker-deployment-764579896b-zm69w
SHLVL=1
HOME=/root
SPECIAL_TYPE_KEY=charm
JAVA_VERSION=8u131
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin
KUBERNETES_PORT_443_TCP_PORT=443
JAVA_OPTS=
KUBERNETES_PORT_443_TCP_PROTO=tcp
LANG=C.UTF-8
SPECIAL_LEVEL_KEY=very
log_level=INFO
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
PWD=/
JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
KUBERNETES_SERVICE_HOST=10.96.0.1
SPRING_PROFILES_ACTIVE=k8s
```

{{% notice warning %}}
ConfigMap이 수정되면 Pod도 다시 배포되어야 적용됩니다.
{{% /notice %}}
 

 
### ConfigMap 생성 (files)

#### 파일로 ConfigMap 생성하기

아래의 redis-config 파일로 ConfigMap을 생성합니다.

`kubectl create configmap` 명령어의 ***--from–file*** 옵션으로
ConfigMap에 파일 데이터를 제공합니다. 

**redis-config**

``` text
maxmemory 2mb
maxmemory-policy allkeys-lru
```

``` bash
$ ls
redis-config
$ kubectl create configmap example-redis-config --from-file=redis-config
configmap "example-redis-config" created
```

  

ConfigMap이 제대로 생성되었는지 확인합니다.

아래와 같이 ConfigMap의 data 섹션에 파일명은 key, 파일내용 전체가
value값이 됩니다.

``` bash
$ kubectl get configmap example-redis-config -o yaml
apiVersion: v1
data:
  redis-config: |-
    maxmemory 2mb
    maxmemory-policy allkeys-lru
kind: ConfigMap
metadata:
  creationTimestamp: 2017-12-07T10:00:37Z
  name: example-redis-config
  namespace: default
  resourceVersion: "12206"
  selfLink: /api/v1/namespaces/default/configmaps/example-redis-config
  uid: 792b3fa8-db35-11e7-a37c-08002780475f
```

####  Pod에서 ConfigMap 데이터 사용 (볼륨에 마운트하여 사용하는 방법)

배포할 Pod **spec** 섹션 에서 ConfigMap의 데이터를 볼륨의 특정경로에
추가하여 사용 가능합니다.

**volumes** 섹션에 ConfigMap 이름을 추가하고 **volumeMounts.mountPath**
에 지정한 디렉토리로 ConfigMap 데이터를 추가합니다.

아래 예제의 경우 config 볼륨이 컨테이너의 **/redis-master** 경로에
마운트되고 **path** 라는 필드를 사용하여 **redis.conf**라는 파일에
**redis-config** key를 추가합니다.

따라서 마운트된 redis config 파일의 경로는 최종적으로
**/redis-master/redis.conf** 가 되고 이 파일은 example-redis-config 라는
ConfigMap 데이터인 redis-config  key의 value로 채워집니다.

**redis-pod.yaml**

``` yml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: kubernetes/redis:v1
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /redis-master-data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: example-redis-config
        items:
        - key: redis-config
          path: redis.conf
```

   아래와 같이 Pod를  배포합니다.

``` bash
$ kubectl apply -f redis-pod.yaml
```

  배포 후 redis Pod에 접속하여 해당 경로에 파일이 존재하는지 확인합니다.

``` bash
$ kubectl exec -it redis sh
# cat /redis-master/redis.conf
maxmemory 2mb
maxmemory-policy allkeys-lru# 
```

Pod에 접속하고 redis-cli를 실행하여 설정이 제대로 적용되었는지
확인합니다.

``` bash
$ kubectl exec -it redis redis-cli
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "2097152"
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"
```
{{% notice tip %}}
마운트된 ConfigMap은 자동으로 업데이트됩니다. Kubelet은 마운트된
ConfigMap이 최신인지 주기적으로 확인합니다.<br/>
그래서 ConfigMap이 업데이트되고 _**동기화 기간 + ConfigMap 캐시의 ttl**_ 만큼의 지연이 있을 수 있습니다.
{{% /notice %}}



###  ConfigMap 관리

#### ConfigMap 생성

`kubectl create configmap` 명령어 또는 yaml 파일을 통해
생성합니다.

자세한 내용은 위의 활용 부분을 참고하세요.

#### ConfigMap 수정 

##### ConfigMap 수정

`kubectl edit configmap` 명령어를 통해 configmap을 수정합니다.
예제에서는 special-config에 special.test:test 를 추가하고 확인해봅니다.

Window OS의 경우 해당 명령어로 수정이 불가합니다. 아래 파일을 직접
수정하는 방식으로 사용하세요.

``` bash
$ kubectl edit configmap special-config
configmap "special-config" edited
$ kubectl describe cm special-config
Name:         special-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
special.how:
----
very
special.test:
----
test
special.type:
----
charm
Events:  <none>
```

  
또는 ConfigMap yaml 파일을 직접 수정한 뒤 적용합니다. 예제에서는
env-config의 log\_level을 ERROR로 변경한 뒤 아래 명령어를 통해
적용합니다.

``` bash
$ kubectl apply -f env-config.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
configmap "env-config" configured

$ kubectl describe cm env-config
Name:         env-config
Namespace:    default
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","data":{"log_level":"ERROR"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"env-config","namespace":"default"}}

Data
====
log_level:
----
ERROR
Events:  <none>
```

##### 수정된 ConfigMap을 Pod에 적용

ConfigMap을 수정한 뒤에 Pod에 적용하기 위해서 Pod를 재생성해야 합니다.

`kubectl replace --force -f` 명령어를 통해 Pod를 재생성합니다.
***-l*** 옵션은 스케일링된 Pod일 경우 동일한 라벨을 가지고 스케일링
되므로 해당 라벨을 가진 모든 Pod에 적용하기 위한 설정입니다.

``` bash
$ kubectl get pod -l app=gs-spring-boot-docker -o yaml | kubectl replace --force -f -
pod "gs-spring-boot-docker-deployment-764579896b-zm69w" deleted
pod "gs-spring-boot-docker-deployment-764579896b-zm69w" replaced
```

  

새로 생성된 Pod에 변경된 ConfigMap이 제대로 적용되었는지 확인합니다. 

``` bash
$ kubectl get pod
NAME                                                READY     STATUS    RESTARTS   AGE
gs-spring-boot-docker-deployment-764579896b-dfdnq   1/1       Running   0          1m    
redis                                               1/1       Running   0          2d                            
```

  

아래 log\_level 값이 ERROR로 변경되었음을 확인합니다.

``` bash
$ kubectl exec -it gs-spring-boot-docker-deployment-764579896b-dfdnq sh
/ # env
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
JAVA_ALPINE_VERSION=8.131.11-r2
HOSTNAME=gs-spring-boot-docker-deployment-764579896b-dfdnq
SHLVL=1
HOME=/root
SPECIAL_TYPE_KEY=charm
JAVA_VERSION=8u131
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin
KUBERNETES_PORT_443_TCP_PORT=443
JAVA_OPTS=
KUBERNETES_PORT_443_TCP_PROTO=tcp
LANG=C.UTF-8
SPECIAL_LEVEL_KEY=very
log_level=ERROR
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
PWD=/
JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
KUBERNETES_SERVICE_HOST=10.96.0.1
SPRING_PROFILES_ACTIVE=k8s
```

####  ConfigMap 삭제

`kubectl delete` 명령어를 통해 ConfigMap 을 삭제합니다.

``` bash
$ kubectl delete -f env-config.yaml
configmap "env-config" deleted
```

  

`kubectl get configmap` 명령어를 통해 configmap 목록을
확인합니다. 제대로 삭제되었는지 확인해봅니다.

``` bash
$ kubectl get configmap
NAME                   DATA      AGE
special-config         3         3d
```
