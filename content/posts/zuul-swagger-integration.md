---
date: "2018-02-27T17:21:07+09:00"
title: "Zuul Swagger 연동"
authors: [bckim0620]
categories:
  - posts
tags:
  - Swagger
  - Zuul
  - Srping Cloud
description: ""
draft: true
---

MSA(Micro Service Architecture) 활용하여 팀 홈페이지를 제작하는 과정에서 User BFF에서 백엔드 서비스를 호출하기 위해 Zuul을 사용하였습니다.
여러 가지 서비스가 Zuul을 통해 User BFF에 노출되고 있어, Zuul을 통해 서비스되는 모든 API를 하나의 Swagger page로 노출하고 싶었습니다.
다행히도 Swagger에서 Zuul 연계하는 기능이 있어 이를 소개합니다.

MSA에 대한 자세한 설명은 차후에 포스팅하기로 하고, 이해를 돕기 위해 간단한 홈페이지 아키텍처 첨부합니다.

{{< mermaid >}}
graph LR;
    A[Client] -->|Request| B(User BFF)
    B -->|Request| C{Zuul}
    C -->|Request| D[Mail service]
    C -->|Request| E[User service]
{{< /mermaid >}}

## 사용법

### 1. Swagger2 라이브러리 추가
Zuul 프로젝트에 swagger2라이브러리를 추가합니다.

```pom.xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.6.1</version>
</dependency>
 
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.6.1</version>
</dependency>
```

### 2. Swagger2 설정
아래 소스와 같이 `SwaggerResourcesProvider`를 상속받아 SwaagerConfiguration을 생성 후 Zuul에 등록된 서비스들의 `API DOC URL`을 입력합니다.

```java
import java.util.ArrayList;
import java.util.List;
 
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;
 
import springfox.documentation.swagger.web.SwaggerResource;
import springfox.documentation.swagger.web.SwaggerResourcesProvider;
import springfox.documentation.swagger.web.UiConfiguration;
import springfox.documentation.swagger2.annotations.EnableSwagger2;
 
@Component
@Primary
@EnableAutoConfiguration
@EnableSwagger2
public class SwaggerConfiguration implements SwaggerResourcesProvider {
 
    @Bean
    public UiConfiguration uiConfig() {
        return new UiConfiguration("validatorUrl", "list", "alpha", "schema",
                UiConfiguration.Constants.DEFAULT_SUBMIT_METHODS, false, true, 60000L);
    }
 
    @Override
    public List<SwaggerResource> get() {
        List<SwaggerResource> resources = new ArrayList<>();
        resources.add(swaggerResource("dtlabs-registration-service", "{SERVICE_API_DOC_URL}", "2.0"));
        resources.add(swaggerResource("dtlabs-data-mgmt-service", "{SERVICE_API_DOC_URL}", "2.0"));
        resources.add(swaggerResource("dtlabs-user-oauth-service", "{SERVICE_API_DOC_URL}", "2.0"));
        return resources;
    }
     
    private SwaggerResource swaggerResource(String name, String location, String version) {
        SwaggerResource swaggerResource = new SwaggerResource();
        swaggerResource.setName(name);
        swaggerResource.setLocation(location);
        swaggerResource.setSwaggerVersion(version);
        return swaggerResource;
    }
     
}
```