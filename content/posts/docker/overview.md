---
date: "2018-03-08T18:47:30+09:00"
title: "[Docker 1/1] Overview"
authors: ["1000jaeh"]
series: ["docker"]
categories:
  - posts
tags:
  - docker
  - container as a service
  - container
cover:
  image: "../images/docker.png"
  caption: ""
description: ""
draft: true
---

## Docker란

Docker란 리눅스의 응용프로그램들을 소프트웨어 Container 안에 배치시키는 일을 자동화하는 오픈 소스 프로젝트로서, Docker 공식 문서에 따르면 **Containers as a Service(CaaS) Platform**으로 정의하고 있습니다.

Docker는 홈페이지에 Docker의 기능을 아래와 같이 명시 하고 있습니다.

> Docker Container는 일종의 소프트웨어를 소프트웨어의 실행에 필요한 모든 것을 포함하는 완전한 파일 시스템 안에 감싼다.
> 여기에는 코드, 런타임, 시스템 도구, 시스템 라이브러리 등 서버에 설치되는 무엇이든 아우른다. 이는 실행 중인 환경에 관계 없이 언제나 동일하게 실행될 것을 보증한다.

Docker는 리눅스에서 운영체제 수준 가상화의 추상화 및 자동화 계층을 추가적으로 제공합니다. 또한, 리눅스 커널, 통합 파일 시스템의 리소스 격리 기능을 사용하며, 이를 통해 독립적인 "Container"가 하나의 리눅스 인스턴스 안에서 실행할 수 있게 함으로써 가상 머신에 대한 유지보수 부담을 덜어줍니다.

## Why Docker

### Application의 구조적인 변화

IT의 진화에 따라, 기존 Application들은 Monolithic Architecture에서 Microservice Architecture로 변화하였습니다. 절대로 변하지 않을 듯한 Infrastructure를 기반으로 OS/Runtime/Middleware는 잘 정돈되어져 있었고, 그 위에 수 많은 기능들을 제공하는 대형 Application이 Running하고 있었습니다.
해당 Application에서 장애란 있을 수 없는 일로 간주되고 있었고, 이는 곧 Application의 유연성을 해치게 되었습니다.

결국, 이런 구조는 Mobile 기기를 필두로 한 변화에 있어서 독이 되었고, Application들의 전체적인 구조는 모두 바뀌게 되었습니다. 대형 Application들은 다양한 기기와 환경들에 맞춰 빠르게 지원하기 위해 잘게 쪼개졌으며, 개발자들은 최상의 성능을 위해서 사용 가능한 Service들을 개별적으로 조립하여 제공하였습니다.

또한, 조립된 Service들에 맞춰 Infrastructure도 Public / Private /Virtualized된 다양한 환경을 구성하여 사용하게 되었습니다.

### Challenges

이러한 변화 속에서, 새로운 문제점이 나타나기 시작했습니다.

- 변화 1: 최상의 성능을 위해서, 사용 가능한 Service들을 개별적으로 조립
  - 다양한 방식으로 조립된 서비스가 일관되게 상호 작용할 수 있도록 보장하는 방법은 무엇인가?
  - 어떻게 하면, 각 Service별 "Dependency Hell"을 피할 수 있는가?
- 변화 2: Public / Private / Virtualized된 다양한 환경을 구성하여 사용
  - 어떻게 Application을 다양한 환경으로 신속하게 Migration 및 확장할 수 있는가?
  - 각 환경에 배포된 Application들의 호환성이 보장 되는가?
- 변화 3: Service와 Infrastructure
  - n개의 Service가 n개의 Infrastructure에서 잘 구동될 수 있도록하는 수많은 설정들을 어떻게 피할 수 있는가?

### Solution: Container

많은 변화에 따른 Challenges의 해결책으로 나타난 것이 **Container**입니다.

Container는 Library, 시스템 도구, Code, Runtime 등 Application에 필요한 모든 것이 포함되어 있는 표준화된 유닛으로, 환경에 구애받지 않고 Application을 신속하게 배포 및 확장 할 수 있는 환경을 제공합니다.

**따라서, Docker는 Container를 관리할 수 있는 Service를 제공해줌으로써, Application을 신속하게 구축, 테스트 및 배포할 수 있는 Platform이라고 할
수 있습니다.**

## Docker의 이점

**동일한 Application을 다양한 환경으로 빠르게 배포할 수 있습니다.**

Docker는 Application 및 Service를 Container를 사용하여 표준화된 환경에서 구동할 수 있도록 함으로써, 개발 주기를 단축시킵니다. 이러한 Container의 이점은 CI/CD Workflow에서 극대화됩니다.

**환경에 따라, 배포와 확장이 자유롭습니다.**

Docker는 개발자의 Laptop, Datacenter의 VM, Cloud 환경 또는 여러 다양한 환경에 쉽게 이식하여 사용할 수 있습니다.

Docker의 이런 특성으로 인해, 비즈니스 요구 사항에 맞춰 Application과 Service를 부하에 따라 동적으로 관리할 수 있으며, 거의 실시간으로 축소 또는 확장할 수 있습니다.

타 Platform에 비해 같은 Hardware에서 더 많은 작업을 수행할 수 있습니다.

Docker는 가볍고 빠르게 동작하기 때문에, Hypervisor 기반의 Virtual Machine 보다 실용적이고 비용 효율적입니다.

따라서, 적은 Resources로 많은 작업을 수행해야하는 중소규모의 배포 환경 및 고밀도 밀집 환경에 이상적입니다.

개발 및 운영 관점에 따른 Docker의 이점은 다음과 같습니다.

| View of Developers: 한번의 Build로, 어디서든 Run | View of Devops: 한번의 Configuration으로, 어디서든 Run |
| --- | --- |
| 다양환 환경으로의 쉬운 이식성 및 이기종간의 호환성 문제 제거 | 다양한 환경에 대한 불일치점 제거 |
| Application 재 배포시, 의존성 및 Package 누락 등의 문제 제거 | 코드 품질 향상 |
| 서로 다른 Library와 의존성을 가진 Application들을 다양한 Version으로 동시 실행 가능 | Application 전체 생명주기에 대한 효율성, 일관성 및 반복성 향상 |
| Testing, Integration, Packaging 등 Script로 실행할 수 있는 모든 것들을 자동화 가능 | CI/CD의 신뢰 및 속도 향상 |
| Virtual Machine의 Overhead 없음 | Virtual Machine의 일반적인 문제(성능, 비용, 배포 및 이식성) 해결 |

## Docker 개발 환경 구성

Docker는 로컬 개발 환경 구성을 위한 Community Edition(CE) 과 실 운영 환경을 위한 Enterprise Edition(EE)의 두 가지 버전으로 제공되고 있습니다.

**Docker Community Edition(CE)**은 각 운영체제 환경에 맞는 설치파일이 제공되고 있습니다(Cloud 환경을 위한 Docker CE도 제공하고 있습니다).

- [Docker for Mac](https://docs.docker.com/docker-for-mac/install/#download-docker-for-mac)
- [Docker for Windows](https://docs.docker.com/docker-for-windows/)

{{% notice note %}}
Docker의 Version 및 지원 Platform에 대한 자세한 정보는 **[Install Docker](https://docs.docker.com/engine/installation/)** 문서에서 확인하실 수 있습니다.
{{% /notice %}}

Docker 개발 환경 구성 방법은 다음과 같습니다.

1. 로컬 환경의 운영체제에 맞는 Docker 설치파일을 다운받아 설치한 뒤, 실행합니다.
1. `docker version`으로 설치된 Docker의 Version을 확인합니다.
  ``` sh
    $ docker version
    Client:
     Version:      17.09.0-ce
     API version:  1.32
     Go version:   go1.8.3
     Git commit:   afdb6d4
     Built:        Tue Sep 26 22:40:09 2017
     OS/Arch:      darwin/amd64

    Server:
     Version:      17.09.0-ce
     API version:  1.32 (minimum version 1.12)
     Go version:   go1.8.3
     Git commit:   afdb6d4
     Built:        Tue Sep 26 22:45:38 2017
     OS/Arch:      linux/amd64
     Experimental: false

    $ docker-compose version
    docker-compose version 1.16.1, build 6d1ac21
    docker-py version: 2.5.1
    CPython version: 2.7.12
    OpenSSL version: OpenSSL 1.0.2j  26 Sep 2016

    $ docker-machine version
    docker-machine version 0.12.2, build 9371605
  ```
1. 구성된 Docker 환경에 Server(Docker Daemon)과 Client외에도 docker-compose(설명에 대한 링크 연결 필요)와 docker-machine Tool(설명에 대한 링크 연결 필요)도 같이 설치된 것을 확인하실 수 있습니다.

1. docker run 명령어로 hello-world Image를 Pull 받아서 Conatiner 생성하여 실행시킵니다.

  ``` sh
    $ docker run hello-world
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    5b0f327be733: Pull complete
    Digest: sha256:07d5f7800dfe37b8c2196c7b1c524c33808ce2e0f74e7aa00e603295ca9a0972
    Status: Downloaded newer image for hello-world:latest

    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the \"hello-world\" image from the Docker Hub.
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
     https://cloud.docker.com/

    For more examples and ideas, visit:
     https://docs.docker.com/engine/userguide/
  ```

1. 위와 같이 정상적으로 메시지가 나타난다면, Docker 개발 환경 구성이 완료된 것입니다.
