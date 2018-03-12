---
date: "2018-03-12T13:23:13+09:00"
title: "CF에 Docker Image 배포"
authors: ["bckim0620"]
categories:
  - posts
tags:
  - Cloud Founcry
  - Docker
description: ""
draft: false
---

Cloud에서 애플리케이션은 Cloud Fondry(CF), Docker 등 다양한 환경에 배포 할 수 있습니다. CF는 웹 애플리케이션 개발에 특화되어 있어 Docker에 비해 웹 애플리케이션 개발에 필요한 다양한 기능을 제공하고 있습니다. 반면에 Docker는 다양한 형태의 애플리케이션 개발이 가능하며 자유도가 높은 장점을 갖고 있습니다. 이 둘의 장점을 모두 사용하기 위해 Docker Image를 CF에 배포하는 리서치를 진행하여 이를 공유합니다.

## 사용법 

### CF 설정
CF에 Docker image를 배포하기 위해서는 `diego_docker`플래그가 설정되어야 합니다. 아래 코드로 설정 할 수 있습니다.
```
cf enable-feature-flag diego_docker
```

### CF Push

#### Docker Hub 사용
아래 명령어를 이용하여 Docker Hub에 배포된 이미지를 CF에 배포할 수 있습니다.

```
cf push APP-NAME --docker-image REPO-NAME/IMAGE-NAME
```

- APP-NAME: 애플리케이션 명
- REPO-NAME: Docker Hub 레파지토리 명
- IMAGE-NAME: Docker Hub에 배포된 Docker image명

#### Private Registry 사용
아래 명령어를 이용하여 Private Registry에 배포된 이미지를 CF에 배포할 수 있습니다.

```
cf push APP-NAME --docker-image MY-PRIVATE-REGISTRY.DOMAIN:PORT/REPO-NAME/IMAGE-NAME
```

- APP-NAME: 애플리케이션 명
- MY-PRIVATE-REGISTRY.DOMAIN: Private Registry 도메인
- REPO-NAME: Docker Hub 레파지토리 명
- IMAGE-NAME: Docker Hub에 배포된 Docker image명

## Example
Docker Hub CloudFoundry에서 제공하는 lattice-app image을 cf에 배포

```cmd
$ cf push image-test-app --docker-image cloudfoundry/lattice-app
Pushing app image-test-app to org *** / space *** as ***@***.com...
Getting app info...
Creating app with these attributes...
+ 이름:           image-test-app
+ docker image:   cloudfoundry/lattice-app
  routes:
+   image-test-app.***.***.***

Creating app image-test-app...
Mapping routes...

Staging app and tracing logs...
   Creating container
   Successfully created container
   Staging...
   Staging process started ...
   Staging process finished
   Exit status 0
   Staging Complete
   Destroying container
   Successfully destroyed container

Waiting for app to start...

이름:                  image-test-app
요청된 상태:           started
인스턴스:              1/1
사용법:                256M x 1 instances
routes:                image-test-app.***.***.***
마지막으로 업로드함:   Mon 12 Mar 16:27:35 KST 2018
스택:                  cflinuxfs2
docker image:          cloudfoundry/lattice-app
start command:         /lattice-app 

     상태      이후                   CPU    메모리      디스크    세부사항
#0   실행 중   2018-03-12T07:33:06Z   0.0%   0 of 256M   0 of 1G   
```