---
date: "2018-02-06T19:47:56+09:00"
title: "Forth-Post"
authors: ["1000jaeh", "blingeeeee"]
categories:
  - Blog
tags:
  - Cloud
draft: false
---
# REST API란 무엇인가?

## 정의

api ==> application programming interface

- 응용프로그램에서 사용할 수 있도록, **운영체제나 프로그래밍 언어가 제공하는 기능을 제어할 수 있게 만든 인터페이스**를 뜻한다. 주로 파일제어, 창제어, 화상처리, 문자제어등을 위한 인터페이스를 제공한다.

rest ==> representational state transer

- www와 같은 분산 하이퍼미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식이다. 이 용어는 로이 필딩의 2000년 박사학위 논문에서 소개되었다. 필딩은 http의 주요 저자 중 한 사람이다. 이 개념은 네트워킹 문화에 널리 퍼졌다.엄격한 의미로 rest는 네트워크 아키텍처 원리의 모음이다. 여기서 '네트워크 아키텍처 원리'란 자원을 정의하고 자원에 대한 주소를 지정하는 방법 전반을 일컫는다. 간단한 의미로는, 웹상의 자료를 http위에서 soap이나 쿠키를 통한 세션 트랙킹 같은 별도의 전송 계층 없이 전송하기 위한 아주 간단한 인터페이스를 말한다. 이 두가지의 의미는 겹치는 부분과 충돌되는 부분이 있다. 필딩의 Rest 아키텍처 형식을 따르면 http나 www이 아닌 아주 커다란 소프트웨어 시스템을 설계하는 것도 가능하다. 또한, 리모트 프로시저 콜 대신에 간단한 xml과 http 인터페이스를 이용해 설계하는 것도 가능하다.
필딩은 rest원리를 따르는 시스템은 종종 restful이란 용어 지칭된다. 열정적인 rest옹호자들은 스스로를 restafrians이라고 부른다.eltm

# 왜 지금 REST API의 관심이 많아졌을까?

## 배경

rest는 왜 나온것인가?
- 전통적으로 RPC API는 CORBA 및 Windows COM과 같은 API 인터페이스 및 방법 측면에서 설계되었습니다. 시간이 갈수록 점점 더 많은 인터페이스와 메소드가 도입됩니다. 최종 결과는 엄청나게 많은 수의 인터페이스와 메소드가 될 수 있으며 각 메소드는 다른 메소드와는 다릅니다. 개발자는 올바르게 사용하기 위해 각 요소를 신중하게 배워야하는데, 이는 시간이 많이 걸리고 오류가 발생하기 쉽습니다.

벤더 종속
사용하기 어렵다.

즉, 이전에는 클라이언트와 백엔드가 견고하게 커플링 되어있다.
--> 어플리케이션의 규모가 커지고 진화하기가 어렵다. --> (분산응용프로그램에서...)

따라서, rest가 이런 견고한 커플링을 끊어주고 (느슨한 결합) 좀더 유연하게 만들수 있게 해준다.

--> 최종 목적은 응용프로그램 개발을 간단하고 사용하기 쉬운 상태를 유지해야하는것!

https://stackoverflow.com/questions/5320003/why-we-should-use-rest
https://stackoverflow.com/questions/1368014/why-do-we-need-restful-web-services

다른 rpc(remote procedure call)을 사용하지 않아야한다는 것은아님(상황에 맞게!!) 

## REST사용의 목적

이걸 어떻게 하면 잘 쓸수 있는가? (낮은 수준의 커플링을 유지할 수 있는가????)


이전 환경들. 
- 요즘은 예전과 달리 소비를 위한 장비 및 프로그램이 다양합니다. 반대로 말하면 예전에는 어떠한 인터넷 컨텐츠를 소비하기위한 장비나 프로그램이 몇개 없었습니다. 그렇기때문에 서버와 클라이언트가 거의 1:1이였습니다. 그러나 요즘은 역시 서버는 1인데 클라이언트가 굉장히 다양해졌습니다. 안드로이드는 OS 버전도 굉장히 다양하고 단말기마다 굉장히 다른 특성을 갖기도 합니다. 또 IOS도 있고, 컴퓨터 브라우져의 종류도 많아졌죠.

- 그래서 예전처럼 하나의 클라이언트를 위한 서버를 구성하는건 비효율적인 일이 되어버렸습니다. 하나의 서버로 여러대의 클라이언트를 대응하도록 할때 필요한것이 RESTFul API입니다.

- client 1 : server 1의 환경들
- -->
- client n : server 1의 환경들로 변화.

- 커플링이 낮아야한다.!!!! --> REST!!

---

레스트의 주요한 목표
1. 구성 요소 상호작용의 규모 확장성
1. 인터페이스의 범용성
1. 구성 요소의 독립적인 배포
1. 중간적 구성요소를 이용해 응답 지연 감소, 보안을 강화, 레거시 시스템을 인캡슐레이션

뭐를 rest라고 부르나??
단순히 url로 호출?? http method로???

# 그러면, 단순하게 URL로 표현한 것들을 REST API라고 할 수 있을까?

REST 원칙

레스트 아키텍처에 적용되는 6가지 제한 조건
다음 제한 조건을 준수하는 한 개별 컴포넌트는 자유롭게 구현할 수 있다.
1. 클라이언트/서버 구조: 일관적인 인터페이스로 분리되어야 한다.
1. 무상태: 각 요청 간 클라이언트의 콘텍스트가 서버에 저장되어서는 안된다.
1. 캐시 처리 가능 : 월드 와이드 웹에서와 같이 클라이언트는 응답을 캐싱할 수 이써야 한다. 잘 관리되는 캐싱은 클라이언트-서버 간 상호작용을 부분적으로 또는 완전하게 제거하여 scalability와 성능을 향상시킨다.
1. 계층화: 클라이언트는 보통 대상 서버에 직접 연결되었는지, 또는 중간 서버를 통해 연결되었는지를 알 수 없다. 중간 서버는 로드 밸런싱 기능이나 공유 캐시 기능을 제공함으로써 시스템 규모 확장성을 향상시키는데 유용하다.
1. code on demand (optional) 자바 애플릿이나 자바 스크립트의 제공을 통해 서버가 클라이언트가 실행시킬 수 있는 로직을 전송하여 기능을 확장시킬 수 있다.
1. 인터페이스 일관성: 아키텍처를 단순화시키고 작은 단위로 분리(decouple)함으로써 클라이언트-서버의 각 파트가 독립적으로 개선될 수 있도록 해준다.

레스트 인터페이스의 원칙에 대한 가이드
1. 자원의 식별
요청 내에 기술된 개별 자원을 식별할 수 있어야 한다. 웹 기반의 레스트 시스템에서의 URI의 사용을 예로 들 수 있다. 자원 그 자체는 클라이언트가 받는 문서와는 개념적으로 분리되어 있다. 예를 들어, 서버는 데이터베이스 내부의 자료를 직접 전송하는 대신, 데이터베이스 레코드를 HTML, XML이나 JSON등의 형식으로 전송한다.
1. 메시지를 통한 리소스의 조작
클라이언트가 어떤 자원을 지칭하는 메시지와 특정 메타데이터만 가지고 있다면 이것으로 서버 상의 해당 자원을 변경, 삭제할 수 있는 충분한 정보를 가지고 있는 것이다.
1. 자기 서술적 메시지
각 메시지는 자신을 어떻게 처리해야 하는지에 대한 충분한 정보를 포함해야 한다. 예를 들어, MIME type과 같은 인터넷 미디어 타입을 전달한다면, 그 메시지에는 어떤 파서를 이용해야 하는지에 대한 정보도 포함해야 한다. 미디어 타입만 가지고도, 클라이언트는 어떻게 그 내용을 처리해야할 지 알 수 있어야 한다. 메시지를 이해하기 위해 그 내용까지 살펴봐야한다면, 그 메시지는 자기 서술적이 아니다. 예를들어, 단순히 application/xml이라는 미디어 타입은, 실제 내용을 다운로드 받지 않으면 그 메시지만 가지고는 무엇을 해야할지에 대해 충분히 알려주지 못한다.
1. 애플리케이션의 상태에 대한 엔진으로서 하이퍼미디어
만약에 클라이언트가 관련된 리소스에 접근하기를 원한다면, 리턴되는 지시자에서 구별될 수 있어야 한다. 충분한 콘텍스트 속에서의  URI를 제공해주는 하이퍼텍스트 링크의 예를 들 수 있겠다.

참고: 위키

# REST API는 어떻게 만드는가?

리소스를 설계
URI를 설계
HTTP 메소드 설계

## 방향성

Resource의 위치를 URI만으로 쉽게 파악할 수 있도록.

HTTP Method 를 통해 어떤 행동을 하는지.

## URI 설계

### 소문자를 사용하라

### 하이픈을 사용하라

### 확장자를 사용하지 말라

### CRUD는 URI에서 하용하지 말라

## HTTP Method

### CRUD

### Code (Success, Error)

## Resource

### 컬렉션과 도큐먼트

## 설계를 할때 REST API Service의 크기는 어느정도일까?

서비스의 사이즈?
https://martinfowler.com/microservices/

http://www.apiacademy.co/resources/api-design-304-api-design-for-microservices/

참고: http://www.rfc-editor.org/rfc/rfc3986.txt

http method의 알맞은 역할
참고: http://blog.remotty.com/blog/2014/01/28/lets-study-rest/

레스트로 설계할때는 리소스에 맞춰서
리소스가 같다면 URI는 하나여야 한다.!

응답상태 코드 설계

참고: http://blog.remotty.com/blog/2014/01/28/lets-study-rest/
https://spoqa.github.io/2013/06/11/more-restful-interface.html
https://spoqa.github.io/2012/02/27/rest-introduction.html

URI란
https://ko.wikipedia.org/wiki/%ED%86%B5%ED%95%A9_%EC%9E%90%EC%9B%90_%EC%8B%9D%EB%B3%84%EC%9E%90

URL??
https://ko.wikipedia.org/wiki/URL
https://spoqa.github.io/2012/02/27/rest-introduction.html
https://beyondj2ee.wordpress.com/2013/03/21/%EB%8B%B9%EC%8B%A0%EC%9D%98-api%EA%B0%80-restful-%ED%95%98%EC%A7%80-%EC%95%8A%EC%9D%80-5%EA%B0%80%EC%A7%80-%EC%A6%9D%EA%B1%B0/
https://hackernoon.com/restful-api-designing-guidelines-the-best-practices-60e1d954e7c9

# 만들어진 API들의 테스트는 어떻게 진행해야 하는가?

REST API 테스트

api 테스트
https://www.slideshare.net/genycho/rest-api-64569519
http://blog.saltfactory.net/create-and-test-rest-conroller-in-spring/

# API 어떻게 배포하고 사용하는가?

배포(ci/cd)
사용

# 만들어진 API들은 어떤식으로 관리해야 할까?

문서화
https://cloud.google.com/apis/design/documentation

버저닝
https://stackoverflow.com/questions/389169/best-practices-for-api-versioning

http://blog.restcase.com/restful-api-versioning-insights/

http://www.baeldung.com/rest-versioning

https://cloud.google.com/apis/design/versioning









https://www.researchgate.net/post/What_is_ROA_and_how_is_it_different_from_SOA


hateoas
https://spring.io/understanding/HATEOAS
https://martinfowler.com/articles/richardsonMaturityModel.html
http://blog.woniper.net/219
http://anster.tistory.com/163
https://en.wikipedia.org/wiki/HATEOAS


spring rest, hateoas 
rest test


https://semaphoreci.com/community/tutorials/testing-rest-endpoints-using-rest-assured

http://rest-assured.io/

given when then
https://martinfowler.com/bliki/GivenWhenThen.html
http://d2.naver.com/helloworld/568425
https://martinfowler.com/articles/microservice-testing/#anatomy-modules

https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/

https://spring.io/guides/gs/rest-hateoas/

계약
https://martinfowler.com/bliki/IntegrationContractTest.html
http://microservices.io/patterns/testing/service-integration-contract-test.html

http://cloud.spring.io/spring-cloud-contract/spring-cloud-contract.html

1. api들의 묶음의 단위크기는 어느정도로?
2. 개발시 유의할게 있나???
3. 하테오아스는?
4. 하테오아스 역할은?
5. 왜 사용하려고 하는거지?
6. 장단점은???
7. 만들어진거 테스트는 어떤식으로 해야하지??
8. 테스트할때 확인해야할 사항들이 따로 있는지??
9. 베스트한 패턴? 이 있는지?

# 운영

1. 배포는 어떻게 해?
1. 에이피아이 버전관리는 어떻게?
1. 이걸 사람들이 사용하게 하려면?? --> 문서화
1. 수동으로? 아니면 다른 방법이 있나?

