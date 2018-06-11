---
date: "2018-03-09T08:27:01+09:00"
title: "[Docker 기본(6/8)] Docker의 Container Ochestartion: Swarm"
authors: ["1000jaeh"]
series: ["docker"]
categories:
  - posts
tags:
  - docker
  - swarm
  - cluster
  - container ochestration
cover:
  image: "../images/swarm.png"
draft: false
---

지금까지는 단일 Docker Machine에서 Container를 실행시켜 서비스를 제공했습니다. 그렇다면, 현 상태로 실제 서비스를 운영할 수 있을까요? 한참 부족합니다! 장애없이 원활한 서비스를 제공하기 위해서는, 내결함성, 고가용성 등등 많은 사항들을 고려하여야 합니다. 결국은 단일 Machine으로는 사용할 수 없고 Cluster 형태로 구성되어야하며, 그 위에서 서비스들은 여러개의 Instance로 부하가 분산되어야 하며, 장애가 발생하더라도 스스로 복구 할 수 있는 구조로 실행되어야 합니다. 마지막으로 이것들이 자동화되어 있어야합니다. 수 십개에서 수 백개로 나뉘어진 Service들을 사람이 직접 관리할 수 없기 때문입니다.

## Container Ochestration: Docker Swarm mode

이 때, 필요한 것이 **Container Ochestration**이며, Docker에서 제공하고 있는 Tool이 바로 Swarm이라 할 수 있습니다. Docker Swarm은 Docker Engine에 Ochestration Layer를 구현하여, Cluster 관리 및 Cluster에 배포되는 Container들에 대한 Resource, Network, Security, Fault Torelance등 Ochestartion 기능을 사용할 수 있게 해줍니다(영어 뜻 그대로, Container들을 지휘하는 지휘자 또는 조율자로 기억하면 쉽게 이해가 될 것 같습니다).

Container Ochestration Tool은 Swarm만 있는 것은 아닙니다. Google에서 Kubernetes, Aphace Mesos등이 있으며, 더 확장해서 Ochestation Tool들을 관리해주는 Rancher라는 것도 있습니다. 현재는 Kubernetes가 가장 많이 사용되고 있으며, 가장 탁월한 성능을 자랑한다고 볼 수 있습니다. 반면 Docker Swarm은 제 개인적인 의견으로는 Kubernetes 보다 가볍고, 사용하기 쉽지만... Kubernetes의 성능과 기능에 비해서 대규모 서비스 운영에 대해서는 부족하다고 생각됩니다. 저도 현재는 Kubernetes를 사용하고 있지만, Container Ochestration에 대한 이해와 Local 환경에서의 직접 조작해볼 수 있는 것(Kubernetes보다는 쉽게 Cluster를 구성하여 Container Ochestration을 접해볼 수 있다)은 Swarm이 간편하게 할 수 있기 때문에 정리해보고자 합니다.

{{% notice info %}}
Kubernetes에 대한 내용은 [Tag: Kubernetes](http://tech.cloudz-labs.io/tags/kubernetes/)에서 확인할 수 있습니다.
{{% /notice %}}

## Architecture

먼저, Swarm은 다음과 같은 구조로 설계되어 있습니다.

{{% center %}}
![swarm-architecture](../images/swarm-architecture.png)[출처: Docker Swarm Architecture](https://medium.com/@ITsolutions/containers-102-continuing-the-journey-from-os-virtualization-to-workload-virtualization-54fe5576969d)
{{% /center %}}

Docker Swarm은 Swarm Mode에서 실행되고 있는 Manager Node 및 Worker Node로 동작하는 여러 Docker Host들로 구성됩니다. 여기서, Docker Host는 Manager, Worker 또는 두 역할 모두 수행 할 수 있습니다.

### Nodes

Node는 Swarm에 참여하는 Docker Engine의 Instance들 입니다. 하나의 물리적인 Server 또는 Cloud Server에서 하나 이상의 Node를 실행할 수 있지만, 일반적으로 Production 환경에서의 Swarm은 여러 물리적인 Server와 Cloud System에 걸쳐 분산된 Docker Node들이 배포되는 형태로 사용됩니다.

#### Manager Node

Application을 Swarm에 배포하기 위해서는, 먼저 Manager Node에 Service라는 이름의 정의서를 전달해야 합니다. 그 후, Manager Node는 Worker Node들에 Task라고 불리는 작업 단위를 전달하여, Service를 정상적으로 생성합니다. 기본적으로, Manager Node는 Service를 Worker Node로 실행하지만, Manager만을 위한 Task를 Manager Node에서만 실행하도록 구성할 수도 있습니다. 또한, Manager Node는 안정적인 Swarm 상태를 유지하는데 필요한 Ochestration 및 Cluster 관리 기능들을 수행합니다. 많은 Manager Node들 중에서 Ochestration 작업을 수행할 단일 리더를 선출하기도 합니다.

- Cluster 상태 유지
- Scheduling Service
- Swarm Mode의 HTTP API Endpoints 제공

#### Worker Node

Worker Node는 Manager Node로부터 Task를 수신받고 Container를 실행합니다. 즉 Schedule에 대한 결정을 내리거나 Swarm Mode의 HTTP API를 제공하지 않고, 오로지 Container를 실행시키는 것이 유일한 목적인 Docker Instance입니다. Worker Node에서는 각각의 Agent들이 존재하며, Worker Node에서 실행되고 할당된 Task의 상태에 대해서 Manager Node에 알립니다. 이를 통해, Manager Node가 Worker Node들이 안정적인 상태를 유지할 수 있도록 합니다.

#### Manager와 Worker Node 구성

Docker Cluster 즉, Swarm Mode를 사용하는 가장 큰 이유 중 하나는 바로 Fault-Tolerance 이점을 활용하기 위해서라고 할 수 있을 것입니다. 이를 위해서, Docker의 고가용성 요구사항에 따라 홀수개의 Node로 구성하는 것을 권장하고 있습니다. Swarm을 Multi Manager로 구성할 경우, Manager Node의 중단 시간 없이 Manager Node에서 발생한 오류를 복구할 수 있습니다. 다음을 참고하여 Swarm의 Manager Node를 구성할 수 있습니다.

- 3-Manager Swarm은 최대 1개 Manager Node의 손실에 대해서 허용 가능
- 5-Manager Swarm은 최대 2개 Manager Node의 동신 손실에 대해서 허용 가능
- n 개의 Manager Cluster는 최대 `(n-1)/2`개의 Manager Node의 손실에 대해서 허용 가능
- Docker는 최대 7개의 Manager Node를 구성할 것을 권장합니다.

{{% notice info %}}
더 많은 Manager Node를 추가한다고 해서, Swarm의 확장성 또는 성능이 향상된다는 의미는 아닙니다.
{{% /notice %}}

Manager Node 하나로만 구성된 Swarm을 구성할 수 있지만, 반대로 Manager Node 없이 Worker Node로만 구성할 수는 없습니다. 기본적으로, 모든 Manager Node들은 Worker의 역할도 같이 수행합니다. Single Manager Node로 구성된 Cluster에서, `docker service create`와 같은 명령을 실행하면, 스케쥴러는 모든 Task를 Local Engine에 배치시킵니다.

#### Node간 역할 변경

`docker node promete`를 사용하여, Worker Node를 Manager Node로 변경할 수 있습니다. 예를들어, Manager Node를 오프라인 상태로 유지해야 할 필요가 있을 때, 임시로 Worker Node를 Manager Node로 변경하여 사용할 수 있을 것입니다. 반대로, `docker node demote`를 사용하여, Manager Node를 Worker Node로 강등시킬 수도 있습니다.

## Docker Swarm 구성

이제부터는 Container Ochestration을 위한 Docker Swarm 환경을 구성해보겠습니다. 구성하고자할 Docker는 Manager Node 1대, Worker Node 2대 입니다.

{{% notice info %}}
Swarm Mode를 사용하기 위해서는 Docker 1.12.0 이상의 Version을 설치하시기 바랍니다.
{{% /notice %}}

### Docker Server 구성

Swarm을 구성하기 위해, `docker-machine`으로 Docker Server를 3대 구성합니다.

``` bash
$ docker-machine create --driver virtualbox default
Running pre-create checks...
(default) Default Boot2Docker ISO is out-of-date, downloading the latest release...
(default) Latest release for github.com/boot2docker/boot2docker is v17.10.0-ce
(default) Downloading /Users/1000jaeh/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v17.10.0-ce/boot2docker.iso...
(default) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(default) Copying /Users/1000jaeh/.docker/machine/cache/boot2docker.iso to /Users/1000jaeh/.docker/machine/machines/default/boot2docker.iso...
(default) Creating VirtualBox VM...
(default) Creating SSH key...
(default) Starting the VM...
(default) Check network to re-create if needed...
(default) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env default

$ docker-machine create --driver virtualbox default-2
Running pre-create checks...
Creating machine...
(default-2) Copying /Users/1000jaeh/.docker/machine/cache/boot2docker.iso to /Users/1000jaeh/.docker/machine/machines/default-2/boot2docker.iso...
(default-2) Creating VirtualBox VM...
(default-2) Creating SSH key...
(default-2) Starting the VM...
(default-2) Check network to re-create if needed...
(default-2) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env default-2

$ docker-machine create --driver virtualbox default-3
Running pre-create checks...
Creating machine...
(default-3) Copying /Users/1000jaeh/.docker/machine/cache/boot2docker.iso to /Users/1000jaeh/.docker/machine/machines/default-3/boot2docker.iso...
(default-3) Creating VirtualBox VM...
(default-3) Creating SSH key...
(default-3) Starting the VM...
(default-3) Check network to re-create if needed...
(default-3) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env default-3
```

생성된 Docker Machine의 목록을 확인합니다.

``` bash
$ docker-machine ls
NAME        ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default     -        virtualbox   Running   tcp://192.168.99.100:2376           v17.10.0-ce   
default-2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.10.0-ce   
default-3   -        virtualbox   Running   tcp://192.168.99.102:2376           v17.10.0-ce  
```

### Manager Node 선정

구성된 docker-machine 중 하나에 ssh로 접속하여, `docker swarm init [OPTIONS]` 명령어를 통해서 **Manager Node**로 설정합니다.

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

docker@default:~$ docker swarm init --advertise-addr 192.168.99.100
Swarm initialized: current node (xhegxfmzkjrqgrv9594zhvhsd) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-36uho8g8hn0dunpx85sx700ahvx27gd18b5x4gfqlf0fcz7fmk-569z62jorhk2d12lhdwjkt6t3 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

### Work Node 구성

Manager Node로 선정한 docker-machine 이외의 docker-machine들을 ssh로 접속하여 `docker swarm join [OPTIONS] HOST:PORT` 명령어를 통해 **Worker Node**로 지정합니다. `docker swarm join`은 `docker swarm init` 실행 후, Manager Node에 접속할 수 있는 Token과 Host, Port정보가 담겨서 자동으로 생성됩니다.

- docker-default-2

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

    docker@default-2:~$ docker swarm join --token SWMTKN-1-36uho8g8hn0dunpx85sx700ahvx27gd18b5x4gfqlf0fcz7fmk-569z62jorhk2d12lhdwjkt6t3 192.168.99.100:2377
    This node joined a swarm as a worker.
    ```

- docker-defulat-3

    ``` bash
    $ docker-machine ssh default-3
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

    docker@default-3:~$ docker swarm join --token SWMTKN-1-36uho8g8hn0dunpx85sx700ahvx27gd18b5x4gfqlf0fcz7fmk-569z62jorhk2d12lhdwjkt6t3 192.168.99.100:2377
    This node joined a swarm as a worker.
    ```

### Docker Swarm 구성 확인

Manager Node에 접속하여 `docker node ls [OPTIONS]`로 Docker의 Node 구성 목록을 확인합니다. Manager Node는 별도로 표시되어 있는 것을 확인할 수 있습니다.

``` bash
docker@default:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
xhegxfmzkjrqgrv9594zhvhsd *   default             Ready               Active              Leader
s06l9jsl6ejucnpvkwh9pyr2d     default-2           Ready               Active
9v5f2zdbof0hdbb82kvaeizyo     default-3           Ready               Active
```

지금까지 진행하면, Docker Swarm을 관리하는 Manager Node 1개, 실질적으로 Container들이 배포되어 실행되는 Worker Node 2개가 묶여 Cluster 환경을 구성한 것입니다. 이런 Swarm 환경에서는 처음에 언급했다시피 Container를 직접 실행시키는 것이 아니라, Service라는 단위로 배포를 합니다. 다음에는 Service를 배포하여, Container들이 Docker Cluster 환경에서 어떻게 동작하는지 확인해보겠습니다.
