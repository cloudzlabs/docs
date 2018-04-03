---
date: "2018-03-08T23:23:28+09:00"
title: "Docker's Skeleton"
authors: ["1000jaeh"]
series: ["docker"]
categories:
  - posts
tags:
  - Docker
  - Docker Daemon
  - Docker Registry
  - Docker Object
cover:
  image: "../images/blue-whale-hintze-hall-news.jpg"
  caption: "Natural History Museum: The blue whale skeleton in its new Hintze Hall home"
draft: false
---

Docker는 Container를 구동시킬 수 있는 환경만 구성되어 있다면, Application들을 한 번의 Build로 어디서든 구동시킬 수 있습니다. 하지만 1차원적으로만 살펴보면, Java Application이 JVM 위에서 실행되는 모습과 크게 다르지 않아 보입니다. Java Application도 JVM만 설치되어 있다면, 어디에서든 실행되죠. 마찬가지로, Python Application도 동일합니다. 그렇다면, 이 모든 것들이 결국은 똑같이 생겼고, 단지 실행되는 주체(Container, JAR, py)만 다른 것일까요?

## Client-Server Model

Docker는 **서비스의 요청자(Docker Client)**와 **제공자(Docker Server)**간의 작업이 분리되어 동작하는 Client-Server Model로 되어있으며, Docker Client는 **REST API**를 사용하여 Docker Server를 제어합니다.

- Docker Client: `docker CLI`
- Docker Server: `docker daemon`
- Docker REST API

그리고, Docker Server(`docker daemon`)는 Docker Client로부터 받은 요청에 따라, 다음의 Docker Object들을 생성하고 관리합니다.

- Image
- Container
- Network
- Data Volumes

{{% center %}}
![component](../images/engine-components-flow.png)[출처: Docker Docs - Docker Engine](https://docs.docker.com/engine/docker-overview/#docker-engine)
{{% /center %}}

## 좀 더 자세히 보면

Docker Client는 Docker Daemon과 UNIX Socket 또는 REST API를 사용하여 통신을 하며, Docker Daemon이 Container를 구축, 실행 및 배포할 수 있도록 합니다. Docker Client와 Daemon은 동일한 시스템에서 실행될 수도 있고, Docker Client를 원격으로 Docker Daemon에 연결하여 사용할 수도 있습니다.

{{% center %}}
![architecture](../images/architecture.svg)[출처: Docker Docs - Docker architecture](https://docs.docker.com/engine/docker-overview/#docker-architecture)
{{% /center %}}

### Docker Daemon

Docker Daemon(`dockerd`)은 Client로부터 API 요청을 수신하고 Image, Container, Network 및 Volume과 같은 Docker Object를 관리합니다. Daemon은 Docker 서비스를 관리하기 위해 다른 Daemon과 통신할 수 있습니다.

### Docker Client

Docker Client(`docker`)는 사용자가 Docker Daemon과 통신하는 주요 방법입니다. `docker run`과 같은 명령을 사용하면 Docker Client는 해당 명령을 Docker Daemon으로 전송하여 명령을 수행하게 합니다. `docker` 명령은 Docker API를 사용하며, Docker Client는 둘 이상의 Docker Daemon과 통신 할 수 있습니다.

### Docker Registry

Docker Registry는 Docker Image를 저장합니다. Docker Hub는 누구나 사용할 수있는 Public Registry이며, Docker는 기본적으로 Docker Hub에서 Image를 찾아 Container를 구성하도록 되어 있습니다. 예를들어, `docker pull`을 사용하여 Image를 Registry에서 Local로 내려받을 수 있으며, `docker push`를 통해 Local의 Image를 Registry에 저장할 수도 있습니다. Docker Registry는 개개인이 구성할 수도 있으며, Docker의 Enterprise Edition에서 제공되는 Docker Trusted Registry이 포함된 Docker Datacenter를 사용할 수도 있습니다.

### Docker Object

Docker Object는 Docker Daemon에 의해, 생성 및 관리되는 Image / Container / Network / Volume 등의 개체를 말합니다.

#### Image

Image는 Docker Container를 생성하기 위한 읽기 전용 Template입니다. Image들은 다른 Image 기반 위에 Customizing이 추가되어 만들어질 수 있으며, 이렇게 만들어진 Image는 Docker Registry에 Push한 뒤 사용할 수 있습니다. Image는 Dockerfile에 Image를 만들고 실행하는 데 필요한 단계를 명령어로 정의하여 생성합니다. Dockerfile에 정의된 각각의 명령어들은 Image의 Layer를 생성하며, 이러한 Layer들이 모여 Image를 구성합니다. Dockerfile을 변경하고 Image를 다시 구성하면 변경된 부분만 새로운 Layer로 생성됩니다. 이러한 Image의 Layer구조는 Docker가 타 가상화 방식과 비교할 때, 매우 가볍고 빠르게 기동할 수 있는 요인이 됩니다.

#### Container

Container는 Docker API 사용하여 생성, 시작, 중지, 이동 또는 삭제 할 수 있는 Image의 실행가능한 Instance를 나타냅니다. Container를 하나 이상의 Network에 연결하거나, 저장 장치로 묶을 수 있으며, 현재 상태를 바탕으로 새로운 Image를 생성할 수도 있습니다. 기본적으로 Container는 Host 또는 다른 Container로부터 격리되어 있으며, Network / Storage와 다른 하위 시스템에 대한 접근을 직접 제어 할 수 있습니다. Container는 생성되거나 시작될 때, 구성 옵션 및 Image로부터 정의됩니다. Container가 제거될 때는 영구 저장소에 저장되지 않은 변경 사항은 모두 해당 Container와 같이 사라집니다.

#### Service

Service를 사용하면, 여러 개의 Docker Daemon들로 이루어진 영역 내에서 Container들을 확장(Scaling)시킬 수 있습니다. Service는 '특정 시간동안 사용 가능한 Service의 Replica 개수'와 같은 상태 정보들을 직접 정의하여 사용할 수 있습니다. 기본적으로 Service는 Docker Daemon들 간의 Load Balancing을 제공하고 있기 때문에, 사용자 관점에서는 단일 Application으로 보입니다.

{{% notice info %}}
여러 개의 Docker Daemon들이 모여 Cluster 형태를 이루고 있는 것을 **Docker Swarm**이라고 합니다. Docker Swarm은 Manager와 Worker Node로 구성되어 있으며, Docker Engine은 Docker Version 1.12 이상에서 Swarm Mode를 지원합니다.
{{% /notice %}}

## Container를 기동시켜 보자

지금까지, Docker가 어떻게 생겼는지 그 구성요소들을 하나씩 확인해 봤습니다. 그렇다면, 간단하게 Public Image로붙터 Container를 기동시키면서, 각각의 요소들을 다시 한번 확인해 보겠습니다.

1. 먼저, `docker search [OPTIONS] TERM`로 Docker Store와 Hub로부터 Image를 찾습니다.
    ```bash
    $ docker search ubuntu
    NAME                                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
    ubuntu                                                 Ubuntu is a Debian-based Linux operating s...   6710                [OK]                
    dorowu/ubuntu-desktop-lxde-vnc                         Ubuntu with openssh-server and NoVNC            139                                     [OK]
    rastasheep/ubuntu-sshd                                 Dockerized SSH service, built on top of of...   112                                     [OK]
    ansible/ubuntu14.04-ansible                            Ubuntu 14.04 LTS with ansible                   87                                      [OK]
    ubuntu-upstart                                         Upstart is an event-based replacement for ...   80                  [OK]                
    neurodebian                                            NeuroDebian provides neuroscience research...   40                  [OK]                
    ubuntu-debootstrap                                     debootstrap --variant=minbase --components...   31                  [OK]                
    nuagebec/ubuntu                                        Simple always updated Ubuntu docker images...   22                                      [OK]
    tutum/ubuntu                                           Simple Ubuntu docker images with SSH access     19                                      
    1and1internet/ubuntu-16-nginx-php-phpmyadmin-mysql-5   ubuntu-16-nginx-php-phpmyadmin-mysql-5          16                                      [OK]
    ppc64le/ubuntu                                         Ubuntu is a Debian-based Linux operating s...   11                                      
    aarch64/ubuntu                                         Ubuntu is a Debian-based Linux operating s...   9                                       
    i386/ubuntu                                            Ubuntu is a Debian-based Linux operating s...   8                                       
    darksheer/ubuntu                                       Base Ubuntu Image -- Updated hourly             3                                       [OK]
    codenvy/ubuntu_jdk8                                    Ubuntu, JDK8, Maven 3, git, curl, nmap, mc...   3                                       [OK]
    1and1internet/ubuntu-16-nginx-php-5.6-wordpress-4      ubuntu-16-nginx-php-5.6-wordpress-4             2                                       [OK]
    1and1internet/ubuntu-16-apache-php-7.0                 ubuntu-16-apache-php-7.0                        1                                       [OK]
    pivotaldata/ubuntu-gpdb-dev                            Ubuntu images for GPDB development              0                                       
    pivotaldata/ubuntu                                     A quick freshening-up of the base Ubuntu d...   0                                       
    1and1internet/ubuntu-16-healthcheck                    ubuntu-16-healthcheck                           0                                       [OK]
    thatsamguy/ubuntu-build-image                          Docker webapp build images based on Ubuntu      0                                       
    1and1internet/ubuntu-16-sshd                           ubuntu-16-sshd                                  0                                       [OK]
    ossobv/ubuntu                                          Custom ubuntu image from scratch (based on...   0                                       
    defensative/socat-ubuntu                                                                               0                                       [OK]
    smartentry/ubuntu                                      ubuntu with smartentry                          0                                       [OK]
    ```

    **Docker Hub**에는 Community Version의 Image들이 포함되어 있습니다. 누구나 새로운 Image를 Docker Hub에 Push할 수 있지만, 해당 Image들의 품질이나 호환성을 Docker가 보장하지 않습니다. 대신, **Docker Store**에는 공인된 업체를 통해 승인된 Image들이 포함되어 있습니다. 이러한 Image들은 Vendor들에 의해, 직접 게시되고 유지/관리됩니다. 또한 Docker Certified 로고는 Image에 대한 품질, 출처 및 지원에 대한 보증을 제공합니다. 공식 Image는 `OFFCIAL`로, 그 외의 Community Image는 `AUTOMATED`에 분류됩니다.

2. Container로 구성할 Image를 Docker Store 및 Hub로부터 `docker pull [OPTIONS] NAME[:TAG|@DIGEST]`를 이용하여 Pull 받습니다.

    ```bash
    $ docker pull ubuntu
    Using default tag: latest
    latest: Pulling from library/ubuntu
    ae79f2514705: Pull complete
    5ad56d5fc149: Pull complete
    170e558760e8: Pull complete
    395460e233f5: Pull complete
    6f01dc62e444: Pull complete
    Digest: sha256:506e2d5852de1d7c90d538c5332bd3cc33b9cbd26f6ca653875899c505c82687
    Status: Downloaded newer image for ubuntu:latest
    ```

    Image를 Pull 받을 때, Image에 대한 Version을 Tag로 지정하여 받을 수 있으며, 지정되지 않을 경우 `latest` Version으로 Pull이 진행됩니다. 각 Image별 상세 정보는 [Docker Store](https://store.docker.com/)에서 Image를 검색하여 나오는 상세 페이지에서 확인할 수 있습니다.

3. `docker images`로 Local Repository에 Pull된 Image 목록을 확인합니다.

    ```bash
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ubuntu              latest              747cb2d60bbe        13 days ago         122MB
    hello-world         latest              05a3bd381fc2        5 weeks ago         1.84kB
    ```

4. Local Repository에 Pull된 Image를 `docker run [OPTIONS] IMAGE[COMMAND] [ARG...]`로 Container를 생성하여 실행시킵니다. 개별 `docker command`의 옵션 및 상세 사용법은 `docker run --help`와 같이 사용하여 확인하실 수 있습니다.

    ```bash
    $ docker run -i -t --name ubuntu-local ubuntu /bin/bash
    root@ebb7937d0325:/#
    ```

5. `docker ps`로 실행되고 있는 Ubuntu Container를 확인할 수 있습니다.

    ``` bash
    $ docker ps
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    ebb7937d0325        ubuntu              "/bin/bash"         3 minutes ago       Up 3 minutes                            ubuntu-local
    ```

## Pull & Run. 끝?

Docker에서 인증받은 공식 Ubuntu Image로 Container를 기동시켜 봤습니다. 진행했던 과정을 요약해보자면, Docker Client는 Docker Daemon에게 Ubuntu Image를 검색시킨 후, Docker Hub로부터 Image를 Local Registry에 내려 받았습니다. 그리고 `docker run`을 Docker Daemon에게 다시 요청하여, 내려받은 Image로부터 Container를 생성 및 기동 시켰습니다.

그렇다면, Pull과 Run이 끝일까요? 이전에 우리는 Docker Hub에 배포된 Image로 **"어디서든 Run"**할 수 있다는 것을 확인했습니다. 이제는 다음 단계로 넘어가, 직접 Image를 Build하고, 해당 Image로부터 Container를 Run해보도록 하겠습니다.

하지만, 그 전에! Build된 Image와 Container가 어떻게 생겼는지 궁금해지네요.
