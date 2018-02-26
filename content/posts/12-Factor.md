---
date: "2018-02-22T11:07:05+09:00"
title: "12-Factor"
authors: ["bckim0620"]
series: ["Go Web Dev"]
categories:
  - posts
tags:
draft: false
---

최근 소프트웨어를 Cloud 환경에 배포하여 서비스 형태로 제공하는 SaaS(Software As A Service)가 보편화 되고있다. SaaS의 개발 방법은 기존의 방식과 많은 차이를 보인다. 12-Factor는 SaaS를 만들기위한 방법론으로 이를 준수 할 경우 아래 SaaS의 특징을 갖출수 있다.

> SaaS의 특징
- 설정 자동화를 위한 절차(declarative) 를 체계화 하여 새로운 개발자가 프로젝트에 참여하는데 드는 시간과 비용을 최소화한다.
- OS에 따라 달라지는 부분을 명확히하고, 실행 환경 사이의 이식성을 극대화 한다.
- 최근 등장한 클라우드 플랫폼 배포에 적합하고, 서버와 시스템의 관리가 필요없게 된다.
- 개발 환경과 운영 환경의 차이를 최소화하고 민첩성을 극대화하기 위해 지속적인 배포가 가능하다.
- 툴, 아키텍처, 개발 방식을 크게 바꾸지 않고 확장(scale up) 할 수 있다.

12-Factor는 이름에서 보여지듯 12가지 항목으로 구성되어있다. 이어지는 내용에서는 12가지 항목을 설명한다.

### 1. CodeBase
애플리케이션의 1개의 코드베이스(Git, SVN)를 통해 관리 되어야 하며, 동일한 코드로 운영/개발에 배포하여야 한다.

- 애플리케이션은 1개의 코드베이스를 가진다.
- 애플리케이션은 1개의 코드베이스를 통해 운영/개발용으로 배포한다.
- CodeBase 항목은 이어지는 타 항목을 준수기위해 기본적으로 준수해야하는 항목이다. 
- CodeBase 항목은 SVN, Git과 같은 코드관리 시스템 사용으로 준수할수 있다.

> 중요도: Non-negotiable

### 2. Dependencies
애플리케이션의 모든 종속성을 명시적으로 선언하여 사용한다. 애플리케이션이 필요로 하는 라이브러리를 dependency manifest 파일에 (Gemfile, POM 등) 명시적으로 선언하여 사용한다.
SaaS는 상황에 따라 다양한 환경(window, mac, linux)에 배포 될수 있다. Gemfile, pom 등을 사용 하여 다양한 환경에서도 SaaS가 정상 동작 할 수 있음을 보장할 수 있다. 예를 들어 curl등을 사용하여 lib를 사용할 경우 os에 따라 오동작 할 수 있다.

- Dependencies 항목 준수 방법
  - Dependencies 항목은 Gemfile, POM 등을 사용하여 준수 할 수 있다.
  - 필요한 모든 라이브러리와 버전을 리스트업하고, 배포할 시 빌드 명령어를 실행. mvn build 또는 build install(Ruby) 등

> 중요도: High

Spring boot의 경우 embedded runtime, external runtime에 따라 dependency 를 명시적으로 선언할 수 있다.

 - 배포방식 : jar or war
 - dependency : spring-boot-starter-tomcat을 사용할 것인지 여부 결정
 - sample code : http://www.slideshare.net/SpringCentral/12-factor-cloud-native-apps-for-spring-developers 19~20 page

### 3. Config
모든 설정 정보는 코드로부터 분리된 공간에 저장되어야 하고, 런타임에서 코드에 의해 읽혀야 한다.
SaaS는 동일한 코드를 여러 환경(운영/개발)에 배포한다. 이를 위해 환경마다 달리 사용되어야 하는 정보를 분리한다.

- 분리되어야 할 정보
  - 데이터베이스나 다른 백업 서비스를 처리하는 리소스
  - 외부 리소스(S3, Twitter 등)의 인증 정보
  - 각 배포마다 달라지는 값(cononical hostname..)
  - dev,test,stage,prod의 배포 단계마다 다를 수 있는 어떤 값들
- 설정정보를 저장하면 안되는 곳
  - code
  - properties file
  - build : one build, many deploy니까
  - app server(jndi database 같은 정보)
- Config 항목 준수 방법
  - 배포 환경(개발/운영)용 설정파일을 작성
  - Spring Cloud Config 사용

> 중요도: Medium

### 4. Backing services
백엔드 서비스를 연결된 리소스로 취급한다.
SaaS의 리소스는 자유롭게 배포에 연결되거나 분리할 수 있고, 코드 수정없이 전환이 가능해야 한다. 예를 들어 DB를 MySQL에서 Amazon RDS로 전환할 때 코드 수정없이 가능해야 한다.

- 백엔드 서비스
  - 네트웍을 통해 이용하는 모든 서비스
  - DB, Cache, SMTP, Messaging/Queueing system
- 준수 방법
  - Config에 백엔드 서비스의 URL이나 Locator를 저장하고, 코드에서는 설정을 읽어서 사용
  - Factor3. Config 기능 사용
- 참고
  - Spring sample
    - https://github.com/joshlong/12factor-backing-services
  - http://www.slideshare.net/SpringCentral/12-factor-cloud-native-apps-for-spring-developers 30~32 page

> 중요도: High



### 5. Build, Release, Run
코드베이스는 build > release > run의 단계를 거쳐 배포로 변환되며, 각 단계는 엄격하게 분리되어야 한다.

- 준수 방법
  - 빌드 단계는 개발자, 배포 단계는 배포툴, 실행 단계는 프로세스 매니저에 의해 시작
- 참고
  - Factor5-Design,Build,Release,Run(12 page): http://www.slideshare.net/SpringCentral/12-factor-cloud-native-apps-for-spring-developers 

> 중요도: Conceptual

### 6. Process
실행 환경에서 앱은 하나 이상의 프로세스로 실행되며, 각 프로세스는 stateless로 아무것도 공유하지 않아야 한다.
SaaS는 여러개의 인스터스로 배포될 수 있다. 각 인스턴스는 메모리 파일 등을 공유 할 수 없으며, 인스턴스가 재실행 될때 local file, session과 같은 상태 정보는 모두 초기화 된다.

- 준수 방법
  - 메모리/파일을 사용할 경우 단일 트랜잭션 내에서 읽고,쓰고 등의 모든 작업을 처리.
  - 세션 상태 데이터의 경우 Memcached 또는 Redis와 같은 데이터 저장소에 저장

> 중요도: High

### 7. Port Binding
배포된 SaaS 애플리케이션을 타 애플리케이션에서 접근 할 수 있도록 포트바인딩을 통해 서비스를 공개한다.
앱도 백엔드 서비스처럼 URL을 제공하고, 라우팅 레이어가 외부에 공개된 호스트명으로 들어온 요청을 포트에 바인딩된 웹 프로세스에 전달한다.
Factor4. Backing services의 확장으로, 포트바인딩에 의해 공개되는 서비스는 HTTP 뿐만 아니라 ejabberd나 Redis 같은 모든 종류의 서버 소프트웨어가 해당된다.

- 준수 방법
  - 보통 dependency에 웹서버 라이브러리를 추가해서 구현
- 참고
  - Spring Cloud Netflix
    - http://cloud.spring.io/spring-cloud-netflix/
  - https://spring.io/blog/2015/01/20/microservice-registration-and-discovery-with-spring-cloud-and-netflix-s-eureka

> 중요도: Medium

### 8. Concurrency
앱은 수평으로 확장할 수 있어야 하고, Factor6. Processes에 의해 동시성을 높일 수 있다.

- 준수 방법
 - 모든 일을 처리하는 하나의 프로세스 대신 기능별로 분리된 프로세스 실행(micro service)
 - 프로세스가 데몬형태가 아니어야 함
 - OS 프로세스 관리자/분산 프로세스 매니저/Foreman 같은 툴에 의존해서 output stream을 관리하고, 충돌이 발생한 프로세스에 대응, 재시작과 종료를 처리해야 함

> 중요도: Low

### 9. Disposability
프로세스는 shutdown 신호를 받았을때 graceful shutdown 해야한다.
SaaS는 요청에 의해 Sacle up/down이 빈번히 발생한다. Disposability를 준수함으로써 이러한 사용에 안정성을 얻을 수 있다.
예를 들어 Sacle down 시점에 graceful shutdown 이 아니라면 db lock등으로 인해 타 프로세스에 영향을 주게된다.

> 중요도: Mediu

### 10. dev/prod parity
development, staging, production 환경을 최대한 비슷하게 유지한다.
SaaS 애플리케이셔은  개발환경과 production 환경의 차이를 작게 유지하여 지속적인 배포가 가능하도록 디자인되어야 한다.

- 준수 방법
  - 시간의 차이를 최소화: 개발자가 작성한 코드는 몇시간 또는 몇분 후에 배포되어야 함
  - 담당자의 차이를 최소화: 코드를 작성한 개발자들이 배포와 production 에서의 모니터링에 깊게 관여함
  - 툴의 차이를 최소화: 개발과 production 환경을 최대한 비슷하게 유지

> 중요도: Medium


### 11. Logs
로그를 이벤트 스트림으로 취급하여 로그를 로컬에 저장하지 않는다.
SaaS는 언제든지 인스턴스가 생성/삭제 될 수 있다. 이때 로컬에 저장된 로그는 초기화 되기 때문에 로그는 스트림으로 취급하여 별도의 저장소에 보관해야 한다.

- 준수 방법
  - 스트림을 버퍼링 없이 stdout, stderr 로 출력함
  - 별도의 로그 저장소를 사용

> 중요도: Low

### 12. Admin Process
admin/maintenance 작업을 일회성 프로세스로 실행해야 한다.

- 일회성 프로세스
  - 데이터베이스 마이그레이션
  - 일회성 스크립트 실행
- 준수 방법
  - 관리/유지보수 작업은 release와 함께 실행
  - release와 동일한 환경에서 실행하고, 같은 코드베이스와 config를 사용
  - admin 코드는 동기화 문제를 피하기 위해 어플리케이션 코드와 함께 배포한다.

>중요도: High


### 참고자료
- The Twenve-Factor App
  - http://12factor.net/12-Factor 
- Apps in Plain English
  - http://www.clearlytech.com/2014/01/04/12-factor-apps-plain-english/
- 12 Factor(Cloud Native) Apps for Spring Developer 
  - http://www.slideshare.net/SpringCentral/12-factor-cloud-native-apps-for-spring-developers
- Building and running 12 factor microservies on docker
  - https://www.packtpub.com/books/content/building-and-running-12-factor-microservices-docker-part-1
  - https://www.packtpub.com/books/content/building-and-running-12-factor-microservices-docker-part-2
- The 12 Factor PHP App
  - http://slashnode.com/the-12-factor-php-app-part-1/
  - http://slashnode.com/the-12-factor-php-app-part-2/
