---
date: "2018-03-09T08:29:07+09:00"
title: "[Docker 기본(8/8)] Docker의 Network"
authors: ["1000jaeh"]
series: ["docker"]
categories:
  - posts
tags:
  - docker
  - service discovery
  - docker network
  - overlay
  - ingress
draft: false
---
Docker Swarm은 두 가지 종류의 Traffic을 생성합니다.

1. 제어 및 관리 영역 Traffic: Docker Swarm에 대한 참가 및 탈퇴 요청과 같은 Docker Swarm의 관리 Message가 포함됩니다. 해당 Traffic은 항상 암호화됩니다.
2. Application Data 영역 Traffic: Container 및 외부 Client와의 Traffic이 포함됩니다.

이 중에서 해당 Post에서는, Application Data 영역의 Traffic에 대해서 확인해보고자 합니다.

{{% notice info %}}
Docker의 Networking에 대한 자세한 내용은 [Docker Networking Reference Architecture](https://success.docker.com/article/Docker_Reference_Architecture-_Designing_Scalable,_Portable_Docker_Container_Networks)문서를 참고하시기 바랍니다.
{{% /notice %}}

## Docker의 Network

먼저, Docker는 **Overlay, Ingress, docker\\_gwbridge**의 세 가지 Network이 존재합니다. 각각에 대한 개념과 역할에 대해서 알아보겠습니다.

### Overlay Network

- Overlay Network는 Docker Swarm에 참여하는 Docker Daemon간의 통신을 관리합니다.
- 독립실행형 Container의 Network를 생성하는 방법과 동일한 방식으로 Overlay Network를 생성할 수 있습니다.
- 기존에 생성된 Overlay Network에 Service를 연결시켜 Service간 통신을 활성화할 수 있습니다.
- Overlay Network는 Overlay Network Driver를 사용합니다.

### Ingress Network

- Ingress Network는 Service의 Node들간에 Load Balancing을 하는 Overlay Network입니다.
- Docker Swarm의 모든 Node가 노출된 Port로 요청을 받게되면, 해당 요청을 IPVS라는 모듈로 전달합니다.
- IPVS는 해당 Service에 참여하는 모든 IP 주소를 추적하고 그 중 하나를 선택한 뒤, 요청을 해당 경로로 Routing합니다.
- Ingress Network는 Docker Swarm을 Init하거나 Join할 때 자동으로 생성됩니다.

### docker\_gwbridge

- docker\_gwbridge는 Overlay Network(Ingress Network 포함)를 개별 Docker Daemon의 물리적 Network에 연결하는 Bridge Network입니다.
- 기본적으로, Service가 실행 중인 각각의 Container는 로컬 Docker Daemon Host의 docker\_gwbridge Network에 연결됩니다.
- docker\_gwbridge Network는 Docker Swarm을 Init하거나 Join할 때 자동으로 생성됩니다.

Docker는 사용자가 정의한 Bridge, Overlay 및 MACVLAN Network들에게 Host 내의 모든 Container의 위치를 제공하는 내부 DNS Server를 갖고 있습니다. 각 Docker Container(또는 Docker Swarm의 Task)에 존재하는 DNS Resolver가, DNS 쿼리를 DNS Server 역할을 하는 Docker Engine으로 전달합니다. 그런 다음 Docker Engine은 DNS 쿼리가 요청한 Container가 Network 내에 포함되어있는지 확인합니다. Docker Engine은 key-value 저장소에서 Container, Task 또는 Service 이름과 일치하는 IP주소를 조회하고, 해당 IP 또는 Service Virtual IP(VIP)를 요청자에게 반환합니다. 이렇게 Docker는 내장 DNS를 사용하여, Single Docker Engine에서 실행되는 Container 및 Docker Swarm에서 실행되는 Task에 대한 Service Discovery기능을 제공합니다.

## Service Discovery

Service Discovery는 Network 범위 내에서 동작합니다. 동일한 Network에 있는 Contrainer나 Task만 내장 DNS 기능을 사용할 수 있음을 의미합니다. 따라서, 동일한 Network에 있지 않은 Container는 서로의 주소를 확인할 수 없습니다. 또한, 특정 Network에 Container 또는 Task가 있는 Node만 해당 Network의 DNS 항목들을 저장합니다. 이러한 특징들이 Docker의 보안 및 성능을 향상시켜 줍니다. 만약 대상 Container 또는 Service가 원본 Container와 동일한 Network에 속하지 않는다면, Doker Engine은 구성된 기본 DNS Server로 DNS 쿼리를 전달합니다.

{{% notice info %}}
Docker Swarm에 내장된 Service Discovery의 경우, Token 기반으로 동작하며, 추가 설정없이 즉시 사용가능합니다. 하지만, 모든 Host들이 Docker Hub에 접근 가능해야 하기 때문에, 전체 Service가 한 곳의 이슈로 중지되는 상황(**a single point of failure**)이 발생할 수 있습니다. 따라서, 해당 Service Discovery는 개발 및 Test 환경에서만 사용하고, Production환경에서는 Consul, etcd, ZooKeeper 등의 Key-Value Store를 구성하여 사용하는 것을 권장합니다. Service Discovery에 대한 자세한 사항은 [Docker Swarm Discovery](https://docs.docker.com/swarm/discovery/)문서를 참고하시기 바랍니다.
{{% /notice %}}

## Service를 위한 Network 구성하기

### Overlay Network 생성

Docker Swarm의 Manager Node에 접속합니다.

```bash
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

`docker network create [OPTIONS] NETWORK`로 새로운 Overlay Network를 생성합니다.

``` bash
docker@default:~$ docker network create -d overlay overnet
fvyh0us5pkb50dd0qbk4z8537
```

생성된 Overlay Network를 `docker network ls [OPTIONS]`로 확인합니다.

``` bash
docker@default:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
af3308efb8e3        bridge              bridge              local
01a00e855484        docker_gwbridge     bridge              local
b40e73017c03        host                host                local
3fbffkmpwe24        ingress             overlay             swarm
4a7e4e260bd5        none                null                local
fvyh0us5pkb5        overnet             overlay             swarm
```

`docker network inspect [OPTIONS] NETWORK [NETWORK...]`로 생성된 Overlay Network의 상세정보를 확인합니다.

``` bash
docker@default:~$ docker network inspect overnet
[
    {
        "Name": "overnet",
        "Id": "fvyh0us5pkb50dd0qbk4z8537",
        "Created": "0001-01-01T00:00:00Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": null,
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4098"
        },
        "Labels": null
    }
]
```

Worker Node에 접속합니다.

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

Worker Node에도 `docker network ls [OPTIONS]`로 Swarm Manager에서 생성한 Overlay Network가 존재하는지 확인합니다.

``` bash
docker@default-2:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
ec983de00160        bridge              bridge              local
33c85f089e4a        docker_gwbridge     bridge              local
48f1f71c59f0        host                host                local
3fbffkmpwe24        ingress             overlay             swarm
b66bb1d6602e        none                null                local
```

Worker Node에서는 아직 생성된 Overlay Network를 확인할 수 없습니다. 이는 Docker가 해당 Overlay Network를 필요할 때만, Host에 확장시키기 때문입니다. 따라서, 해당 Network에서 생성된 Service에서 Task될 때, Network 목록에 나타나게 됩니다.

### Service Network 연결

Overlay Network에 연결할 Service를 `docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]`으로 생성합니다. `--network` Option을 설정하여, 생성된 Overlay Network에 해당 Service를 추가할 수 있습니다.

``` bash
docker@default:~$ docker service create --name myservice \
> --network overnet \
> --replicas 3 \
> ubuntu sleep infinity
0pqg1f550559cuoxcjvhenb30
overall progress: 3 out of 3 tasks
1/3: running   [==================================================>]
2/3: running   [==================================================>]
3/3: running   [==================================================>]
verify: Service converged
```

생성된 Service와 Container의 정보를 확인합니다.

``` bash
docker@default:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
0pqg1f550559        myservice           replicated          3/3                 ubuntu:latest

docker@default:~$ docker service ps myservice
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
7u0odxhvmqam        myservice.1         ubuntu:latest       default-2           Running             Running 25 seconds ago
0ylu27keha3b        myservice.2         ubuntu:latest       default             Running             Running 25 seconds ago
pfeozzgdnu97        myservice.3         ubuntu:latest       default-3           Running             Running 25 seconds ago

docker@default:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
fb057acbf8fa        ubuntu:latest       "sleep infinity"    29 seconds ago      Up 29 seconds                           myservice.2.0ylu27keha3b3cieivu4vh4bd
docker@default:~$
```

Worker Node에서 Overlay Network가 생성되었는지 `docker network ls [OPTIONS]`와 `docker network inspect [OPTIONS] NETWORK [NETWORK...]`로 확인합니다.

``` bash
docker@default-2:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
ec983de00160        bridge              bridge              local
33c85f089e4a        docker_gwbridge     bridge              local
48f1f71c59f0        host                host                local
3fbffkmpwe24        ingress             overlay             swarm
b66bb1d6602e        none                null                local
fvyh0us5pkb5        overnet             overlay             swarm

docker@default-2:~$ docker network inspect overnet
[
    {
        "Name": "overnet",
        "Id": "fvyh0us5pkb50dd0qbk4z8537",
        "Created": "2017-11-09T08:14:08.731645101Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "40c48c6eb4e56fab5f6f222c31bfe55915126033dcb416267d78ff197c712583": {
                "Name": "myservice.1.7u0odxhvmqam2s6zcx722bfvx",
                "EndpointID": "a5d198bb2f1f31cb01e0368e8095d222288d0844ef56bcc5789e6f081425093d",
                "MacAddress": "02:42:0a:00:00:06",
                "IPv4Address": "10.0.0.6/24",
                "IPv6Address": ""
            },
            "overnet-sbox": {
                "Name": "overnet-endpoint",
                "EndpointID": "aeea6fa5e54ec37d640ffcbe81d3206550dc972382265de9b5592e9b2bd1a3f2",
                "MacAddress": "02:42:0a:00:00:03",
                "IPv4Address": "10.0.0.3/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4098"
        },
        "Labels": {},
        "Peers": [
            {
                "Name": "99176f06426d",
                "IP": "192.168.99.101"
            },
            {
                "Name": "ac69cf321f95",
                "IP": "192.168.99.102"
            },
            {
                "Name": "8c65bb22c846",
                "IP": "192.168.99.100"
            }
        ]
    }
]
```

### Overlay Network로 통신 확인하기

Worker Node에서 `docker network inspect [OPTIONS] NETWORK [NETWORK...]`로 Overlay Network의 상세정보를 확인하여 실행 중인 Container의 IP정보를 확인합니다.

``` bash
docker@default-2:~$ docker network inspect overnet
[
    {
        "Name": "overnet",
        "Id": "fvyh0us5pkb50dd0qbk4z8537",
        "Created": "2017-11-09T08:14:08.731645101Z",
        "Scope": "swarm",
        "Driver": "overlay",

        ... 생략 ...

        "Containers": {
            "40c48c6eb4e56fab5f6f222c31bfe55915126033dcb416267d78ff197c712583": {
                "Name": "myservice.1.7u0odxhvmqam2s6zcx722bfvx",
                "EndpointID": "a5d198bb2f1f31cb01e0368e8095d222288d0844ef56bcc5789e6f081425093d",
                "MacAddress": "02:42:0a:00:00:06",
                "IPv4Address": "10.0.0.6/24",
                "IPv6Address": ""
            },
            "overnet-sbox": {
                "Name": "overnet-endpoint",
                "EndpointID": "aeea6fa5e54ec37d640ffcbe81d3206550dc972382265de9b5592e9b2bd1a3f2",
                "MacAddress": "02:42:0a:00:00:03",
                "IPv4Address": "10.0.0.3/24",
                "IPv6Address": ""
            }
        },

        ... 생략 ...

    }
]
```

다시 Docker Swarm의 Manager Node에 접속하고 실행 중인 Container ID를 확인한 뒤, Container에 접속합니다.

``` bash
docker@default:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
fb057acbf8fa        ubuntu:latest       "sleep infinity"    29 seconds ago      Up 29 seconds                           myservice.2.0ylu27keha3b3cieivu4vh4bd

docker@default:~$ docker exec -it fb057acbf8fa /bin/bash
root@fb057acbf8fa:/#
```

Networking Test를 위해, iptuils-ping을 다음의 명령어를 실행하여 설치합니다.

``` bash
root@fb057acbf8fa:/# apt-get update && apt-get install iputils-ping

... 생략 ...

Processing triggers for libc-bin (2.23-0ubuntu9) ...
```

앞에서 확인한 Worker Node에서 실행 중인 Container의 IP로 Networking Test를 진행합니다.

``` bash
root@fb057acbf8fa:/# ping 10.0.0.6
PING 10.0.0.6 (10.0.0.6) 56(84) bytes of data.
64 bytes from 10.0.0.6: icmp_seq=1 ttl=64 time=0.785 ms
64 bytes from 10.0.0.6: icmp_seq=2 ttl=64 time=0.811 ms
64 bytes from 10.0.0.6: icmp_seq=3 ttl=64 time=0.688 ms
64 bytes from 10.0.0.6: icmp_seq=4 ttl=64 time=0.558 ms
64 bytes from 10.0.0.6: icmp_seq=5 ttl=64 time=0.519 ms
64 bytes from 10.0.0.6: icmp_seq=6 ttl=64 time=0.510 ms
64 bytes from 10.0.0.6: icmp_seq=7 ttl=64 time=0.535 ms
64 bytes from 10.0.0.6: icmp_seq=8 ttl=64 time=0.534 ms
^C
--- 10.0.0.6 ping statistics ---
8 packets transmitted, 8 received, 0% packet loss, time 6999ms
rtt min/avg/max/mdev = 0.510/0.617/0.811/0.119 ms
```

### Overlay Network로 Service Discovery 확인하기

다시 Manager Node의 Container로 돌아와, 아래의 명령어를 실행합니다.

``` bash
root@fb057acbf8fa:/# cat /etc/resolv.conf
nameserver 127.0.0.11
options ndots:0
```

여기서 관심을 가져야할 값은 nameserver 127.0.0.11입니다. 이 값은 Container 내부에서 실행되고 있는 DNS Resolver들에게 보내지고, 모든 Docker Container는 이 주소가 포함되어있는 내장 DNS Server를 실행합니다. Service이름으로 `ping`명령을 실행하여 Networking Test를 진행합니다.

``` bash
root@fb057acbf8fa:/# ping myservice
PING myservice (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=0.026 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=0.058 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=0.050 ms
64 bytes from 10.0.0.5: icmp_seq=4 ttl=64 time=0.047 ms
64 bytes from 10.0.0.5: icmp_seq=5 ttl=64 time=0.047 ms

^C
--- myservice ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 3997ms
rtt min/avg/max/mdev = 0.026/0.045/0.058/0.012 ms
```

Container내에서 `exit`명령으로 빠져나와 `docker service inspect [OPTIONS] SERVICE [SERVICE...]`로 Service의 상세정보를 확인합니다. ping을 통해 나온 결과에 출력된 10.0.0.5의 주소가 현재 Service의 Virtual
IP인 것을 확인할 수 있습니다.

``` bash
root@fb057acbf8fa:/# exit
exit

docker@default:~$ docker service inspect myservice
[
    {
        "ID": "0pqg1f550559cuoxcjvhenb30",
        "Version": {
            "Index": 289
        },
        "CreatedAt": "2017-11-09T08:14:08.625888401Z",
        "UpdatedAt": "2017-11-09T08:14:08.627636861Z",
        "Spec": {
            "Name": "myservice",
            "Labels": {},
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "ubuntu:latest@sha256:6eb24585b1b2e7402600450d289ea0fd195cfb76893032bbbb3943e041ec8a65",
                    "Args": [
                        "sleep",
                        "infinity"
                    ],
                    "StopGracePeriod": 10000000000,
                    "DNSConfig": {}
                },

            ... 생략 ...

            },
            "EndpointSpec": {
                "Mode": "vip"
            }
        },
        "Endpoint": {
            "Spec": {
                "Mode": "vip"
            },
            "VirtualIPs": [
                {
                    "NetworkID": "fvyh0us5pkb50dd0qbk4z8537",
                    "Addr": "10.0.0.5/24"
                }
            ]
        }
    }
]
```

Worker Node에서도 Service이름으로 ping명령을 실행하게 되면, Service Virtual IP로 실행되어 출력되는 것을 확인할 수 있습니다.

``` bash
root@40c48c6eb4e5:/# ping myservice
PING myservice (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=0.030 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=0.054 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=0.046 ms
64 bytes from 10.0.0.5: icmp_seq=4 ttl=64 time=0.048 ms
64 bytes from 10.0.0.5: icmp_seq=5 ttl=64 time=0.084 ms
64 bytes from 10.0.0.5: icmp_seq=6 ttl=64 time=0.053 ms

--- myservice ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 4998ms
rtt min/avg/max/mdev = 0.030/0.052/0.084/0.017 ms
```

왜 Docker라는 것이 나타났는가란 주제에서 시작하여, Docker Cluster구성과 내부 Network까지, 총 8가지의 주제로 Docker System에 대해서 생각해봤습니다. Image Registry, Container 보안 등 더 살펴봐야할 내용은 많지만, Docker에 대한 주제는 이쯤에서 일단 마무리 짓고, 이제부터는 Container 환경에서 실질적으로 Application들이 어떤 식으로 구축되고 운영되어야 하는지에 대해서 먼저 고민해 봐야겠습니다.
