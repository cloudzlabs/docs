---
date: "2018-02-28T10:59:06+09:00"
title: "Docker in docker"
authors: ["bckim0620"]
categories:
  - posts
tags:
  - Docker in docker
  - Docker
  - dind
draft: false
---

지난 한주 동안 애플리케이션을 Kubernetes 배포하는 CI/CD 파이프라인 구성했습니다. 
K8S CI/CD 파이프라인 구성은 차후에 포스팅하기로 하고, K8S용 파이프라인 구성에 사용된 Docker in docker 기술을 소개하겠습니다.

DinD(Docker In Docker)기술은 컨테이너 내부에 Docker를 사용하는 기술을 말하며 일반적으로 K8S CI/CD 구성, Docker 엔진 개발 등에 사용됩니다.

## DinD 원리
DinD 원리를 이해하기 위해서는 Docker의 명령어 실행 구조를 이해해야 합니다.

Docker를 설치/실행하면 Docker daemon과 Docker CLI가 설치/실행됩니다.
콘솔 화면에서 Docker 명령어를 입력할 경우 Docker CLI가 명령어를 받아서 Socket을 통해 Docker daemon에 전달합니다.
Docker 설치 경로 중 `/var/run/docker.sock` 파일을 볼 수가 있는데, 이 파일이 Docker daemon에게 명령을 내릴수 있는 인터페이스입니다.
Docker CLI가 `/var/run/docker.sock`를 통해 daemon에 명령어를 전달한다는 것은, 외부의 Docker CLI도 저파일에 접근 할수 있다면 해당 Docker에 명령을 내릴 수 있는 것 입니다.

결국 `/var/run/docker.sock` 파일만 공유를 하면 Local에 설치된 Docker를 컨테이너 내부에서 사용할 수 있고,
또한 컨터에너 내부에 설치된 Docker를 다른 컨테이너 내부에서 사용이 가능해집니다.

## 사용해보기

로컬에 설치된 Docker를 컨테이너에서 사용하기 위해 아래 명령어를 통해 수행합니다.

``docker run --rm -v /var/run/docker.sock:/var/run/docker.sock docker docker ps``

`-v` 옵션을 통해 로컬 Docker의 `/var/run/docker.sock`를 컨테이너의 `/var/run/docker.sock`로 공유합니다.
사용된 이미지는 Docker가 설치된 이미지로 원래는 컨테이너 내부에서 자신의 Docker를 사용하도록 만들어진 이미지. 단 위의 명령어는 `/var/run/docker.sock`를 local것으로 교체하여 local Docker에 명령어 전달합니다.
이미지에 대한 자세한 내용은 [Docker image](https://hub.docker.com/_/docker/)에서 참고하세요.

#### 결과
로컬 PC의 `Docker ps` 결과물과 동일한 내용이 출력됩니다.

```cmd
$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock docker docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                  PORTS               NAMES
8614e57a7099        docker              "docker-entrypoint.s…"   1 second ago        Up Less than a second                       clever_morse
```
