---
date: "2018-02-07T13:24:01+09:00"
title: "New-Post"
authors: ["1000jaeh"]
categories:
  - Spring Cloud
  - Cloud
  - Java
tags:
  - Spring
  - Cloud
  - Spring Cloud
cover:
  image: /docs/images/container-layers.jpg
  caption: Eden Farm Children's Village by Gareth Harper on Unsplash
draft: false
---

- [Staticfile Buildpack](#staticfile-buildpack)
    - [Overview](#overview)
    - [What](#what)
        - [Staticfile app](#staticfile-app)
        - [Staticfile buildpack](#staticfile-buildpack)
    - [Staticfile Requirement](#staticfile-requirement)
    - [Memory Usage](#memory-usage)
    - [“Hello World” Tutorial](#%E2%80%9Chello-world%E2%80%9D-tutorial)
        - [Step / Action](#step-action)
    - [Buildpack Configuration & Application push](#buildpack-configuration-application-push)
        - [Config option](#config-option)
    - [Consideration](#consideration)

## Overview

이 주제는 Staticfile buildpack 설정 및 web에 push하는 방법에 대해서 설명합니다. 그리고 staticfile buildpack을 사용하여, 간단한 "Hello World"를 어떻게 출력하는지 보여줍니다.

 > Note: BOSH configured custom trusted certificates are not supported by the Staticfile buildpack.

## What

### Staticfile app

buildpack에서 제공하는 NGINX 웹서버 외의 Backend 코드가 필요 없는 App 또는 Content. staticfile app의 예는 프론트엔드 javascript 앱, static html content, 그리고 html/javascript 폼들

### Staticfile buildpack

staticfile buildpack은 staticfile app과 백엔드가 다른 곳에서 호스팅되는 앱에 대한 런타임 지원을 제공한다. 현재 staticfile buildpack에서 사용되는 Nginx의 버전을 찾기 위해서는 staticfile buildpack 릴리즈 노트를 참고 하십시오.

## Staticfile Requirement

Staticfile buildpack을 앱과 같이 사용하기 위해서는, root 디렉토리 밑에 Staticfile이라는 이름의 파일이 필요합니다.

## Memory Usage

NGinx는 static assets을 제공하기 위해 20메가의 메모리가 필요합니다. staticfile buildpack을 사용할때, 기본적으로 컨터이너에 할당된 기본 1기가에서 램할당을 줄이려면 -m 64M옵션을 사용하여 app을 푸시하는 것이 좋습니다.

## “Hello World” Tutorial

Staticfile buildpack을 사용하여 spa 앱을 만들고 푸시하려면 아래 절차를 따르세요.

### Step / Action

1. 워크스페이스 안에 샘플 앱을 위한 루트 디렉토리를 만들고 이동:

``` sh
$ mkdir sample
$ cd sample
```

2. index.html 파일을 만들고 임의의 스트링을 넣어라:

``` sh
$ echo 'Hello World' > index.html
```

3. Staticfile이라는 이름의 비어있는 파일을 생성:

``` sh
$ touch Staticfile
```

4. `cf login` 명령어를 사용하여 Cloud Foundry에 로그인하라.

좀 더 자세한 정보는 CLI 문서 Getting Started with the cf CLI Documentation의 Login Section을 확인하라.

``` sh
$ cf login
```

5. 앱을 푸쉬:

``` sh
$ cf push hello -m 64M
```

6. 결과 안에서 앱의 url을 찾아라.
결과 항목들은 아래와 같다.:

``` sh
Creating app hello in org sample-org / space sample-space as username@example.com...
OK

…
requested state: started
instances: 1/1
usage: 64M x 1 instances
urls: hello.example.com
```

7. url로 이동하여 실행중인 앱을 확인하라.

## Buildpack Configuration & Application push

이 섹션에서는 Staticfile buildpack 옵션으로 설정하는 방법과 Staticfile 앱을 푸쉬하는 방법에 대해서 설명한다.

### Config option

이 테이블은 구성 할 수 있는 Staticfile 빌드팩의 몇가지 측면을 설명합니다.

1. Alternative root
- 기본값 이외의 루트 디렉토리(index.html이 위치한)를 지정할 수 있습니다. 예를 들어, buildpack이 index.html과 HTML/CSS/Javascript 파일이 있는 대체 폴더 (dist/ or public/ 등)의 다른 모든 애셋을 제공하도록 지정할 수 있습니다.

2. Directory list
- 사이트의 디렉토리 index를 표시하는 html page입니다. 디렉토리 목록의 샘플은 다음과 같습니다.
- [사진]
- 사이트에 index.html이 없는 경우, 표준 404 에러 대신 디렉토리 목록이 표시됩니다.

3. SSI	
- SSI(Server Side Includes)를 사용하면 웹서버의 웹 페이제 파일 내용을 표시 할 수 있습니다. SSI에 대한 일반 정보는 위키피디아의 Server Side includes 항목을 참조하십시오.
- https://httpd.apache.org/docs/current/ko/howto/ssi.html

4. Pushstate routing
- 여러 경로를 제공하는 클라이언트 측 자바 스크립트 앱에서 브라우저가 표시되는 URL을 깨끗하게 유지합니다. 예를 들어, 푸시 스테이트 라우팅을 사용하면 하나의 Javascript 파일이 `/some#path1` 대신 `/some/path` 처럼 보이는 앵커 태그가 지정된 여러 Url로 라우팅 할 수 있습니다.

5. GZip file serving and compressing
- gzip_static 및 gunzip 모듈은 기본적으로 사용됩니다. 이를 통해 nginx는 압축된 gz 형식으로 저장된 파일을 제공하고, 압축된 내용이나 응답을 지원하지 않는 클라이언트의 압축을 풀 수 있습니다.
- 예를 들어, 아주 오래된 브라우저 클라이언트에 게재하는 경우와 같이 특정 상황에서 압축을 사용하지 않도록 설정할 수 있습니다.

6. Basic authentication	
- 앱에 간단한 액세스 컨트롤을 배치 할 수 있습니다.
- [그림]

7. Proxy support
- staging하는 동안 디펜던시들을 다운로드하기 위해 프록시를 사용할 수 있습니다.
 
8. Force HTTPS
- 모든 요청이 https를 통해 전송되도록하는 방법입니다.
- 이렇게 하면, 비 https요청이 https 요청으로 리다이렉션됩니다.
- https 강제 실행을 피하는 예는 역방향 프록시가 있는 force_https 정보를 참조하십시오.

9. Dot Files
- 기본적으로, 숨긴파일(.으로 시작하는)은 빌드팩에서 지원하지 않습니다.

10. HTTP Strict Transport Security
- nginx가 strict-transport-security:max-age=31536000헤더로 모든 요청에 응답하게 합니다. 이렇게 하면, 브라우저가 https를 통해 모든 후속 요청을 수행합니다.

11. Custom Location configuration
- nginx의 설정파일 위치를 사용자 정의하려면 다음단계를 수행하십시오.
  1. 루트 디렉토리를 설정하세요.(location_include는 루트와만 연동됩니다.)
  2. 위치 범위가 지정된 ngninx 지시문을 사용하여 파일을 만듭니다. 사이트 방문자가 x-mysiteName HTTP 헤더를 받도록하는 다음 예를 참조하십시오.:
  ``` yml
  File: nginx/conf/includes/custom_header.conf
  Content: add_header X-MySiteName BestSiteEver;
  ```
  3. Staticfile의 location_include 변수를 이전 단계의 파일 경로로 설정하십시오:
    ``` yml
    root: public
    location_include: includes/*.conf
    ```
    nginx 지시문에 대한 자세한 내용은 nginx 설명서를 참조하십시오.

12. Custom NGINX configuration
- 파일 확장자를 mime 유형에 매핑하는 것과 같은 추가 nginx 구성을 적용할 수 있습니다. 예제는 nginx 설명서를 참조하십시오.

## Consideration

1. Staticfile의 존재여부
1. 기본 구조: nodejs로 테스트
1. nginx.conf 파일 설정 방법
