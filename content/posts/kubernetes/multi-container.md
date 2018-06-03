---
date: "2018-06-03T13:47:20+09:00"
title: "Multi-Container"
authors: ["1000jaeh"]
categories:
  - posts
tags:
  - kubernetes
  - k8s
  - multi-container
  - ambsssdor
cover:
  image: ""
draft: true
---

## Kubernets의 Multi Container 설계 패턴

### Sidecar

사이드카 Containers는 "main" 컨테이너를 확장 및 강화하고, 기존 컨테이너들을 사용하여 컨테이너를를 개선합니다. 예를 들어, Nginx 웹 서버를 실행하는 컨테이너를 고려하십시오. 파일 시스템을 git 저장소와 동기화하는 다른 컨테이너를 추가하고, 컨테이너간에 파일 시스템을 공유하면 Git push-to-deploy가 빌드됩니다. 그러나 모듈화 된 방식으로 git 동기화 기가 다른 팀에 의해 구축 될 수 있으며 여러 다른 웹 서버 (Apache, Python, Tomcat 등)에서 재사용 할 수 있습니다. 이 모듈성 때문에 git 동기화 기는 한번만 작성하고 테스트하고 수많은 앱에 다시 사용해야합니다. 다른 사람이 글을 쓰면 그걸 할 필요조차 없습니다.

![](https://3.bp.blogspot.com/-IVsNKDqS0jE/WRnPX21pxEI/AAAAAAAABJg/lAj3NIFwhPwvJYrmCdVbq1bqNq3E4AkhwCLcB/s1600/Example%2B%25231-%2BSidecar%2Bcontainers%2B.png)ㅍ




사례 # 2 : 앰배서더 컨테이너
앰배서더 컨테이너는 세계와 로컬 연결을합니다. 예를 들어, 읽기 복제본과 단일 쓰기 마스터가있는 Redis 클러스터를 생각해보십시오.

 당신은 레디스 대사 컨테이너와 함께 너의 메인 애플리케이션을 그룹화해서 팟을 만들 수 있습니다.
 
 대사는 읽기와 쓰기를 분리하여 적절한 서버로 보내는 역할을 프록시가 담당합니다. 
 
 왜냐하면 이 두 컨테이너는 네트워크 네임 스페이스를 공유하기 때문에, IP 주소를 공유하므로 응용 프로그램은 "localhost"에서 연결을 열고 서비스 디스커버리없이 프록시를 찾을 수 있습니다.
 
  귀하의 주요 응용 프로그램에 관한 한, 그것은 로컬 호스트의 Redis 서버에 연결하는 것입니다.

 이는 서로 다른 팀이 구성 요소를 쉽게 소유 할 수 있다는 사실뿐 아니라 개발 환경에서 프록시를 건너 뛰고 localhost에서 실행되는 Redis 서버에 직접 연결할 수 있기 때문에 강력합니다.

![](https://4.bp.blogspot.com/-yEmqGZ86mNQ/WRnPYG1m3jI/AAAAAAAABJo/94DlN54LA-oTsORjEBHfHS_UQTIbNPvcgCEw/s1600/Example%2B%25232-%2BAmbassador%2Bcontainers.png)



예제 # 3 : 어댑터 컨테이너
어댑터 컨테이너는 출력을 표준화하고 표준화합니다. N 개의 다른 응용 프로그램을 모니터링하는 작업을 고려하십시오. 각 응용 프로그램은 모니터링 데이터를 내보내는 다른 방법으로 빌드 될 수 있습니다. (예 : JMX, StatsD, 응용 프로그램 별 통계) 모든 모니터링 시스템은 수집하는 모니터링 데이터에 대해 일관되고 균일 한 데이터 모델을 기대합니다. 복합 컨테이너의 어댑터 패턴을 사용하면 변환 수행 방법을 알고있는 어댑터로 어플리케이션 컨테이너를 그룹화하는 포드를 작성하여 다른 시스템의 이기종 모니터링 데이터를 단일 통합 표현으로 변환 할 수 있습니다. 이 포드가 네임 스페이스와 파일 시스템을 공유하기 때문에이 두 컨테이너의 조정은 간단하고 간단합니다.

![](https://4.bp.blogspot.com/-4rfSCMwvSwo/WRnPYLLQZqI/AAAAAAAABJk/c29uQgM2lSMHaUL013scJo_z4O8w38mJgCEw/s1600/Example%2B%25233-%2BAdapter%2Bcontainers%2B.png)





이 모든 경우에 컨테이너 경계를 캡슐화 / 추상화 경계로 사용하여 모듈화 된 재사용 가능한 구성 요소를 구축하여 응용 프로그램을 구축 할 수 있습니다. 이러한 재사용을 통해 여러 개발자간에 컨테이너를보다 효과적으로 공유하고 코드를 여러 응용 프로그램에 재사용 할 수 있으며보다 안정적이고 강력한 분산 시스템을보다 신속하게 구축 할 수 있습니다. 포드와 복합 컨테이너 패턴을 사용하여 강력한 분산 시스템을보다 신속하게 구축하고 컨테이너 코드 재사용을 달성하는 방법을 살펴 보셨기를 바랍니다. 자신의 응용 프로그램에서 이러한 패턴을 직접 사용해보십시오. 오픈 소스 인 Kubernetes 또는 Google Container Engine을 사용해 보시기 바랍니다.

