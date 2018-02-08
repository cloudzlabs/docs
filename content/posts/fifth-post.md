---
date: "2018-02-07T10:25:22+09:00"
title: "Spring Cloud Contract Test"
authors: ["1000jaeh"]
categories:
  - Spring
tags:
  - Spring
  - Test
  - Spring Cloud
  - CDC
description: "alksdjflaksdjflakdsjflkasdjfkladsjflakdsjfalksd"
draft: false
---
spring-cloud-contract

스펙 

왜 나왔나. 

용어 


* 소비자 주도 계약 (Consumer Driven Contract): 소비자가 생산자의 API변화를 주도하는 접근 법
* Producer: API를 제공하는 서비스
* Consumer: 프로듀서의 API를 소비하는 서비스
* Contract: 생산자와 소비자 사이의 합의된 API의 모습


* DSL: 계약 정의 언어


spring cloud contract 


이런환경에서 
￼
테스트환경이 유닛 / 통합 테스트에서 다른 마이크로서비스를 모의하는 경우에 초점을 맞춰 
* 장점
    * 매우 빠른 피드백
    * 인프라 요구 사항 없음
* 단점
    * 서비스의 구현자는 스텁을 생성하므로 실제 서비스와 관련이 없다.
    * 잘못된 것이 테스트에 통과하여 프로덕션으로 감




앞서 언급한 문제를 해결하기 위해 스텁러너가 있는 스프링 클라우드컨트랙트 베리파이어가 생성됨 
즉, 주요 아이디어는 마이크로 서비스의 전체 세계를 설정할 필요 없이 매우 빠른 피드백을 제공하는 것!! 


￼
스텁에서 작업하는 경우, 필요한 응용프로그램은 응용 프로그램에서 직접 사용하는 응용 프로그램뿐입니다. 
￼
스프링 클라우드 컨트랙트 베리파이어는 사용중인 스텁이 호출중인 서비스에 의해 생성되었음을확실하게 보여줍니다. 
또한 당신이 그것을 사용할 수 있다면 그것은 그들이 생산자 측에 대해 테스트 되었다는 것을 의미합니다. —> 테스트 코드로 작성하니깐 
다른 말로 하면 - 당신은 그 스텁을 신뢰할 수 있습니다. 


스텁러너가 있는 스프링 클라우드 컨트랙트 베리파이어의 주요 목적은 다음과 같습니다 
* 와이어목/메시징 스텁(클라이언트 개발시 사용됨)이 실제 서버 측 구현이 수행하는 작업을 정확히 수행하는지 확인하기 위해
* ATDD방법 및 msa 촉진시키기위해
* 양쪽에서 즉시 볼 수 있는 계약의 변경사항을 게시할 수 있는 방법을 제공
* 서버측에서 사용되는 보일러플레이트 테스트코드 생성


클라이언트 측 
테스트중에 와이어목 인스턴스/메시징 라우트를 가동시키고 서비스 Y를 시뮬레이트하는 것을 원합니다. 적절한 스텁 정의로 인스턴스를 공급하려고 합니다. 
그 스텁 정의는 유효해야하며 서버 측에서 재사용가능해야합니다. 
—> 이 측면에서는 스텁 정의에서 요청 스텁에 패턴을 사용할 수 있으며 응답에 대한 정확한 값이 필요합니다. 


서버 측 
스텁을 개발할 때부터 서비스 Y가 되므로 실제로 구현과 닮았는지 확인해야합니다. 스터브가 한가지 방식으로 작동하고 프로덕션 응용프로그램이 다른 방식으로 작동하는 상황을 가질 수 없습니다. 
그렇기 때문에 제공된 스텁 수락 테스트가 생성되어 스텁에서 정의한 것과 동일한 방식으로 애플리케이션이 작동하는지 확인할 수 있습니다. 
요약: 이 부분에서는 스텁정의에서 요청으로 정확한 값이 필요하며 응답확인을 위해 패턴/메소드를 사용할 수 있습니다. 




spring cloud contract verifier 기능 


요약 
* HTTP / Messaging 스텁(클라이언트 개발 시 사용됨)이 실제로 서버 측 구현이 수행하는 작업을 정확히 수행하는지 확인함
* Tdd와 MSA를 수용
* 커뮤니케이션의 양면에서 즉시 볼 수 있는 계약을 게시하는 방법 제공
* 서버측에서 사용되는 보일러블레이트 테스트 코드 생성


JVM 기반 애플리케이션의 CDC 개발을 가능하게하는 툴 
계약 정의 언어 DSL와 함꼐 제공 
아래 내용을 제공 
* 클라이언트 코드(클라이언트 테스트)에 대한 통합 테스트를 수행할 때, Wiremock이 사용할 JSON스텁 정의. 테스트코드는 여전히 손으로 작성해야하며, 테스트 데이터는 스프링 클라우드 컨트랙터 베리파이어가 작성한다.
* 메시징 경로를 사용하고 있다면 스프링 스트림, 스프링 에이엠큐피, 카멜과 통합 가능 —> 개별 설정 가능
* API의 서버 측 구현이 계약 (서버 테스트)을 준수하는지 확인하는데 사용되는 수락테스트(JUnit 또는 Spock). 전체 테스트는 스프링 클라우드 컨트랙트 베리파이어에의해 생성됩니다.


베리파이어  HTTP 
베리파이어 메시징: 스프링 클라우드 컨트랙트 베리파이어는 메시징을 통신수단으로 사용하는 애플리케이션을 검증할 수 있게 한다. 
스텁러너: 코어, 제이유닛 룰(스텁을 쉽게 다운로드함), 스프링클라우드, 메시징,  인티그레이션, 스트림, amqp, 카멜 
    스프링클라우드 컨트랙트 베리파이어를 사용하는 동안 발생하ㄴㄹ 수 있는 문제 중 하나는 생성된 와이어목 JSON 스텁을 서버측에서 클라이언트 측 (또는 다양한 클라이언트)으로 전달하는 것이 엇습니다. 메시징의 클라이언트 측 세대 측면에서도 마찬가지 입니다. JSON 파일을 복사하거나 수동으로 메시징용 클라이언트측을 설정하는 것은 문제가 되지 않습니다. 그래서 우리는 자동으로 스텁을 다운로드하고 실행할 수 잇는 스프링클라우드컨트랙트 스텁 러너를 소개할 것입니다. 
DSL: 그루비로 작성되었지만 아주 작은 하위 집합(즉, 리터럴, 메소드 호출 및 클로저)만을 사용하므로 실제로 필요하지 않다(그루비 지식은). 게다가 dsl은 dsl자체에 대한 지식이 없어도 프로그래머가 읽을 수 있도록 설계되었습니다. 정적으로 입력됨 




적용방법 


스프링 클라우드 콘트랙트 
1. wiremock 작성
2. 스텁을 자동으로 등록 (src/test/resources/mappings와 stubs의 사용자정의 위치에서 자동 매핑, )
3. 스텁바디 지정
    1. 파일을 사용하여 스텁바디를 지정 (src/test/resources/_files
    2. JUnit 룰을 사용
4. wiremock 과 spring mvc mock: 와이어목 스터브를 스프링 목레스트서비스서버에 로드할 수 잇음
5. 레스트닥을 이용하여 스텁 자동 생성 (target/snippets/stubs에 생성) —> wiremock(), jsonPath(), contentType()ㅇ로 생성가능하나 같이 사용못함
6. 레스트닥을 이용하여 계약 생성 -> dsl파일과 문서 생성 —> wiremock과 결합하면 계약과 스텁을 같이 얻는다.


스프링 클라우드 콘트랙트 베리파이어 




구조 






어떤식으로 동작하는가. 


* Consumer flow 1
    * TDD 시작 - 기능 테스트 작성
    * 로컬에서 API를 변경하기 위해서 프로듀서 코드를 복제
    * 복제된 프로듀서 코드 안에서 계약을 스텁으로 변경하고 그것을 로컬에 설치
    * 소비자 코드에서 스텁러너를 오프라인 모드로 전환
    * 프로듀서의 스텁을 다운로드 하기 위해서 스텁러너를 설정
    * api와 테스트들 위에서 red-green-refactor
    * test가 그린이 되어 api가 허용될때까지 위의 프로세스를 반복
    * 계약 제안서와 함께 제작자에게 홍보를 보낸다.
* producer flow
    * PR을 받는다.
    * 자동생성된 테스트를 통과시키는 누락된 구현을 작성한다.
    * PR을 머지하고, app과 스텁을 같이 jar로 배포한다.
* consumer flow 2
    * 제작자가 스텁을 업로드하면 스텁러너의 오프라인 모드를 끈다.
    * 스텁이 포함된 저장소의 URL을 제공하여 스텁러너를 구성한다.
    * 만약 프로듀서가 api를 깨고 변경하면 테스트가 중단된다.


왜 스프링 클라우드 컨트랙트 베리파이어? 


* 메시징을 이용해 CDC가 가능하다.
* 명확하고 사용하기 쉽고 정적으로 입력된 DSL
* 정의된 계약으로부터 테스트 자동 생성
* 스텁러너 기능 - 스텁은 자동으로 넥서스나 아티팩토리로부터 런타임에 다운로드 한다.
* 스프링 클라우드 인테그레이션 - 디스커버리 없이 통합 테스트 가능




참고 


샘플 깃: https://github.com/spring-cloud-samples/spring-cloud-contract-samples 
슬라이드: https://www.slideshare.net/MarcinGrzejszczak/consumer-driven-contracts-and-your-microservice-architecture?qid=a5a4dbef-6d55-42ae-b0db-068274b9c37e&v=&b=&from_search=4 
소비자 주도 계약: http://blog.java2game.com/225 
스프링 클라우드 컨트랙트: https://cloud.spring.io/spring-cloud-contract/spring-cloud-contract.html#_spring_cloud_contract 
https://pt.slideshare.net/MarcinGrzejszczak/spring-cloud-contract-and-your-microservice-architecture

http://specto.io/blog/spring-cloud-contract


http://specto.io/blog/spring-cloud-contract2017년 3월 16일 


http://specto.io/blog/spring-cloud-contract


http://free-strings.blogspot.kr/2016/01/soa.html


https://martinfowler.com/articles/microservice-testing/#testing-integration-introduction
