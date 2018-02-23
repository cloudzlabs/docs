---
date: "2018-01-08T14:02:23+09:00"
title: "Container-Layer"
authors: ["1000jaeh"]
series: ["Go Web Dev"]
categories:
  - posts
tags:
  - Docker
  - Cloud
  - Container As a Service
draft: false
---
# Container Layer

- [Container Layer](#container-layer)
    - [Images와 Layers](#images%EC%99%80-layers)
    - [Container와 Layers](#container%EC%99%80-layers)
    - [Disk상의 Container 크기](#disk%EC%83%81%EC%9D%98-container-%ED%81%AC%EA%B8%B0)
    - [Copy-on-Write(CoW) 전략](#copy-on-writecow-%EC%A0%84%EB%9E%B5)
        - [공유는 더 작은 Image를 조장합니다](#%EA%B3%B5%EC%9C%A0%EB%8A%94-%EB%8D%94-%EC%9E%91%EC%9D%80-image%EB%A5%BC-%EC%A1%B0%EC%9E%A5%ED%95%A9%EB%8B%88%EB%8B%A4)
        - [복사는 Container를 효율적으로 만듭니다](#%EB%B3%B5%EC%82%AC%EB%8A%94-container%EB%A5%BC-%ED%9A%A8%EC%9C%A8%EC%A0%81%EC%9C%BC%EB%A1%9C-%EB%A7%8C%EB%93%AD%EB%8B%88%EB%8B%A4)
    - [Data Volume과 Storage Driver](#data-volume%EA%B3%BC-storage-driver)
    - [샘플 텍스트](#%EC%83%98%ED%94%8C-%ED%85%8D%EC%8A%A4%ED%8A%B8)
    - [샘플 표](#%EC%83%98%ED%94%8C-%ED%91%9C)

## Images와 Layers

Docker Image는 일련의 Layer로 구성됩니다. 각 Layer는 Image의 Dockerfile에 있는 명령을 나타냅니다. 마지막 Layer를 제외한 각 Layer는 읽기 전용입니다.

아래의  Dockerfile에 있는 `FROM`, `COPY`, `RUN`, `CMD` 4개의 명령이  각각  Layer를 만듭니다.

```dockerfile
# 4개의 Layer
FROM ubuntu:15.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

`FROM` 은 `ubuntu:15.04` Image에서 Layer를 생성하는 것으로 시작합니다.  `COPY` 는 Docker Client의 현재 디렉토리에서 일부 파일을 추가합니다. `RUN`은 `make`명령어를 사용하여, 응용프로그램을 빌드합니다. 마지막 Layer는 Container 내에서 실행할 명령을 지정합니다.

각각의 Layer는 이전 Layer와의 차이점 Set입니다. Layer는 서로 위에 겹쳐져 있습니다. 새로운 Container를 만들 때, 기본 Layer 위에 새로운 쓰기 가능한 Layer를 추가합니다. 이 Layer를 종종 "Container Layer"라고 합니다. 새 파일 작성, 기존 파일 수정과 삭제와 같은 Container 기동 중에 발생하는 모든 변화들은 Thin Writable Container Layer에 쓰여집니다.

아래 다이어그램은 Ubuntu 15.04 Image를 기반으로하는 Container의 구조입니다.

![image](/docs/images/container-layers.jpg)

*Storage Driver*가 Layers간 상호 작용 방식에 대한 세부 사항들을 처리합니다. 다양한 Storage Driver를 사용할 수 있으며, 상황에 따라 장점과 단점이 있습니다.

## Container와 Layers

Container와 Image의 주된 차이점은 최상단 Writable Layer의 존재입니다. 새 데이터를 추가하거나 기존 데이터를 수정하는 Container에서 발생하는 행위는 이 Writable Layer에 저장됩니다. Container가 삭제되면, 해당 Writable Layer도 삭제됩니다. 기본 Image는 변경되지 않습니다.

각 Container는 자체적으로 Writable Layer를 갖고 있으며, 모든 변경된 내용은 해당 Layer에 저장되지 때문에, 여러 Container가 동일한 기본 Image를 공유하여 사용하면서 Container 자신의 Data 상태를 가질 수 있습니다. 아래 다이어그램은 동일한 Ubuntu 15.04 Image를 공유하는 여러 Container를 보여줍니다.

![multiple container](http://docs.docker.com/engine/userguide/storagedriver/images/sharing-layers.jpg)

> **참고:** 만약 여러 Image들이 동일한 데이터에 대한 접근을 공유할 필요한 경우, 해당 데이터를 Docker Volume에 저장하고 Container들에 마운트하십시오.

Docker는 Storage Driver를 사용하여, Image Layers와 Writable Container Layer의 Contents들을 관리합니다. 각 Storage Driver는 구현은 다르게 되어있지만, 모든 드라이버는 Stackable Image Layers와 copy-on-write(CoW) 전략을 사용합니다.

## Disk상의 Container 크기

실행중인 Container의 대략적인 크기를 보려면 이 `docker ps -s`명령을 사용할 수 있습니다. 두 개의 열이 크기와 관련이 있습니다.

```shell
$ docker ps -s
CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS              PORTS               NAMES               SIZE
5d4c529606f9        acme/my-final-image:1.0   "bash"              2 hours ago         Up 2 hours                              my_container_5      0B (virtual 107MB)
0caa7313f99d        acme/my-final-image:1.0   "bash"              2 hours ago         Up 2 hours                              my_container_4      0B (virtual 107MB)
4001e7dd703c        acme/my-final-image:1.0   "bash"              2 hours ago         Up 2 hours                              my_container_3      0B (virtual 107MB)
b2a14a58e3b6        acme/my-final-image:1.0   "bash"              2 hours ago         Up 2 hours                              my_container_2      0B (virtual 107MB)
285b36287b5e        acme/my-final-image:1.0   "bash"              2 hours ago         Up 2 hours                              my_container_1      0B (virtual 107MB)
```

- `size`: 각 Container의 Writable Layer에서 사용되는 디스크상의 데이터 양
- `virtual size`: Container가 사용하는 Read-only Image에서 사용된 데이터의 양. 여러 Container들이 일부 또는 모든 Read-only Image 데이터를 공유할 수 있습니다. 동일한 Image에서 시작된 두 개의 Container는 Read-only 데이터의 100% 공유하는 반면, 서로 다른 Image를 가진 두 개의 Container는 공통된 Layer만을 공유합니다. 따라서, `virtual size`들을 합산할 수 없습니다. 이로 인해서 잠재적으로 전체 Disk 사용량이 예상치 못하게 과대평가될 수 있습니다.

Disk 상에서 실행 중인 모든 Container들이 사용하는 전체 Disk 공간은 각 Container의 `size`와 `virtual size`값의 일부 조합입니다. 만약 여러 Container들의 `virtual size`가 동일하다면, 동일한 Image에서 만들어졌을 가능성이 큽니다.

또한, 아래와 같은 Container가 Disk 공간을 차지할 수 있는 추가 방법들은 계산하지 않습니다:

- `json-file` Logging Driver를 사용하는 경우, 로그 파일들이 차지하는 Disk 공간. 만약 Container가 대량의 로깅 데이터를 생성하고, 로그 로테이션에 대한 설정이 되어있지 않다면, 이는 중대한 문제가 될 수 있습니다.
- Container가 사용하는 Volume들과 Bind Mounts
- 일반적으로 크기가 작은 Container 설정 파일이 사용하는 Disk 공간
- Disk에 기록된 Memory(Swapping이 활성화된 경우.)
- Checkpoints, 만약 실험적으로 checkpoint/restore Feature를 사용하는 경우.

## Copy-on-Write(CoW) 전략

Copy-On-Write는 파일을 공유하고 복사하여 최대한의 효율성을 높이는 전략입니다. 파일 또는 디렉토리가 Image의 하위 Layer에 존재하거나, 다른 Layer(Writable Layer 포함)에서 해당 파일 또는 디렉토리에 대한 읽기 권한이 필요한 경우, 기존 파일을 사용하기만 합니다. 처음으로 다른 Layer가 파일을 수정해야 할 때 (Image를 만들거나 Container를 실행할 때), 파일은 해당 Layer에 복사되고 수정됩니다. 이렇게 하면, I/O와 각 후속 Layer의 크기가 최소화됩니다. 

### 공유는 더 작은 Image를 조장합니다

Repository로 부터 Image를 Pull하기 위해 `docker pull`를 사용하거나, 아직 로컬에 존재하지 않는 Image에서 Container를 생성하려할 때, 각 Layer는 별도로 분리되어 Docker의 로컬 Repository(일반적으로 Linux에서 `/var/lib/docker/`에 위치)에 저장됩니다. 다음 예제에서 이러한 Layer가 표시되는지 확인할 수 있습니다.

[Ubuntu Image](https://store.docker.com/images/ubuntu) 비교

```shell
$ docker pull ubuntu:latest
latest: Pulling from library/ubuntu
ae79f2514705: Pull complete
5ad56d5fc149: Pull complete
170e558760e8: Pull complete
395460e233f5: Pull complete
6f01dc62e444: Pull complete
Digest: sha256:506e2d5852de1d7c90d538c5332bd3cc33b9cbd26f6ca653875899c505c82687
Status: Downloaded newer image for ubuntu:latest

$ docker pull ubuntu:14.04
14.04: Pulling from library/ubuntu
bae382666908: Pull complete
29ede3c02ff2: Pull complete
da4e69f33106: Pull complete
8d43e5f5d27f: Pull complete
b0de1abb17d6: Pull complete
Digest: sha256:6e3e3f3c5c36a91ba17ea002f63e5607ed6a8c8e5fbbddb31ad3e15638b51ebc
Status: Downloaded newer image for ubuntu:14.04
```

15.04 Version과 최신버전의 Ubuntu Image를 Pull한 경우, 최초 한번 받은 Layer들은 이후에는 Complete처리가 되어 다운받지 않고, 차이나는 Layer만 다운받는 것을 확인할 수 있습니다.

[Ubuntu Image](https://store.docker.com/images/ubuntu) 버전에 따른 Dockerfile 비교

```diff
FROM scratch

# 16.04 버전
- ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /

# 14.04 버전
+ ADD ubuntu-trusty-core-cloudimg-amd64-root.tar.gz /

# a few minor docker-specific tweaks
# see https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap
RUN set -xe \

.... 생략 ....
```

16.04와 14.04 Version을 Pull했을 때, 발생하는 Layer의 차이는 각 Dockerfile의 `ADD`이 달라서 발생하는 것으로 확인할 수 있습니다.

각각의 Layer들은 Docker Host의 로컬 저장 영역의 자체 디렉토리에 저장됩니다. 파일 시스템에서 `/var/lib/docker/<storage-driver>/layers/`의 경로에서 Layer목록을 확인할 수 있습니다.

저장된 Docker Image Layer (Mac 기준)

```shell
# Mac의 Local에 저장된 Docker Machine 위치로 이동
$ cd $HOME/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux

# Docker 내부 Linux 접속
$ screen tty

# overlay2 Storage Driver를 이용하여 Layer들이 저장되어 있음
/ # ls var/lib/docker/overlay2

002250e05883243184641ce260926c4607dccd9625fb1ee1663ac42631f672d4
10df38c294a0149749f53eb97b48fda9cd78d37bae7de7b86d5384763b645584
3116449a1ca87b76339002ccaefddc91bb0dbedb2e0551a82010fee16d18316d
31ebc71440c0f6a7d2fcb6b12e28b12d9566484d1aa4055d0c4b9d6829a78c8c
325fc0f904fb3674b318ce69b2fe654227adb49a31555ca5ac77679e473f408a
3a605d39d1352bf2b13ef71daf5a42e63f846727652875b25d310a842dd5df56
4380ee463d618466f897ea4c264471b9620717388ab5306802d129a8ebc181ab
478a539e00b4204240f6d8416989a1e0c435d3053848595154b03d8e42e64644
60149faa751f305766569e80ec173f5ca4b39d049c706925a6796bbf662ce84f
6194213a09f571f0f547765bad1ca7cb5d943aa3115100050c7fb4a41da24c4e
6234e8752d64c03f6fa0f47ff7ef510b6370a17303851e3984d8ca0f94513c7d
6d0a16d294fad611993afde029ea68682f566347399ef149b57715b0e5eeb41a
711a89189adc9e08ad6e07ff577c1ef5e762fe5e305ceff1ac6cf861c24a5c7e
82563175b179a431fa484f332bf42833fbbcdb275297cbf622b08f921ca80915
88722473c7d6fad60f7bf52012eccdac15f3f8980e6eed75c5fd3d1b29279cb8
8a592adefdb1ef4f2135ba20bcb729bbe1ed2bebade89a86da334e15d80433e4
9a5e7728f198289f7db0a91f38d1914351b25b0b8465b70c39664b73aa051980
a5530b711ac91c90170869d0b4780b872e85bd67ec5a46851513c0fbc76e4b36
b0bd726a3362929e553f8d656ead086040bf90f7887ea1ec3389388f59dc05de
b39c395831be79c026a44b991d8607da3c082d797689779e0964499bc4bf5025
b7703280e8750b01868b412e940b55167a74820be8238c3a523be75cbe4e17f3
bbbdd9bcaa8265367c62e95b3eba7ea5919f26d8227243ba93a7650ae813b9d9
bc0090ac7930cfb5d0f4681454c067a21f8636c45b2129102622dd90c432de33
bc19e59650c274246b42c059bbf3ae0720b588dd987c9d1d04efbe9730e3f315
c0679fc908e6f65cf3c4d5624ea44a0d01d86bd086379535433e85aa569defff
cf57af5ddc83c27e26d12d3ec978cd4d9f04023da08c1d01d39797c7ad45b837
d2d881bf298918be4c06ea9460c13a9ed2dabbb8d58eb53d87c92c07d941bbbd
dcc1018b84f658affe7149c74421e188e76f19e7d09d47e846e1907e06a14c77
e219e54eff96eda67614b2f4ad4b50958e8d738b9fc036a4cbb18617cc452331
e3a72c56645c2b91c22ab41f1750a709f8b7702a7b613d1e0e371c3ef7d99565
e4e4778a9877173a43852c041d136e9be42ab8708bf08dcd6f7e1b7cfd0e9d3c
e88759133ebc683e06b1611286cfaf15a035e3a61292fda03f2309bd98c6034b
eb583742774f5036bcab811f7bf583ad449a549269ae8e59cc105d2a98a4e783
f593e9831624b839ac7f298464d833bb437a4455703bacf916e8c2672f576502
f7ec4b5b98e04d9e9d75a8854efb13744c3e46bf4164ddd34ac65cd5c9d7be3c
fbe39d72842b0969949ea269e615ddc74cd59508f40e9861dbd411f958f3afb5
fffe916816bf624b5d994ea5c29980562c088e029e78215e0df1079d39372fef
l
```

> overlay2 Storage Driver로 Layer들이 생성되어 있으며 각 디렉토리 이름은 Layer ID와 일치하지 않습니다.(Docker 1.10 이후 사실로 확인됨)

Container간 Layer 차이 확인

Ubuntu를 이용하여, `acme/my-base-image:1.0`  Base Image를 생성합니다.

```dockerfile
FROM ubuntu:16.10
COPY . /app
```

`acme/my-base-image:1.0` Image에 몇 가지 추가 Layer를 추가하여 새로운 Image를 생성합니다.

```dockerfile
FROM acme/my-base-image:1.0
CMD /app/hello.sh
```

두번째 Image에는 Base Image의 모든 Layer와 `CMD` 명령이 포함된 새 Layer 및 Read-Write Container Layer가 포함됩니다. Docker는 이미 Base Image의 모든 Layer를 가지고 있으므로, 다시 Pull할 필요가 없습니다. 두 Image는 공통된 Layer를 공유합니다.

두 Dockerfile에서 Image를 빌드하는 경우, `docker images` 및 `docker history`를 사용하여 공유 Layer의 암호화 ID가 동일한지 확인할 수 있습니다.

1. `cow-test/` 디렉토리를 생성합니다.

2. `cow-test/` 안에 다음과 같은 내용으로 `hello.sh`을 만듭니다.

   ``` shell
   #!/bin/sh
   echo "Hello world"
   ```

   파일을 저장하고 실행 가능한 상태로 변경합니다.

   ```sh
   chmod +x hello.sh
   ```

3. 위의 Base Image의 Dockerfile 내용을 `Dockerfile.base`라는 이름으로 생성합니다.

4. 위의 Image의 Dockerfile 내용을 `Dockerfile`라는 이름으로 생성합니다.

5. 생성된 파일들은 다음과 같습니다.

   ```shell
   /cow-test $ tree
   .
   ├── Dockerfile
   ├── Dockerfile.base
   └── hello.sh
   ```

6. `cow-test/`내에서 Dockerfile.base로 Base Image를 생성합니다.

   ```shell
   $ docker build -t acme/my-base-image:1.0 -f Dockerfile.base .
   Sending build context to Docker daemon  4.096kB
   Step 1/2 : FROM ubuntu:16.10
   16.10: Pulling from library/ubuntu
   dca7be20e546: Pull complete 
   40bca54f5968: Pull complete 
   61464f23390e: Pull complete 
   d99f0bcd5dc8: Pull complete 
   120db6f90955: Pull complete 
   Digest: sha256:8dc9652808dc091400d7d5983949043a9f9c7132b15c14814275d25f94bca18a
   Status: Downloaded newer image for ubuntu:16.10
    ---> 7d3f705d307c
   Step 2/2 : COPY . /app
    ---> 71a49d51b7cb
   Successfully built 71a49d51b7cb
   Successfully tagged acme/my-base-image:1.0
   ```

7. Dockerfile로 두번째 Image를 생성합니다.

   ```shell
   $ docker build -t acme/my-final-image:1.0 -f Dockerfile .
   Sending build context to Docker daemon  4.096kB
   Step 1/2 : FROM acme/my-base-image:1.0
    ---> 71a49d51b7cb
   Step 2/2 : CMD /app/hello.sh
    ---> Running in 989ec753c09f
    ---> e242fd50d9e6
   Removing intermediate container 989ec753c09f
   Successfully built e242fd50d9e6
   Successfully tagged acme/my-final-image:1.0
   ```

   > 6번과 7번의 과정을 비교해보면, 이미 생성된 Layer들이 있기 때문에 7번 과정에서는 빠르게 Image가 생성된다.

8. `docker images`로 생성된 두 Image의 크기를 확인할 수 있습니다.

   ```shell
   $ docker images

   REPOSITORY            TAG                 IMAGE ID            CREATED              SIZE
   acme/my-final-image   1.0                 e242fd50d9e6        34 seconds ago       107MB
   acme/my-base-image    1.0                 71a49d51b7cb        About a minute ago   107MB
   <none>                <none>              59c82072bc1b        22 hours ago         1.86GB
   <none>                <none>              0a69fdd3eae7        7 days ago           131MB
   <none>                <none>              ac2743c9efcc        7 days ago           131MB
   <none>                <none>              f57c2f4774e8        7 days ago           129MB
   <none>                <none>              eb0d577762c9        7 days ago           129MB
   <none>                <none>              53a9298ee024        8 days ago           129MB
   redis                 <none>              1fb7b6c8c0d0        9 days ago           107MB
   portainer/portainer   <none>              a64783906447        2 weeks ago          11MB
   ubuntu                16.10               7d3f705d307c        3 months ago         107MB
   ubuntu                15.04               d1b55fd07600        21 months ago        131MB
   ```

9. `docker history IMAGE`로 각 Image를 구성하는 Layer를 확인할 수 있습니다.

   Base Image:

   ```shell
   $ docker history 71a49d51b7cb
   IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
   71a49d51b7cb        2 minutes ago       /bin/sh -c #(nop) COPY dir:feccc62f3388b3d...   107B                
   7d3f705d307c        3 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
   <missing>           3 months ago        /bin/sh -c mkdir -p /run/systemd && echo '...   7B                  
   <missing>           3 months ago        /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   2.78kB              
   <missing>           3 months ago        /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
   <missing>           3 months ago        /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745B                
   <missing>           3 months ago        /bin/sh -c #(nop) ADD file:6cd9e0a52cd1520...   107MB  
   ```

   두번째 Image:

   ```shell
   $ docker history e242fd50d9e6
   IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
   e242fd50d9e6        2 minutes ago       /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/a...   0B                  
   71a49d51b7cb        3 minutes ago       /bin/sh -c #(nop) COPY dir:feccc62f3388b3d...   107B                
   7d3f705d307c        3 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
   <missing>           3 months ago        /bin/sh -c mkdir -p /run/systemd && echo '...   7B                  
   <missing>           3 months ago        /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   2.78kB              
   <missing>           3 months ago        /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
   <missing>           3 months ago        /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745B                
   <missing>           3 months ago        /bin/sh -c #(nop) ADD file:6cd9e0a52cd1520...   107MB         
   ```

   두번째 Image의 맨 위 Layer를 제외한, 모든 Layer가 동일한 것을 확인할 수 있습니다. 그 외의 모든 Layer는 두 Image간에 공유되며, `/var/lib/docker/<storage_driver>/`에 한번만 저장됩니다. 새로 생성된 Layer는 파일을 변경하지 않고, `CMD` 명령만 실행하기 때문에 실제로 아무런 공간도 차지하지 않습니다.

   > **참고:** `docker history`안의 `<missing>`행은 해당 Layer가 다른 시스템에 의해 작성되었으며, Local에서 사용할 수 없음을 나타낸 것입니다.

10. 결과

  ![스크린샷 2017-10-19 오후 5.52.25](../../../스크린샷 2017-10-19 오후 5.52.25.png)

  > 생성된 Image들이 Layer별로 분리되어 위와 같이 저장되었습니다.

### 복사는 Container를 효율적으로 만듭니다

Container를 시작할 때, Thin Writable Container Layer가 다른 Layer 위에 추가됩니다. Container가 파일 시스템에 대해 수행한 모든 변경 사항이 여기에 저장됩니다. Container가 변경하지 않는 파일은 Writable Layer에 복사되지 않습니다. 즉, Writable Layer는 가능한 작게 유지되고 있음을 의미합니다.

Container의 기존 파일이 수정되면 Storage Driver는 copy-on-write를 수행합니다. 관련된 구체적인 단계는 특정 Storage Driver에 따라 다릅니다. `aufs` 드라이버와 `overlay` 및 `overlay2` 드라이버를 기본으로, copy-on-write의 대략적인 순서는 다음과 같습니다:

- Image Layer를 검색하여 업데이트할 파일을 찾습니다. 이 과정은 최신 Layer에서 시작하여, 한 번에 한 Layer씩 Base Layer로 내려가며 작업합니다. 발견되면, 향후 작업 속도를 높이기 위해 캐시에 추가됩니다.
- 찾은 파일의 첫번째 복사본에 대해, 파일을 Container의 Writable Layer에 복사하는 `copy_up`을 수행합니다.
- 해당 파일의 복사본이 수정되면, Container는 하위 Layer에 있는 파일의 Read-only 복사본을 볼 수 없습니다.

Btrfs, ZFS 및 다른 드라이버는 copy-on-write를 다르게 처리합니다. 

많은 양의 데이터를 쓰는 Container는 그렇지 않은 Container보다 더 많은 공간을 사용합니다. 이는 대부분의 쓰기 작업이 Container의 최상위에 있는 Writable Layer에서 새로운 공간을 사용하기 때문입니다.

> **참고:** 쓰기가 많은 응용 프로그램의 경우, Container에 데이터를 저장하지 마십시오. 대신 Docker Volume을 사용하십시오. Docker 볼륨은 실행중이 Container와 독립적이며, I/O을 효율적으로 할 수 있도록 설계되었습니다. 또한, Volume을 Container들간 공유가 가능하며, Container의 Writable Layer 크기는 증가하지 않습니다.

`copy_up`은 성능 오버헤드가 발생할 수 있습니다. 이 오버헤드는 사용 중인 Storage Driver에 따라 다릅니다. 대용량 파일, 많은 Layer 및 Deep Directory Tree들이 오버헤드를 더 두드러지게 만들 수 있습니다. 각 `copy_up` 동작이 주어진 파일이 처음 수정 될 때만 발생한다는 사실은, 이러한 오버헤드를 완화시킵니다.

copy-on-write가 작동 방식을 확인하기 위해, `acme/my-final-image:1.0` Image 기반의 Container 5개를 동작시키고 차지하는 공간을 확인해보겠습니다.

> **참고:** 이 절차는 Mac 용 Docker 또는 Windows 용 Docker에서 작동하지 않습니다.

1. Docker Host의 터미널에서 `docker run`을 실행하십시오. 각 Container의 ID가 출력됩니다.
   ```shell
   $ docker run -dit --name my_container_1 acme/my-final-image:1.0 bash \
   && docker run -dit --name my_container_2 acme/my-final-image:1.0 bash \
   && docker run -dit --name my_container_3 acme/my-final-image:1.0 bash \
   && docker run -dit --name my_container_4 acme/my-final-image:1.0 bash \
   && docker run -dit --name my_container_5 acme/my-final-image:1.0 bash

   285b36287b5e35398149dc484a0a6cd0c770459fd5a35d0ebede461143b64d66
   b2a14a58e3b6f6d94a8e82c070761c3416069b5505b99be8a8d1c9e6d6ac2456
   4001e7dd703c00bafebfcf3452dbc19555d7353096fc8b1013dc811f873dddd2
   0caa7313f99dc4fdc68d79f1625fc33d457f63f2629085456759a610676ecd8b
   5d4c529606f97c9be869d066c95882be9e731329889d8ecad75f04b668a8d39f
   ```

2. `docker ps -s`를 실행하면, 5개 Container가 실행되었는지 확인할 수 있습니다.

   ```shell
   $ docker ps -s    
   CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS              PORTS               NAMES               SIZE
   5d4c529606f9        acme/my-final-image:1.0   "bash"              3 minutes ago       Up 3 minutes                            my_container_5      0B (virtual 107MB)
   0caa7313f99d        acme/my-final-image:1.0   "bash"              3 minutes ago       Up 3 minutes                            my_container_4      0B (virtual 107MB)
   4001e7dd703c        acme/my-final-image:1.0   "bash"              3 minutes ago       Up 3 minutes                            my_container_3      0B (virtual 107MB)
   b2a14a58e3b6        acme/my-final-image:1.0   "bash"              3 minutes ago       Up 3 minutes                            my_container_2      0B (virtual 107MB)
   285b36287b5e        acme/my-final-image:1.0   "bash"              3 minutes ago       Up 3 minutes                            my_container_1      0B (virtual 107MB)
   ```

3. 로컬에 저장된 Container의 목록을 확인하십시오.

   ```shell
   /var # ls /var/lib/docker/containers
   0caa7313f99dc4fdc68d79f1625fc33d457f63f2629085456759a610676ecd8b
   285b36287b5e35398149dc484a0a6cd0c770459fd5a35d0ebede461143b64d66
   4001e7dd703c00bafebfcf3452dbc19555d7353096fc8b1013dc811f873dddd2
   5d4c529606f97c9be869d066c95882be9e731329889d8ecad75f04b668a8d39f
   b2a14a58e3b6f6d94a8e82c070761c3416069b5505b99be8a8d1c9e6d6ac2456
   ```

4. 저장된 실제 크기를 확인하십시오:

   ```shell
   /var # du -sh /var/lib/docker/containers/*
   32.0K   /var/lib/docker/containers/0caa7313f99dc4fdc68d79f1625fc33d457f63f2629085456759a610676ecd8b
   32.0K   /var/lib/docker/containers/285b36287b5e35398149dc484a0a6cd0c770459fd5a35d0ebede461143b64d66
   32.0K   /var/lib/docker/containers/4001e7dd703c00bafebfcf3452dbc19555d7353096fc8b1013dc811f873dddd2
   32.0K   /var/lib/docker/containers/5d4c529606f97c9be869d066c95882be9e731329889d8ecad75f04b668a8d39f
   32.0K   /var/lib/docker/containers/b2a14a58e3b6f6d94a8e82c070761c3416069b5505b99be8a8d1c9e6d6ac2456
   ```

   각각의 Container들은 파일 시스템상의 32KB 공간 만을 차지합니다.

copy-on-write는 공간을 절약할 뿐만 아니라, Container의 기동 시간을 줄여줍니다. Container(또는 동일한 Image에서 여러 Container)를 시작하려할 때, Docker는 단지 Writable Container Layer만 만들면 됩니다.

Docker가 새 Container를 시작할 때마다 기본 Image Stack의 전체 복사본을 만들어야 하는 경우, Container 기동 시간과 사용하는 Disk 공간은 크게 늘어납니다. 이는 Virtual Machine이 동작하는 방식과 유사하며, Virtual Machine당 하나 이상의 Virtual Disk가 있는 것과 같습니다.

## Data Volume과 Storage Driver

Container가 삭제되면, **데이터 볼륨**에 저장되지 않은 Container에 쓰여진 모든 Data가 Container와 함께 삭제됩니다.

Data Volume은 Container에 직접 마운트된 Docker Host의 파일 시스템에 있는 디렉토리 또는 파일입니다. Data Volume은 Storage Driver에 의해 제어되지 않습니다. Data Volume에 대한 읽기 및 쓰기는 Storage Driver를 우회하고 기본 Host 속도로 작동합니다. 원하는 수의 Data Volume을 Container에 마운트할 수 있습니다. 여러 Container들은 하나 이상의 Data Volume을 공유할 수도 있습니다.

아래 다이어그램은 두 개의 Container를 실행하는 단일 Docker Host를 보여줍니다. 각 Container는 Docker Host의 로컬 저장소 영역 (`/var/lib/docker/...`) 내에 있는 자체 주소 공간 내에 있습니다. Docker Host의 `/data`에 단일 공유 Data Volume이 있습니다. 이것은 두 Container에 직접 마운트됩니다.

![Volume](https://docs.docker.com/engine/userguide/storagedriver/images/shared-volume.jpg)

```shell
$ ls data

# MySql DB Container의 Data Volume
dtlabs-mysql
```

Data Volume은 Docker Host의 로컬 스토리지 영역 외부에 있으므로, Storage Driver의 제어로부터 독립성이 더욱 강화됩니다. Container가 삭제되면, Data Volume에 저장된 모든 Data는 Docker Host에 유지됩니다.

## 샘플 텍스트

*기울임꼴*

**굵기**

_기울임꼴_

__굵게__

## 샘플 표
| 항목              | 내용                        | 비고                         |
|-----------------|---------------------------|----------------------------|
| /docs           | 프로젝트 홈                    |                            |
| archetypes      | 컨텐츠 기본 구조 정의              | default.md 파일에서 마크다운 구조 설정 |
| content/posts   | 블로그에 올라갈 마크다운 파일 위치       |                            |
| data            | 태그, 카테고리, 저자 등 기타 항목 정의   |                            |
| layouts, static | 블로그의 템플릿 및 정적 리소스 위치      | 현재는 테마를 사용하기 때문에 사용안함      |
| public          | 블로그 빌드 타켓 폴더              | gh-pages 브랜치에 Push될 결과물    |
| themes          | 사용할 Hugo 테마 위치            |                            |
| .editorconfig   | 프로젝트 내의 코딩 컨벤션 설정 파일      |                            |
| .travis.yml     | 빌드/배포를 위한 Travis CI 설정 파일 |                            |
| config.toml     | Hugo 블로그 전체 설정 파일         |                            |

Cat
: Fluffy animal everyone likes

Internet
: Vector of transmission for pictures of cats

This is a footnote.[^1]

[^1]: the footnote text.

~~Vector of transmission for pictures of cats~~

4/5

asdf
---

1--2
