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

### Docker Service Discovery 기능

User Defined Bridge Network를 지정하면 컨테이너 명으로 HOST정보를 찾을 수 있습니다. 또한, 컨테이너 외부 Port를 노출하지 않고도 컨테이너 간 통신이 가능합니다.

Spring Boot - RestTemplate으로 컨테이너 간 통신 테스트를 해보겠습니다.
- 컨테이너 정보
  - jisang-ms1
    - port: 8080
    - RestController로 Endpoint 호출시 jisang-ms2로 통신해 String을 리턴합니다.
  - jisang-ms2
    - port: 8081
    - RestController로 Endpoint 호출시 "This is jisang"을 리턴합니다.
- 테스트 case
  - 

#### IP주소 + Port Expose 한 경우
- `des: http://192.168.99.100:8081`
  > docker for windows의 경우, docker-machine이 운용중인 virtual box의 가상 ip로 컨테이너에 접근 가능합니다.
  >
  > $ docker-machine url
  >
  > tcp://192.168.99.100:2376
- 컨테이너 상태

- 호출 결과


#### IP주소 + Port Expose 안 한 경우

#### 컨테이너 명 + Port Expose 한 경우

#### 컨테이너 명 + Port Expose 안 한 경우



