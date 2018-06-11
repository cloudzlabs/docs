---
date: "2018-03-09T08:27:58+09:00"
title: "[Docker 기본(7/8)] Docker Swarm의 구조와 Service 배포하기"
authors: ["1000jaeh"]
series: ["docker"]
categories:
  - posts
tags:
  - docker
  - swarm
  - service
  - rolling update
  - container ochestration
draft: false
---

이제는 구성된 Docker Swarm에 Application을 배포해보겠습니다. Docker Swarm에 Application Image를 배포하기 위해선, Service를 생성해야 합니다. Service는 큰 Application Context 내의 Microservice들의 Image를 의미하며, 예로 HTTP Server, Database 또는 분산 환경에서 실행하고자 하는 다양한 유형의 Runtime Program들이 여기에 속한다고 할 수 있습니다. Service를 생성하고자 할 때, 사용할 Container Image와 Container 내에서 실행할 명령을 지정합니다. 또한, 다음과 같은 Option들을 정의하여 사용합니다.

- Docker Swarm 외부에서 접속할 수 있는 Port
- Docker Swarm 내부의 다른 Service와 통신하기 위한 Overlay Network
- CPU 및 Memory 사용에 대한 정책
- Rolling Update 정책
- Image의 Replica 개수

## Service

Service가 독립형 Container들을 직접 실행하는 것에 비해 갖는 주요 장점 중 하나는, 수동으로 Service를 다시 시작할 필요없이 연결된 Network 및 Volume 등의 구성을 수정할 수 있습니다. Docker는 Configuration을 수정하고 만료된 Configuration의 Service Task를 중지할 것이며, 원하는 Configuration과 일치하는 새로운 Task를 생성할 것입니다. Docker가 Swarm Mode에서 실행 중이라고 한다면, Service들 뿐만 아니라, Swarm에 참여하고 있는 모든 Docker Host들 위에서도 독립 실행형 Container들을 실행할 수 있습니다. Service와 독립형 Container의 주요 차이점은 Service는 Manager Node에서만 관리할 수 있고, 독립형 Container들은 모든 Docker Daemon에서 실행될 수 있다는 점입니다.

### Service가 Task를 Task가 Container를

Service를 배포할 때, 먼저 Manager Node는 Service 정의서로 생성할 Service에 대한 상태 및 설정 정보를 받아들입니다. 그런 다음, 타 Node들에서 Service에 대한 복제 Task가 실행되도록 Scheduling합니다. 이 때, Node들에 위치한 Task는 서로 독립적으로 실행됩니다. Container는 격리된 Process이며, 각각의 Task는 정확히 하나의 Container를 호출하게 됩니다. 일단 Container가 기동되면, Scheduler는 Task가 실행 중인 상태임을 인식하고, 만약 Container의 상태 검사를 실패하거나 종료된다면, Task도 종료됩니다.

Task는 Docker Swarm의 가장 작은 Scheduling 단위입니다. Serivce를 생성하거나 수정하기 위해 Service의 상태를 정의하면, Ochestrator는 Task를 Scheduling하여 정의된 상태로 Service를 구체화시킵니다. 예를들어, Ochestrator에게 Microservice를 항상 3개의 Instance로 유지하도록 Service를 정의한다고 했을 때, Ochestrator는 3개의 Task를 생성시킵니다. 각각의 Task는 Scheduler가 Container를 생성하여 채우는 **Slot**이라고 할 수 있으며 Container는 Task가 인스턴스화된 것으로, 할당 - 준비 - 실행의 단방향 메커니즘으로 진행됩니다. 만약 Task가 실행에 실패하면, Ochestrator는 해당 Task와 Container를 제거한 다음, 이를 대체할 새로운 Task를 생성합니다. 아래 그림은 Docker Swarm에서 Service 생성 요청을 받아들이고, Task를 Worker Node에 Scheduling하는 방법을 나타내고 있습니다.

{{% center %}}
![Service Lifecycle](../images/service-lifecycle.png)[출처: Docker Docs - How services work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#tasks-and-scheduling)
{{% /center %}}

### Service 배포 유형

Service 배포에는 Replicatied와 Global이라는 두 가지 유형이 있습니다. Replicated Service는 사용자가 원하는 수만큼 Task를 동일하게 생성하여 실행되는 Service입니다(가장 일반적인 Service유형이며, 일반적인 Application을 배포할 때 사용된다고 생각하면 됩니다). Global Service는 모든 Node에서 하나의 Task를 실행하는 Service이며, 미리 지정된 Task의 개수가 없습니다. Swarm에 Node를 추가할 때마다, Ochestrator는 Task를 만들고 Scheduler는 Task를 새로운 Node에 할당합니다. Global Service는 Monitoring Agent, Virus 백신 Scanner 또는 Swarm의 모든 Node에서 실행해야하는 Container들을 들 수 있습니다. 아래 그림은 황색으로 표현된 3개의 Replicated Service와 회색으로 표현된 Global Service를 보여줍니다.

{{% center %}}
![Replicated VS Global](../images/replicated-vs-global.png)[출처: Docker Docs - How services work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#replicated-and-global-services)
{{% /center %}}

### Service의 부하 분산 처리

Docker Swarm의 Manager Node는 Service가 수 많은 Request를 수용할 수 있도록 하기위해, Ingress Load Balancing이라는 기능을 사용합니다. 이를 위해, Manager Node가 자동으로 해당 Service의 노출 Port(PublishedPort)를
지정하거나, 관리자가 직접 해당 Service를 위한 PublishedPort를 설정할 수도 있습니다(만약 Port를 지정하지 않을 경우, Manager Node는 30000 ~ 32767 범위내의 Port를 자동으로 할당합니다). 이 후 외부에서는 Node가 현재 Task를 실행 중인지 여부와 관계없이, Cluster의 모든 Node의 PublishedPort를 통해서 Service에 접근할 수 있습니다.

또한, Docker Swarm은 내부 DNS Component를 갖고 있어서, DNS 항목에 있는 각 Service를 자동으로 할당할 수 있습니다. Manager Node는 내부 Load Balancing을 사용하여, Service의 DNS 이름에 따라 Cluster 내의 Service 간에 요청을 분산시킵니다. Cluster 내에 위치한 Container간 내부 Load Balancing과 Cluster에 진입하는 Traffic들에 대한 외부 Load Balancing 모두 처리가 가능하며, 이는 Docker Engine에 내장된 Load Balancing 기능에 의해서 수행됩니다.

## Service 배포하기

이제부터는 직접 Service를 Docker Swarm에 배포해 보겠습니다(아래의 예제는 Docker 공식문서에서 .

### Swarm Manager에서 Service 배포

Docker Swarm의 Manager Node에 접속합니다.

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

Service를  `docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]`으로 생성합니다.

``` bash
docker@default:~$ docker service create -p 80:80 --name web nginx:latest
1lc92552oo4mxh25l4iur54pj
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```

Service가 생성되었는지 `docer service ls [OPTIONS]`으로 Service 목록을 확인합니다.

``` bash
docker@default:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
1lc92552oo4m        web                 replicated          1/1                 nginx:latest        *:80->80/tcp
```

생성된 Service가 정상적으로 기동되었는지 `docker service ps [OPTIONS] SERVICE [SERVICE...]`로 확인합니다.

``` bash
docker@default:~$ docker service ps web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
u3pmniq55z8a        web.1               nginx:latest        default             Running             Running 43 seconds ago
```

`docker service inspect [OPTIONS] SERVICE [SERVICE...]`로 생성된 Serivce의 상세정보를 확인합니다. `--pretty` Option으로 JSON형식의 Data를 사람이 읽기 쉽운 형태로 변환하여 확인할 수 있습니다.

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

Docker Swarm의 Manager Node에서 빠져나와 `docker-machine ls [OPTIONS] [ARG...]`로 각 Worker Node의 `URL`을 확인한 뒤, 접속합니다.

``` bash
$ docker-machine ls
NAME        ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default     -        virtualbox   Running   tcp://192.168.99.100:2376           v17.10.0-ce
default-2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.10.0-ce
default-3   -        virtualbox   Running   tcp://192.168.99.102:2376           v17.10.0-ce
```

- Manager Node
  ![manager-node:default](../images/manager-node.png)
- Worker Node
  ![worker-node:default-2](../images/worker-node-1.png)
  ![worker-node:default-3](../images/worker-node-2.png)

### Worker Node에서 Service 배포

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

Service를 `docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]`로 생성합니다. Service는 생성되지 않으며, Swarm Manager가 아닌 Node에서는 Cluster 상태를 변경할 수 없다는 Error Message를 확인할 수 있습니다.

``` bash
docker@default-2:~$ docker service create -p 80:80 --name web nginx:latest
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.
```

## 배포된 Service 수정하기

지금부터는 Docker Swarm에 배포된 Service를 통해, Scaling과 배포된 Application의 Version Update/Rollback 기능을 사용해보겠습니다.

### Scaling

`docker service ls [OPTIONS]`으로 Service 목록을 확인합니다. 현재 생성된 Service의 Replica 개수가 1인 것을 알 수 있습니다.

``` bash
docker@default:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
te07odz7sd0d        web                 replicated          1/1                 nginx:latest        *:80->80/tcp
```

Service 목록 중 Service 하나를 선택하여 `docker service scale SERVICE=REPLICAS [SERVICE=REPLICAS...]`로 원하는 개수만큼 Scaling합니다.

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

Service가 Scaling 되었는지 `docker service ls [OPTIONS]`으로 확인합니다. REPLICAS열에서 지정한 수만큼 변경되었는지 확인할 수 있습니다.

``` bash
docker@default:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
1lc92552oo4m        web                 replicated          5/5                 nginx:latest        *:80->80/tcp
```

`docker service ps [OPTIONS] SERVICE [SERVICE...]`로 실제로 몇 개의 Task가 실행중인지 확인합니다. 지정한 개수만큼 Task가 늘어난 것을 확인할 수 있습니다.

``` bash
docker@default:~$ docker service ps web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
u3pmniq55z8a        web.1               nginx:latest        default             Running             Running about a minute ago
p8i67ito9574        web.2               nginx:latest        default             Running             Running 13 seconds ago
lxtyceb71nk9        web.3               nginx:latest        default-2           Running             Running 13 seconds ago
yd7ijk5qyqaa        web.4               nginx:latest        default-3           Running             Running 13 seconds ago
q84hr00ompib        web.5               nginx:latest        default-3           Running             Running 13 seconds ago
```

### Rollilng Update

`docker service inspect [OPTIONS] SERVICE [SERVICE...]`로 생성된 Serivce의 상세정보를 확인합니다. 해당 Service는 Latest Version(1.13.6)의 Nginx Image를 바탕으로 생성된 것을 확인할 수 있습니다.

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

Service의 Image를 `docker service update [OPTIONS] SERVICE`를 사용하여 다른 Version의 Image로 변경합니다.

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

Service가 Update되었는지, `docker service ps [OPTIONS] SERVICE [SERVICE...]`로 확인합니다. 기존 Task의 상태는 Shutdown으로 변경되었으며, 각각의 REPLICA마다 새로운 Task들이 실행중임을 확인할 수 있습니다.

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

추가적으로, `docker service inspect [OPTIONS] SERVICE [SERVICE...]`로 Service의 Image에 대한 상세정보를 확인합니다. Image가 지정한 Version의 Image로 변경되었음을 확인할 수 있습니다.

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

### Roll Back

`docker service rollback [OPTIONS] SERVICE`로 이전 상태의 버전으로 복구합니다.

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

추가적으로, `docker service inspect [OPTIONS] SERVICE [SERVICE...]`로 Service의 Image에 대한 상세정보를 확인합니다. 1.12.2 Version으로 생성된 Service의 상태가 Shutdown으로 변경되었고, 다시 Latest Version으로 Task가 생성되어 Running 상태임을 확인할 수 있습니다.

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

추가적으로, `docker service inspect [OPTIONS] SERVICE [SERVICE...]`로 Service의 Image에 대한 상세정보를 확인합니다. Image가 초기 지정한 Version의 Image로 변경되었음을 확인할 수 있습니다.

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

`docker service ls [OPTIONS]`와 `docker service ps [OPTIONS] SERVICE [SERVICE...]`로 현재 실행중인 Service 목록을 확인합니다.

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

`docker service rm SERVICE [SERVICE...]`으로 Service를 삭제 합니다.

``` bash
docker@default:~$ docker service rm web
web
```

Service가 정상적으로 삭제되었는지 `docker service ls [OPTIONS]`와 `docker service ps [OPTIONS] SERVICE [SERVICE...]`로 확인합니다.

``` bash
docker@default:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS

docker@default:~$ docker service ps web
no such service: web
```

Docker Swarm의 Service는 배포된 Application들의 고가용성을 보장하고 있습니다. Rolling Update와 Roll Back 기능을 제공함으로써, 배포 시 발생할 수 있는 서비스 중단 상황에 대한 대비도 할 수 있도록 되어 있으며, 정의된 Replica의 개수만큼 정상동작하는 Container들을 유지시켜주기도 합니다. 또한, Traffic에 따라 Load Balancing이 적절히 이루어지도록 관리됩니다. 이 모든 것들이 유기적으로 동작할 수 있었던 점은, Cluster Node간 또는 배포된 Service나 Container간 연결된 Docker의 Network 구조 때문이라 생각할 수 있습니다. 따라서, 다음 챕터에서는 Docker의 Network에 대해서 알아보도록 하겠습니다.
