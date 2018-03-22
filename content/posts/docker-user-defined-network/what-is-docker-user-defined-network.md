---
date: "2018-03-20T18:12:19+09:00"
title: "[Docker-User Defined Network 활용(1/3)] Docker User Defined Network 란?"
authors: [jisangyun]
categories:
  - posts
tags:
  - Docker
  - User Defined Network
  - Spring Boot
cover:
  image: docker-logo.png
description: "Docker User Defined Bridge Network를 간단 동작을 확인합니다."
draft: false
---

Docker는 몇가지 네트워크 드라이버를 기본 제공하여 강력한 네트워크 기능을 활용할 수 있게 합니다.

- bridge: 기본 네트워크 드라이버. docker0이라는 이름의 bridge 네트워크를 생성됩니다. 설정없이 컨테이너를 생성하게되면 docker0 bridge에 컨테이너를 binding 해서 네트워크 기능을 수행합니다.
- host: 컨테이너 네트워크가 독립/격리되지 않고, Host의 네트워크를 직접적으로 사용합니다.
- overlay: 서로 다른 Docker Host에서 실행되는 컨테이너 간 통신이 필요하거나, Docker Swarm 상에서 여러 컨테이너를 동시에 운영할 때 유용합니다.
- Macvlan: Mac주소를 컨테이너에 할당합니다. Docker 데몬은 트래픽을 컨테이너의 MAC 주소로 라우팅합니다.

상세 내용은 Docker Docs를 참고하시기 바랍니다. ( [Docker Docs - Network](https://docs.docker.com/network/) )

## What

1. Docker Network의 사용법을 익힙니다.
2. User Defined Bridge Network를 활용한 다양한 컨테이너 간 통신을 테스트합니다.
3. User Defined Bridge Network로 private network상의 API 서버를 구현합니다.

## Why

###### link option의 legacy화
기존에 컨테이너 간 통신에 사용하던 docker run --link 옵션은 legacy 기능이 되었습니다. 새로운 컨테이너 간 통신 방법을 확인할 필요가 있습니다.
( [Docker Docs - link](https://docs.docker.com/network/links/) )

###### 쉽고 가벼운 네트워크 사용

기본적으로 컨테이너는 IP주소로만 접근할 수 있습니다. 반면에, user defined bridge network 적용시 컨테이너의 명으로 접근해 편리하게 네트워크 기능을 수행할 수 있습니다.

또한, docker network 명령어로 네트워크에 컨테이너 추가/해제가 컨테이너의 생성/삭제 없이 쉽고 가볍게 이루어집니다. 이를 활용해 시스템 요구사항에 맞는 다양한 활용법을 적용할 수 있습니다. (배포전략 등)  

###### Private Network 구성

동일한 user defined bridge network 에 연결된 컨테이너는 모든 Port를 서로에게 노출시키고 외부에는 Port를 노출하지 않습니다.
이를 통해 private 네트워크를 구성해 시스템의 보안 level을 높일 수 있습니다.


## How

Docker Network의 기본동작을 확인하고, 다양한 경우의 수를 대비한 컨테이너 간 통신을 테스트합니다. 마지막으로 private network 상의 API 서버를 구현해 실제 적용할 수 있는 최소단위의 구성을 해보겠습니다.

###### Docker Network 기본 동작

docker network 명령어의 기본 동작을 확인해보겠습니다. 상세한 옵션 등은 Docker Docs나 Docker cli 상의 help option을 확인하시면 됩니다.

- ls
  - docker network 의 목록을 조회합니다.
  - `docker network ls`

    ```Shell
    $ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    251862fde7a0        bridge              bridge              local
    aa6913be5189        host                host                local
    c455da940d9b        none                null                local
    ```
- create
  - user defined network를 생성합니다. 기본 driver는 bridge로 생성됩니다.
  - `docker network create {network 명}`
    
    ```Shell
    $ docker network create js-network
    b89583ee19716b63c30d59a5d2d21a39f3cd30ecbcced537793e9fe6a4407195

    $ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    251862fde7a0        bridge              bridge              local
    aa6913be5189        host                host                local
    b89583ee1971        js-network          bridge              local
    c455da940d9b        none                null                local
    ```
- inspect
  - docker network의 상세정보를 조회합니다.
  - `docker network inspect {network 명}`
    
    ```Shell
    $ docker network inspect js-network
    [
        {
            "Name": "js-network",
            "Id": "b89583ee19716b63c30d59a5d2d21a39f3cd30ecbcced537793e9fe6a4407195",
            "Created": "2018-03-21T07:14:28.207599689Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {},
            "Options": {},
            "Labels": {}
        }
    ]
    ```
- connect
  - docker network에 컨테이너를 연결합니다.
  - `docker network connect {network 명} {container 명}`
    
    ```Shell
    $ docker network connect js-network jisang-ms1

    $ docker network inspect js-network
    [
        {
            "Name": "js-network",
            "Id": "b89583ee19716b63c30d59a5d2d21a39f3cd30ecbcced537793e9fe6a4407195",
            "Created": "2018-03-21T07:14:28.207599689Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {
                "fe50dfe7788f46b4df3e25a128bec292f4263167dafa116ef4e68a47aeaccdcb": {
                    "Name": "jisang-ms1",
                    "EndpointID": "c7c9a46ad6715cc65250a502bbd28c1bc389514d1f1db6136b90f12e483681e0",
                    "MacAddress": "02:42:ac:12:00:02",
                    "IPv4Address": "172.18.0.2/16",
                    "IPv6Address": ""
                }
            },
            "Options": {},
            "Labels": {}
        }
    ]
    ```
- disconnect
  - docker network에서 컨테이너 연결을 해제합니다.
  - `docker network disconnect {network 명} {container 명} `

    ```Shell
    $ docker network disconnect js-network jisang-ms1

    $ docker network inspect js-network
    [
        {
            "Name": "js-network",
            "Id": "b89583ee19716b63c30d59a5d2d21a39f3cd30ecbcced537793e9fe6a4407195",
            "Created": "2018-03-21T07:14:28.207599689Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {},
            "Options": {},
            "Labels": {}
        }
    ]
    ```
- rm
  - user defined network를 삭제합니다.
  - `docker network rm {network 명}`
    
    ```Shell
    $ docker network rm js-network
    js-network

    $ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    251862fde7a0        bridge              bridge              local
    aa6913be5189        host                host                local
    c455da940d9b        none                null                local
    ```
- docker run
  - 컨테이너 수행시 user defined netowrk에 연결합니다.
  - `docker run --network={network 명} {컨테이너 image}`
   
    ```Shell
    $ docker run -d --network=js-network -e SPRING_PROFILES_ACTIVE=docker --name jisang-ms1 jisang-ms1:latest
    4e1c75a6f44763a4ac0266271698448491905388dd69397aaf84eda641464058

    $ docker network inspect js-network
    [
        {
            "Name": "js-network",
            "Id": "3e7b219d5c06c4a817ebf499316a4758f016d747b6be8a0d5b83ba108620e42c",
            "Created": "2018-03-21T07:38:35.244071424Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {
                "4e1c75a6f44763a4ac0266271698448491905388dd69397aaf84eda641464058": {
                    "Name": "jisang-ms1",
                    "EndpointID": "4c3b9ea660520f6ed6463575e6ec44f7a2652a43c075ee5f640367f00ec6a2ec",
                    "MacAddress": "02:42:ac:12:00:02",
                    "IPv4Address": "172.18.0.2/16",
                    "IPv6Address": ""
                }
            },
            "Options": {},
            "Labels": {}
        }
    ]
    ```

## Conclusion

지금까지 Docker의 user defined network에 대해 알아보고 docker network 명령어의 간단 동작을 수행해보았습니다.

다음 포스팅에서는 컨테이너 간 통신에 연관된 factors를 확인하고 경우의 수를 찾아 테스트를 진행해보겠습니다.