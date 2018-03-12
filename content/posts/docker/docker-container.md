---
date: "2018-03-09T08:22:39+09:00"
title: "Container"
authors: ["1000jaeh"]
categories:
  - posts
tags:
  - docker
  - container
  - virtualization
  - layer
cover:
  image: "../images/docker-official.svg"
draft: true
---
Container란, 운영체제의 커널이 하나의 사용자 공간 인스턴스가 아닌, 여러 개의 격리된 사용자 공간 인스턴스를 갖출 수 있도록 하는 서버 가상화 방식입니다. 이러한 인스턴스들은 소유하고 있는 Host Machine과 사용자의 관점에서 바라보면 실제 서버인 것 처럼 보입니다. 먼저 가상화에 대해 알아보고, Docker Container의 구조와 특징에 대해서 자세히 살펴보겠습니다.

## 가상화(Virtualization)

흔히 우리가 알고있는 가상화(Virtualization)는, 하드웨어와 소프트웨어를 결합하여 가상 머신(VM)을 만들어 사용하는 플랫폼 가상화를 의미합니다. 플랫폼 가상화는 주어진 하드웨어 플랫폼 위에서 제어 프로그램, 즉 호스트 소프트웨어를 통해 실행됩니다. 호스트 소프트웨어는 호스트 환경 내의 게스트 소프트웨어에 맞춰 가상 머신(VM)을 만들어 냅니다. 게스트 소프트웨어는 완전한 운영체제이며, 독립된 하드웨어 플랫폼에 설치된 것처럼 실행됩니다.

### Virtual Machine

가상 머신(VM)은 하나의 서버를 여러 서버로 전환시키는 물리적 하드웨어의 추상화로 **하이퍼바이저(Hypervisor) 기반의 시스템 가상화입니다.** 하이퍼바이저는 기반이 되는 시스템 안에 또 다른 시스템을 구동 시킬 수 있게 시스템의 각 요소들을 가상화해서 제공합니다. 이 때 기반 시스템을 보통 호스트 시스템이라고 하고 하이퍼바이저 위에서 돌아가는 시스템을 가상 시스템을 의미합니다. 하이퍼 바이저는 호스트 시스템의 자원을 기반으로 가상 시스템이 독립적으로 움직일 수 있도록 합니다. 따라서, 하이퍼바이저를 사용하여 여러 대의 VM을 단일 시스템에서 실행할 수 있지만, 호스트 시스템의 하드웨어 자원에 제한을 받습니다.

![](https://www.docker.com/sites/default/files/VM%402x.png)

참고
하이퍼바이저(hypervisor)는 호스트 컴퓨터에서 다수의 운영 체제(operating system)를 동시에 실행하기 위한 논리적 플랫폼(platform)을 말한다. 가상화 머신 모니터(virtual machine monitor, 줄여서 VMM)라고도 부른다.

### Container

반면 Container는 공유된 운영 체제에서 격리되어 실행할 수 있는 형식으로 소프트웨어를 가상화하는 방법입니다. 즉, 하이퍼바이저처럼 시스템의 전반적인 것을 가상화하는 것이 아닌 애플리케이션을 구동할 수 있는 환경 즉, CPU와 메모리 영역 등을 가상화하고 구동하는데 필요한 운영체제나 라이브러리는 호스트 시스템과 공용으로 사용합니다.

### VM VS Container

VM은 CPU, 메모리, 디스크는 물론이고 그 안에서 구동 중인 OS와 각종 라이브러리까지 전체 영역을 독립적으로 구분되어 운영됩니다. 따라서, 각 VM에는 운영체제의 전체 번들, 하나 이상의 응용 프로그램, 필요한 바이너리 및 라이브러리 전체가 포함되어 있기 때문에, 수십 GB를 차지합니다. 반면, Container는 전체 운영 체제를 번들로 제공하지 않습니다. 단지,
소프트웨어가 실행할 때 필요한 라이브러리 및 설정들만 포함되기 때문에 VM보다 적은 공간을 차지합니다. 일반적으로 Container를 생성하는 Image의 크기는 수십 MB정도 입니다. 따라서, Container는 VM보다 공간 효율적이고,
가볍습니다.

VM과 Container는 동작방식에서도 차이가 있습니다. 하이퍼바이저 기반의 가상 시스템에서 애플리케이션을 구동하기 위해서는
해당 가상 시스템이 완전히 실행된 후에 애플리케이션이 동작됩니다. 따라서, 실제 애플리케이션이 구동되기까지의 시간이 길게 걸리지만, Container는 이미 시스템이 동작되고 있는 상태에서 애플리케이션 부분만 가상화되어 실행되기 때문에 속도가 훨씬 빠릅니다(거의 즉시 시작됩니다). 또한, 가상화된 시스템이 OS등을 동작할 때 사용하는 자원을 Docker에서는 사용하고 있지 않기 때문에 자원의 소비율 역시 VM에 비해서 Container는 적습니다.

### Docker Container의 한계

하지만 Docker Container에도 한계점은 존재합니다. 기존 하이퍼바이저 기반의 가상화 기술보다 Container가 뛰어난 부분은 앞서 얘기한 것처럼 속도가 빠르고 자원 소비율이 적기 때문에, 같은 자원을 갖고 있는 호스트 시스템에서 더 많은 어플리케이션을 가상화 기반 위에 동작시킬 수 있습니다. 하지만 Docker의 기반이 되는 Container기술은 애플리케이션의
구동 환경을 가상화하기 때문에, 호스트 시스템에서 제공하는 OS와 같은 환경에서만 제공한다는 단점이 있습니다. 즉, 일반 Docker 환경(Linux)에서 Windows 애플리케이션 실행은 현재로선 어렵다는 것입니다. 이는 Container 기술 자체가 OS와 실행 바이너리, 라이브러리를 공유하면서 애플리케이션 자체의 실행 환경만 가상화하여 제공하는 기술이기 때문에 어쩔 수 없는 상황입니다.

참고
현재는, Enterprise Edition에서 Windows환경을 지원하고 있습니다. 자세한 내용은 [Install Docker](https://docs.docker.com/engine/installation/)에서 Supported platforms항목을 확인하시기 바랍니다.

## Container

Docker Container는 다음과 같은 특징을 갖고 있습니다.

| Lightweight | Standard | Secure |
| --- | --- | --- |
| Docker Container는 Host OS의 Kernel을 공유하여 사용합니다. 컨테이너들은 즉시 시작할 수 있고, RAM 및 계산 시간이 덜 소모됩니다. 이미지는 파일 시스템 레이어로 구성되며 공통 파일을 공유합니다. 이렇게하면 디스크 사용이 최소화되고 이미지 다운로드가 훨씬 빨라집니다. | Docker 컨테이너는 개방형 표준을 기반으로 하며 모든 주요 Linux 배포판, Microsoft Windows 및 VM, 베어 메탈 및 클라우드를 포함한 모든 인프라에서 실행됩니다. | Docker 컨테이너는 서로 또는 기본 인프라로부터 응용 프로그램을 격리합니다. Docker는 응용 프로그램 문제를 전체 시스템이 아닌 단일 컨테이너로 제한하기 위해 가장 강력한 기본 격리를 제공합니다. |

### Image와 Container의 Layer

먼저 Container의 구조에 대해서 살펴보기 전에, Container 실행의 주체가 되는 Image에 대해 알아보겠습니다.

#### Image와 Layer

Docker Image는 Dockerfile이라는  Build 명세서를 바탕으로 생성됩니다. 아래는 Latest Version(16.04 Version) Ubuntu Image의 Dockerfile입니다([Github: docker-brew-ubuntu-core/xenial/Dockerfile](https://github.com/tianon/docker-brew-ubuntu-core/blob/5fce3945d95630c2fc03c21ef8665d92bd824642/xenial/Dockerfile)).

``` text
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /

# a few minor docker-specific tweaks
# see https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap
RUN set -xe \

    ... 중략 ...

# delete all the apt list files since they're big and get stale quickly
RUN rm -rf /var/lib/apt/lists/*
# this forces "apt-get update" in dependent images, which is also good

# enable the universe
RUN sed -i 's/^#\s*\(deb.*universe\)$/\1/g' /etc/apt/sources.list

# make systemd-detect-virt return "docker"
# See: https://github.com/systemd/systemd/blob/aa0c34279ee40bce2f9681b496922dedbadfca19/src/basic/virt.c#L434
RUN mkdir -p /run/systemd && echo 'docker' > /run/systemd/container

# overwrite this with 'CMD []' in a dependent Dockerfile
CMD ["/bin/bash"]
```

Ubuntu Docker 파일은 다음의 명령어들로 구성되어 있습니다.

- FROM
- ADD
- 4개의 RUN
- CMD

Dockerfile로 부터 Image가 구성될 때, Dockerfile의 각각의 명령어가 Layer로 구성됩니다.

만약 `docker build \[OPTIONS\] PATH \| URL \| -` 를 사용하여 위의 Dockerfile로부터 Ubuntu Image를 Build할 때, FROM을 제외한 6개의 명령어가 각각의 Layer로 구성됩니다.

참고
해당 Dockerfile의 **FROM scratch**는 비어있는 Image를 생성하는 명령어로, Layer를 구성하지 않습니다. 해당 Image의 Layer가 실제로 Dockerfile의 명령어로 이루어져 있는지 `docker history \[options\] IMAGE`를 사용하여 확인해보겠습니다.

``` bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              747cb2d60bbe        2 weeks ago         122MB

$ docker history 747cb2d60bbe
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
747cb2d60bbe        2 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           2 weeks ago         /bin/sh -c mkdir -p /run/systemd && echo '...   7B                  
<missing>           2 weeks ago         /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   2.76kB              
<missing>           2 weeks ago         /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
<missing>           2 weeks ago         /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745B                
<missing>           2 weeks ago         /bin/sh -c #(nop) ADD file:5b334adf9d9a225...   122MB
```

총 6개의 행을 확인할 수 있으면, CREATED BY열에서 해당 Layer가 어떤 명령어로 생성되었는지 확인할 수 있습니다.

참고
IMAGE열이 &lt;missing&gt;으로 표현된 Layer는 해당 Layer가 다른 시스템에 의해 작성되었으며, Local에서 사용할 수 없음을 의미합니다. 이렇게 생성된 Layer를 Image Layer라 하며, Image Layer는 수정이 불가능하며, 오로지 읽기만 가능합니다.

### Container와 Layer

Container는 Image Layer 최상단에 쓰기가능한 Layer가 추가되어, 실행됩니다. 추가된 Layer를 Container Layer라고 부릅니다. Container Layer에는 Container 기동 중 발생하는 모든 변경 사항들(파일 생성, 수정, 삭제 등)이 저장됩니다. 아래는 현재 Local Repository에 실제로 저장된 Ubuntu의 Image Layer입니다.

``` diff
/ # ls /var/lib/docker/overlay2/
14ed0fd37eebdc612b798270d3685a6a6c1a6c42dde696aa4048533dfe1a86fe
21535fcab696330a4d6de5bed2a06e65f8547ca6d3cdaacf6dd636f41a4d27cf
8803ef05c1bd6845eade31d9aba92f538700971e630cf274b3ccd11dc6a8a304
fc25a819ae26279427b3203b1acd4309cb622de19479fe1ce5890635c0052740
fc83da02f8a1ec7f9cc40e512aafe550dd1030883ddc82c56e59d229dcda60f2
l
```

참고
Docker의 Image와 Container가 저장된 위치는 `/var/lib/docker/&lt;storage\_driver&gt;/` 경로로 확인할 수 있습니다. 단, Mac과 Window 개발환경은 설치된 Docker 가상 Machine에 직접 접속하여야 확인 가능합니다.

그럼 Ubuntu Image로 부터 Container를 기동하겠습니다.

``` bash
$ docker run -dit --name ubuntu-local ubuntu /bin/bash
8c0ee2a29f48b2e9ac27ed26f87aa4f32705dd52e3bea228528e6d9fce210e76

$ docker ps -s
CONTAINER ID        IMAGE               COMMAND             CREATED                  STATUS              PORTS               NAMES               SIZE
8c0ee2a29f48        ubuntu              "/bin/bash"         Less than a second ago   Up 4 seconds                            ubuntu-local        0B (virtual 122MB)
```

Container가 기동되면서, Layer가 추가된 것을 확인할 수 있습니다.

``` diff
/ # ls /var/lib/docker/overlay2/
14ed0fd37eebdc612b798270d3685a6a6c1a6c42dde696aa4048533dfe1a86fe
21535fcab696330a4d6de5bed2a06e65f8547ca6d3cdaacf6dd636f41a4d27cf
8803ef05c1bd6845eade31d9aba92f538700971e630cf274b3ccd11dc6a8a304
+ c7a75746f5f12bcbb5ae03c20a65f080b2e7c9f1ec897491309ac3843454cff8
+ c7a75746f5f12bcbb5ae03c20a65f080b2e7c9f1ec897491309ac3843454cff8-init
fc25a819ae26279427b3203b1acd4309cb622de19479fe1ce5890635c0052740
fc83da02f8a1ec7f9cc40e512aafe550dd1030883ddc82c56e59d229dcda60f2
l
```

기동 중인 Container를 삭제하겠습니다.

``` bash
$ docker stop ubuntu-local
ubuntu-local

$ docker rm ubuntu-local
ubuntu-local

$ docker ps -s
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES               SIZE
```

Container Layer만 삭제되고, Image Layer만 남아있는 것을 알 수 있습니다.

``` diff
/ # ls /var/lib/docker/overlay2/
14ed0fd37eebdc612b798270d3685a6a6c1a6c42dde696aa4048533dfe1a86fe
21535fcab696330a4d6de5bed2a06e65f8547ca6d3cdaacf6dd636f41a4d27cf
8803ef05c1bd6845eade31d9aba92f538700971e630cf274b3ccd11dc6a8a304
fc25a819ae26279427b3203b1acd4309cb622de19479fe1ce5890635c0052740
fc83da02f8a1ec7f9cc40e512aafe550dd1030883ddc82c56e59d229dcda60f2
l
```

정리하면, 다음과 같습니다.

- Image는 Layer의 Set으로 이루어져 있으며, 각각의 Layer는 Dockerfile의 명령어에 해당합니다.
- Image Layer는 읽기전용 Layer로 수정할 수 없습니다.
- Container를 기동하면, 쓰기 가능한 Container Layer가 Image Layer Set의 최상단에 추가됩니다.
- Container 기동 중 발생한 모든 행위는 Container Layer에 기록되며, Container가 삭제되면 해당 Container Layer도 삭제됩니다(단, Image Layer는 삭제되지 않습니다).

아래는 Image와 Container Layer를 도식화한 것입니다.

![](attachments/35440165/41447689.png)

#### Contariner들간의 Layer 공유

나아가, 하나의 Image로 부터 여러개의 Container가 기동될 때는 어떻게 되는지 알아보겠습니다. 먼저 Ubuntu Image로 부터 **ubuntu-local-NUM** 이름의 Container들을 기동시킵니다.

``` bash
$ docker run -dit --name ubuntu-local-1 ubuntu /bin/bash \
&& docker run -dit --name ubuntu-local-2 ubuntu /bin/bash \
&& docker run -dit --name ubuntu-local-3 ubuntu /bin/bash
cc2f62db98b4ed5aadc6904229db1c1f2f58d69f3ef4c120eb78402a12a96f7d
f921b84e42d42bdad629f58398ac18d942c25c3451238a361f841b0b8d630523
87fccd76cd1d195bc25dd0e30203c3d3f056bb69a2851a1feef8026386ccb440

$ docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
87fccd76cd1d        ubuntu              "/bin/bash"         About an hour ago   Up About an hour                        ubuntu-local-3
f921b84e42d4        ubuntu              "/bin/bash"         About an hour ago   Up About an hour                        ubuntu-local-2
cc2f62db98b4        ubuntu              "/bin/bash"         About an hour ago   Up About an hour                        ubuntu-local-1
```

Local Docker에 실제로 저장된 Container와 Layer입니다. 각 Container 별 Container Layer가 추가되었으나, Image Layer는 추가되지 않았습니다.

``` diff
/ # ls /var/lib/docker/containers
87fccd76cd1d195bc25dd0e30203c3d3f056bb69a2851a1feef8026386ccb440
cc2f62db98b4ed5aadc6904229db1c1f2f58d69f3ef4c120eb78402a12a96f7d
f921b84e42d42bdad629f58398ac18d942c25c3451238a361f841b0b8d630523

/ # ls /var/lib/docker/overlay2/
14ed0fd37eebdc612b798270d3685a6a6c1a6c42dde696aa4048533dfe1a86fe
+ 15123a9c6da09e3ac10a96846059a7af7bdfc040d7895eae40f22df12c8f3e5a
+ 15123a9c6da09e3ac10a96846059a7af7bdfc040d7895eae40f22df12c8f3e5a-init
21535fcab696330a4d6de5bed2a06e65f8547ca6d3cdaacf6dd636f41a4d27cf
+ 2de0c7fc0c717cb2b834b33857d369b92599bfd53078a16bfa7690af2200b727
+ 2de0c7fc0c717cb2b834b33857d369b92599bfd53078a16bfa7690af2200b727-init
8803ef05c1bd6845eade31d9aba92f538700971e630cf274b3ccd11dc6a8a304
+ ed0d6800996f0b1e0d7f0ae845128a89c9029aa6019dd3ea2efed965e4bee760
+ ed0d6800996f0b1e0d7f0ae845128a89c9029aa6019dd3ea2efed965e4bee760-init
fc25a819ae26279427b3203b1acd4309cb622de19479fe1ce5890635c0052740
fc83da02f8a1ec7f9cc40e512aafe550dd1030883ddc82c56e59d229dcda60f2
```

기동 중인 Container의 대락젹인 크기는 `docker ps -s`으로 확인할 수 있습니다.

``` bash
$ docker ps -s
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES               SIZE
87fccd76cd1d        ubuntu              "/bin/bash"         About an hour ago   Up About an hour                        ubuntu-local-3      0B (virtual 122MB)
f921b84e42d4        ubuntu              "/bin/bash"         About an hour ago   Up About an hour                        ubuntu-local-2      0B (virtual 122MB)
cc2f62db98b4        ubuntu              "/bin/bash"         About an hour ago   Up About an hour                        ubuntu-local-1      0B (virtual 122MB)
```

- **size**: 각 Container의 Container Layer에서 사용하고 있는 Data 양
- **virtual size**: Image Layer에서 사용하고 있는 Data 양. 

Local Docker의 File System에 저장된 실제 Container 크기입니다.

``` bash
/ # du -sh /var/lib/docker/containers/*
32.0K   /var/lib/docker/containers/87fccd76cd1d195bc25dd0e30203c3d3f056bb69a2851a1feef8026386ccb440
32.0K   /var/lib/docker/containers/cc2f62db98b4ed5aadc6904229db1c1f2f58d69f3ef4c120eb78402a12a96f7d
32.0K   /var/lib/docker/containers/f921b84e42d42bdad629f58398ac18d942c25c3451238a361f841b0b8d630523
```

지금까지의 과정에서 다음의 2가지 사실을 확인할 수 있습니다.

- 같은 Image를 바탕으로 Container가 기동될 때, Container Layer만 추가된다.
- 대략적으로 보이는 Container의 크기와 실제 저장된 크기는 차이가 있다. (docker ps -s로 나타난 각 Container의 크기는 122MB로 보이지만, 실제 File System에 저장된 Container는 32.0K입니다.)

이를 종합적으로 봤을 때, 동일한 Image를 바탕으로 기동된 Container는 각각의 Container Layer만 추가되며, Image Layer는 공유하여 사용한다는 사실을 알 수 있습니다(실제로 Docker는 동일한 Image에서 기동된 Container들은 Image Layer를 100%공유하여 사용합니다). 또한, Container는 개별적으로 Container Layer를 갖고 해당 Layer에 모든 변경된 내용이 저장되기 때문에, 여러 Container가 동일한 기본 Image를 공유하여 사용하면서 Container 자시의 Data 상태를 가질 수 있습니다. 이러한 특성으로 인해, Docker Container는 더욱 가벼워지고 기동속도가 빨라질 수 있는 것입니다.

참고
Container Layer의 파일 쓰기 및 변경 대한 자세한 내용은 Docker Documents에서 [About images, containers, and storage drivers](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/)의 copy-on-write Strategy를 참고하시기 바랍니다.

이를 다이어그램으로 표현하면 다음과 같습니다.

![](https://images.techhive.com/images/article/2016/06/docker-images-vs-containers-100664049-large.idge.png)
