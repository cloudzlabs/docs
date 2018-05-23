---
date: "2018-02-22T14:41:42+09:00"
title: "[Kubernetes 활용(2/8)] Rolling Update(무중단 배포)"
authors: ["blingeeeee"]
series: ["k8s"]
categories:
  - posts
tags:
  - Kubernetes
  - Container
  - Container Orchestration
  - Rolling Update
description: ""
draft: false
---
지난 챕터에서는 Kubernetes 환경에서 애플리케이션을 배포하고 접속하는 방법을 알아보았습니다. <br/>그렇다면 이미 배포되어 있는 애플리케이션을 업데이트할 때 중단 없이 처리할 수 있을까요??<br/><br/>
네. 가능합니다!<br/>
Kubernetes에서는 중단 없이 애플리케이션을 배포할 수 있도록 Rolling Update라는 기능을 지원하고 있습니다.<br/>
지난 챕터에서 간단히 나오긴 했지만 다시 한번 자세히 알아봅시다.

## Rolling Update 란?
------------------------------------------------------------------------
서비스 중단 없이 애플리케이션을 업데이트 하기 위해서, Kubernetes에서는
rolling update라는 기능을 지원합니다. 이 기능을 통해서 전체 Pod을 일시에
중단/업데이트 하는 것이 아니라, 한번에 n개씩 Pod을 순차적으로 업데이트할
수 있습니다. 이를 통해 서비스 중단 현상 없이 애플리케이션 버전 업데이트
및 롤백을 할 수 있습니다.

`kubectl rolling-update` 커맨드의 사용은 [Replication
Controllers](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)를
이용해 애플리케이션을 배포한 경우에만 사용 합니다. 최신 버전의
Kubernetes에서는
[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)를
이용한 애플리케이션 배포를 권장 합니다.

  

## Rolling Update 하기

------------------------------------------------------------------------

### 애플리케이션 배포

우선 Rolling Update 테스트를 위한 애플리케이션 버전 1.0을 아래와 같이
배포 합니다. (Deployment, Service 생성 필요)

아래의 설정 내용 중, Image 설정이 'dtlabs/gs-spring-boot-docker:1.0' 임을 통해 Image 버전(1.0)을 확인 합니다.

  

1.  Deployment 생성

    **gs-spring-boot-docker-deployment.yaml**

    ```yaml
    apiVersion: apps/v1beta1 # for versions before 1.8.0 use apps/v1beta1
    kind: Deployment
    metadata:
      name: gs-spring-boot-docker-deployment
      labels:
        app: gs-spring-boot-docker
    spec:
      replicas: 3
      minReadySeconds: 10
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 0
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
            imagePullPolicy: Always
            ports:
            - containerPort: 8080
    ```

    ###### line8 spec.replicas

    Pod의 복제 수를 나타냅니다. rolling update 테스트를 위해 초기 설정을
    Pod 3개 복제로 설정하였습니다.

    ###### line9 spec.minReadySeconds

    Pod이 Ready 단계가 된 후, Available 단계가 될 때 까지의 시간 차이를
    의미합니다. 테스트 결과, minReadySeconds를 설정하지 않으면,
    Ready에서 곧 바로 Available이 되고, 몇 초간 순단 현상이 있음이
    확인되었습니다. 반면, minReadySeconds를 통해 10초 정도 여유 시간을
    주었을 때, 순단이 최소화 되는 것을 확인 하였습니다. Pod의 컨테이너가
    초기화 되는 시간을 고려하여 적절한 시간 minReadySeconds를 설정하는
    것을 권장합니다.

    ###### line10 spec.strategy

    RollingUpdate에 대한 상세 설정을 합니다.

    ###### line11 spec.strategy.type

    "Recreate" or "RollingUpdate"를 설정 가능 합니다. 기본값은
    "RollingUpdate" 입니다. Recreate의 경우 Pod가 삭제된 후 재생성 됩니다.

    ###### line12 spec.strategy.rollingUpdate

    spec.strategy.type에서 "RollingUpdate"를 설정한 경우,
    RollingUpdate에 대한 상세 설정을 합니다.

    ###### line13 spec.strategy.rollingUpdate.maxSurge

    rolling update 중 정해진 Pod 수 이상으로 만들 수 있는 Pod의 최대 개수입니다.
    기본값은 25% 입니다.

    ###### line14 spec.strategy.rollingUpdate.maxUnavailable

    rolling update 중 unavailable 상태인 Pod의 최대 개수를 설정 합니다.
    rollgin update 중 사용할 수 없는 Pod의 최대 개수입니다.
    값은 0보다 큰 정수를 통해 Pod의 절대 개수 설정이 가능하고, "25%"와
    같이 percentage 표현 또한 가능 합니다. maxUnavailable에서 percentage
    계산은 rounding down(내림) 방식이며 기본값은 25% 입니다.
    maxSurge와 maxUnavailable 값이 동시에 0이 될 수는 없습니다.

    -   replica: 3인 경우, 25%는 0.75개 이지만, 최소값이 1이기 때문에
        maxUnavailable은 1개로 계산 됩니다.

    -   replica: 9인 경우, 25%는 2.25개 이지만, rounding down(내림) 하여
        maxUnavailable은 2개로 계산 됩니다.

    ``` bash
    # Deployment 생성을 통해 애플리케이션을 배포 합니다.
    # kubectl create 명령어에서 -f 옵션을 통해 파일명이 gs-spring-boot-docker-deployment.yaml 임을 인자로 전달합니다.
    $ kubectl create --save-config -f ./gs-spring-boot-docker-deployment.yaml
    deployment "gs-spring-boot-docker-deployment" created
    ```

2.  Service 생성

    **gs-spring-boot-docker-service.yaml**

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: gs-spring-boot-docker-service
    spec:
      ports:
        - name: http
          port: 8081
          targetPort: 8080
      selector:
        app: gs-spring-boot-docker
      type: NodePort
    ```

    ``` bash
    # Service 생성
    # kubectl create 명령어에서 -f 옵션을 통해 파일명이 gs-spring-boot-docker-service.yaml 임을 인자로 전달합니다.
    $ kubectl create --save-config -f ./gs-spring-boot-docker-service.yaml
    service "gs-spring-boot-docker-service" created
    ```

  Deployment 및 Service 생성 관련 상세 방법은 바로 이전 챕터를 확인하시기 바랍니다.

### Rolling Update 및 확인

Deployment를 이용한 애플리케이션 배포 방식에서는 rolling update를 위해
`kubectl set image` 명령어를 사용합니다.

1.  Rolling Update 진행 모니터링

    `kubectl get pod` 명령어에서 옵션으로 '-w '을 사용하게 되면, 실시간으로 갱신되는 정보를 지속 조회할 수 있습니다.<br/>
    별개의 커맨드창을 사용하여 버전 업데이트 수행 전부터 실시간 갱신 정보의 모니터링을 시작합니다.<br/>
    Deployment를 통해 'replica: 3' 을 설정 하였기 때문에 3개의 Pod이 Running 되는 것을 확인할 수 있습니다. 

    ``` bash
    $ kubectl get pod -w
    NAME                                                READY     STATUS    RESTARTS   AGE
    gs-spring-boot-docker-deployment-2374595156-47dz4   1/1       Running   0          1m
    gs-spring-boot-docker-deployment-2374595156-5fvft   1/1       Running   0          59s
    gs-spring-boot-docker-deployment-2374595156-zb0nt   1/1       Running   0          1m
    ```

2.  `kubectl set image` 명령어를 통한 Image 2.0 버전 업데이트 수행

    ``` bash
    $ kubectl set image deployment/gs-spring-boot-docker-deployment gs-spring-boot-docker=dtlabs/gs-spring-boot-docker:2.0
    deployment "gs-spring-boot-docker-deployment" image updated
    ```

3.  Rolling Update 확인

    ``` bash
    $ kubectl get pod -w
    NAME                                                READY     STATUS    RESTARTS   AGE
    gs-spring-boot-docker-deployment-2374595156-5fvft   1/1       Running   0          59s          <- Old Pod #1
    gs-spring-boot-docker-deployment-2374595156-47dz4   1/1       Running   0          1m           <- Old Pod #2
    gs-spring-boot-docker-deployment-2374595156-zb0nt   1/1       Running   0          1m           <- Old Pod #3

    gs-spring-boot-docker-deployment-313620219-m0hgq   0/1       Pending   0         0s             <- New Pod #1 생성 시작
    gs-spring-boot-docker-deployment-313620219-m0hgq   0/1       Pending   0         0s
    gs-spring-boot-docker-deployment-313620219-m0hgq   0/1       ContainerCreating   0         0s
    gs-spring-boot-docker-deployment-313620219-m0hgq   1/1       Running   0         1s             <- New Pod #1 생성 완료

    gs-spring-boot-docker-deployment-2374595156-5fvft   1/1       Terminating   0         1m        <- Old Pod #1 Terminating 시작

    gs-spring-boot-docker-deployment-313620219-78q8s   0/1       Pending   0         0s             <- New Pod #2 생성 시작
    gs-spring-boot-docker-deployment-313620219-78q8s   0/1       Pending   0         0s
    gs-spring-boot-docker-deployment-313620219-78q8s   0/1       ContainerCreating   0         0s
    gs-spring-boot-docker-deployment-313620219-78q8s   1/1       Running   0         0s             <- New Pod #2 생성 완료

    gs-spring-boot-docker-deployment-2374595156-5fvft   0/1       Terminating   0         1m        <- Old Pod #1 Terminating 완료
    gs-spring-boot-docker-deployment-2374595156-47dz4   1/1       Terminating   0         1m        <- Old Pod #2 Terminating 시작

    gs-spring-boot-docker-deployment-313620219-q3js4   0/1       Pending   0         0s             <- New Pod #3 생성 시작
    gs-spring-boot-docker-deployment-313620219-q3js4   0/1       Pending   0         0s
    gs-spring-boot-docker-deployment-313620219-q3js4   0/1       ContainerCreating   0         0s
    gs-spring-boot-docker-deployment-313620219-q3js4   1/1       Running   0         0s             <- New Pod #3 생성 완료

    gs-spring-boot-docker-deployment-2374595156-5fvft   0/1       Terminating   0         1m        <- Old Pod #1 Terminating 완료
    gs-spring-boot-docker-deployment-2374595156-47dz4   1/1       Terminating   0         1m        <- Old Pod #2Terminating 완료
    gs-spring-boot-docker-deployment-2374595156-zb0nt   0/1       Terminating   0         1m        <- Old Pod #3 Terminating 시작
    gs-spring-boot-docker-deployment-2374595156-zb0nt   0/1       Terminating   0         1m        <- Old Pod #3 Terminating 완료
    ```

