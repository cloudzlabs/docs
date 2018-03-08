---
date: "2018-03-09T08:27:01+09:00"
title: "Swarm"
authors: ["1000jaeh"]
categories:
  - posts
tags:
  - docker
  - swarm
  - cluster
  - container ochestration
cover:
  image: "../images/docker-official.svg"
draft: true
---

## **Docker Cluster 환경: Swarm Mode**

------------------------------------------------------------------------

Swarm은 Docker Engine에 Ochestration Layer를 구현하여, Cluster 관리
및 Ochestration 기능을 사용할 수 있게 해주는 별도의 Project입니다.

Swarm은 Swarm Mode에서 실행되고 있는 Manager(구성원 및 위임 관리 역할)
및 Worker(Swarm 서비스를 실행)로 동작하는 여러 Docker Host들로
구성됩니다.

여기서, Docker Host는 Manager, Worker 또는 두 역할 모두 수행 할 수
있습니다.

  

Swarm Mode에서는 Service단위로 배포되고 실행됩니다.

사용가능한 복제된 Service 개수, Network 및 Storage Resource 개수,
Service의 외부에 노출된 Port 정보 등 Service에 대한 상태를 먼저
정의하고, 그를 기준으로 Service가 생성됩니다. 그리고 Docker는 정의된
상태를 유지하기 위해 노력합니다.

예를 들어, 특정 Worker Node를 사용할 수 없게 되었을 때, Docker는 해당
Node의 작업을 다른 Node에서 수행할 수 있도록 Scheduling합니다.

이러한 Service는 Swarm Manager가 관리하는 실행 Container인 Task들로
이루어져 있습니다.

  

Swarm Service가 독립형 Container들에 비해 갖는 주요 장점 중 하나는,
수동으로 Service를 다시 시작할 필요없이 연결된 Network 및 Volume등의
Service의 구성을 수정할 수 있습니다.

Docker는 Configuration을 수정할 것이고, 만료된 Configuration의 Service
Task를 중지할 것이며, 원하는 Configuration과 일치하는 새로운 Task를
생성할 것입니다.

Docker가 Swarm Mode에서 실행 중이라고 한다면, Swarm Service들 뿐만아니라
Swarm에 참여하고 있는 어떠한 Docker Host들 위에서 독립 실행형
Container를 계속 실행할 수 있습니다.

Swarm Service와 독립형 Container의 주요 차이점은 Service는 Swarm
Manager로 선정된 Docker Daemon만 관리할 수 있고, 독립형 Container들은
모든 Docker Daemon에서 실행 될 수 있다는 점입니다.

참고

Swarm Mode를 사용하기 위해서는 Docker 1.12.0 이상의 Version을 설치하시기
바랍니다.

## **Swarm의 제공 기능**

------------------------------------------------------------------------

Docker Swarm은 다음의 기능들을 제공하고 있습니다.

<table>
<thead>
<tr class="header">
<th>특징</th>
<th>상세</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><strong>Docker Engine과 통합된 Cluster 관리 환경 제공</strong></p></td>
<td><p>Docker Engine CLI를 그대로 사용하여, Application Service를 배포할 수 있습니다.</p>
<p>Swarm을 생성하거나 관리하기 위한, Ochestration Software를 추가할 필요가 없습니다.</p></td>
</tr>
<tr class="even">
<td><strong>분산된 설계</strong></td>
<td><p>배포 시, 각각의 Node들을 역할 별로 분리하여, 맡은 역할에서 필요한 기능들을 전문적으로 처리할 수 있도록 설계되었습니다.</p>
<p>이렇게 역할별로 나뉜 두 종류의 Node는 Manager와 Worker로 분류되어 배포됩니다.</p></td>
</tr>
<tr class="odd">
<td><strong>선언적 Service Model</strong></td>
<td><p>Docker Engine은 선언적 접근 방식을 사용하여, Application Stack에서 다양한 Service들의 상태를 정의하여 사용할 수 있습니다.</p>
<p>예를 들어, Message Queueing Service와 함께 있는 Web Front Service와 Database Backend로 구성된 Application을 작성하여 배포할 수 있습니다.</p></td>
</tr>
<tr class="even">
<td><strong>Scaling</strong></td>
<td><p>각 Service들에 대해, 실행할 Task 개수를 지정할 수 있습니다.</p>
<p>Scale Up 또는 Down할 때, Swarm Manager는 지정된 상태를 유지하기 위해서, Task들을 추가하거나 제거하여 자동으로 조정할 수 있습니다.</p></td>
</tr>
<tr class="odd">
<td><strong>안정적 상태 유지</strong></td>
<td><p>Swarm Manager는 지속적으로 Cluster 상태를 Monitoring하고, 현 상태와 사용자가 원하는 상태와의 차이점을 조정합니다.</p>
<p>예를 들어, Container의 Replica 10개를 실행하도록 서비스를 설정하고, 그 중 2개의 Replica를 Hosting하고 있는 Worker Node에서 충돌이 발생하면 Swarm Manager는 충돌한 2개의 Replica를 대체할 새로운 Replica를 만듭니다.</p></td>
</tr>
<tr class="even">
<td><strong>다중 호스트 네트워킹</strong></td>
<td><p>Service에 대한 Overlay Network를 지정할 수 있습니다.</p>
<p>Swarm Manager는 Application을 초기화하거나 업데이트 할 때 자동으로 Overlay Network의 Container에 주소를 할당합니다.</p></td>
</tr>
<tr class="odd">
<td><p><strong>Servicec Discovery</strong></p></td>
<td><p>Swarm Manager는 Swarm의 각 Service에 고유한 DNS 이름을 할당하고, 실행 중인 Container들의 부하를 골고루 분산시킵니다.</p>
<p>Swarm에 포함된 DNS 서버를 통해, Swarm에서 실행 중인 모든 Container에 질의할 수 있습니다.</p></td>
</tr>
<tr class="even">
<td><p><strong>Load Balacing</strong></p></td>
<td><p>Service의 Port를 외부 부하 분산 장치에 노출할 수 있습니다.</p>
<p>내부적으로는 Node 간 Service Container들을 분배하는 방법을 지정할 수 있습니다.</p></td>
</tr>
<tr class="odd">
<td><strong>보안</strong></td>
<td><p>Swarm의 각 Node는 자체와 다른 모든 노드 간의 통신을 보호하기 위해 <code>TLS</code> 상호 인증 및 암호화를 시행합니다.</p>
<p>사용자가 지정한 별도의 인증기관에서 서명된 인증서를 옵션을 통해 설정하여 사용할 수 있습니다.</p></td>
</tr>
<tr class="even">
<td><p><strong>Rolling Updates</strong></p></td>
<td><p>Role Out 시간에 Node들에 점진적으로 Service Update를 적용 할 수 있습니다.</p>
<p>Swarm Manager를 사용하면, 여러 Node Set에 배포로 인해 발생하는 Service 지연에 대한 이슈를 제어할 수 있습니다.</p>
<p>또한, 배포된 Service에서 문제가 발생했을 경우, 이전 Version의 Service로 Roll Back도 가능합니다.</p></td>
</tr>
</tbody>
</table>

## **Docker Swarm 환경 구성하기**

------------------------------------------------------------------------

Docker Swarm으로 Cluster 환경을 구성해보겠습니다.

### Docker Server 구성

Swarm을 구성하기 위해, ***docker-machine***으로 Docker Server를 3대
구성합니다.

**docker machine을 virtualbox로 설치**

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

**docker-machine ls**

``` bash
$ docker-machine ls
NAME        ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default     -        virtualbox   Running   tcp://192.168.99.100:2376           v17.10.0-ce   
default-2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.10.0-ce   
default-3   -        virtualbox   Running   tcp://192.168.99.102:2376           v17.10.0-ce  
```

### Manager Node 선정

구성된 docker-machine 중 하나에 ssh로 접속하여, ***docker swarm
init \[OPTIONS\]*** 명령어를 통해서 **Manager Node**로 설정합니다.

-   **default**

    **Swarm Manager**

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

Manager Node로 선정되지 못한 docker-machine들을 ssh로 접속하여 ***docker
swarm join \[OPTIONS\] HOST:PORT*** 명령어를 통해 **Worker Node**로
지정합니다.

***docker swarm join***은 ***docker swarm init** *실행 후, Manager
Node에 접속할 수 있는 Token과 Host, Port정보가 담긴 담겨서 자동으로
생성됩니다.

-   **docker-default-2**

    **Worker Node**

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

-   **docker-defulat-3**

    **Worker Node**

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

Manager Node에 접속하여 ***docker node ls \[OPTIONS\]***로 Docker의 Node
구성 목록을 확인합니다.

Manager Node는 별도로 표시되어 확인할 수 있습니다.

**Swarm Manager**

``` bash
docker@default:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
xhegxfmzkjrqgrv9594zhvhsd *   default             Ready               Active              Leader
s06l9jsl6ejucnpvkwh9pyr2d     default-2           Ready               Active              
9v5f2zdbof0hdbb82kvaeizyo     default-3           Ready               Active                        
```
