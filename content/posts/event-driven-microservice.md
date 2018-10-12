---
date: "2018-10-11T10:14:07+09:00"
title: "Event Driven Microservice 란?"
authors: [jisangyun]
series: []
categories:
  - posts
tags:
  - event driven
  - microservices
  - spring cloud stream
  - 비동기 통신
  - reative
  - serverless
  - rabbitmq
  - kafka
  - event store
  - message queue
cover:
  image: event_driven_microservice.png
  caption: ""
description: "우리는 Monolitic Architecture의 단점을 보완하고 새로운 이점을 얻기 위해 MicroService Architecture를 도입했습니다. 우리의 모든 문제들이 해결되었나요? 새로운 문제가 발생하지는 않았나요? 걱정하지 마세요. Event Driven MircoService를 소개합니다."
draft: true
---

MicroService Architecture(MSA)는 loosely coupled를 기반으로 빠른 배포주기, 폴리글랏 프로그래밍, 관심사의 집중 등의 장점을 발휘해 Enterprise IT에서 가장 주목받고 있는 아키텍처 입니다. 또한, 분해된 서비스의 scalabililty, resiliency 등 컨테이너 기반의 플랫폼(Kubernetes 등)과 잘 어우러지는 성향으로 서로 끌어주고 밀어주며 발전하고 있습니다.  

하지만 MSA를 도입한 이후 새로운 문제점은 발생하지 않았나요 ? Database Per Service 라는 새로운 요구사항은 잘 지켜지나요 ? rest 통신(synchronized)으로 인한 제약사항은 없나요 ? 분산된 서비스 간 트랜잭션 처리 / 반정규화 된 데이터의 동기 처리는 잘 이루어지고 있나요 ?

MSA가 적용된 시스템을 보완하는 Event Driven MircoService를 소개합니다.
