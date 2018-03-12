---
date: "2018-03-09T08:23:38+09:00"
title: "Docker-Flow"
authors: ["1000jaeh"]
categories:
  - posts
tags:
  - docker
  - image
  - container
  - dockerfile
cover:
  image: "../images/docker-official.svg"
draft: true
---
## 신규 Image 생성 및 배포

Docker의 신규 Image 생성 및 배포 과정은 다음과 같습니다.

![](https://dzone.com/storage/temp/5288806-docker-stages.png)

### Dockerfile 작성

**docker-flow-test** 폴더를 생성한 뒤, 다음의 내용이 포함된 **Dockerfile**을 생성합니다.

``` text
FROM ubuntu:latest
COPY . /app
```

**docker-flow-test** 구조는 다음과 같습니다.

``` bash
/docker-flow-test $ tree
.
└── Dockerfile
```

### Image Build

**docker-flow-test** 폴더 내에서 생성한 Dockerfile로
**acme/my-base-image:1.0**이란 이름의 Image를 Build합니다.

``` bash
/docker-flow-test $ docker build -t acme/my-base-image:1.0 -f Dockerfile .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM ubuntu:latest
 ---> 747cb2d60bbe
Step 2/2 : COPY . /app
 ---> 1ff6e3572bac
Successfully built 1ff6e3572bac
Successfully tagged acme/my-base-image:1.0
```

생성된 Image를 확인합니다.

``` bash
/docker-flow-test $ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
acme/my-base-image   1.0                 1ff6e3572bac        55 seconds ago      122MB
ubuntu               latest              747cb2d60bbe        2 weeks ago         122MB
$ docker run -dit --name my_container acme/my-base-image:1.0 bash
931f7b08a348f282969eaf42952fea86dd1ccaa82b581b7108ce6aebd094e307

$ docker ps -s
CONTAINER ID        IMAGE                    COMMAND             CREATED                  STATUS              PORTS               NAMES               SIZE
931f7b08a348        acme/my-base-image:1.0   "bash"              Less than a second ago   Up 2 seconds                            my_container        0B (virtual 122MB)
```

### Container 기동

Local에서 생성한 Image로 부터 Container를 기동하고, 정상적으로
기동되었는지 확인합니다.

``` bash
/docker-flow-test $ docker run -dit --name my_container acme/my-base-image:1.0 bash
e5231f8509329acdfe2645c541afb13eba6dc2a61e001fc750d77a4f521dbe14

/docker-flow-test $ docker ps -s
CONTAINER ID        IMAGE                    COMMAND             CREATED                  STATUS              PORTS               NAMES               SIZE
e5231f850932        acme/my-base-image:1.0   "bash"              Less than a second ago   Up 2 seconds                            my_container        0B (virtual 122MB)
```

### Image Commit (작성중)

commit 합니다.

### Image Push (작성중)

이상이 없는 Image를 Docker Registry에 등록합니다. 

## Image 수정 후, 재 배포

![](http://pointful.github.io/docker-intro/docker-img/changes-and-updates.png)

### Dockerfile 변경

기 생성된 Dockerfile을  **Dockerfile.base**로 변경 한 후, acme/my-base-image:1.0 Image에 CMD Layer가 추가된 Image를 생성하는 **Dockerfile**을 새로 생성합니다.

``` text
FROM acme/my-base-image:1.0
CMD /app/hello.sh
```

다음과 같은 내용으로 **hello.sh**를 생성합니다.

``` bash
#!/bin/sh
echo "Hello world"
```

파일을 저장하고 실행가능한 상태로 변경합니다.

``` bash
/docker-flow-test $ chmod +x hello.sh
```

변경된  폴더 구조는 다음과 같습니다.

``` bash
/docker-flow-test $ tree
.
├── Dockerfile
├── Dockerfile.base
└── hello.sh
```

### Image Build

변경된 Dockerfile로 **acme/my-final-image:1.0**이란 이름의 Image를 Build합니다.

``` bash
/docker-flow-test $ docker build -t acme/my-final-image:1.0 -f Dockerfile .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM acme/my-base-image:1.0
 ---> 1ff6e3572bac
Step 2/2 : CMD /app/hello.sh
 ---> Running in 292debab576f
 ---> ebd5179a37b3
Removing intermediate container 292debab576f
Successfully built ebd5179a37b3
Successfully tagged acme/my-final-image:1.0
```

생성된 Image를 확인합니다.

``` bash
/docker-flow-test $ docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
acme/my-final-image   1.0                 ebd5179a37b3        26 seconds ago      122MB
acme/my-base-image    1.0                 1ff6e3572bac        5 minutes ago       122MB
ubuntu                latest              747cb2d60bbe        2 weeks ago         122MB
```

`docker history \[OPTIONS\] IMAGE`로 각 Image를 구성하는 Layer를 확인합니다. 공유하는 Layer의 암호화 ID가 동일한지 확인할 수 있습니다.

- **acme/my-base-image:1.0**

    ``` bash
    /docker-flow-test $ docker history 1ff6e3572bac
    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    1ff6e3572bac        7 minutes ago       /bin/sh -c #(nop) COPY dir:b623b1c4cd3c38e...   31B                 
    747cb2d60bbe        2 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
    <missing>           2 weeks ago         /bin/sh -c mkdir -p /run/systemd && echo '...   7B                  
    <missing>           2 weeks ago         /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   2.76kB              
    <missing>           2 weeks ago         /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
    <missing>           2 weeks ago         /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745B                
    <missing>           2 weeks ago         /bin/sh -c #(nop) ADD file:5b334adf9d9a225...   122MB     
    ```

- **acme/my-final-image:1.0**

    ``` bash
    /docker-flow-test $ docker history ebd5179a37b3
    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    ebd5179a37b3        3 minutes ago       /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/a...   0B                  
    1ff6e3572bac        8 minutes ago       /bin/sh -c #(nop) COPY dir:b623b1c4cd3c38e...   31B                 
    747cb2d60bbe        2 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
    <missing>           2 weeks ago         /bin/sh -c mkdir -p /run/systemd && echo '...   7B                  
    <missing>           2 weeks ago         /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   2.76kB              
    <missing>           2 weeks ago         /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
    <missing>           2 weeks ago         /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745B                
    <missing>           2 weeks ago         /bin/sh -c #(nop) ADD file:5b334adf9d9a225...   122MB   
    ```

### Image Push

**acme/my-final-image:1.0 Image**를 Docker Registry에 Push합니다. Docker Registry에는 acme/my-base-image:1.0 Image의 Layer들이 이미 존재하고 있기 때문에, 변경된 Layer(Dockerfile에 추가된 `CMD`)만 Push됩니다.

### Container Update(작성중)

기동중인 Container를 Update합니다.
