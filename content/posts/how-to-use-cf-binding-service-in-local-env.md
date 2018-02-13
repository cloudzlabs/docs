---
date: "2018-02-13T08:34:45+09:00"
title: "Spring Cloud Connector로 PaaS의 binding 서비스를 로컬에서 사용하기"
authors: ["jisangYun"]
tags:
  - Spring Cloud Connector
  - Cloud Foundry
  - MariaDB
  - H2
  - PaaS
  - Spring Cloud
cover:
  image: /docs/images/112411124214.PNG
  caption: "asdfasdf"
  style: full
draft: false
---

##What?

#### PaaS의 binding 된 서비스를 로컬 개발 환경에서 사용하자.

## Why?

로컬 개발 환경과 PaaS 환경(dev, stg, prod 등) 을 분리하여 개발환경을 구성하는 경우 몇가지 **불편한 점**이 있다.

그 중 하나가 로컬 개발 환경에서 사용하는 서비스와 PaaS 환경에서 사용하는 서비스가 다른 경우다.



예를 들면,

1. 로컬 환경에서 H2 DB 를 쓰다가 MariaDB로 배포하는 경우 쿼리가 다르다.

   H2:

   ``` sql
   DROP TABLE IF EXISTS users CASCADE;
   CREATE TABLE IF NOT EXISTS users (
     	id 			INTEGER,
     	username 	VARCHAR(100) NOT NULL,
     	age 		INTEGER NOT NULL,
     	job      	VARCHAR(100) NOT NULL
   );
   ALTER TABLE users MODIFY id INTEGER NOT NULL AUTO_INCREMENT;
   ```

   MariaDB:

   ``` sql
   DROP TABLE IF EXISTS users CASCADE;
   CREATE TABLE IF NOT EXISTS users (
     	id 			INTEGER,
     	username 	VARCHAR(100) NOT NULL,
     	age 		INTEGER NOT NULL,
     	job      	VARCHAR(100) NOT NULL
   );
   ALTER TABLE users MODIFY id INTEGER PRIMARY KEY NOT NULL AUTO_INCREMENT;
   ```

   다른 점은 ... ?      MariaDB는 PRIMARY KEY가 빠지면 에러다.

2. local 환경에서 초기 데이터를 만들기 번거롭다. 

   테스트 데이터를 매번 생성 or 초기화 하는 절차가 추가된다.

   ![1518484130041](/docs/content/images/1518484130041.png)

3. 어플리케이션 수정 후 PaaS 환경에서 테스트를 할 때마다 매번 배포를 해야한다. -> CI/CD 배포 pipeline이 없다면 번거로운 절차다.

4. 로컬에서는 잘 되는게 PaaS에서는 안된다. -> 서비스의 버전이 다른 경우,,,, 로컬에서 아무리 잘되도 서비스하는 환경에서 안되면 말짱 꽝이다. 버전 맞추는 것도 번거롭다.

## How?

Spring Cloud Connector 와 STS/Eclipse 의 Run Configuration(환경변수 주입) 을 활용하자.

- 개발 환경

  - STS(Spring Tool Suite)
  - Spring Boot 1.5.9
  - [Spring Cloud Connector](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-connectors.html)
  - Cloud Foundry 기반의 OpenPaaS
    - MariaDB 인스턴스

  ​

- 미리 준비한 것

  - MariaDB 인스턴스 생성 : js-test-MariaDB

  - 어플리케이션 : js-local-paas-service-conn

    - dependency 추가 - Spring Cloud Connector

      ![1518495055690](/docs/images/1518495055690.png)

    - datasource 설정

      - Bean 생성

        ``` java
        @Configuration
        @Profile({"dev"})
        public class CloudConfiguration extends AbstractCloudConfig {
        	
        	@Value("${services.datasource.name}")
        	private String datasourceName;

        	@Value("${services.datasource.initial-size}")
        	private int minPoolSize;

        	@Value("${services.datasource.maximum-pool-size}")
        	private int maxPoolSize;

        	@Value("${services.datasource.max-wait-time}")
        	private int maxWaitTime;

        	/**
        	 * configure datasource.
        	 * @return dataSource object
        	 */
        	@Bean
        	public DataSource dataSource() {
        		PoolConfig poolConfig = new PoolConfig(minPoolSize, maxPoolSize, maxWaitTime);
        		DataSourceConfig dbConfig = new DataSourceConfig(poolConfig, null);
        		return connectionFactory().dataSource(datasourceName, dbConfig);
        	}
        }

        ```

      - application-dev.yml - datasource 설정

        ``` yaml
        services:
          datasource: 
            initial-size: 1
            maximum-pool-size: 100
            max-wait-time: 3000
            name: js-test-mariadb
            initialize: false
        ```

  - js-local-paas-service-conn 을 PaaS에 배포한 후 js-test-MariaDB 와 binding 한다.

    ![1518484942742](/docs/images/1518484942742.png)

  - 로컬 환경과 PaaS 환경의 데이터가 다른 것을 확인한다.

    ![151231](/docs/images/151231-8495551138.png)

  - 준비 끝!

- 상세 적용 방법

  1. MariaDB ssh 연동

     PaaS 환경의 서비스 인스턴스는 로컬에서 연동시 바로 연동하지 못하고, ssh로 연동한다.

     ``` bash
     cf ssh -N -L 63306:172.132.14.32:3306 js-local-paas-service-conn
     ```

  2. STS - Run Configuration - Spring Boot - Profiles 설정

     어플리케이션의 수행 profile을 dev로 설정한다.

     ![1518489287635](/docs/images/1518489287635.png)

  3. STS - Run Configuration - Environment - Environment variables

     어플리케이션의 PaaS 환경 변수(VCAP_APPLICATION, VCAP_SERVICES)를 설정한다.

     - VCAP_APPLCATION : {} 로 세팅한다.

     - VCAP_SERVICES : cf env {어플리케이션명} 으로 조회된 value를 엔터키 없이 복사해서 넣는다.

       ![141231](/docs/images/141231.png)

       ![1518493193055](/docs/images/1518493193055.png)

       > cf env로 조회된 mariaDB credential 중 hostname과 port 정보를 ssh 연동한 정보로 수정이 필요하다.
       >
       > ex. "hostname": "172.132.14.32" => "hostname": "127.0.0.1"

  4. 로컬에서 PaaS 데이터 확인

     ![1518495702705](/docs/images/1518495702705.png)

  5. 성공!

  ​

## Conclusion

#### Spring Cloud Connector와 STS/Eclipse의 Run Configuration(환경변수 주입)을 사용해, 로컬 개발 환경에서 PaaS의 binding 된 서비스를 사용할 수 있다.



> 가나다라

# 가

## 나

### 다

#### 라

##### 마

###### 바
