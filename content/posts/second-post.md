---
date: "2018-02-06T18:47:29+09:00"
title: "Docker"
authors: ["1000jaeh"]
categories:
  - 블로그
tags:
  - Docker
draft: false
---
# Docker
- [Docker](#docker)
  - [Overview](#overview)
    - [Docker Platform](#docker-platform)
    - [Docker 구성](#docker-%EA%B5%AC%EC%84%B1)
    - [Why Docker](#why-docker)
    - [What can I use Docker for?](#what-can-i-use-docker-for)
    - [Docker는 무엇을 사용할 수 있습니까?](#docker%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%A0-%EC%88%98-%EC%9E%88%EC%8A%B5%EB%8B%88%EA%B9%8C)
    - [Solution: Container](#solution-container)
  - [Architecture](#architecture)
    - [Docker Daemon](#docker-daemon)
    - [Docker Client](#docker-client)
    - [Docker Registry](#docker-registry)
    - [Docker Object](#docker-object)
      - [Image](#image)
      - [container](#container)
      - [service](#service)
  - [Container](#container)
    - [Container에 대하여](#container%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC)
    - [Container의 특징](#container%EC%9D%98-%ED%8A%B9%EC%A7%95)
      - [Lightweight](#lightweight)
      - [Standard](#standard)
      - [Secure](#secure)
    - [Container VS VM](#container-vs-vm)
      - [Container](#container)
      - [Virtual Machines(흔히 아는 가상화방식: VMWare, VirtualBox)](#virtual-machines%ED%9D%94%ED%9E%88-%EC%95%84%EB%8A%94-%EA%B0%80%EC%83%81%ED%99%94%EB%B0%A9%EC%8B%9D-vmware-virtualbox)
      - [요약](#%EC%9A%94%EC%95%BD)
      - [Container and Virtual machines Together](#container-and-virtual-machines-together)
  - [Flow](#flow)
    - [Basic](#basic)
    - [Change or Update](#change-or-update)
      - [Image와 Container의 Layer](#image%EC%99%80-container%EC%9D%98-layer)
  - [Get Started](#get-started)

## Overview

**"Docker is the leading Containers as a Service(CaaS) platform"**

Docker는 애플리케이션을 신속하게 구축, 테스트 및 배포할 수 있는 소프트웨어 플랫폼입니다. Docker는 소프트웨어를 Container라는 표준화된 유닛으로 패키징하며, 이 Container에는 라이브러리, 시스템 도구, 코드, 런타임 등 소프트웨어를 실행하는 데 필요한 모든 것이 포함되어 있습니다. Docker를 사용하면 환경에 구애받지 않고 애플리케이션을 신속하게 배포 및 확장할 수 있으며 코드가 문제없이 실행될 것임을 확신할 수 있습니다.



### Docker Platform

Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure in the same ways you manage your applications. By taking advantage of Docker’s methodologies for shipping, testing, and deploying code quickly, you can significantly reduce the delay between writing code and running it in production.



Docker는 응용 프로그램을 개발, 운송 및 실행하기위한 개방형 플랫폼입니다. Docker를 사용하면 인프라에서 응용 프로그램을 분리하여 소프트웨어를 신속하게 제공 할 수 있습니다. Docker를 사용하면 응용 프로그램을 관리하는 것과 동일한 방법으로 인프라를 관리 할 수 있습니다. 코드를 신속하게 선적, 테스트 및 배포하는 Docker의 방법론을 활용하면 코드 작성과 프로덕션 환경에서의 실행 간의 지연을 크게 줄일 수 있습니다.



Docker provides the ability to package and run an application in a loosely isolated environment called a container. The isolation and security allow you to run many containers simultaneously on a given host. Containers are lightweight because they don’t need the extra load of a hypervisor, but run directly within the host machine’s kernel. This means you can run more containers on a given hardware combination than if you were using virtual machines. You can even run Docker containers within host machines that are actually virtual machines!

Docker provides tooling and a platform to manage the lifecycle of your containers:

- Develop your application and its supporting components using containers.
- The container becomes the unit for distributing and testing your application.
- When you’re ready, deploy your application into your production environment, as a container or an orchestrated service. This works the same whether your production environment is a local data center, a cloud provider, or a hybrid of the two.



Docker는 컨테이너라고하는 느슨한 환경에서 응용 프로그램을 패키지화하고 실행할 수있는 기능을 제공합니다. 격리 및 보안을 통해 주어진 호스트에서 여러 컨테이너를 동시에 실행할 수 있습니다. 컨테이너는 하이퍼 바이저의 추가로드가 필요 없기 때문에 경량이지만 호스트 시스템의 커널 내에서 직접 실행됩니다. 즉, 가상 시스템을 사용하는 경우보다 특정 하드웨어 조합에서 더 많은 컨테이너를 실행할 수 있습니다. Docker 컨테이너는 실제로 가상 시스템 인 호스트 시스템 내에서 실행할 수도 있습니다!

Docker는 컨테이너의 수명주기를 관리하는 툴링 및 플랫폼을 제공합니다.

- 컨테이너를 사용하여 애플리케이션 및 지원 구성 요소를 개발하십시오.
- 컨테이너는 응용 프로그램을 배포하고 테스트하는 단위가됩니다.
- 준비가되면 응용 프로그램을 컨테이너 또는 통합 서비스로 프로덕션 환경에 배포하십시오. 이는 프로덕션 환경이 로컬 데이터 센터, 클라우드 공급자 또는이 둘의 하이브리드에 상관없이 동일하게 작동합니다.



### Docker 구성

*Docker Engine* 은 다음과 같은 주요 구성 요소가있는 클라이언트 - 서버 응용 프로그램입니다.

- 데몬 프로세스 ( `dockerd`명령) 라고하는 장기 실행 프로그램 유형의 서버입니다 .
- 프로그램이 데몬과 대화하고 무엇을해야하는지 지시하는 데 사용할 수있는 인터페이스를 지정하는 REST API.
- 명령 행 인터페이스 (CLI) 클라이언트 ( `docker`명령).

CLI는 Docker REST API를 사용하여 스크립팅 또는 직접 CLI 명령을 통해 Docker 데몬을 제어하거나 상호 작용합니다. 다른 많은 Docker 응용 프로그램은 기본 API 및 CLI를 사용합니다.

데몬은 이미지, 컨테이너, 네트워크 및 볼륨과 같은 Docker *객체를* 만들고 관리합니다 .

![Docker Engine Components Flow](https://docs.docker.com/engine/article-img/engine-components-flow.png)

### Why Docker

그렇다면, 왜 현재 Docker를 많은 사람들이 사용하는 것인가?

**Application의 구조적인 변화**

IT의 진화에 따라, 기존 Application들은 Monolithic Architecture에서 Microservice Architecture로 변화하였습니다.

절대로 변하지 않을 듯한 Infrastructure를 기반으로 OS/Runtime/Middleware는 잘 정돈되어져 있었고, 그 위에 수 많은 기능들을 제공하는 대형 Application이 Running하고 있었습니다. 해당 Application에서 장애란 있을 수 없는 일로 간주되고 있었고, 이는 곧 Application의 유연성을 해치게 되었습니다.

결국, 이런 구조는 모바일 기기를 필두로 한 변화에 있어서 독이 되었고, Application들의 전체적인 구조는 모두 바뀌게 되었습니다.

대형 Application들은 다양한 기기와 환경들에 맞춰 빠르게 지원하기 위해 잘게 쪼개졌으며,개발자들은 **최상의 성능을 위해서, 사용 가능한 서비스들을 개별적으로 조립**하여 제공하였습니다. 또한, 조립된 서비스들에 맞춰 Infrastructure도 **Public/Private/Virtualized된 다양한 환경을 구성**하여 사용하게 되었습니다.

**Challenges**

변화속에서, 새로운 문제점이 나타나기 시작해습니다.

- 최상의 성능을 위해서, 사용 가능한 서비스들을 개별적으로 조립
  - 다양한 방식으로 조립된 서비스가 일관되게 상호 작용할 수 있도록 보장하는 방법
  - 각 서비스별, 의존성 지옥을 피하는 방법
- Public/Private/Virtualized된 다양한 환경을 구성
  - 신속하게 마이그레이션 및 확장하는 방법
  - 호환성 보장
- Service와 Infrastructure
  - n X n의 다른 설정들을 피할 수 있는 방법

이런 환경 속에서, **서비스와 앱이 적절하게 상호 작용할 수 있습니까?** 또한, **원활하고 신속하게 마이그레이션을 할 수 있습니까?**

1960년 이전에 화물 선적도 이와 비슷한 상황이었습니다. 다양한 상품들간의 상호작용, 빠르게 배에서 기차나 트럭에 싣는 방법, 마지막으로 개별 상품들과 선적방법의 수많은 다른 고려사항들이 존재했습니다.



###  What can I use Docker for?

**Fast, consistent delivery of your applications**

Docker streamlines the development lifecycle by allowing developers to work in standardized environments using local containers which provide your applications and services. Containers are great for continuous integration and continuous development (CI/CD) workflows.

Consider the following example scenario:

- Your developers write code locally and share their work with their colleagues using Docker containers.
- They use Docker to push their applications into a test environment and execute automated and manual tests.
- When developers find bugs, they can fix them in the development environment and redeploy them to the test environment for testing and validation.
- When testing is complete, getting the fix to the customer is as simple as pushing the updated image to the production environment.

**Responsive deployment and scaling**

Docker’s container-based platform allows for highly portable workloads. Docker containers can run on a developer’s local laptop, on physical or virtual machines in a data center, on cloud providers, or in a mixture of environments.

Docker’s portability and lightweight nature also make it easy to dynamically manage workloads, scaling up or tearing down applications and services as business needs dictate, in near real time.

**Running more workloads on the same hardware**

Docker is lightweight and fast. It provides a viable, cost-effective alternative to hypervisor-based virtual machines, so you can use more of your compute capacity to achieve your business goals. Docker is perfect for high density environments and for small and medium deployments where you need to do more with fewer resources.



### Docker는 무엇을 사용할 수 있습니까?

**애플리케이션의 빠르고 일관된 전달**

Docker는 개발자가 응용 프로그램 및 서비스를 제공하는 로컬 컨테이너를 사용하여 표준화 된 환경에서 작업 할 수 있도록하여 개발 수명주기를 간소화합니다. 컨테이너는 지속적인 통합 및 지속적인 개발 (CI / CD) 워크 플로우에 유용합니다.

다음 예제 시나리오를 고려하십시오.

- 개발자는 코드를 로컬에서 작성하고 Docker 컨테이너를 사용하여 동료와 작업을 공유합니다.
- [Docker](#docker)
  - [Overview](#overview)
    - [Docker Platform](#docker-platform)
    - [Docker 구성](#docker-%EA%B5%AC%EC%84%B1)
    - [Why Docker](#why-docker)
    - [What can I use Docker for?](#what-can-i-use-docker-for)
    - [Docker는 무엇을 사용할 수 있습니까?](#docker%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%A0-%EC%88%98-%EC%9E%88%EC%8A%B5%EB%8B%88%EA%B9%8C)
    - [Solution: Container](#solution-container)
  - [Architecture](#architecture)
    - [Docker Daemon](#docker-daemon)
    - [Docker Client](#docker-client)
    - [Docker Registry](#docker-registry)
    - [Docker Object](#docker-object)
      - [Image](#image)
      - [container](#container)
      - [service](#service)
  - [Container](#container)
    - [Container에 대하여](#container%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC)
    - [Container의 특징](#container%EC%9D%98-%ED%8A%B9%EC%A7%95)
      - [Lightweight](#lightweight)
      - [Standard](#standard)
      - [Secure](#secure)
    - [Container VS VM](#container-vs-vm)
      - [Container](#container)
      - [Virtual Machines(흔히 아는 가상화방식: VMWare, VirtualBox)](#virtual-machines%ED%9D%94%ED%9E%88-%EC%95%84%EB%8A%94-%EA%B0%80%EC%83%81%ED%99%94%EB%B0%A9%EC%8B%9D-vmware-virtualbox)
      - [요약](#%EC%9A%94%EC%95%BD)
      - [Container and Virtual machines Together](#container-and-virtual-machines-together)
  - [Flow](#flow)
    - [Basic](#basic)
    - [Change or Update](#change-or-update)
      - [Image와 Container의 Layer](#image%EC%99%80-container%EC%9D%98-layer)
  - [Get Started](#get-started)

**응답 성있는 배포 및 확장**

Docker의 컨테이너 기반 플랫폼은 이동성이 뛰어난 워크로드를 허용합니다. Docker 컨테이너는 개발자의 로컬 랩톱, 데이터 센터의 물리적 또는 가상 컴퓨터, 클라우드 공급자 또는 여러 환경에서 실행할 수 있습니다.

Docker의 이식성과 가벼운 특성으로 인해 워크로드를 동적으로 관리하고 비즈니스 요구 사항에 따라 응용 프로그램과 서비스를 거의 실시간으로 축소 또는 축소 할 수 있습니다.

**동일한 하드웨어에서 더 많은 작업 부하 실행**

Docker는 가볍고 빠릅니다. Hypervisor 기반 가상 시스템 대신 실용적이고 비용 효율적인 대안을 제공하므로 더 많은 컴퓨팅 용량을 사용하여 비즈니스 목표를 달성 할 수 있습니다. Docker는 고밀도 환경과 적은 리소스로 더 많은 작업을 수행해야하는 중소 규모 배포 환경에 이상적입니다.



### Solution: Container

해결책으로 나타난 것이 **Container**입니다. 실제적인 모든 상품들이 최종 납품까지 봉인된 형태로 표준 Container에 적재되고, 그 사이에 Container를 적재 및 반출, 쌓아올리고, 장거리 운송을 효율적으로 수행할 수 있으며, 한 운송 수단에서 다른 운송 수단으로 쉽게 이전할 수 있습니다.

이를 참고하여, 만단 것이 코드용 Shipping Container System **Docker**이다.

- 개발자: **한번 빌드해서 어디서든지 실행** --> 어떠한 Payload든지 가볍고, 휴대 가능하며, 자체 완비한 Container로 캡슐화 할 수있게 해주는 엔진
- 운영자: **한번 설정해서 어디서든지 실행** --> 표준 작업을 사용하여 조작하고, 사실상 모든 hadrware 플랫폼에서 일관되게 실행

**Why Container matter: Adventages**

|                                  | Docker                                   |
| -------------------------------- | ---------------------------------------- |
| Content Agnostic                 | 어떠한 Payload들과 그 종속성을 함께 캡슐화 가능           |
| Hardware Agnostic                | 운영체제 프리미티브를 사용하면, 수정없이 가상머신, 베어메탈, 오픈스택, 공용 IaaS등 거의 모든 하드웨어에서 일관되게 실행할 수 있음 |
| Content Isolation and Iteraction | Resource, Network, Content가 독립적이어서, 의존성 지옥을 피할 수 있음 |
| Automation                       | run, start, stop, commit, search 등의 표준 명령어 기반Devops용으로도 완벽: CI, CD, Autoscaling, Hybrid Clouds |
| Highly Efficient                 | 경량, 스타트-업 또는 성능 패널티가 없음, 빠른 이동 및 조작      |

**View of Developers**

한번 빌드. 어디서든 Run

- 앱을 위한 깨끗하고 안전하며 위생적이고 이식 가능한 런타임 환경
- 후속 배포시, 의존성 및 패키지 누락 등의 문제점에 대해서 걱정할 필요 없음
- 독립된 Container에서 각 Application이 실행되기 때문에, 라이브러리와 다른 의존성을 가진 각 Application들을 다양한 버전으로 실행 가능
- Testing, Integration, Packaging 등 스크립트로 실행할 수 있는 모든 것들을 자동화
- 다른 플랫폼들간의 호환성 문제 제거
- 저렴
- 배포시, 패널티가 없음,
- VM의 Overhead 없음, 즉시 시작 가능하며, Image Snapshot으로 재시작 가능

**View of Devops**

한번 설정, 어디서든 Run

- 전체 라이프 사이클의 효율성, 일관성 및 반복성 향상
- 코드 품질 향상
- 각 환경에 따른 불일치점 제거
- Container별 직무 분리
- CI/CD의 속도와 신뢰 향상
- 경량 Container이기 때문에, VM에서 발생하는 일반적인 문제들(성능, 비용, 배포 및 이식성 등등) 해결




## Architecture

Docker는 Client - Server Architecture를 사용합니다. Docker Client는 Docker Daemon과 통신합니다. Docker Daemon은 Container를 구축, 실행 및 배포하는 작업을 수행합니다. Docker Client와 Daemon은 동일한 시스템에서 실행되거나, Docker Client를 원격으로 Docker Daemon에 연결할 수 있습니다. Docker Client와 Daemon은 UNIX Socket 또는 REST API를 사용하여 통신합니다.

![아키](https://docs.docker.com/engine/article-img/architecture.svg)



### Docker Daemon

Docker Daemon은 Docker API 요청을 수신하고 Image, Container, Network 및 Volume과 같은 Docker Object를 관리합니다. Daemon은 Docker 서비스를 관리하기 위해 다른 Daemon과 통신 할 수도 있습니다.

### Docker Client

Docker Client(`docker`)는 많은 Docker 사용자가 Docker와 상호 작용하는 주요 방법입니다. `docker run`과 같은 명령을 사용하면 Client는 해당 명령을 Docker Daemon로 전송하여 이를 수행합니다. `docker` 명령은 Docker API를 사용합니다. Docker Client는 둘 이상의 Docker Daemon과 통신 할 수 있습니다.

### Docker Registry

Docker Registry는 Docker Image를 저장합니다. Docker Hub 및 Docker Cloud는 누구나 사용할 수있는 Public Registry이며, Docker는 기본적으로 Docker Hub에서 Image를 찾아 Container를 구성하도록 되어 있습니다. 자신의 개인 Registry를 구성할 수도 있습니다. [Docker Datacenter(DDC)](https://blog.docker.com/2016/02/docker-datacenter-caas/)를 사용하는 경우, [Docker Trusted Registry(DTR)](https://docs.docker.com/datacenter/dtr/2.0/#built-in-security-and-access-control)가 포함됩니다.

Docker Store를 사용하면, Docker Image를 사고 팔거나 무료로 배포 할 수 있습니다. 예를 들어, 소프트웨어 공급 업체의 응용 프로그램 또는 서비스가 포함된 Docker Image를 구입하고, Image를 사용하여 응용 프로그램을 테스트, 준비 및 프로덕션 환경에 배포할 수 있습니다. Image의 새 버전을 기반으로 Container를 다시 배포하여 응용 프로그램을 업그레이드 할 수도 있습니다.

### Docker Object

#### Image

이미지는 Docker 컨테이너를 생성하기 위한 읽기 전용 템플릿입니다. 종종 이미지는 다른 이미지를 기반으로 하며, 몇 가지 추가 사용자 정의가 있습니다. 예를 들어, 우분투 이미지를 기반으로 이미지를 만들 수 있지만, Apache 웹 서버와 애플리케이션을 설치할 수 있을 뿐만 아니라, 애플리케이션을 실행하는 데 필요한 세부 설정도 할 수 있습니다.

나만의 이미지를 만들거나 다른 사람들이 만들고 레지스트리에 게시한 이미지만 사용할 수 있습니다. 자신의 이미지를 만들려면 이미지를 만들고 실행하는 데 필요한 단계를 정의하기 위한 간단한 구문으로 Dockerfile을 만듭니다. Dockerfile의 각 명령은 이미지에 레이어를 만듭니다. Dockerfile을 변경하고 이미지를 다시 작성하면 변경된 레이어만 재구성됩니다. 이는 다른 가상화 기술과 비교할 때 이미지를 매우 가볍고 작고 빠르게 만드는 요소의 일부입니다.

#### container

#### service

## Container

**Container는 공유된 운영 체제에서 격리되어 실행할 수 있는 형식으로 소프트웨어를 패키지화하는 방법**입니다. VM과 달리, Container는 전체 운영 체제를 번들로 제공하지 않습니다. - 단지, 소프트웨어가 실행할 때 필요한 라이브러리 및 설정만 필요합니다. 이는 효율적이고, 가벼우며, 독립적인 시스템을 구축하고, 배포 위치에 상관없이 소프트웨어가 항상 동일하게 실행되도록 보장합니다.

![Container](https://www.docker.com/sites/default/files/Package%20software%40x2.png)

### Container에 대하여

**Development, Shipment, Deployment**를 위해 표준화된 단위로 패키지된 소프트웨어

Container 이미지는 코드, 런타임, 시스템 도구, 시스템 라이브러리, 설정 등 소프트웨어를 실행하는 데 필요한 모든 것을 포함하는 **경량의 독립 실행형 실행 패키지**입니다. Linux 및 Windows 기반 응용 프로그램 모두에서 사용할 수 있는 Container화된 소프트웨어는 환경에 관계없이 항상 동일하게 실행됩니다. 컨테이너는 Devlopment 환경과 Staging 환경의 차이점과 같은 주변 환경으로부터 소프트웨어를 격리하고, 동일한 인프라에서 다른 소프트웨어를 실행하는 팀들 간의 충돌을 줄이는 데 도움이 됩니다(?).

### Container의 특징

#### Lightweight

단일 컴퓨터에서 실행되는 Docker 컨테이너는 해당 컴퓨터의 운영 체제 커널을 공유합니다. 컨테이너들은 즉시 시작할 수 있고, RAM 및 계산 시간이 덜 소모됩니다. 이미지는 파일 시스템 레이어로 구성되며 공통 파일을 공유합니다. 이렇게하면 디스크 사용이 최소화되고 이미지 다운로드가 훨씬 빨라집니다.

#### Standard

Docker 컨테이너는 개방형 표준을 기반으로 하며 모든 주요 Linux 배포판, Microsoft Windows 및 VM, 베어 메탈 및 클라우드를 포함한 모든 인프라에서 실행됩니다.

#### Secure

Docker 컨테이너는 서로 또는 기본 인프라로부터 응용 프로그램을 격리합니다. Docker는 응용 프로그램 문제를 전체 시스템이 아닌 단일 컨테이너로 제한하기 위해 가장 강력한 기본 격리를 제공합니다.

### Container VS VM

Container와 VM은 리소스 격리 및 할당 이점이 비슷하지만, Container는 하드웨어가 아닌 운영 체제를 가상화하기 때문에 기능이 다르게 작동합니다. Container는 VM보다 더 휴대 가능하고 효율적입니다.

#### Container

![Container](https://www.docker.com/sites/default/files/Container%402x.png)

Container는 코드와 종속성을 함께 패키징하는 앱 계층의 추상화입니다. 여러 Container를 동일한 시스템에서 실행하고 OS 커널을 다른 Container와 공유할 수 있습니다. 각 Container는 사용자 공간에서 분리 된 프로세스로 실행됩니다. Container는 VM보다 공간을 적게 차지하며 (Container 이미지는 일반적으로 수십 MB 크기 임) 거의 즉시 시작됩니다.



#### Virtual Machines(흔히 아는 가상화방식: VMWare, VirtualBox)

![VM](https://www.docker.com/sites/default/files/VM%402x.png)

가상 머신(VM)은 하나의 서버를 여러 서버로 전환시키는 물리적 하드웨어의 추상화입니다. 하이퍼 바이저를 사용하면 여러 대의 VM을 단일 시스템에서 실행할 수 있습니다. 각 VM에는 운영 체제의 전체 복사본, 하나 이상의 응용 프로그램, 필요한 바이너리 및 라이브러리가 포함되어 있습니다 (수십 GB를 차지함). VM도 부팅 속도가 느릴 수 있습니다.

> 하이퍼바이저(hypervisor)는 호스트 컴퓨터에서 다수의 운영 체제(operating system)를 동시에 실행하기 위한 논리적 플랫폼(platform)을 말한다. 가상화 머신 모니터(virtual machine monitor, 줄여서 VMM)라고도 부른다.
>
> 참고: [하이퍼바이저](https://ko.wikipedia.org/wiki/%ED%95%98%EC%9D%B4%ED%8D%BC%EB%B0%94%EC%9D%B4%EC%A0%80)

#### 요약

![Summary](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/vm-vs-docker.png)

#### Container and Virtual machines Together

![img](http://www.docker.com/sites/default/files/Container-vms-together%402x.png)



## Flow

Layer 부분은 적절히 찢어서 설명

### Basic

![default](http://pointful.github.io/docker-intro/docker-img/basics-of-docker-system.png)





### Change or Update

![update](http://pointful.github.io/docker-intro/docker-img/changes-and-updates.png)



#### Image와 Container의 Layer

![container](http://docs.docker.com/engine/userguide/storagedriver/images/container-layers.jpg)

## Get Started
