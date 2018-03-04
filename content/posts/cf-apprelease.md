---
date: "2018-02-27T11:07:05+09:00"
title: "CF에 Cloud Native Application 배포하기 "
authors: ["sya"]
categories:
  - posts
tags:
  - Cloud Native Application
  - Cloud Foundry
  - CF
draft: true
---

PaaS에 Cloud Application을 배포하는 전 과정을 정리해보았습니다. Java로 만든 Cloud Native Application을 CF에 배포하는 간단한 실습인데요, 개발 환경 세팅부터 PaaS에 애플리케이션 배포하고 애플리케이션 동작을 조작할 수 있습니다.  
초보 개발자 위주의 간단한 실습이라 처음 접하는 사람도 쉽게 따라할 수 있으실꺼예요. 처음부터 끝까지 따라하면서 Cloud Native Application 개발을 시작해보세요! 

## 실습 과정
----------------
아래 과정의 순서대로 진행합니다.

## 준비 사항
----------------
### 전제조건
- **CF 기반의 PaaS**
    - PaaS에 애플리케이션을 배포, 업데이트, 삭제, 스케일링 등의 작업을 할 수 있는 CF기반의 PaaS가 있어야 합니다.
- 시스템 요구사항
    - Java (1.8 권장)

### 개발 환경 설정
Cloud Native Application을 만들기 위해 필요한 개발 환경에 대한 설명과 설치 방법입니다.

#### Framework
##### [Spring Boot](http://projects.spring.io/spring-boot)
  - Spring Framework를 사용하는 프로젝트를 쉽고 빠르게 개발할 수 있는 Spring Framework의 서브 프로젝트입니다.

#### Language
##### Java
  - JDK와 같이 설치됩니다.

#### Tool
개발에 필요한 도구들입니다.

##### [Maven](https://maven.apache.org) 
 - 프로젝트 의존성 관리, 라이브러리 관리, 프로젝트 생명주기 관리, 빌드 기능을 제공하는 프로젝트 관리 도구입니다.
 - 설치 방법
   - STS에서 제공하는 Maven 플러그인을 사용하므로 별도의 설치가 필요하지 않습니다.

##### STS 
 - STS는 spring에서 제공하는 이클립스 기반의 Spring 개발에 최적화되어 있는 개발 도구입니다.
 - 설치 방법

    1. https://spring.io/tools/sts/all 에 접속합니다.
    2. 컴퓨터 OS 사양에 맞는 최신 파일을 다운로드합니다. 
    3. 다운로드한 .zip 파일을 원하는 폴더에 압축 해제 한 후, sts-version.RELEASE 폴더 안에 STS 실행 파일을 실행시킵니다. 
    4. worksapce를 지정해주면 STS 설치가 완료됩니다.

##### JDK
  - JDK는 Java Development Kit으로, Java로 애플리케이션을 개발하기 위해 설치해야 합니다.
  - JDK 버전은 안정성을 위해 최신 버전보다 1.8버전 사용을 권장합니다.  
  - 설치 방법

    1. JDK 설치 전, 설치 여부를 확인합니다.

        > a. 명령 프롬프트 실행
        >
        > - Window OS: 윈도우키 + R 을 눌러 실행창에 cmd 입력하면 명령 프롬프트가 실행됩니다. 
        > - Mac OS: 터미널을 실행합니다.
        >
        > b. 명령 프롬프트에 `java -version` 입력  
        > c. 화면에 Java의 버전이 표시되지 않으면 JDK가 미설치되어있으므로 2번 과정을 이어서 하세요.

    2. [JDK 다운로드 페이지](http://www.oracle.com/technetwork/java/javase/downloads/index.html)에 접속합니다.
    3. JDK → Accept License Agreement 체크 → 컴퓨터 OS 정보에 맞는 File 선택, 다운로드 후 설치합니다.
    4. Java 환경변수 설정 (Window OS만 해당)  
    JDK를 새로 설치한 경우, Java 환경 변수 설정이 필요합니다.  

        > a. 제어판 → 시스템 → 고급시스템 설정을 실행시킵니다.  
        > b. 다이얼로그 하단 오른쪽 환경 변수 클릭합니다.  
        > c. 사용자 변수 항목에서 새로 만들기를 클릭합니다.  
        >    
        > > - JAVA_HOME 설정
        >       - 변수 이름: **JAVA_HOME**
        >       - 변수 값: JDK 경로
        >            - 예시. C:\Program Files\Java\jdk1.8.0_77
        >
        > d. 시스템 변수 항목에서 환경 변수를 다음과 같이 설정합니다.
        >
        > >  - 변수 이름: **PATH**
        >    - 변수 값
        >       - **%JAVA_HOME%\bin**
        >       - PATH 변수가 이미 등록되어있는 경우: 변수 값 뒤에 ;**%JAVA_HOME%\bin** 입력
        >
        > e. 확인을 클릭하여 모든 창을 닫습니다.
    4. 1번 과정을 따라서 JDK가 정상적으로 설치되었는지 확인합니다.

#### CF CLI
  - PaaS에 애플리케이션을 배포할 때 사용할 명령어 도구 CF CLI( Cloud foundry Command
    line interface)를 설치합니다. 
  - 자세한 설치 방법이나 CF 명령어 정보는 [CF CLI 사용하기]()포스트를 참고하세요.
  - 설치 방법
    1. https://github.com/cloudfoundry/cli/releases에 접속합니다.
    2. 컴퓨터 OS 정보에 맞는 파일을 다운로드합니다. 
    3. 다운로드한 파일의 압축을 풀어서, CF CLI를 설치합니다. 
    4. 명령 프롬프트에 `cf` 명령어를 실행합니다.  
    화면에 cf 정보가 표시되면 정상적으로 설치가 완료된 것입니다.

#### 버전 관리 & 원격 저장소 
##### Github 
 - 버전 관리 도구입니다.
 - 설치 방법
    - STS에서 제공하는 플러그인을 사용하므로 별도의 설치가 필요하지 않습니다.

## 샘플 프로젝트 개발하기
Cloud Native Application 샘플 프로젝트를 Github에서 다운로드해서, STS에서 실행해 봅니다. 

### 샘플 프로젝트 다운로드 
Spring에서 제공하는 샘플 프로젝트를 사용합니다.

1. 프로젝트 소스를 다운로드 받기 위해 https://github.com/cloudfoundry-samples/hello-spring-cloud로 이동합니다.
2. **Clone or download** 버튼을 클릭하여 샘플 프로젝트의 주소를 복사합니다.
3. STS를 실행합니다. 
4. Git Repositories view를 추가합니다. STS 메뉴의 **Window → Show view → Other.... → Git → Git Repositories**을 선택하세요. 
5. Git Repositories View 상단에, **Clone a repository** 또는, 상단의 왼쪽에서 세 번째 아이콘 **Clone a Git repogitory and add the clone to this view**를 클릭하고 **CloneURI**를 선택합니다.
6. 복사한 Github 주소가 **Location URI**에 입력되었는지 확인합니다.
7. **Next →  Next → Finish**를 선택해서 소스 코드를  로컬 레파지토리에 다운로드합니다.

### 샘플 프로젝트 STS 설정
1. STS의 Git Repositories View에서 **hello-spring-cloud** 프로젝트 선택 후 마우스 오른쪽 버튼 클릭 → **Import Maven Projects**를 선택합니다.
2. **hello-spring-cloud** 프로젝트가 Package Explorer View에 애플리케이션이 정상적으로 추가되었는지 확인합니다. 
3.  JDK 설정  

   > a. STS 메뉴 **Window → Preference → Java → Installed
        JREs**를 선택합니다.  
   > b. 화면에 JRE 만 있다면 JDK를 추가합니다.  
   > c. **Add → Next → Directory** 선택 후 JDK 경로를 입력합니다.   
   > d. 추가한 JDK의 체크박스에 체크하여 defalut로 지정해 줍니다. 

### 샘플 프로젝트 설명
hello-spring-cloud 프로젝트는 spring-boot-starter Maven 프로젝트입니다.  
Spring Boot의 자세한 가이드는  [Spring Boot
Document](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-documentation)를
참고하세요.

#### hello-spring-cloud 특징
-   hello-spring-cloud는 클라우드 환경과, 기존과 같은 물리 서버 환경
    모두에서 서비스할 수 있는 형태로 구성되어 있습니다. 
-   hello-spring-cloud 프로젝트에 연결되는 서비스는 아래와 같습니다. 
    -   **Maria DB**
    -   **Redis**
    -   **Mongo DB**
    -   **Rabbit MQ**

#### hello-spring-cloud 구조
-   hello-spring-cloud
    -   src/main/java 
    -   src/main/resource
        -   static : CSS, Javascript의 Static Resource 파일
        -   template : UI Templates 파일
        -   application.properties : Spirng boot 기본 설정 파일 
    -   src/test/java
    -   src
    -   manifest.yml : PaaS 애플리케이션 배포 부가정보 설정 파일로
         애플리케이션 이름 및 바인딩 되는 서비스 정보들을 기입
    -   pom.xml: maven 설정 파일

### 샘플 프로젝트 빌드하기
#### 로컬 실행
hello-spring-cloud 프로젝트를 로컬 환경에서 구동해봅니다.
    
1. 애플리케이션 run을 하는 방법 2가지가 있습니다.
    - STS의 Package Explorer View에서 **hello-spring-cloud** 선택 → 마우스 오른쪽 버튼 → **Run AS** → **Spring Boot App** 선택 
    - Boot Dashboard View에서 **local** 하위의 **hello-spring-cloud** 프로젝트 선택 → 마우스
            오른쪽 버튼 → **(Re)start** 선택

2. 웹 브라우저에서 [http://localhost:8080](http://localhost:8080/welcome)으로 이동합니다.
3. 화면에 **Spring Cloud Demo Application** 메시지가 출력되면 정상적으로 실행된 것입니다.
 
 ![Image of URL](docs/contents/images/cfurl.png)

#### 빌드 
PaaS에 배포할 JAR 파일을 만들기 위해서 빌드를 진행합니다.

- 빌드 설정 
    1. STS의 Package Explorer View에서 **hello-spring-cloud** 선택
        → 마우스 오른쪽 버튼 → **Run AS** → **Run configurations...**
        → **Maven build**를 더블 클릭합니다.
    2.  configuration 창에서 다음 설정 정보를 입력합니다.
        - Name: **hello-spring-cloud**
        - Goal : **clean install**
    3.  **Apply** 선택해서 설정 정보를 저장합니다. 

-  빌드 
    1. STS의 Package Explorer View에서 **hello-spring-cloud** 선택
        → 마우스 오른쪽 버튼 → **Run AS** → **Run configurations...**
        → **Maven build**를 선택하여 빌드합니다.
    2. workspace의 샘플 프로젝트 하위에 **tartget** 폴더에 JAR 파일이 생성되었는지 확인합니다. 

### 애플리케이션 배포하기
샘플 애플리케이션을 CF CLI를 이용해서 PaaS에 배포해보겠습니다. 

#### PaaS에 애플리케이션 배포하기 
##### 1. manifest.yml 작성
manifest.yml은 PaaS에 애플리케이션을 배포하는데 필요한 정보들을 작성하는 설정 파일입니다.

```Emacs
applications:
- name: hello-spring-cloud
    instances: 1
    host: hello-spring-cloud-${random-word}
    path: target/hello-spring-cloud-0.0.1.BUILD-SNAPSHOT.jar
```


##### 2. PaaS 로그인
1. 명령 프롬프트을 실행합니다. 
2. 명령 프롬프트에서 **hello-spring-cloud** 프로젝트 경로로 이동합니다.
- 예시

    ```Emacs
    $ cd C:/Users/hello-spring-cloud
    ```
3.  PaaS에 로그인을 하기 위해 ` cf login -a [PaaS API URL]` 같이 명령어를 입력합니다.
    - `[PaaS API URL]` 로그인할 PaaS API URL
    - 예시.  
        ```Emacs
        $ cf login -a http://api.paas.sk.com/
        ```

        전 제가 사용하고 있는 Open PaaS의 PaaS API인  http://api.paas.sk.com URL을 사용하였어요. 이 PaaS는 SK 주식회사 C&C에서 만든 PaaS로, 권한이 있는 사람만 사용이 가능해서 예시처럼 사용하진 못해요. 준비하신 PaaS 테스트하시길 바랍니다.

4.  PaaS에 가입한 Email과 Password 정보를 입력합니다. 아래 예시를 참고하세요.
        
    ```Emacs
    hello-spring-cloud seoyoungahn$ cf login -a http://api.paas.sk.com/
    API 엔드포인트: http://api.paas.sk.com/

    Email> abc@sk.com

    Password> 
    인증 중...
    확인
    ```

5.  애플리케이션을 배포할 조직(Org)과 영역 (Space)을 선택합니다.   
    로그인이 성공하면 애플리케이션을 배포할 곳을 선택하는 창이 나옵니다.  
    조직을 선택하면 조직 내 존재하는 영역 목록이 표시되고, 숫자키를 눌러서 내가 배포할 위치를 선택합니다. 선택을 마치면 API 엔드포인트, 사용자, 내가 선택한 조직과 영역 정보가 표시되요.  
    전 dtlab 조직의 dev 영역에 애플리케이션을 배포할꺼예요.

    ```Emacs
    조직 선택(또는 Enter를 눌러 건너뜀):
    1.sollab
    2.dtlab

    Org> 2
    대상 지정된 조직 dtlab

    영역 선택(또는 Enter를 눌러 건너뜀):
    1.dev
    2.prod

    Space> 1
    대상 지정된 영역 dev


    API 엔드포인트:   http://api.paas.sk.com(API 버전: 2.69.0)
    사용자:           abc@sk.com
    조직:             dtlab
    영역:             dev
    ```


##### 3. 서비스 생성
애플리케이션에서 서비스를 사용할 수 있도록 생성, 바인딩 하는
작업이 필요합니다. 

hello-spring-cloud 애플리케이션은 아래의 서비스를 사용할 것입니다.

- MariaDB
- Redis 
- MongoDB 
- Rabbit MQ 

서비스 생성하는 과정은 요약하면 다음과 같습니다.

1. 사용할 서비스의 인스탄스가 이미 생성되어 있는 지 확인.   
2. 생성되어 있는 인스탄스를 그대로 사용하려면 서비스 생성 과정 필요 없음.  
없으면 서비스 인스탄스를 생성해야 한다.  
마켓플레이스에 cf에서 제공하는 서비스 정보를 이용하여 서비스 인스탄스를 생성!

그럼 사용할 서비스 인스탄스가 생성되어있는지 확인부터 해보겠습니다.

###### 3.1 서비스 확인
명령 프롬프트에 `cf services` 명령어를 입력하면 선택한 영역에서 사용할 수 있는 서비스의 목록이 표시됩니다. 

```Emacs
    $ cf services
```

제가 선택한 조직/영역에서 사용할 수 있는 서비스 인스탄스가 뭐가 있는지 볼까요?

    ```Emacs
    $ cf services
    abc@sk.com(으)로 dtlab 조직/dev 영역의 서비스를 가져오는 중...
    확인

    이름                        서비스             플랜                    바인딩된 앱                          마지막 조작                                                                                                                                     
    amqp-service               RabbitMQ         standard                                                 create 성공                          
    mariadb-dtlabs             MariaDB          Mysql-Plan2-100con     dtlabs-registration-service       update 성공                                                                                      
    test-mongo                 Mongo-DB         default-plan           dtlabs-registration-service       create 성공                        
    userbff_autoscaler         CF-AutoScaler    free                   dtlabs-user-bff-service           create 성공
    ```

RabbitMQ, MariaDB, Mongo-DB, CF-AutoSacler 서비스 인스탄스들이 이미 존재하네요. 대부분의 서비스가 앱에 바인딩되어 사용 중인걸 보니 저희 팀원이 만들어서 사용 중인 서비스인 것 같아요.  
전 RabbitMQ 서비스는 이미 만들어진 amqp-service를 사용하고, MariaDB 서비스 인스탄스는 이미 존재하는 인스탄스 말고 제가 새로 만들어서 사용해볼께요.
그 외 서비스는 서비스가 생성되어 있지 않으니 서비스를 직접 생성해야겠네요.

###### 3.2 서비스 생성  
서비스 인스탄스를 생성하려면 내가 사용할 서비스가 CF PaaS에 있어야 합니다.
명령 프롬프트에 `cf marketplace` 입력해서 PaaS에서 제공하는 서비스 중 우리가 사용할 서비스가 있는 지 확인해봅니다. 
 
```Emacs
$ cf marketplace
abc@sk.com(으)로 dtlab 조직/dev 영역에서 서비스를 가져오는 중...
확인

서비스                플랜                                        설명
App-Autoscaler-beta   autoscaler-free-plan                        (Beta Version) Automatically increase or decrease the number of application instances based on a policy you define.
CF-AutoScaler         free                                        Automatically increase or decrease the number of application instances based on a policy you define.
MariaDB               Mysql-Plan1-5con, Mysql-Plan2-100con*       A simple mysql implementation
Mongo-DB              default-plan                                A simple mongo implementation
Mongo-DB-Dev          default*                                    A simple MongoDB service broker implementation-Test
Object-Storage        object-storage-1GB, object-storage-100GB*   A simple object-storage implementation
RabbitMQ              standard                                    RabbitMQ is a robust and scalable high-performance multi-protocol messaging broker.
Redis                 shared-vm                                   Redis service to provide a key-value store
Redis-dev             Free Plen*                                  Shared Redis server

* 해당 서비스 플랜에 연관된 비용이 있습니다. 서비스 인스턴스를 작성하면 이 비용이 발생합니다.

팁: 주어진 서비스의 개별 플랜에 대한 설명을 보려면 'cf marketplace -s SERVICE'를 사용하십시오.

```

우리가 사용할 서비스 MariaDB, Redis, MongoDB, Rabbit MQ 정보를 확인할 수 있어요. 이 정보로 서비스를 생성해볼께요.

서비스 생성 명령어 `cf create-service [서비스 명][plan][생성할 서비스명]`를 이용합니다.  
`[서비스명][서비스 플랜][생성할 서비스 인스타스명]`에 `cf marketplace`로 조회한 서비스 정보를 입력하면 됩니다.  
서비스를 생성한 후엔, 잘 생성되었는 지 `cf services` 명령어를 입력하여 확인합니다.

그럼 서비스를 하나씩 생성해보겠습니다.

- MariaDB 서비스 생성

    1. 마켓플레이스에서 MariaDB 서비스 정보 확인
        ```Emacs
        $ cf marketplace
        abc@sk.com(으)로 dtlab 조직/dev 영역에서 서비스를 가져오는 중...
        확인

        서비스                플랜                                        설명
        MariaDB             Mysql-Plan1-5con, Mysql-Plan2-100con*      A simple mysql implementation

        (생략)
        ```

    2. MariaDB 서비스 생성  
        마켓플레이스에서 확인한 아래의 정보를 토대로, `cf create` 명령어를 실행합니다.  
        `[생성할 서비스 인스타스명]`은 원하는 서비스명을 정해서 입력하면 됩니다. **maridb-service**라고 지어볼까요?

        |  서비스  | 서비스 plan |생성할 서비스명|
        |------  | ---------  |-----------|
        | MariaDB | Mysql-Plan1-5con, Mysql-Plan2-100con*   |maridb-service|

        명령 프롬프트창에 `cf create-service [서비스 명][ 서비스 plan][생성할 서비스 인스타스명]`을 실행합니다.

        ```Emacs
        $ cf create-service MariaDB Mysql-Plan1-5con mariadb-service

        abc@sk.com(으)로 dtlab 조직/dev 영역에 서비스 인스턴스 mariadb-service 작성 중...
        확인
        ```
        '확인' 글자가 표시되면서 서비스가 생성되었습니다.

    3. 명령 프롬프트에 `cf services` 명령어를 입력해서 내 space에
    서비스가 제대로 생성되었는지 확인합니다. 

        ```Emacs
        $ cf services

        이름                        서비스           플랜                   바인딩된 앱            마지막 조작
        mariadb-service            MariaDB        Mysql-Plan1-5con                           create 성공
        ```
    **mariadb-service**  서비스가 잘 생성되었네요. 이제 막 생성한 서비스 인스탄스라 바인딩된 앱이 없어서 아무 정보가 없음을 확인할 수 있습니다.

같은 방식으로, Redis, Mongo DB, Rabbit MQ 서비스 인스탄스를 생성하고, `cf service` 명령어로 확인하면 됩니다. 아래를 참고하세요.

-  Redis, Mongo DB Rabbit MQ 서비스 생성
    1. 마켓플레이스에서 서비스 정보 확인
        
        ```Emacs
        $ cf marketplace
        abc@sk.com(으)로 dtlab 조직/dev 영역에서 서비스를 가져오는 중...
        확인

        서비스          플랜                     설명
        Mongo-DB      default-plan            A simple mongo implementation
        RabbitMQ      standard                RabbitMQ is a robust and scalable high-performance multi-protocol messaging broker.
        Redis         shared-vm               Redis service to provide a key-value store
        
        (생략)
        ```
        
    2. Redis, MongoDB, RabbitMQ 서비스 생성
        - 서비스 정보
        
            |서비스  | 서비스 plan |생성할 서비스명|
            |------  | ---------  |-----------|
            | Mongo-DB | default-plan|mongodb-service|
            | RabbitMQ | standard|amqp-service|
            | Redis | shared-vm|redis-service|

        - MongoDB 서비스 생성
            ```
            $ cf create-service Mongo-DB default-plan mongodb-service
            abc@sk.com(으)로 dtlab 조직/dev 영역에 서비스 인스턴스 mongodb-service 작성 중...
            확인
            ```

        - RabbitMQ 서비스 생성

            ```Emacs 
            $ cf create-service RabbitMQ standard amqp-service
            ```
            
            위에서 말했듯이 전 RabbitMQ 서비스 인스탄스는 이미 생성되어 있는 **amqp-service**를 사용할 것이지만 위의 `cf create-service` 명령어를 실행해보면,  
            **amqp-service** 서비스 인스탄스가 이미 생성되어 있어서 아래와 같이 'Service amqp-service이(가) 이미 있음' 메세지를 볼 수 있죠.

            ```Emacs
            $ cf create-service RabbitMQ standard amqp-service
            abc@sk.com(으)로 dtlab 조직/dev 영역에 서비스 인스턴스 amqp-service 작성 중...
            확인
            Service amqp-service이(가) 이미 있음
            ```

        - Redis 서비스 생성
            
            ```Emacs 
            $ cf create-service Redis shared-vm redis-service
            abc@sk.com(으)로 dtlab 조직/dev 영역에 서비스 인스턴스 redis-service 작성 중...
            확인
            ```

        서비스 생성을 모두 완료했습니다.

    3. 명령 프롬프트에 `cf services` 명령어를 입력해서 내 영역에 서비스가 제대로 생성되었는지 확인해볼께요.

        ```Emacs
        $ cf services
        sya@sk.com(으)로 dtlab 조직/dev 영역의 서비스를 가져오는 중...
        확인

        이름                   서비스           플랜                   바인딩된 앱           마지막 조작
        amqp-service          RabbitMQ       standard                                 create 성공
        mariadb-service       MariaDB        Mysql-Plan1-5con                         create 성공
        mongodb-service       Mongo-DB       default-plan                             create 성공
        redis-service         Redis          shared-vm                                create 성공

        (생략)
        ```
    우리가 방금 만든 4가지 서비스가 모두 잘 생성되어있음 확인할 수 있습니다.

##### 3. 애플리케이션 배포 
1. 명령 프롬프트에서 hello-spring-cloud 폴더 경로로 이동합니다.
    - 예시.
            
            $ cd Documents/workspace/hello-spring-cloud

2. 하위에 **manifest.yml** 파일이 있는 지 확인합니다.
    - 예시. Linux

            $ ls
            1			LICENSE			manifest.yml		src			system.properties
            2			README.md		pom.xml			sya@sk.com		target

    - Window OS

            $ dir

2. 명령 프롬프트에 `cf push`를 입력합니다.

    ```Emacs
    $ cf push
    ```

3. 명령 프롬프트에 표시되는 애플리케이션의 배포 상태를 확인합니다. 애플리케이션이 시작되었다는 메시지가 확인되면 정상적으로 애플리케이션이 배포된 상태입니다.

    ```Emacs
    Manifest 파일 /Users/seoyoungahn/git/hello-spring-cloud/manifest.yml 사용

    sya@sk.com(으)로 dtlab 조직/dev 영역에서 hello-spring-cloud 앱 업데이트 중...
    확인

    hello-spring-cloud-persistent-hypnotization.paas.sk.com 라우트 작성 중...
    확인

    hello-spring-cloud에 hello-spring-cloud-persistent-hypnotization.paas.sk.com 바인드 중...
    확인

    hello-spring-cloud 업로드 중...
    업로드 중인 앱 파일 원본 위치: /var/folders/mc/0_v3hb1j2j71t_g53qg8qjg40000gn/T/unzipped-app423622616
    462.7K, 99 파일 업로드
    Done uploading               
    확인


    abc@sk.com(으)로 dtlab 조직/dev 영역에서 hello-spring-cloud 앱 시작 중...
    Downloading liberty_buildpack...
    Downloading staticfile_buildpack...
    Downloading go_buildpack...
    Downloading java_buildpack...
    Downloading java_offline_buildpack...
    Downloaded staticfile_buildpack
    Downloading ruby_buildpack...
    Downloaded go_buildpack
    Downloading nodejs_buildpack...
    Downloaded liberty_buildpack
    Downloading dotnet_core_buildpack...
    Downloaded java_offline_buildpack
    Downloading python_buildpack...
    Downloaded java_buildpack
    Downloading binary_buildpack...
    Downloaded ruby_buildpack
    Downloading nodejs_buildpack_v167...
    Downloaded nodejs_buildpack
    Downloading php_buildpack...
    Downloaded dotnet_core_buildpack
    Downloading java-test...
    Downloaded python_buildpack
    Downloaded binary_buildpack
    Downloaded java-test
    Downloaded nodejs_buildpack_v167
    Downloaded php_buildpack
    Creating container
    Successfully created container
    Downloading app package...
    Downloaded app package (30.7M)
    Staging...
    -----> Java Buildpack Version: v3.11 | https://github.com/cloudfoundry/java-buildpack.git#eba4df6
    -----> Downloading Open Jdk JRE 1.8.0_111 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_111.tar.gz (0.9s)
        Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.2s)
    -----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (0.0s)
        Memory Settings: -Xss349K -Xmx681574K -XX:MaxMetaspaceSize=104857K -Xms681574K -XX:MetaspaceSize=104857K
    -----> Downloading Spring Auto Reconfiguration 1.12.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.12.0_RELEASE.jar (0.0s)
    Exit status 0
    Staging complete
    Uploading droplet, build artifacts cache...
    Uploading droplet...
    Uploading build artifacts cache...
    Uploaded build artifacts cache (44.9M)
    Uploaded droplet (75.8M)
    Uploading complete
    Destroying container
    Successfully destroyed container

    0 / 1 인스턴스 실행 중, 1 시작 중
    1 / 1 인스턴스 실행 중

    앱 시작됨


    확인

    `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher` 명령을 사용하여 hello-spring-cloud 앱이 시작되었습니다.

    sya@sk.com(으)로 dtlab 조직/dev 영역에서 hello-spring-cloud 앱의 상태 표시 중...
    확인

    요청된 상태: started
    인스턴스: 1/1
    사용법: 256M x 1 인스턴스
    URL: hello-spring-cloud-unskillful-contractor.paas.sk.com, hello-spring-cloud-persistent-hypnotization.paas.sk.com
    마지막으로 업로드함: Sun Mar 4 10:02:58 UTC 2018
    스택: cflinuxfs2
    빌드팩: java-buildpack=v3.11-https://github.com/cloudfoundry/java-buildpack.git#eba4df6 java-main open-jdk-like-jre=1.8.0_111 open-jdk-like-memory-calculator=2.0.2_RELEASE spring-auto-reconfiguration=1.12.0_RELEASE

        상태      이후                     CPU    메모리        디스크      세부사항
    #0   실행 중   2018-03-04 07:08:50 PM   0.0%   944K / 256M   1.3M / 1G
    skcc09n00914:hello-spring-cloud seoyoungahn$ 
    ```


##### 4. 서비스 바인딩
애플리케이션이 정상적으로 배포되었다면, '3. 서비스 생성'과정에서 생성한 서비스를 애플리케이션에 연결시켜야 합니다.  
이것을 서비스 바인딩이라고 하는데요, 서비스 바인딩할 때는 애플리케이션을 재시작해주어 서비스를 적용시켜야 합니다.

그 과정을 명령어를 통해서 보자면,  
1. 명령 프롬프트에 `cf bind-service [애플리케이션명][바인딩할 서비스명]` 명령어로 애플리케이션에 서비스를 바인딩한다.
2. 명령 프롬프트에 `cf restage [애플리케이션명]`으로 애플리케이션을 재시작한다.  
입니다.

mariadb-service를 바인딩해보겠습니다.

- mariadb-service 바인딩

    ```Emacs
    $ cf bind-service hello-spring-cloud mariadb-service
    ```
    ```Emacs
    $ cf restage hello-spring-cloud
    ```

    제가 실행해보니,

    ```Emacs
    $ cf bind-service hello-spring-cloud mariadb-service
    확인
    팁: 환경 변수 변경사항을 적용하려면 'cf restage hello-spring-cloud'을(를) 사용하십시오.
    $ cf restage hello-spring-cloud
    abc@sk.com(으)로 dtlab 조직/dev 영역에서 hello-spring-cloud 앱 다시 스테이징 중...

    Staging app and tracing logs...
   Downloading java_buildpack...
   Downloading go_buildpack...
   Downloading java_offline_buildpack...
   Downloading binary_buildpack...
   Downloading liberty_buildpack...
   Downloading python_buildpack...
   Downloaded go_buildpack
   Downloading php_buildpack...
   Downloaded liberty_buildpack
   Downloading dotnet_core_buildpack...
   Downloaded java_offline_buildpack
   Downloading java-test...
   Downloaded binary_buildpack
   Downloaded java_buildpack
   Downloading ruby_buildpack...
   Downloading nodejs_buildpack_v167...
   Downloading nodejs_buildpack...
   Downloaded php_buildpack
   Downloaded python_buildpack
   Downloading staticfile_buildpack...
   Downloaded dotnet_core_buildpack
   Downloaded java-test
   Downloaded ruby_buildpack
   Downloaded nodejs_buildpack_v167
   Creating container
   Downloaded nodejs_buildpack
   Downloaded staticfile_buildpack
   Successfully created container
   Downloading app package...
   Downloading build artifacts cache...
   Downloaded app package (30.7M)
   Staging...
   Downloaded build artifacts cache (44.9M)
   -----> Java Buildpack Version: v3.11 | https://github.com/cloudfoundry/java-buildpack.git#eba4df6
   -----> Downloading Open Jdk JRE 1.8.0_111 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_111.tar.gz (found in cache)
          Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.3s)
   -----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
          Memory Settings: -Xmx681574K -Xms681574K -XX:MetaspaceSize=104857K -XX:MaxMetaspaceSize=104857K -Xss349K
   -----> Downloading Spring Auto Reconfiguration 1.12.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.12.0_RELEASE.jar (found in cache)
   Uploading droplet...
   Staging complete
   Exit status 0
   Uploading droplet, build artifacts cache...
   Uploading build artifacts cache...
   Uploaded build artifacts cache (44.9M)
   Uploaded droplet (75.8M)
   Uploading complete
   Destroying container

    Waiting for app to start...

    이름:                  hello-spring-cloud
    요청된 상태:           started
    인스턴스:              1/1
    사용법:                256M x 1 instances
    routes:                hello-spring-cloud-unskillful-contractor.paas.sk.com, hello-spring-cloud-persistent-hypnotization.paas.sk.com
    마지막으로 업로드함:   Sun 04 Mar 19:02:58 KST 2018
    스택:                  cflinuxfs2
    빌드팩:                java-buildpack=v3.11-https://github.com/cloudfoundry/java-buildpack.git#eba4df6 java-main open-jdk-like-jre=1.8.0_111
                        open-jdk-like-memory-calculator=2.0.2_RELEASE spring-auto-reconfiguration=1.12.0_RELEASE
    start command:         CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k..
                        -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) &&
                        JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval
                        exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher

        상태      이후                   CPU    메모리          디스크         세부사항
    #0   실행 중   2018-03-04T10:17:48Z   0.0%   76.2M of 256M   157.3M of 1G  
    ``` 
    이런 메세지와 함께, 서비스가 바인딩되고, 앱이 재시작하는 것을 확인할 수 있습니다.

마찬가지로, **redis-service, mongodb-service, amqp-service**도 `cf bind-service`로 서비스를 바인딩하고, `cf restage`로 애플리케이션에 서비스를 적용시킵니다. 3가지 서비스를 한꺼번에 적용하고 restage할께요.

- redis-service 바인딩

    ```Emacs
    $ cf bind-service hello-spring-cloud redis-service
    ```

- mongodb-service 바인딩

    ```Emacs
    $ cf bind-service hello-spring-cloud mongodb-service
    ```

- amqp-service 바인딩
    
    ```Emacs
    $ cf bind-service hello-spring-cloud amqp-service
    ```

- 앱 restage
    바인딩한 서비스를 적용합니다.
    
    ```Emacs
    $ cf restage hello-spring-cloud
    ```

- 앱 서비스 인스탄스 Credential 확인  
명령프롬프트에 `cf env hello-spring-cloud`을 입력하여, **hello-spring-cloud** 앱에 바인딩된 서비스의 Credential 정보가 출력됩니다.

    ```Emacs
    abc@sk.com(으)로 dtlab 조직/dev 영역의 hello-spring-cloud 앱에 사용할 환경 변수를 가져오는 중...
    확인

    시스템 제공:
    {
    "VCAP_SERVICES": {
    "MariaDB": [
    {
        "credentials": {
        "hostname": "123.12.34.56",
        "name": "op_bf120a4d_69d8_401f_bd19_f231d363bddb",
        "password": "9d113775",
        "port": "3306",
        "uri": "mysql://9d113775835c56eb:9d113775835c56eb@172.16.21.32:3306/op_bf120a4d_69d8_401f_bd19_f231d363bddb",
        "username": "9d113775835c56eb"
        },
        "label": "MariaDB",
        "name": "mariadb-service",
        "plan": "Mysql-Plan1-5con",
        "provider": null,
        "syslog_drain_url": null,
        "tags": [
        "mysql",
        "document"
        ],
        "volume_mounts": []
    }
    ],
    "Mongo-DB": [
    {
        "credentials": {
        "hosts": [
        "123.45.67.88:9999"
        ],
        "name": "c573258d-455e-4d6b-84cd-120af2f93657",
        "password": "60635047-3298-46cb-9b9f-1123343ddd",
        "uri": "mongodb://1734a4d7-0da1-4abe-848c-00d37e6ac715:60635047-3298-46cb-9b9f-145da122ef34@155.66.77.99:88888/c5d-455e-4d6b-84cd-120af2657",
        "username": "1734a4d7-0da1-4abe-848c-00d37e6ac715"
        },
        "label": "Mongo-DB",
        "name": "mongodb-service",
        "plan": "default-plan",
        "provider": null,
        "syslog_drain_url": null,
        "tags": [
        "mongodb",
        "document"
        ],
        "volume_mounts": []
    }
    ],

    (생략)
    ```


##### 5. 애플리케이션 확인 
1. `cf apps`로 배포한 애플리케이션의 URL정보를 확인합니다.
    ```Emacs
        $ cf apps
    ```

2. 화면에 표시되는 URL로 접속해서 애플리케이션이 정상 동작하는 지 확인합니다. 

    ```Emacs
    $ cf apps
    sya@sk.com(으)로 dtlab 조직/dev 영역의 앱 가져오는 중...
    확인
    이름                               요청된 상태   인스턴스   메모리   디스크   URL
    hello-spring-cloud                 started       1/1        256M     1G       hello-spring-cloud-unskillful-contractor.paas.sk.com, hello-spring-cloud-persistent-hypnotization.paas.sk.com

    (생략)
    ```

3. 웹 브라우저로 URL에 접속했을 때, 아래 그림과 같이 **Spring Cloud Demo Application** 메시지가 출력되면 정상적으로 실행된 것입니다. 
    URL 값에 표시된 hello-spring-cloud-unskillful-contractor.paas.sk.com 으로 접속해보겠습니다.
    ![Image of URL result](docs/contents/images/cfurl_final.png)


## 마치며..
---
지금까지 Cloud Native Application을 개발하기 위한 간단한 개발 환경을 만들어보고,
실제로 샘플 프로젝트를 통해서 애플리케이션 빌드와 PaaS에 애플리케이션 배포까지 전반의 과정을 같이 해보았습니다.
어렵지 않죠? CF의 다양한 기능을 알고 싶다면 Cloud Foundry 공식 홈페이지에 가서 도움을 받을 수 있습니다. 참고하세요!

  
