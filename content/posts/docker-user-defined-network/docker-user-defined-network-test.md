---
date: "2018-03-21T11:16:00+09:00"
title: "[Docker-User Defined Network 활용(2/3)] Docker User Defined Bridge Network 테스트"
authors: [jisangyun]
categories:
  - posts
tags:
  - Docker
  - User Defined Network
  - Spring Boot
cover:
  image: docker-logo.png
description: "Docker User Defined Bridge Network를 Spring Boot Application으로 테스트합니다."
draft: true
---

지난 포스팅에서 docker user defined network의 간단한 설명과 동작을 확인했습니다.

이번에는 docker user defined network를 활용한 컨테이너 간 통신에 연관된 factors를 확인하고 경우의 수에 따라 테스트를 진행하겠습니다.

## Docker Network 기능

![docker-network](dockernetwork.png)

### Service Discovery 

User Defined Bridge Network를 지정하면 컨테이너 명으로 HOST정보를 찾을 수 있습니다. 또한, 컨테이너 외부 Port를 노출하지 않고도 컨테이너 간 통신이 가능합니다.

컨테이너명으로 통신이 가능하다. 내부적으로 어케 관리하는거지 ? etcd 이런거인데 쿠버는 ... 

### Port Expose 

컨테이너 포트 열고 닫고 에 따른 통신 관련 그림 하나 넣고 설명 넣고

### user defined network을 사용한 Private Network 구성

이거로 user defined network에 컨테이너를 연결하고 port를 expose 하지 않으면 private network를 구성할 수 있다.
프론트 백엔드 등 네트워크 영역을 분리해서 보안을 높일 수 있다. 
그래서 편리하게 사용 가능하다

## 테스트

###### 준비물

- Application 정보
  - jisang-ms1
    - port: 8080
    - RestController로 Endpoint 호출시 jisang-ms2로 통신해 String을 리턴합니다.
  - jisang-ms2
    - port: 8081
    - RestController로 Endpoint 호출시 "This is jisang"을 리턴합니다.

###### factors

- Host 정보
  - 컨테이너 명
  - IP주소
    
    > 테스트 환경은 Docker for Windows입니다. 
    >
    > Virtual Box에 Docker Machine을 구성해서 사용하는 경우, 기본 IP주소는 localhost가 아닌 **Docker Machine의 IP주소입니다.**
    > 
    > ```Shell
    > $ docker-machine url
    > tcp://192.168.99.100:2376
    > ```
- Port  
  - 노출
  - 격리
- user defined network
  - 연결
  - 해제

### Test Case 1

| Host | Port expose | Network Status |
| --- | --- | --- |
| IP주소 | 노출 | disconnect |

### Test Case 2

| Host | Port expose | Network Status |
| --- | --- | --- |
| IP주소 | 격리 | disconnect |

### Test Case 3

| Host | Port expose | Network Status |
| --- | --- | --- |
| 서비스명 | 노출 | disconnect |

### Test Case 4

| Host | Port expose | Network Status |
| --- | --- | --- |
| 서비스명 | 격리 | connect |

### Test Case 5

| Host | Port expose | Network Status |
| --- | --- | --- |
| IP주소 | 노출 | connect |

### Test Case 6

| Host | Port expose | Network Status |
| --- | --- | --- |
| IP주소 | 격리 | connect |

### Test Case 7

| Host | Port expose | Network Status |
| --- | --- | --- |
| 서비스명 | 노출 | connect |

### Test Case 8

| Host | Port expose | Network Status |
| --- | --- | --- |
| 서비스명 | 격리 | connect |




## Conclusion



