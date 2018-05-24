---
date: "2018-03-09T08:23:38+09:00"
title: "docker build & push"
authors: ["1000jaeh"]
series: ["docker"]
categories:
  - posts
tags:
  - docker
  - image
  - container
  - dockerfile
draft: false
---
Docker 환경에 Service 및 Application을 구동시키 위한 전체적인 과정은 다음과 같습니다.

{{< mermaid >}}
graph BT;
  A[Dockerfile] -->|build| B[Images]
    B -->|push| C[Docker Registry]
    C -->|pull| B
    subgraph Local Docker Instance
      B -->|Run| D[Container]
    end
{{< /mermaid >}}

지금까지는, Docker Registry(Docker Hub)에 배포되어 있는 있는 Image를 이용해 Container를 구동시켰습니다. 이제부터는 위에서 정리한 과정대로, Dockerfile로 Image를 생성하고 다른 Machine에서 배포된 Image로 Container를 실행시켜 보겠습니다.

{{% notice note %}}
본 예제는 [About storage drivers](https://docs.docker.com/storage/storagedriver/)에 있는 예제를 참고하여 진행했습니다.
{{% /notice %}}

## 신규 Image 생성에서 배포까지

### Dockerfile 작성

_**image-build-test**_ 폴더를 생성한 뒤, 다음의 내용이 포함된 **Dockerfile**을 생성합니다.

```dockerfile
FROM ubuntu:16.10
COPY . /app
```

_**image-build-test**_ 폴더의 내부 구조는 다음과 같습니다.

```bash
/image-build-test $ tree
.
└── Dockerfile

0 directories, 1 file
```

### Image Build

_**image-build-test**_ 폴더 내에서 생성한 Dockerfile로 `${username}/my-base-image:1.0`이란 이름의 Image를 Build합니다.

{{% notice info %}}
여기서, `${username}`는 Docker에 로그인 가능한 계정입니다.
{{% /notice %}}

```bash
/image-build-test $ docker build -t ${username}/my-base-image:1.0 -f Dockerfile .
Sending build context to Docker daemon  11.26kB
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
 ---> 6087ccdf7873
Successfully built 6087ccdf7873
Successfully tagged ${username}/my-base-image:1.0
```

처음 Image를 Build하게 되면, Local Instance에 저장된 Base Image(여기서는 Ubuntu Image입니다)가 존재하지 않기 때문에, Docker Registry(Docker Hub)로부터 Pull을 받습니다. 그 후, Dockerfile의 **COPY**명령어가 새로운 Layer로 생성되고, `${username}/my-base-image:1.0` Image가 Build됩니다.

생성된 Image를 확인해 보겠습니다.

```bash
/image-build-test $ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
${username}/my-base-image   1.0                 6087ccdf7873        22 seconds ago      107MB
ubuntu                      16.10               7d3f705d307c        8 months ago        107MB
```

### Container를 Run

방금 생성한 Image를 Local에서 실행시켜 보겠습니다. 정상적으로 실행된다면, 아래와 같이 **Container ID**가 나타나고 `docker ps`로 실행중인 해당 Container를 목록에서 확인할 수 있습니다.

```bash
/image-build-test $ docker run -dit --name ${username}_container ${username}/my-base-image:1.0 bash
8517d3ff5b8a4233ea716190beb73bfdbb4401afee656dff7d694307d304b2e4

/image-build-test $ docker ps -s
CONTAINER ID        IMAGE                           COMMAND             CREATED             STATUS              PORTS               NAMES                   SIZE
8517d3ff5b8a        ${username}/my-base-image:1.0   "bash"              3 seconds ago       Up 4 seconds                            ${username}_container   0B (virtual 107MB)
```

### Image를 Docker Hub에 배포

이제는 생성한 Image를 Docker Registry에 배포해 보겠습니다. 따로, Docker Registry를 구성하지 않았기 때문에 배포되는 장소는 Docker Hub이며, 정상적으로 배포하기 위해서 먼저 Login을 하겠습니다.

```bash
/image-build-test $ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username (${username}): ${username}
Password:
Login Succeeded
```

로그인이 정상적으로 되었다면 `docker push`로 생성한 Image를 Docker Registry에 등록하겠습니다.

```bash
/image-build-test $ docker push ${username}/my-base-image:1.0
The push refers to repository [docker.io/${username}/my-base-image]
1b9e33c4403d: Pushed
fcc11235a441: Mounted from library/ubuntu
0ba4c05a8843: Mounted from library/ubuntu
47a3ebbaa644: Mounted from library/ubuntu
31eed92f5a23: Mounted from library/ubuntu
57145c01eb80: Mounted from library/ubuntu
1.0: digest: sha256:baa607ec007033e47ad2d26c5f38da7d95023efad2a76da2aa857e1e4d5a756f size: 1564
```

Console에 찍힌 로그를 보면, Unbuntu Image는 이미 Docker Registry에 존재하기 때문에 Mount만 되고, 그 외에 생성한 Layer만 추가로 Push되는 것을 볼 수 있습니다.

{{% notice info %}}
[Docker Hub](https://hub.docker.com/)에서 Push된 Image를 확인할 수 있습니다.
{{% /notice %}}

## Hello World를 만들어 보자

처음에 `hello-world`를 실행시켜 봤었습니다. 지금부터는 직접 구현해 보겠습니다.

### Dockerfile 변경

기 생성된 Dockerfile을  **Dockerfile.base**로 변경 한 후, `${username}/my-base-image:1.0` Image에 `CMD` Layer가 추가된 Image를 생성하는 **Dockerfile**을 새로 생성합니다.

```dockerfile
FROM ${username}/my-base-image:1.0
CMD /app/hello.sh
```

다음과 같은 내용으로 **hello.sh**를 생성합니다.

```sh
#!/bin/sh
echo "Hello world"
```

**hello.sh** 파일을 저장하고 실행가능한 상태로 변경합니다.

```bash
/image-build-test $ chmod +x hello.sh
```

변경된 폴더 구조는 다음과 같습니다.

```bash
/image-build-test $ tree
.
├── Dockerfile
├── Dockerfile.base
└── hello.sh

0 directories, 3 files
```

### Hello World Image Build

변경된 Dockerfile로 `${username}/my-final-image:1.0`이란 이름의 Image를 Build합니다.

```bash
/image-build-test $ docker build -t ${username}/my-final-image:1.0 -f Dockerfile .
Sending build context to Docker daemon  11.26kB
Step 1/2 : FROM ${username}/my-base-image:1.0
 ---> 6087ccdf7873
Step 2/2 : CMD /app/hello.sh
 ---> Running in 8b7b0383d636
Removing intermediate container 8b7b0383d636
 ---> c0d79183f707
Successfully built c0d79183f707
Successfully tagged ${username}/my-final-image:1.0
```

처음에 Build했을 때 보다는 월등히 빠르게 Build되는 것을 알 수 있습니다. `${username}/my-final-image:1.0`의 Base Image가 이미 Local에 존재하기 때문에 `CMD`에 해당하는 Layer만 생성하면 되기 때문입니다. 

생성된 Image를 확인합니다.

```bash
/image-build-test $ docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
${username}/my-final-image    1.0                 c0d79183f707        14 seconds ago      107MB
${username}/my-base-image     1.0                 6087ccdf7873        4 minutes ago       107MB
ubuntu                        16.10               7d3f705d307c        8 months ago        107MB
```

`docker history [OPTIONS] IMAGE`로 각 Image를 구성하는 Layer를 한번 확인해 보겠습니다. 두 Image 사이에서 공유하는 Layer는 암호화된 ID가 동일하다는 것을 확인할 수 있습니다.

{{% center %}}
**${username}/my-base-image:1.0**
{{% /center %}}

```bash
/image-build-test $ docker history ${username}/my-base-image:1.0
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
6087ccdf7873        4 minutes ago       /bin/sh -c #(nop) COPY dir:d9ce0b6a08037dcd1…   6.26kB
7d3f705d307c        8 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>           8 months ago        /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B
<missing>           8 months ago        /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$…   2.78kB
<missing>           8 months ago        /bin/sh -c rm -rf /var/lib/apt/lists/*          0B
<missing>           8 months ago        /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   745B
<missing>           8 months ago        /bin/sh -c #(nop) ADD file:6cd9e0a52cd152000…   107MB
```

{{% center %}}
**${username}/my-final-image:1.0**
{{% /center %}}

``` bash
/image-build-test $ docker history ${username}/my-final-image:1.0
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
c0d79183f707        50 seconds ago      /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/app…   0B
6087ccdf7873        5 minutes ago       /bin/sh -c #(nop) COPY dir:d9ce0b6a08037dcd1…   6.26kB
7d3f705d307c        8 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>           8 months ago        /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B
<missing>           8 months ago        /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$…   2.78kB
<missing>           8 months ago        /bin/sh -c rm -rf /var/lib/apt/lists/*          0B
<missing>           8 months ago        /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   745B
<missing>           8 months ago        /bin/sh -c #(nop) ADD file:6cd9e0a52cd152000…   107MB
```

### 다시 Push

`${username}/my-final-image:1.0` Image를 Docker Registry에 Push합니다. Docker Registry에는 ${username}/my-base-image:1.0 Image의 Layer들이 이미 존재하고 있기 때문에, 추가된 Layer(Dockerfile에 추가된 `CMD`)만 Push됩니다.

```bash
/image-build-test $ docker push ${username}/my-final-image:1.0
The push refers to repository [docker.io/${username}/my-final-image]
1b9e33c4403d: Mounted from ${username}/my-base-image
fcc11235a441: Mounted from ${username}/my-base-image
0ba4c05a8843: Mounted from ${username}/my-base-image
47a3ebbaa644: Mounted from ${username}/my-base-image
31eed92f5a23: Mounted from ${username}/my-base-image
57145c01eb80: Mounted from ${username}/my-base-image
1.0: digest: sha256:cd26aa2cb82dbecdb064b524e3f3796493b106fd2dae4a9a2320856c46288dad size: 1564
```

## 다른 곳에서 Hello World를 실행해보자

새로운 환경에서 지금 만든 Image를 Container로 실행시켜 보겠습니다. 그 전에 `docker-machine create`로 새로운 Docker 환경을 구성하겠습니다.

```bash
/image-build-test $ docker-machine create ${username}
Running pre-create checks...
(${username}) Default Boot2Docker ISO is out-of-date, downloading the latest release...
(${username}) Latest release for github.com/boot2docker/boot2docker is v18.03.0-ce
(${username}) Downloading /Users/${username}/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.03.0-ce/boot2docker.iso...
(${username}) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(${username}) Copying /Users/${username}/.docker/machine/cache/boot2docker.iso to /Users/${username}/.docker/machine/machines/${username}/boot2docker.iso...
(${username}) Creating VirtualBox VM...
(${username}) Creating SSH key...
(${username}) Starting the VM...
(${username}) Check network to re-create if needed...
(${username}) Waiting for an IP...
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
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env ${username}
```

새로 구성한 Docker 환경에 `docker-machine ssh`로 접속하겠습니다.

```bash
/image-build-test $ docker-machine ssh ${username}
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
Boot2Docker version 18.03.0-ce, build HEAD : 404ee40 - Thu Mar 22 17:12:23 UTC 2018
Docker version 18.03.0-ce, build 0520e24
docker@${username}:~$
```

지금 만든 Image를 Pull받고,..

```bash
docker@${username}:~$ docker pull ${username}/my-final-image:1.0
1.0: Pulling from ${username}/my-final-image
dca7be20e546: Pull complete
40bca54f5968: Pull complete
61464f23390e: Pull complete
d99f0bcd5dc8: Pull complete
120db6f90955: Pull complete
cb2cf9c669f2: Pull complete
Digest: sha256:cd26aa2cb82dbecdb064b524e3f3796493b106fd2dae4a9a2320856c46288dad
Status: Downloaded newer image for ${username}/my-final-image:1.0
```

새로 구성된 환경이라 Image의 모든 Layer들을 내려받습니다. 그리고, `docker run`을 실행하면 !!!

```bash
docker@${username}:~$ docker run ${username}/my-final-image:1.0
Hello world
```

처음에 정리했던 Build - Push - Pull - Run의 과정으로 Container가 실행되는 것을 확인할 수 있었습니다. Docker Engine이 설치된 어떤 환경에서도 지금 만든 **Hello world**는 실행될 것입니다. 여기서 더 나아가 Hello world는 **다양한 메시지를 입력받거나 혹은, 저장되어 있는 메시지들을 출력하는 Application**으로 발전할 수도 있습니다. 그렇다면, 이 Application의 기능이 Upgrade되어, 새로운 버전의 Image를 만들고 새로운 Container를 기동시켜야 한다면, 변경된 내용(입력받거나 혹은 저장된 메시지들)들은 어떻게 될까요?

우리는 이전에 Image와 Container의 Layer 구조를 살펴보면서, 해당 Container에 읽기/쓰기가 가능한 Layer내에 변경된 내용들이 저장된다는 것을 알고 있습니다. 그리고, 이 데이터들은 해당 Container와 같이 소멸된다는 것도 알고 있습니다. 그럼, Container가 소멸되었다가 새로 생성되어도, 이전의 데이터들을 유지하여 사용할 방법은 없을까요? 이 다음에는 Docker의 Volume을 이용하여, 데이터를 보존하는 방법에 대해서 확인해볼 필요가 있겠네요.
