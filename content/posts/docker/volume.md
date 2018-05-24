---
date: "2018-03-09T08:25:25+09:00"
title: "Volume을 활용한 Data 관리"
authors: ["1000jaeh"]
series: ["docker"]
categories:
  - posts
tags:
  - docker
  - volume
draft: false
---

우리는 Container의 Writable Layer에 Data를 저장할 수 있다는 것을 알고 있습니다. 하지만, 여기에는 몇 가지 문제점이 존재합니다.

1. Container가 삭제되면 Data도 같이 삭제됩니다. 또한, 다른 프로세스에서 Container에 저장된 Data를 사용하기 어렵습니다.
2. Container의 Writable Layer에는 Container가 실행 중인 Host Machine과 밀접하게 연결됩니다. 따라서, Data를 다른 곳으로 쉽게 옮길 수 없습니다.
3. Container의 Writable Layer에 Data를 저장하기 위해서는 File System을 관리하는 Storage Driver가 필요합니다. Storage Driver는 Linux 커널을 사용하여 공용 File System을 제공합니다. 이 기능은 Host File System에 직접 쓰는 `data volume`보다 성능이 떨어집니다.

Docker는 Data를 안전하게 존속시킬 수 있는 방식으로 `volume, bind mounts, tmpfs`의 3가지 방식을 제공합니다(어떤 것을 사용해야할 지 모를 때는 `volume`를 사용하시기 바랍니다). 아래는 Container의 Data 관리 방식들과 사용 사례에 대해서 자세히 살펴보도록 하겠습니다.

## 올바른 Mount 유형 선택 방법

어떤 유형의 Mount를 사용하든, Data는 Container 내에서 동일하게 보이며, Container File System의 폴더나 개별적인 파일들로 표시됩니다. 올바른 Mount유형을 선택할 때 기준이 될 `volume, bind mounts, tmpfs mount`간의 가장 큰 차이점은, Data가 Docker Host내에서 ***어디에 존재***하는지 입니다.

{{% center %}}
![types of mounts and where they live on the Dockerhost](../images/types-of-mounts-volume.png)[출처: Docker Docs - Use volumes](https://docs.docker.com/storage/volumes/)
{{% /center %}}

### Data 저장을 위한 최선의 선택

- `volume`는 Docker(Linux에서는 `/var/lib/docker/volume/`)가 관리하는 Host File System의 일부에 Data가 저장됩니다.
- Non-Docker 프로세스들이 File System의 해당 부분을 수정해서는 안됩니다.
- Docker에서 Data를 존속시킬 수 있는 **Best한 방법**입니다.

### Host의 File System 원하는 곳에 저장

- `bind mount`는 Data가 Host System의 어디에든지 저장될 수 있습니다.
- 저장되는 Data는 System File이거나 Directory일 수 있습니다.
- Docker Host 또는 Docker Container의 Non-Docker 프로세서들이 언제든지 저장된 Data를 수정할 수 있습니다.

### Host System의 Memory에 저장

- `tmpfs mount`는 Host System의 Memory에만 Data가 저장되며, 절대로 Host의 File System에는 저장되지 않습니다.

## Mount 유형

각 Mount 유형에 대해서 자세히 살펴보겠습니다.

### volume

- Docker가 생성하고 관리하는 방식입니다. `docker volume create` 명령을 사용하여 명시적으로 voluems를 생성하거나, Container나 Service 생성 중에 `volume`을 생성할 수 있습니다.
- `volume`이 생성되면, Data는 Docker Host의 디렉토리에 저장됩니다. 해당 `volume`을 Container에 Mount하면, Host의 디렉토리가 Mount가 됩니다. 이는 `volume`이 Docker에 의해 관리되고 Host System과 분리된다는 점을 제외한다면, `bind mount`와 유사하게 동작합니다.
- 동시에 `volume`을 여러 Container에 Mount할 수 있습니다.
- 실행 중인 Container가 `volume`을 사용하지 않아도, 해당 `volume`은 Docker에서 계속 사용할 수 있으며, 자동으로 삭제되지 않습니다.
- `docker volume prune`을 사용하여 사용하지 않는 `volume`을 정리할 수 있습니다.
- `volume`을 Mount할 때, 이름을 명시적으로 지정하여 사용할 수 있으며, 익명으로도 사용할 수 있습니다.
- 익명 `volume`은 처음 Mount될 때 명시적으로 이름이 부여되지 않기 때문에, Docker는 주어진 Docker Host내에서 고유한 임의의 이름을 해당 `volume`에 부여합니다.
- 이름이 지정된 방식과는 상관 없이 `volume`은 동일한 방식으로 작동합니다.
- 또한, `volume`은 원격 Host와 Cloud Provider가 Data를 저장할 수 있는 `volume drivers` 사용을 지원하고 있습니다.

### bind mount

- `bind mount`는 Docker 초기부터 사용할 수 있었던 방식으로 `volume`에 비해 기능이 제한적입니다.
- `bind mount`를 사용하면, Host System의 파일 또는 디렉토리가 Container에 Mount됩니다.
- 파일 또는 디렉토리는 Host System의 전체 경로로 참조되지만, 미리 Docker Host에 존재할 필요는 없습니다. 없을 경우, 참조된 경로로 파일 또는 디렉토리가 생성됩니다.
- `bind mount`는 매우 효과적이지만, Host Machine의 File System 디렉토리 구조에 의존적입니다.
- Docker CLI 명령어로 `bind mount`를 관리할 수 없습니다.

{{% notice warning %}}
`bind mount`는 Container에서 실행 중인 프로세스들이 Host File System의 중요한 시스템 파일들이나 디렉토리의 생성, 수정, 삭제 명령을 통해 변경시킬 수 있습니다. 이는 Host System의 Non-Docker 프로세스들에게 충돌이 발생하거나, 보안에 큰 영향을 줄 수 있습니다. 따라서, `bind mount`보다는 `volume`를 사용하는 것을 권장합니다.
{{% /notice %}}

### tmfs mount

- `tmpfs mount`는 Docker Host 또는 Container내의 디스크에서 Data가 유지되지 않습니다.
- 비영구적인 상태 정보나 민감 정보들 같이 Container의 생명주기와 맞춰서 Data를 보존하고자 할 때 사용할 수 있습니다.
- 예를 들어, Docker Cluster인 Swarm Service는 내부적으로 `tmps mount`를 사용하여 Secret 정보를 Service의 Container에 Mount하여 사용합니다.

`bind mount` 및 `volume`는 `-v` 또는 `--volume` Flag를 사용하여 Container에 모두 Mount할 수 있지만, 각 구문은 약간 차이가 있습니다. `tmpfs mount`의 경우, `--tmpfs` Flag를 사용합니다. 그러나 Docker 17.06 이상에서는 bind mount, volume, tmpfs mount에 대해 Container와 Service 모두에 `--mount` Flag를 사용하는 것이 구문이 더 명확이기 때문에 권장합니다.

## Mount 유형별 사용 사례

### volume 사용 사례

`volume`은 Docker Container 및 Service에서 Data를 유지하는 기본적인 방법으로, 다음과 같은 경우에 `volume`을 사용합니다.

- **실행 중인 여러 Container 간에 Data 공유가 필요한 경우**
  - `volume`생성에 대해서 명시하지 않은 경우, Container에 처음 Mount될 때 생성됩니다.
  - Container가 중지되거나 제거되더라도 생성된 `volume`은 그대로 유지됩니다.
  - 여러 Container가 동일한 `volume`을 읽기/쓰기 또는 읽기 전용으로 동시에 Mount할 수 있습니다.
  - `volume`은 명시적으로 선언하여 제거합니다.
- **Docker Host가 특정 디렉토리나 파일 구조를 가질 수 없는 경우**
  - `volume`은 Container Runtime으로부터 Docker Host의 설정을 분리하는데 도움이 됩니다.
- **로컬외의 원격 Host 및 Cloud Provider에 Container의 Data를 저장하려는 경우**
  - 한 Docker Host에서 다른 Docker Host로 Data를 백업이나 복구 또는 마이그레이션을 하는 경우 `volume`을 사용하면 더 좋습니다.
  - `volume`을 사용하여 Container를 중지한 다음 `volume`의 디렉토리(예: `/var/lib/docker/volume/<volume-name>`)를 백업할 수 있습니다.

### bind mount 사용 사례

일반적인 경우, 가능하면 `bind mount`보다 `volume`를 사용하십시오. 다음과 같은 경우에 `bind mount`를 사용합니다.

- **Host Machine에서 Container로 설정 파일을 공유해야하는 경우**
  - Docker는 Host Machine의 `/etc/resolv.conf`를 각 Container에 `bind mount`하여 DNS Resolution을 제공하고 있습니다.
- **Docker Host 개발 환경과 Container 간 소스코드 또는 빌드된 아티팩트를 공유**
  - 예를 들어, Maven의 `target/` 디렉토리를 Container에 Mount하고, Docker Host에서 Maven Project를 빌드할 때마다, Container에서 재 작성된 작성된 JAR/WAR에 접근할 수 있습니다.
  - 만약 이런 방식으로 개발을 진행한다면, Production Dockerfile은 `bind mount`에 의존하지 않고도, Production-Ready 아티팩트를 Image에 직접 복사할 수 있습니다.
- **Docker Host의 파일 또는 디렉토리 구조가 Container가 요구하는 bind mount와 일치하는 경우**

#### volume 또는 bind mounts 사용 시, 유의사항

`volume` 또는 `bind mount`를 사용하는 경우, 다음의 사항에 대해 유의하시기 바랍니다.

- File 또는 Directory 있는 Container 내의 Directory에 비어있는 `volume`를 Mount하면, 해당 File 또는 Directory가 `volume`으로 전달(복사)됩니다. 마찬가지로, 아직 존재하지 않는 특정 `volume`을 지정하고 Container를 시작하면, 비어있는 `volume`이 생성됩니다. 이는 다른 Container에 필요한 Data를 미리 갖출 수 있는 좋은 방법입니다.
- File 또는 Directory가 있는 Container 내의 Directory에 `bind mount` 또는 비어있지 않은 `volume`를 Mount한다면, Linux Host에서 File을 `/mnt`에 저장하고 나서 USB 드라이브가 `/mnt`에 Mount되는 것처럼 기존에 존재하던 File 또는 Directory가 Mount에 의해 가려지게 됩니다. `/mnt`의 컨텐츠들은 USB 드라이브가 Unmount될 때까지, USB 드라이브의 컨텐츠들에 의해서 가려지게 됩니다. 가려진 파일들은 제거되거나 대체된 것은 아니지만, `bind mount` 또는 `volume`이 Mount되어 있는 동안은 접근할 수 없습니다.

### tmpfs mount 사용 사례

`tmpfs mount`는 Host System이나 Container 내에서 Data를 유지하지 않아야할 때 가장 적합합니다. 이는 보안상의 이유가 될 수도 있고, Application이 많은 양의 비영구적인 상태의 Data를 작성해야 할 때, Container의 성능을 보호하기 위한 것일 수도 있습니다.

## volume Mount 하기

volume을 사용하기 전에, volume을 Mount하기 위한 Flag에 대해서 확인해 보겠습니다.

### -v 또는 --mount 선택

원래는 독립형 Container에서는 `-v` 또는 `--volume`가 사용되었고, Docker Cluster인 Swarm Mode의 Service에서는 `--mount`가 사용되었습니다. 하지만, Docker 17.06부터 독립형 Container에서도 `--mount`를 사용할 수 있게 되었습니다. 두 Flag 간 가장 큰 차이점은 `-v`구문은 모든 옵션들을 하나의 Field에 결합하여 사용하고, `--mount`구문은 옵션을 분리하여 사용합니다. 따라서, `--mount`가 보다 명확하고 자세하게 정보를 확인할 수 있습니다.

{{% notice info %}}
`-v` 또는 `--volume` 보다 `--mount`의 사용성이 더 쉽기 때문에 `--mount`를 사용하는 것을 권장합니다. 만약 특정 `volume driver` 옵션들이 필요하다면, `--mount`를 사용해야 합니다.
{{% /notice %}}

다음은 Flag별 사용법입니다.

#### `-v` 또는 `--volume`

- 콜론(:)으로 구분된 세 개의 필드로 구성되어있습니다. 필드의 순서가 정확해야 하며, 각 필드의 의미는 명확하지 않습니다.
- 이름이 명시된 `volume`의 경우, 첫번째 필드는 `volume`의 이름이며, Host Machine에서 유일해야 합니다. 익명 `volume`의 경우, 첫번째 필드는 생략됩니다.
- 두번째 필드는 File 또는 Direcoty가 Container에 Mount될 경로입니다.
- 세번째 필드는 선택사항이며, 쉼표(,)로 구분된 옵션 목록(예: `ro`(읽기전용을 나타내는 옵션))입니다.

#### `--mount`

- 쉼표(,)로 구분되고 각각 여러 key-value 쌍으로 구성됩니다. `--mount` 구문은 `-v` 또는 `--volume`보다 길지만, key의 순서는 중요치 않으며 Flag의 값을 이해하기 쉽게 되어있습니다.
- `type`은 `bind, volume, tmpfs`로 Mount 유형을 지정합니다.
- `source`는 `volume`의 이름입니다. 익명 `volume`을 사용하고자 할 때는, 해당 필드를 생략해도 됩니다. `source` 또는 `src`로 지정할 수 있습니다.
- `destination`은 파일 또는 디렉토리가 Container에 Mount될 경로를 나타냅니다. `destination, dst` 또는 `target`으로 지정할 수 있습니다.
- `readonly` 옵션이 있으면 Container에 읽기 전용으로 Mount됩니다.
- 두 번 이상 지정할 수 있는 `volume-opt`옵션은 옵션 이름과 해당 값으로 구성된 key-value 쌍을 사용합니다.

### -v와 --mount 사이의 동작 차이

`bind mount`와 달리, `volume`의 모든 옵션들을 `--mount`와 `-v` Flag에서 사용할 수 있습니다. 단, Service에서 `volume`을 사용하고자 한다면,  `--mount`만 지원됩니다.

### volume 생성 및 삭제하기

`docker volume create [OPTIONS] [VOLUME]`로 `volume`을 생성합니다.

```bash
$ docker volume create my-vol
my-vol
```

`docker volume ls [OPTIONS]`로 `volume`의 전체 목록을 확인합니다.

```bash
$ docker volume ls
DRIVER              VOLUME NAME
local               c6d60670aa3b05a06956839587ce608c67dbc0b14e5f13590f526ff149382bb0
local               my-vol
```

`docker volume inspect [OPTIONS] VOLUME [VOLUMES...]`로 생성한 `volume`의 상세 정보를 확인합니다.

```bash
$ docker volume inspect my-vol
[
    {
        "CreatedAt": "2017-11-10T06:08:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volume/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

`docker volume rm [OPTIONS] VOLUME [VOLUME...]`로 생성한 `volume`를 삭제합니다.

```bash
$ docker volume rm my-vol
my-vol
```

### Container에 volume Mount하기

`--mount` Flag를 사용하여 Container 기동 시, `volume`을 Mount합니다.

```bash
$ docker run -d \
> -it \
> --name devtest \
> --mount source=myvol2,target=/app \
> nginx:latest
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
bc95e04b23c0: Pull complete
a21d9ee25fc3: Pull complete
9bda7d5afd39: Pull complete
Digest: sha256:9fca103a62af6db7f188ac3376c60927db41f88b8d2354bf02d2290a672dc425
Status: Downloaded newer image for nginx:latest
6de449868f75c83a425ae33f46dde69f3287edb56af88dc616582fb0c3622166
```

생성된 Container의 상세 정보를 확인합니다. Mounts 필드에서 `volume`에 대한 상세정보를 확인할 수 있습니다. Mount 형식은 `volume`으로 source 및 destination의 경로를 확인할 수 있으며, 읽기/쓰기가 모두 가능한 것을 확인할 수 있습니다.

```bash
$ docker inspect devtest
[
    {
        "Id": "6de449868f75c83a425ae33f46dde69f3287edb56af88dc616582fb0c3622166",
        "Created": "2017-11-10T06:17:32.273492731Z",
        "Path": "nginx",
        "Args": [
            "-g",
            "daemon off;"
        ],

        ... 생략 ...

        "Mounts": [
            {
                "Type": "volume",
                "Name": "myvol2",
                "Source": "/var/lib/docker/volume/myvol2/_data",
                "Destination": "/app",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],

        ... 생략 ...

        }
    }
]
```

Container를 중지시키고, 생성한 Container와 `volume`을 제거합니다.

```bash
$ docker container stop devtest
devtest

$ docker container rm devtest
devtest

$ docker volume rm myvol2
myvol2
```

Docker Cluster인 Swarm Mode의 Service도 Container와 동일한 방식으로 `volume`을 사용할 수 있습니다. Service를 시작하고 `volume`을 정의하면, 각 Service Container들은 자체 로컬 `volume`을 사용합니다. 로컬 `volume driver`를 사용하는 경우, Container들 중 어느 것도 해당 Data들을 공유할 수 없지만 일부 `volume driver`는 공유 Storage를 지원합니다. Docker for AWS와 Docker for Azure는 모두 Cloudstor 플러그인을 사용하여 영구 저장소를 지원합니다.

### Container로 volume에 Data 적재하기

Container를 시작할 때 새로운 `volume`을 생성하고, 해당 Container에 Mount할 Directory 내에 File 또는 Directory가 있으면(예: `/app/`) Directory의 내용이 `volume`으로 복사됩니다. 그러면 Container는 `volume`을 Mount하여 사용하고, 다른 Container도 미리 채워진 `volume`내의 컨턴츠에 접근할 수 있습니다.

먼저 Container를 실행할 때, `destination`을 `/usr/share/nginx/html`로 지정하여, 해당 Directory의 내용으로 새로운 `nginx-vol`라는 이름의 `volume`에 Data가 채워질 수 있도록 설장합니다. 해당 Directory는 Nginx가 기본 HTML 컨텐츠를 저장하는 곳입니다.

```bash
$ docker run -d \
> -it \
> --name=nginxtest \
> --mount source=nginx-vol,destination=/usr/share/nginx/html \
> nginx:latest
777e64d14c6ae173c00372d23130e43fe8d14f6160c94ef4dd7ac7b247ec110d
```

생성된 Container의 상세 정보를 확인합니다. Mounts 필드에서 `volume`에 대한 상세정보를 확인할 수 있습니다.

```bash
$ docker inspect nginxtest
[
    {
        "Id": "777e64d14c6ae173c00372d23130e43fe8d14f6160c94ef4dd7ac7b247ec110d",
        "Created": "2017-11-10T06:34:08.21801753Z",
        "Path": "nginx",
        "Args": [
            "-g",
            "daemon off;"
        ],

        ... 생략 ...

        "Mounts": [
            {
                "Type": "volume",
                "Name": "nginx-vol",
                "Source": "/var/lib/docker/volume/nginx-vol/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],

        ... 생략 ...

        }
    }
]
```

Container를 중지시키고, 생성한 Container와 `volume`을 제거합니다.

```bash
$ docker container stop nginxtest
nginxtest

$ docker container rm nginxtest
nginxtest

$ docker volume rm nginx-vol
nginx-vol
```

### 읽기전용 volume Mount하기

일부 Application 개발의 경우, Container가 Docker Host로 변경 사항을 다시 전달할 수 있도록 `bind mount`하여 사용합니다. 반대로, 다른 Container들은 단지 Data를 읽기만 하여야 하고, 수정하지 못해야 할 때도 있습니다. 즉, 여러 Container가 동일한 `volume`을 Mount할 수 있으며, 일부 Container는 읽기/쓰기로, 그 외 Container는 읽기전용으로 Mount하여 사용할 수도 있습니다.

`--mount`에 `readonly` 옵션을 추가하여 Container와 volume을 생성합니다.

```bash
$ docker run -d \
> -it \
> --name=nginxtest \
> --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
> nginx:latest
44f1138b38c1b4df9b1bb4b4d14e7181af05ee0dcc153e634c7db5b17c509635
```

생성된 Container의 상세 정보를 확인합니다. Mounts 필드에서 `volume`에 대한 상세정보를 확인할 수 있습니다. `readonly` 옵션이 적용되어, `RW` Key의 value가 `false`로 되어 있는 것을 확인할 수 있습니다.

```bash
$ docker inspect nginxtest
[
    {
        "Id": "44f1138b38c1b4df9b1bb4b4d14e7181af05ee0dcc153e634c7db5b17c509635",
        "Created": "2017-11-10T06:38:25.060891516Z",
        "Path": "nginx",
        "Args": [
            "-g",
            "daemon off;"
        ],

        ... 생략 ...

        "Mounts": [
            {
                "Type": "volume",
                "Name": "nginx-vol",
                "Source": "/var/lib/docker/volume/nginx-vol/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "z",
                "RW": false,
                "Propagation": ""
            }
        ],

        ... 생략 ...

        }

    }

]
```

Container를 중지시키고, 생성한 Container와 `volume`을 제거합니다.

```bash
$ docker container stop nginxtest
nginxtest

$ docker container rm nginxtest
nginxtest

$ docker volume rm nginx-vol
nginx-vol
```

이렇게 `volume`을 사용하여, Container의 Data들을 영구적으로 존속시킬 수 있었습니다. 또한 다양한 Driver를 사용하여 NFS 또는 AWS의 S3와 같은 외부 Storage System과의 연동을 통해 Data를 Docker Machine간에 Data를 공유할 수도 있습니다. 이 때, `volume driver`는 Storage System을 추상화하여 사용하기 때문에, Application의 로직 변경없이도 자유롭게 Data를 유지시킬 수 있습니다.

Container란 무엇인가라는 주제부터 시작해서 Volume을 활용한 Container Data관리까지, Application이 Container화하여 어떤 식으로 동작하고, 무엇을 고려해야할지에 대해서 기초적인 항목들을 확인하였습니다. 현재까지는 모든 것들을 단일 Docker Machine 환경으로 한정하여 생각했습니다. 다음에는 Docker Cluster인 Swarm을 구성하여 이런 Container들을 어떤 방식으로 Orchestration할 수 있는지 알아봐야겠네요. 
