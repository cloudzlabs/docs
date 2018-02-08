---
date: "2018-02-07T10:26:47+09:00"
title: "Swarm"
authors: ["1000jaeh"]
categories:
  - CaaS
tags:
  - Docker
  - Swarm
draft: false
---
# Swarm

Docker의 현재 버전에는 swarm이라고 불리는 Docker 엔진의 클러스터를 기본적으로 관리하기 위한 Swarm 모드가 포함되어 있습니다. Docker CLI를 사용하여 swarm을 만들고, swarm에 응용 프로그램 서비스들을 배포하고, swarm의 행동을 관리하십시오.

> Swarm 모드에서 Docker를 사용하려면 Docker 1.12.0 이상을 설치하십시오. 모든 플랫폼에 대한 설치 지침이 있습니다.

### Feature highlights

#### Docker Engine과 통합 된 클러스터 관리
Docker Engine CLI를 사용하여 응용 프로그램 서비스를 배포 할 수있는 Docker 엔진 Swarm을 만듭니다. swarm을 만들거나 관리하기 위해서 오케스트레이션 소프트웨어를 추가할 필요가 않습니다.

#### 분산된 설계
배포시 노드 역할 간의 차별화를 처리하는 대신, Docker 엔진으로 런타임시 모든 전문화(specialization)를 처리합니다(?). Docker Engine을 사용하여 두 종류의 노드, 관리자 및 작업자를 배포할 수 있습니다. 즉, 하나의 디스크 이미지에서 전체 swarm을 만들 수 있습니다.

#### 선억적 서비스 모델
Docker Engine은 선언적 접근 방식을 사용하여 응용 프로그램 스택에서 다양한 서비스의 원하는 상태를 정의 할 수있게합니다. 예를 들어, 메시지 큐잉 서비스와 함께 있는 웹 프론트엔드 서비스와 데이터베이스 백엔드로 구성된 애플리케이션을 작성할 수 있습니다.

#### 스케일링
각 서비스에 대해, 실행할 작업 수를 선언할 수 있습니다. scale up 또는 down할 때, swarm 관리자는 원하는 상태를 유지하기 위해 작업을 추가하거나 제거하여 자동으로 조정합니다.

#### Desired state reconciliation
swarm 관리자 노드는 지속적으로 클러스터 상태를 모니터링하고 실제 상태와 사용자가 원하는 원하는 상태 사이의 차이점을 조정합니다. 예를 들어, 컨테이너의 복제본 10 개를 실행하도록 서비스를 설정하고 두 개의 복제본을 호스팅하는 작업자 시스템이 충돌하면 관리자는 충돌 한 복제본을 대체 할 두 개의 새 복제본을 만듭니다. swarm manager는 새로운 복제본을 실행중인 사용 가능한 작업자에게 할당합니다.

#### 다중 호스트 네트워킹
서비스에 대한 오버레이 네트워크를 지정할 수 있습니다. swarm 관리자는 응용 프로그램을 초기화하거나 업데이트 할 때 자동으로 오버레이 네트워크의 컨테이너에 주소를 할당합니다.

#### Servicec discovery
swarm 관리자 노드는 swarm의 각 서비스에 고유한 DNS 이름을 할당하고 실행중인 컨테이너의 load balances합니다. swarm에 포함된 DNS 서버를 통해 swarm에서 실행중인 모든 컨테이너에 질의할 수 ​​있습니다.

#### Load balacing
서비스용 포트를 외부 부하 분산 장치에 노출 할 수 있습니다. 내부적으로 swarm은 노드 간에 서비스 컨테이너를 distribute하는 방법을 지정할 수 있게 합니다.

#### 보안
swarm의 각 노드는 자체와 다른 모든 노드 간의 통신을 보호하기 위해 `TLS` 상호 인증 및 암호화를 시행합니다. 사용자 지정 루트 CA에서 자체 서명된 루트 인증서 또는 인증서를 사용하는 옵션이 있습니다.

#### Rolling updates
롤아웃 시간에 노드에 점진적으로 서비스 업데이트를 적용 할 수 있습니다. swarm manager를 사용하면 여러 노드 세트에 대한 서비스 배포 사이의 지연을 제어할 수 있습니다. 문제가 발생하면 작업을 이전 버전의 서비스로 롤백할 수 있습니다.

### Swarm Architecture

![swarm](https://learning-continuous-deployment.github.io/assets/images/docker-swarm.png)

추가적으로 필요한거
swarm load balancing
swarm discovery (server side)

swarm 장단점

### k8s

### swarm vs k8s

## 참고

### Docker

- [Docker](https://www.docker.com)
- [Docker란 무엇입니까?](https://aws.amazon.com/ko/docker/)
- [Why Docker](https://www.slideshare.net/dotCloud/why-docker)
- [Docker Docs](https://docs.docker.com/engine/docker-overview/)
- [초보를 위한 도커 안내서](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
- [Docker Swarm을 이용한 분산서버관리](https://subicura.com/2017/02/25/container-orchestration-with-docker-swarm.html)
- `https://www.slideshare.net/SufyaanKazi/cloud-foundry-vs-docker-vs-kubernetes`
- [Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/)
