---
date: "2018-03-09T08:27:58+09:00"
title: "Swarm-Architecture"
authors: ["1000jaeh"]
categories:
  - posts
tags:
  - docker
  - swarm
  - container ochestration
cover:
  image: "../images/docker-official.svg"
draft: true
---
## Swarm의 구조

![swarm](https://learning-continuous-deployment.github.io/assets/images/docker-swarm.png)

구성된  Docker Swarm의 구조는 위의 그림과 같으며, 각 항목에 대해서 자세히 살펴보도록 하겠습니다.

### Nodes

Node는 Swarm에 참여하는 Docker Engine의 Instance들 입니다. 하나의 물리적인 Server 또는 Cloud Server에서 하나 이상의 Node를 실행할 수 있지만, 일반적으로 Production 환경에서의 Swarm은 여러 물리적인
Server와 Cloud System에 걸쳐 분산된 Docker Node들이 배포되는 형태로 사용됩니다.

#### Manager Node

Application을 Swarm에 배포하기 위해서는, 먼저 Manager Node에 Service 정의서를 제출해야합니다. 그 후, Manager Node는 Worker Node들에 Task라고 불리는 작업 단위를 전달하여, Service를 정상적으로 생성합니다. 기본적으로, Manager Node는 Service를 Worker Node로 실행하지만, Manager 전용 Task를 Manager 전용 Node에서만 실행하도록 구성할 수도 있습니다. 또한, Manager Node는 안정적인 Swarm 상태를 유지하는데 필요한 Ochestration 및 Cluster 관리 기능들을 수행합니다. 많은 Manager Node들 중에서 Ochestration 작업을 수행할 단일 리더를 선출하기도 합니다.

#### Worker Node

Worker Node는 Manager Node로부터 Task를 수신받고 Container를 실행합니다. Worker Node에서는 각각의 Agent들이 존재하며, Worker Node에서 실행되고 할당된 Task의 상태에 대해서 Manager Node에 알립니다. 이를 통해, Manager Node가 Worker Node들이 안정적인 상태를 유지할 수 있도록 합니다.

참고
역할별 Node의 상태에 대한 자세한 내용은 [Manage node in a Swarm](https://docs.docker.com/engine/swarm/manage-nodes/)문서를 참고하시기 바랍니다.

### Services and tasks

#### Service

Service는 Manager 또는 Worker Node에서 실행되는 Task에 대한 정의라고 할 수 있습니다. Service는 Swarm System의 중심적인 구성요소이며, Swarm과 사용자 사이에서 상호작용하고 있는 주요 근본입니다.

Service를 생성할 때, 사용할 Container Image와 Container에서 실행할 명령을 지정할 수 있으며, Replicated된 Service Model에서는, Swarm Manager에 의해 Scale된 Node들의 Replica 개수만큼 Task를 배포합니다. Global Service들의 경우, Swarm은 Cluster에서 사용 가능한 모든 Node에서 해당 Service에 대해 하나의 Task만을 실행합니다.

#### Task

Task는 Docker Container와 Container 내에서 실행될 명령을 전달하는 역할을 하며, Swarm에서 가장 작은 Scheduling 단위입니다. Manager Node는 Service Scale에 설정된 Replica 개수에 따라 Worker Node에
Task를 할당합니다. 일단 Task가 Node에 할당되면, 해당 Task는 다른 Node로 이동할 수 없으면, 지정된 Node에서만 실행되거나 중지될 수 있습니다.

## Swarm Node 동작 방식

![](https://docs.docker.com/engine/swarm/images/swarm-diagram.png)

### Manager nodes

Manager Node는 Cluster 관리 Task를 처리합니다.

- Cluster 상태 유지
- Scheduling Service
- Swarm Mode의 HTTP API Endpoints 제공

Raft 합의 알고리즘을 사용하여 구현되었기 때문에, Manager들은 Swarm 내의 모든 실행 중이 Service들의 내부 상태를 일관되게 유지할 수 있습니다.

참고
Raft 합의 알고리즘에 대한 자세한 내용은 [Raft Consensus Algorithm](https://raft.github.io/)문서를 참고하시기 바랍니다. Swarm을 Test 목적으로 사용하기 위해서는, Single Manager로 구성하여 사용하는 것이 좋습니다. 단, Single Manager로 구성된 Swarm의 Manager Node에서 이슈가 발생했을 경우, Swarm상의 Service들은 정상적인 상태로 계속 실행되지만, 새로운 Cluster를 구성하거나 Manager를 복구시켜야 합니다.

Swarm Mode의 Fault-Tolerance 이점을 활용하기 위해서는, Docker의 고가용성 요구사항에 따라 홀수개의 Node로 구성하는 것을 권장합니다. Swarm을 Multi Manager로 구성할 경우, Manager Node의 중단 시간 없이
Manager Node에서 발생한 오류를 복구할 수 있습니다.

다음을 참고하여 Swarm의 Manager Node를 구성할 수 있습니다.

- 3-Manager Swarm은 최대 1개 Manager Node의 손실에 대해서 허용 가능
- 5-Manager Swarm은 최대 2개 Manager Node의 동신 손실에 대해서 허용 가능
- n 개의 Manager Cluster는 최대 `(n-1)/2`개의 Manager Node의 손실에 대해서 허용 가능
- Docker는 최대 7개의 Manager Node를 구성할 것을 권장합니다.

참고
더 많은 Manager Node를 추가한다고 해서, Swarm의 확장성 또는 성능이 향상된다는 의미는 아닙니다.

### Worker Nodes

Worker Node는 Container를 실행하는 것이 유일한 목적인 Docker Engine의 Instance입니다. Worker Node는 Manager Node와 다르게, Raft 분산 상태에 참여하지 않으며, Schedule에 대한 결정을 내리거나, Swarm Mode의 HTTP API를 제공하지 않습니다.

Manager Node 하나로만 구성된 Swarm을 구성할 수 있지만, 반대로 Manager Node 없이 Worker Node로만 구성할 수는 없습니다. 기본적으로, 모든 Manager Node들은 Worker의 역할도 같이 수행합니다. Single Manager Node로 구성된 Cluster에서, `docker service create`와 같은 명령을 실행하면, 스케쥴러는 모든 Task를 Local Engine에 배치시킵니다.

스케쥴러가 Multi Node로 구성된 Swarm내의 Manager Node에 Task를 배치시키는 것을 막으려면, Manager Node의 가용성 Option을 Drain으로 설정하시기 바랍니다. 그러면 스케쥴러는 Drain Mode의 Node에서는 Task를 정상적으로 중지(Gracefully Stop)하고, 활성화된 Node에 Task를 예약합니다. 스케쥴러는 Drain Mode의 가용성을 가진 Node에는 새로운 Task를 할당하지 않습니다.

### Node간 역할 변경

`docker node promete`를 사용하여, Worker Node를 Manager Node로 변경할 수 있습니다. 예를들어, Manager Node를 오프라인 상태로 유지해야 할 필요가 있을 때, 임시로 Worker Node를 Manager Node로 변경하여 사용할 수 있을 것입니다. 반대로, `docker node demote`를 사용하여, Manager Node를 Worker
Node로 강등시킬 수도 있습니다.

## Swarm Service 동작 방식

Swarm Mode에서 Application Image를 배포하기 위해선, Service를 생성해야 합니다. 이런 Service는 큰 Application Context 내의 Microservice의 Image를 의미하기도 합니다. Service의 예로, HTTP Server, Database 또는 분산 환경에서 실행하고자 하는 다양한 유형의 Runtime Program들이 있습니다.

Service를 생성하고자 할 때, 사용할 Container Image와 Container 내에서 실행할 명령을 지정합니다. 또한, 다음과 같은 Option들을 정의하여 사용합니다.

- Swarm 외부에서 접속할 수 있는 Port
- Swarm 내부의 다른 Service와 통신하기 위한 Overlay Network
- CPU 및 Memory
- Rolling Update 정책
- Image의 Replica 개수

### Services, tasks, and containers

Service를 배포할때, 먼저 Swarm Manager는 Service 정의로 생성할 Service의 상태를 받아들입니다. 그런 다음, Swarm Node들에서 Service에 대한 복제 Task이 실행되도록 예약시켜 놓습니다. 이 때, Swarm Node에 위치한 Task는 서로 독립적으로 실행됩니다. 예를 들어, HTTP Listener를 3개의 Instance로 구성하여 Load Balancing 하는 것을 생각해 보십시오.

아래 그림은 3개의 Replica가 있는 HTTP Listener Service를 보여줍니다. Listener의 세개 Instance 각각은 Swarm의 Task입니다.

![](https://docs.docker.com/engine/swarm/images/services-diagram.png)

Container는 격리된 Process입니다. Swarm Mode Model에서 각각의 Task는 정확히 하나의 Container를 호출합니다. Task는 스케쥴러가 Container를 배치하는 "Slot"과 유사하다고 할 수 있습니다. 일단 Container가 기동되면, 스케쥴러는 Task가 실행 중인 상태임을 인식합니다. 만약 Container의 상태 검사를 실패하거나 종료된다면, Task도 종료됩니다.

### Tasks and scheduling

Task는 Swarm내 스케쥴링의 가장 작은 단위입니다. Serivce를 생성하거나 수정하기 위해 Service 상태를 정의하면, Ochestrator는 Task를 스케쥴링하여 정의된 상태로 Service를 실체화 시킵니다. 예를들어, Ochestrator에게 HTTP Listener를 항상 3개의 Instance로 유지하도록 Service를 정의한다고 했을 때, Ochestrator는 3개의 Task를 생성시킵니다.

각각의 Task는 스케쥴러가 Container를 생성하여 채우는 "Slot"이라고 할 수 있으며, Container는 Task가 인스턴스화된 것입니다. 만약, HTTP Listener Task가 상태 확인에 실패하거나 충돌이 발생했을 경우,
Ochestrator는 새 Container를 생성하는 새로운 Replica Task를 생성합니다. Task는 단방향 메커니즘입니다. 할당, 준비, 실행의 순서로 단방향으로 진행됩니다. 만약 Task가 실행에 실패하면, Ochestrator는 해당 Task와 Container를 제거한 다음, 이를 대체할 새로운 Task를 생성합니다.

Docker Swarm Mode의 기초를 이루고 있는 Locig는 범용 스케쥴러 및 Ochestrator입니다. Service 및 Task 추상화 자체는 그들이 구현하는 Container를 인식하지 못합니다. 가정하여, 가상 Machine Task 또는 비 Container 처리 Task 같은 다른 유형의 Task를 구현할 수도 있습니다. 스케쥴러와 Ochestrator는 Task 유형에 대해 알 수 없습니다. 그러나, Docker의 현재 버전은 Container Task만 지원하고 있습니다. 아래 그림은 Swarm Mode가 Service생성 요청을 받아들이고, Task를 Worker Node에 스케쥴링하는 방법을 나타내고 있습니다.

![](https://docs.docker.com/engine/swarm/images/service-lifecycle.png)

### Pending services

Swarm내의 어떠한 Node들도 Task들을 실행할 수 없도록 Service를 설정할 수도 있습니다. 이 경우에, Service는 Pending 상태로 유지됩니다.

다음은 Service가 Pending 상태인 경우 인 예 입니다.

- 만약 모든 Node들이 일시 중지되거나 Drain상태일 때 Service를 생성하면, Node가 사용가능해질 때까지 Pending 상태로 유지됩니다.(실제로, 첫 번째 Node를 사용할 수 있게 되면 모든 Task가 수행되므로, Production 환경에서는 수행하기에 좋지 않습니다.)
- Service에 필요한 Memory를 특정 값을 설정할 수 있습니다. 만약 Swarm의 Node에 필요한 Memory가 없는 경우, 해당 Task를 시랭할 수 있는 Node를 사용할 수 있을 때까지 Service가 Pending 상태로 유지됩니다. 만약 500GB와 같이 매우 큰 값을 지정한다면, 해당 조건을 만족시킬 수 있는 Node가 나타날 때까지 Task는 영원히 Pending 상태가 유지됩니다.
- Service에 특정 조건을 설정할 수 있으며, 주어진 시간안에 조건을 만족하지 못할 경우 Pending 상태가 유지됩니다.

참고
만얀, Service 배포를 막으려는 경우, Service를 Pending 상태로 하는 것보다 0으로 Scaling하는 것을 권장합니다. 이러한 동작들은 Task의 요구 사항 및 구성이 Swarm의 현재 상태와 밀접하게 연관되어 있지 않음을 보여줍니다. 즉 Swarm의 관리자로서, Swarm의 상태가 안정적으로 유지되도록 지시하고, Swarm Manager는 Swarm의 Node들이 해당 상태를 유지되도록 작동합니다. 직접 Swarm내의 Task까지 세부적으로 관리할 필요가 없습니다.

### Replicated and global services

Service 배포에는 Replicatied와 Global이라는 두 가지 유형이 있습니다. Replicated Service는 사용자가 원하는 수만큼 Task를 동일하게 생성하여 실행되는 Service입니다. 예를 들어, 각각 동일한 컨텐츠를 제공하는 세개의 Replicated HTTP Service를 배포할 수 있습니다.

Global Service는 모든 Node에서 하나의 Task를 실행하는 Service이며, 미리 지정된 Task의 개수가 없습니다. Swarm에 Node를 추가할 때마다, Ochestrator는 Task를 만들고 스케쥴러는 Task를 새로운 Node에 할당합니다. Global Service의 예로는, Monitoring Agent, Virus 백신 Scanner 또는 Swarm의 모든 Node에서 실행해야하는 Container들을 들 수 있습니다.

아래 그림은 황색으로 표현된  3개의 Replicated Service와 회색으로 표현된 Global Service를 보여줍니다.

![](https://docs.docker.com/engine/swarm/images/replicated-vs-global.png)

### Load balancing

Swarm Manager는 Service가 외부의 많은 요청들을 수용할 수 있도록 하기위해, Ingress Load Balancing을 사용합니다. 이를 위해, Swarm Manager가 자동으로 해당 Service의 PblishedPort를
지정하거나, 관리자가 직접 해당 Service를 위한 PublishedPort를 설정할 수 있습니다(만약 Port를 지정하지 않을 경우, Swarm Manager는 30000 ~ 32767 범위내의 Port를 자동으로 할당합니다).

Cloud Load Balancer와 같은 외부 구성요소는 Node가 현재 Service를 위한 Task를 실행 중인지 여부와 관계없이, Cluster의 모든 Node의 PublishedPort를 통해서 Service에 접근할 수 있습니다. 

Swarm Mode는 내부 DNS Component를 갖고 있어, DNS 항목에 있는 각 Service를 자동으로 할당합니다. Swarm Manager는 내부 Load Balancing을 사용하여, Service의 DNS 이름에 따라 Cluster 내의 Service 간에 요청을 분산시킵니다.

Docker Swarm Cluster에는 내장된 Docker Engine에 내부 및 외부 요청에 대한 Load Balancing 기능이 내장되어 있습니다. 내부 Load Balancing은 동일한 Swarm 또는 UCP Cluster내에 위치한 Container간 Load Balancing을 제공합니다. 외부 Load Balancing은 Cluster에 진입하는 Traffic들에 대한 Load
Balancing을 제공합니다.

## Service 배포하기

### Swarm Manager에서 Service 배포

Swam Manager에 접속합니다.

``` bash
$ docker-machine ssh default
                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 17.10.0-ce, build HEAD : 34fe485 - Wed Oct 18 17:16:34 UTC 2017
Docker version 17.10.0-ce, build f4ffd25

docker@default:~$
```

Service를  `docker service create \[OPTIONS\] IMAGE\[COMMAND\] \[ARG...\]`으로 생성합니다.

``` bash
docker@default:~$ docker service create -p 80:80 --name web nginx:latest
1lc92552oo4mxh25l4iur54pj
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>]
verify: Service converged
```

Service가 생성되었는지 `docer service ls \[OPTIONS\]`으로 Service 목록을 확인합니다.

``` bash
docker@default:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
1lc92552oo4m        web                 replicated          1/1                 nginx:latest        *:80->80/tcp
```

생성된 Service가 정상적으로 기동되었는지 `docker service ps \[OPTIONS\] SERVICE \[SERVICE...\]`로 확인합니다.

``` bash
docker@default:~$ docker service ps web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
u3pmniq55z8a        web.1               nginx:latest        default             Running             Running 43 seconds ago                       
```

`docker service inspect \[OPTIONS\] SERVICE \[SERVICE...\]`로 생성된 Serivce의 상세정보를 확인합니다.

`--pretty` Option으로 JSON형식의 Data를 사람이 읽기 쉽운 형태로 변환하여 확인할 수 있습니다.

``` bash
docker@default:~$ docker service inspect web
[
    {
        "ID": "1lc92552oo4mxh25l4iur54pj",
        "Version": {
            "Index": 68
        },
        "CreatedAt": "2017-11-07T01:19:38.721156306Z",
        "UpdatedAt": "2017-11-07T01:19:38.721966563Z",
        "Spec": {
            "Name": "web",
            "Labels": {},
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "nginx:latest@sha256:9fca103a62af6db7f188ac3376c60927db41f88b8d2354bf02d2290a672dc425",
                    "StopGracePeriod": 10000000000,
                    "DNSConfig": {}
                },
                "Resources": {
                    "Limits": {},
                    "Reservations": {}
                },
                "RestartPolicy": {
                    "Condition": "any",
                    "Delay": 5000000000,
                    "MaxAttempts": 0
                },
                "Placement": {
                    "Platforms": [
                        {
                            "Architecture": "amd64",
                            "OS": "linux"
                        },
                        {
                            "OS": "linux"
                        },
                        {
                            "Architecture": "arm64",
                            "OS": "linux"
                        },
                        {
                            "Architecture": "386",
                            "OS": "linux"
                        },
                        {
                            "Architecture": "ppc64le",
                            "OS": "linux"
                        },
                        {
                            "Architecture": "s390x",
                            "OS": "linux"
                        }
                    ]
                },
                "ForceUpdate": 0,
                "Runtime": "container"
            },
            "Mode": {
                "Replicated": {
                    "Replicas": 1
                }
            },
            "UpdateConfig": {
                "Parallelism": 1,
                "FailureAction": "pause",
                "Monitor": 5000000000,
                "MaxFailureRatio": 0,
                "Order": "stop-first"
            },
            "RollbackConfig": {
                "Parallelism": 1,
                "FailureAction": "pause",
                "Monitor": 5000000000,
                "MaxFailureRatio": 0,
                "Order": "stop-first"
            },
            "EndpointSpec": {
                "Mode": "vip",
                "Ports": [
                    {
                        "Protocol": "tcp",
                        "TargetPort": 80,
                        "PublishedPort": 80,
                        "PublishMode": "ingress"
                    }
                ]
            }
        },
        "Endpoint": {
            "Spec": {
                "Mode": "vip",
                "Ports": [
                    {
                        "Protocol": "tcp",
                        "TargetPort": 80,
                        "PublishedPort": 80,
                        "PublishMode": "ingress"
                    }
                ]
            },
            "Ports": [
                {
                    "Protocol": "tcp",
                    "TargetPort": 80,
                    "PublishedPort": 80,
                    "PublishMode": "ingress"
                }
            ],
            "VirtualIPs": [
                {
                    "NetworkID": "3fbffkmpwe24mhpphqjxyudo9",
                    "Addr": "10.255.0.12/16"
                }
            ]
        }
    }
]

docker@default:~$ docker service inspect --pretty web
ID:     1lc92552oo4mxh25l4iur54pj
Name:       web
Service Mode:   Replicated
 Replicas:  1
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:     nginx:latest@sha256:9fca103a62af6db7f188ac3376c60927db41f88b8d2354bf02d2290a672dc425
Resources:
Endpoint Mode:  vip
Ports:
 PublishedPort = 80
  Protocol = tcp
  TargetPort = 80
  PublishMode = ingress
```

Swarm Manager에서 빠져나와 `docker-machine ls \[OPTIONS\] \[ARG...\]`로 각 Node의 `URL`을 확인한 뒤, 접속합니다.

``` bash
$ docker-machine ls
NAME        ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default     -        virtualbox   Running   tcp://192.168.99.100:2376           v17.10.0-ce   
default-2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.10.0-ce   
default-3   -        virtualbox   Running   tcp://192.168.99.102:2376           v17.10.0-ce   
```

- Manager Node
  ![](attachments/35442314/35451421.png)
- Worker Node
  ![](attachments/35442314/35451422.png)
  ![](attachments/35442314/35451423.png)

### Worker Node 에서 Service 배포

구성한 Worker Node들 중 한 Node에 접속합니다.

``` bash
$ docker-machine ssh default-2
                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 17.10.0-ce, build HEAD : 34fe485 - Wed Oct 18 17:16:34 UTC 2017
Docker version 17.10.0-ce, build f4ffd25

docker@default-2:~$
```

Service를  `docker service create \[OPTIONS\] IMAGE \[COMMAND\] \[ARG...\]`로 생성합니다. Service는 생성되지 않으며, Swarm Manager가 아닌 Node에서는 Cluster 상태를 변경할 수 없다는 Error Message를 확인할 수 있습니다.

``` bash
docker@default-2:~$ docker service create -p 80:80 --name web nginx:latest
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.
```

## 배포된 Service 수정하기

### Service Scaling

`docker service ls \[OPTIONS\]`으로 Service 목록을 확인합니다.

현재 생성된 Service의 Replica 개수가 1인 것을 알 수 있습니다.

``` bash
docker@default:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
te07odz7sd0d        web                 replicated          1/1                 nginx:latest        *:80->80/tcp
```

 Service 목록 중 Service 하나를 선택하여 docker service scale
SERVICE=REPLICAS \[SERVICE=REPLICAS...\]로 원하는 개수만큼
Scaling합니다.

``` bash
docker@default:~$ docker service scale web=5
web scaled to 5
overall progress: 5 out of 5 tasks 
1/5: running   [==================================================>]
2/5: running   [==================================================>]
3/5: running   [==================================================>]
4/5: running   [==================================================>]
5/5: running   [==================================================>]
verify: Service converged
```

Service가 Scaling 되었는지 `docker service ls \[OPTIONS\]`으로 확인합니다. REPLICAS열에서 지정한 수만큼 변경되었는지 확인할 수 있습니다.

``` bash
docker@default:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
1lc92552oo4m        web                 replicated          5/5                 nginx:latest        *:80->80/tcp
```

`docker service ps \[OPTIONS\] SERVICE \[SERVICE...\]`로 실제로 몇 개의 Task가 실행중인지 확인합니다. 지정한 개수만큼 Task가 늘어난 것을 확인할 수 있습니다.

``` bash
docker@default:~$ docker service ps web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
u3pmniq55z8a        web.1               nginx:latest        default             Running             Running about a minute ago                       
p8i67ito9574        web.2               nginx:latest        default             Running             Running 13 seconds ago                           
lxtyceb71nk9        web.3               nginx:latest        default-2           Running             Running 13 seconds ago                           
yd7ijk5qyqaa        web.4               nginx:latest        default-3           Running             Running 13 seconds ago                           
q84hr00ompib        web.5               nginx:latest        default-3           Running             Running 13 seconds ago                           
```

### Service Rollilng Update

`docker service inspect \[OPTIONS\] SERVICE \[SERVICE...\]`로 생성된 Serivce의 상세정보를 확인합니다. 해당 Service는 Latest Version(1.13.6)의 Nginx Image를 바탕으로 생성된 것을 확인할 수 있습니다.

``` bash
docker@default:~$ docker service inspect --pretty web
ID:     1lc92552oo4mxh25l4iur54pj
Name:       web
Service Mode:   Replicated
 Replicas:  5
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:     nginx:latest@sha256:9fca103a62af6db7f188ac3376c60927db41f88b8d2354bf02d2290a672dc425
Resources:
Endpoint Mode:  vip
Ports:
 PublishedPort = 80
  Protocol = tcp
  TargetPort = 80
  PublishMode = ingress
```

Service의 Image를 docker service update \[OPTIONS\] SERVICE를 사용하여 다른 Version의 Image로 변경합니다.

``` bash
docker@default:~$ docker service update --image nginx:1.12.2 web
web
overall progress: 5 out of 5 tasks
1/5: running   [==================================================>] 
2/5: running   [==================================================>] 
3/5: running   [==================================================>] 
4/5: running   [==================================================>] 
5/5: running   [==================================================>] 
verify: Service converged
```

Service가 Update되었는지, `docker service ps \[OPTIONS\] SERVICE \[SERVICE...\]`로 확인합니다.

기존 Task의 상태는 Shutdown으로 변경되었으며, 각각의 REPLICA마다 새로운
Task들이 실행중임을 확인할 수 있습니다.

``` bash
docker@default:~$ docker service ps web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                 ERROR               PORTS
p3zonnmr967b        web.1               nginx:1.12.2        default-2           Running             Running about a minute ago                        
u3pmniq55z8a         \_ web.1           nginx:latest        default             Shutdown            Shutdown 2 minutes ago                            
t6tlduvxpjlx        web.2               nginx:1.12.2        default-2           Running             Running 40 seconds ago                            
p8i67ito9574         \_ web.2           nginx:latest        default             Shutdown            Shutdown 41 seconds ago                           
oww8cmtlnc14        web.3               nginx:1.12.2        default-3           Running             Running 44 seconds ago                            
lxtyceb71nk9         \_ web.3           nginx:latest        default-2           Shutdown            Shutdown 44 seconds ago                           
yl4af19swwk7        web.4               nginx:1.12.2        default             Running             Running 48 seconds ago                            
yd7ijk5qyqaa         \_ web.4           nginx:latest        default-3           Shutdown            Shutdown about a minute ago                       
yj7q0bxxm2zh        web.5               nginx:1.12.2        default-3           Running             Running 2 minutes ago                             
q84hr00ompib         \_ web.5           nginx:latest        default-3           Shutdown            Shutdown 3 minutes ago                            
```

추가적으로, `docker service inspect \[OPTIONS\] SERVICE \[SERVICE...\]`로 Service의 Image에 대한 상세정보를 확인합니다. Image가 지정한 Version의 Image로 변경되었음을 확인할 수 있습니다.

``` bash
docker@default:~$ docker service inspect --pretty web

ID:     1lc92552oo4mxh25l4iur54pj
Name:       web
Service Mode:   Replicated
 Replicas:  5
UpdateStatus:
 State:     completed
 Started:   3 minutes ago
 Completed: 28 seconds ago
 Message:   update completed
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:     nginx:1.12.2@sha256:5269659b61c4f19a3528a9c22f9fa8f4003e186d6cb528d21e411578d1e16bdb
Resources:
Endpoint Mode:  vip
Ports:
 PublishedPort = 80
  Protocol = tcp
  TargetPort = 80
  PublishMode = ingress
```

### Service Roll Back

`docker service rollback \[OPTIONS\] SERVICE`로 이전 상태로 복구합니다.

``` bash
docker@default:~$ docker service rollback web
web
rollback: manually requested rollback 
overall progress: rolling back update: 5 out of 5 tasks 
1/5: running   [>                                                  ] 
2/5: running   [>                                                  ] 
3/5: running   [>                                                  ] 
4/5: running   [>                                                  ] 
5/5: running   [>                                                  ] 
verify: Service converged
```

추가적으로, `docker service inspect \[OPTIONS\] SERVICE \[SERVICE...\]`로 Service의 Image에 대한 상세정보를 확인합니다. 1.12.2 Version으로 생성된 Service의 상태가 Shutdown으로 변경되었고, 다시 Latest Version으로 Task가 생성되어 Running상태임을 확인할 수 있습니다.

``` bash
docker@default:~$ docker service ps web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
ksilhkwx7lkj        web.1               nginx:latest        default-2           Running             Running 21 seconds ago                        
p3zonnmr967b         \_ web.1           nginx:1.12.2        default-2           Shutdown            Shutdown 22 seconds ago                       
u3pmniq55z8a         \_ web.1           nginx:latest        default             Shutdown            Shutdown 3 hours ago                          
9ctk9l8n1lmp        web.2               nginx:latest        default-2           Running             Running 18 seconds ago                        
t6tlduvxpjlx         \_ web.2           nginx:1.12.2        default-2           Shutdown            Shutdown 18 seconds ago                       
p8i67ito9574         \_ web.2           nginx:latest        default             Shutdown            Shutdown 3 hours ago                          
cv5z21ep2u6p        web.3               nginx:latest        default-3           Running             Running 25 seconds ago                        
oww8cmtlnc14         \_ web.3           nginx:1.12.2        default-3           Shutdown            Shutdown 26 seconds ago                       
lxtyceb71nk9         \_ web.3           nginx:latest        default-2           Shutdown            Shutdown 3 hours ago                          
xetjjjb5ed5t        web.4               nginx:latest        default             Running             Running 10 seconds ago                        
yl4af19swwk7         \_ web.4           nginx:1.12.2        default             Shutdown            Shutdown 10 seconds ago                       
yd7ijk5qyqaa         \_ web.4           nginx:latest        default-3           Shutdown            Shutdown 3 hours ago                          
ftcfg2xwuj8u        web.5               nginx:latest        default             Running             Running 14 seconds ago                        
yj7q0bxxm2zh         \_ web.5           nginx:1.12.2        default-3           Shutdown            Shutdown 14 seconds ago                       
q84hr00ompib         \_ web.5           nginx:latest        default-3           Shutdown            Shutdown 3 hours ago   
```

추가적으로, `docker service inspect \[OPTIONS\] SERVICE \[SERVICE...\]`로 Service의 Image에 대한 상세정보를 확인합니다. Image가 초기 지정한 Version의 Image로 변경되었음을 확인할 수 있습니다.

``` bash
docker@default:~$ docker service inspect --pretty web

ID:     1lc92552oo4mxh25l4iur54pj
Name:       web
Service Mode:   Replicated
 Replicas:  5
UpdateStatus:
 State:     rollback_completed
 Started:   4 minutes ago
 Message:   rollback completed
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:     nginx:latest@sha256:9fca103a62af6db7f188ac3376c60927db41f88b8d2354bf02d2290a672dc425
Resources:
Endpoint Mode:  vip
Ports:
 PublishedPort = 80
  Protocol = tcp
  TargetPort = 80
  PublishMode = ingress 
```

### Service 삭제하기

`docker service ls \[OPTIONS\]`와 `docker service ps \[OPTIONS\] SERVICE \[SERVICE...\]`로 현재 실행중인 Service 목록을 확인합니다.

``` bash
docker@default:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
1lc92552oo4m        web                 replicated          5/5                 nginx:latest        *:80->80/tcp

docker@default:~$ docker service ps web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
ksilhkwx7lkj        web.1               nginx:latest        default-2           Running             Running 21 seconds ago                        
p3zonnmr967b         \_ web.1           nginx:1.12.2        default-2           Shutdown            Shutdown 22 seconds ago                       
u3pmniq55z8a         \_ web.1           nginx:latest        default             Shutdown            Shutdown 3 hours ago                          
9ctk9l8n1lmp        web.2               nginx:latest        default-2           Running             Running 18 seconds ago                        
t6tlduvxpjlx         \_ web.2           nginx:1.12.2        default-2           Shutdown            Shutdown 18 seconds ago                       
p8i67ito9574         \_ web.2           nginx:latest        default             Shutdown            Shutdown 3 hours ago                          
cv5z21ep2u6p        web.3               nginx:latest        default-3           Running             Running 25 seconds ago                        
oww8cmtlnc14         \_ web.3           nginx:1.12.2        default-3           Shutdown            Shutdown 26 seconds ago                       
lxtyceb71nk9         \_ web.3           nginx:latest        default-2           Shutdown            Shutdown 3 hours ago                          
xetjjjb5ed5t        web.4               nginx:latest        default             Running             Running 10 seconds ago                        
yl4af19swwk7         \_ web.4           nginx:1.12.2        default             Shutdown            Shutdown 10 seconds ago                       
yd7ijk5qyqaa         \_ web.4           nginx:latest        default-3           Shutdown            Shutdown 3 hours ago                          
ftcfg2xwuj8u        web.5               nginx:latest        default             Running             Running 14 seconds ago                        
yj7q0bxxm2zh         \_ web.5           nginx:1.12.2        default-3           Shutdown            Shutdown 14 seconds ago                       
q84hr00ompib         \_ web.5           nginx:latest        default-3           Shutdown            Shutdown 3 hours ago  
```

`docker service rm SERVICE \[SERVICE...\]`으로 Service를 삭제 합니다.

``` bash
docker@default:~$ docker service rm web
web
```

Service가 정상적으로 삭제되었는지 `docker service ls \[OPTIONS\]`와 `docker service ps \[OPTIONS\]SERVICE \[SERVICE...\]`로 확인합니다.

``` bash
docker@default:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS

docker@default:~$ docker service ps web
no such service: web
```
