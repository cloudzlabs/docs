---
date: "2018-02-22T10:37:45+09:00"
title: "Bluemix Auto-scaling 서비스 테스트"
authors: [jisangYun]
categories:
  - posts
tags:
  - Bluemix
  - BuildPack
  - Java BuildPack
  - Liberty BuildPack
  - Cloud Foundry
  - Auto Scaling
  - Spring Auto Reconfiguration
cover:
  image: /docs/contents/images/bluemix-autoscaling-test/auto-scaling.png
  caption: ""
draft: true
---

## What ?

PaaS 사용의 이점으로 항상 언급되는 것이 바로 Auto-Scaling 이다.

Bluemix는 어플리케이션에 Auto-Scaling 서비스를 binding 해 Auto-Scaling을 적용할 수 있다.
![bluemix-auto-scaling-info](docs/contents/images/bluemix-autoscaling-test/bluemix-auto-scaling-info.PNG)

- [Bluemix Auto-Scaling 서비스](https://console.bluemix.net/catalog/services/auto-scaling/)
  - 어플리케이션의 scaling 정책을 정의할 수 있다.
  - 정의된 정책에 따라 어플리케이션의 instance를 동적으로 조정할 수 있다.
  - metrics(Heap/Memory/ThroughPut/Response Time)을 visualization 해준다. 
  - 시간, 상태에 따른 Scaling History를 visualization 해준다.

해당 기능을 테스트 해보자.

## Why ?
Bluemix Auto-Scaling 서비스의 사용 방법 및 [DOCS](https://console.bluemix.net/docs/services/Auto-Scaling/index.html)를 확인하니, 몇가지 제약사항이 언급되어 있다.

![restriction](docs/contents/images/bluemix-autoscaling-test/retriction.PNG)

> 참고: Auto-Scaling 메트릭 데이터를 수집하려면 Liberty 웹 컨테이너를 통해 HTTP/HTTPS 측정 요청이 처리되도록 애플리케이션이 Liberty 웹 앱으로 배치되어야 합니다. 

자바 어플리케이션을 Bluemix에 배포할 때 준비해야할 사항이 무엇인지 확인할 필요가 있다.


## How ?

1. 처리량 임계치를 설정한다.

2. Jmeter로 어플리케이션 부하테스트를 수행한다. 

3. 처리량 임계치를 넘었을 때, Auto-Scaling이 수행되는지 확인한다.

4. 대쉬보드에서 metrics가 정상적으로 조회되는지 확인한다.

### 준비물
- 자바 어플리케이션
  - liberty-for-java buildpack / liberty 웹 컨테이너 (dtlabs-service-activity)
  - liberty-for-java buildpack / tomcat (dtlabs-service-acrivity-tomcat)
  - java buildpack (dtlabs-service-activity-javabuildpack)
- 위의 어플리케이션 Bluemix에 배포
- 어플리케이션에 Bluemix Auto-Scaling 서비스 binding
- 테스트용 Jmeter 및 script

준비 끝!

### dtlabs-service-activity

#### Auto-Scaling 정책 설정

#### 테스트

#### 결과

### dtlabs-service-acrivity-tomcat

#### Auto-Scaling 정책 설정

#### 테스트

#### 결과

### dtlabs-service-activity-javabuildpack

#### Auto-Scaling 정책 설정

#### 테스트

#### 결과

## Conclusion

블루믹스에서 오토스케일링 서비스를 이용하려면 자바 어플리케이션은 리버티 빌드팩을 적용해야한다.

어플리케이션을 블루믹스에 배포하고 오토스케일링 서비스를 바인딩 한 후, 정책에 따라 동적으로 어플리케이션 인스턴스를 조정할 수 있다.