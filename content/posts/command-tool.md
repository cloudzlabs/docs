---
date: "2018-02-20T16:55:36+09:00"
title: "CF CLI 사용하기"
authors: ["sya"]
categories:
  - posts
tags:
  - cloud foundry
  - cf
  - cf cli
  - paas
draft: false
---
{{% notice note %}}
오픈소스 CF CLI에서 **자주 사용하는 CF CLI 명령어**를 사용하는 방법을 정리해보았습니다.  
CF CLI에서 사용 가능한 모든 명령어의 사용법은 [CF 공식 홈페이지 문서](https://docs.cloudfoundry.org/cf-cli/)를 참고하길 바랍니다.
{{% /notice%}}

## CF

CF CLI 사용 방법을 알아보기에 앞서 CF가 무엇인지 간단히 살펴보겠습니다. CF는 Cloud Foundry의 약자로, 2011년 VMWare가 업계 최초로 만든 오픈소스 PaaS 플랫폼입니다. 지금은 Cloud Foundry 재단에서 관리하고 있습니다.

### 오픈소스 CF 제품

Cloud Foundry의 오픈소스 Cloud Foundry 기술을 기반으로 만든 제품은 다음과 같습니다.

- PCF (Pivotal Cloud Foundry)
  - [Pivotal](https://pivotal.io/)에서 오픈소스 Cloud Foundry로 만든 상용 PaaS
- [Bluemix] (https://www.ibm.com/cloud-computing/bluemix)
  - IBM에서 오픈 소스 Cloud Foundry로 만든 상용 PaaS

## CF CLI

GO 언어로 개발된 Cloud Foundry의 Command Line Interface입니다. 사용자가 CF CLI를 설치하면 명령 프롬프트를 통해서 CF기반의 PaaS 환경에 자신이 만든 애플리케이션을 배포하거나 서비스 생성하여 바인등하는 등의 다양한 기능을 사용할 수 있습니다.

### CF CLI 설치 방법

CLI Installer를 다운로드하여 간단히 설치할 수 있습니다.

1. [CF CLI 설치 가이드 페이지](https://github.com/cloudfoundry/cli/releases) 에 접속합니다.
2. Installer 문단에서 컴퓨터 OS 정보에 맞는 파일을 다운로드합니다.  
3. 다운로드한 파일을 실행하여 CF CLI를 설치합니다.
4. 명령어 프롬프트에 `cf`를 실행하여 정상 설치 여부를 확인합니다. 다음과 같은 정보가 표시되면 정상적으로 설치된 것을 확인할 수 있습니다.

```sh
$ cf

cf 버전 6.32.0+0191c33d9.2017-09-26, Cloud Foundry command line tool
Usage: cf [global options] command [arguments...] [command options]

Before getting started:
  config    login,l      target,t
  help,h    logout,lo

Application lifecycle:
  apps,a        run-task,rt    events
  push,p        logs           set-env,se
  start,st      ssh            create-app-manifest
  stop,sp       app
  restart,rs    env,e
  restage,rg    scale

Services integration:
  marketplace,m        create-user-provided-service,cups
  services,s           update-user-provided-service,uups
  create-service,cs    create-service-key,csk
  update-service       delete-service-key,dsk
  delete-service,ds    service-keys,sk
  service              service-key
  bind-service,bs      bind-route-service,brs
  unbind-service,us    unbind-route-service,urs

Route and domain management:
  routes,r        delete-route    create-domain
  domains         map-route
  create-route    unmap-route

...
```

  > **Tip. 컴퓨터가 32bit인지 64bit인지 확인 방법**  
  > 1. 명령 프롬프트에서 `systeminfo`명령어를 입력합니다.  
  > 2. 다음과 같이 시스템 종류 정보가 표시됩니다.  
  > - x86-based PC -> 32bit  
  > - x64-based PC -> 64bit

## Getting Started

### cf help

CF CLI 명령어 도움말입니다. CF CLI에서 사용 가능한 모든 명령어에 대한 도움말을 표시합니다.

#### 사용법

```sh
cf help

cf help [명령어]
```

#### 옵션

- 옵션 없이 사용할 시, 사용 가능한 모든 명령어를 표시합니다.
- `[명령어]` 정보를 알고 싶은 특정 명령어

#### 예시

```sh
$ cf help

cf 버전 6.32.0+0191c33d9.2017-09-26, Cloud Foundry command line tool
Usage: cf [global options] command [arguments...] [command options]

Before getting started:
  config    login,l      target,t
  help,h    logout,lo

Application lifecycle:
  apps,a        run-task,rt    events
  push,p        logs           set-env,se
  start,st      ssh            create-app-manifest
  stop,sp       app
  restart,rs    env,e
  restage,rg    scale

Services integration:
  marketplace,m        create-user-provided-service,cups
  services,s           update-user-provided-service,uups
  create-service,cs    create-service-key,csk
  update-service       delete-service-key,dsk
  delete-service,ds    service-keys,sk
  service              service-key
  bind-service,bs      bind-route-service,brs
  unbind-service,us    unbind-route-service,urs

Route and domain management:
  routes,r        delete-route    create-domain
  domains         map-route
  create-route    unmap-route
...
```

```sh
$ cf help apps

이름:
apps - 대상 영역에 모든 앱 나열
사용법:
cf apps
별명:
a
SEE ALSO:
events, logs, map-route, push, restart, scale, start, stop
```

### cf api

CF CLI의 대상(API End Point)을 설정합니다. 설정된 API Endpoint를 대상으로 이후 모든 작업이 진행됩니다.

#### 사용법 

```sh
cf api [URL]
```

`[URL]` 접속하려는 PaaS의 API End Point

#### 옵션

- `--skip-ssl-validation` 비보안 Endpoint를 사용합니다.
- `--unset` 설정되어 있는 EndPoint를 삭제합니다.

#### 예시

```sh
$ cf api http://api.paas.sk.com

API 엔드포인트를 http://api.paas.sk.com(으)로 설정 중...
Warning: Insecure http API endpoint detected: secure https API endpoints are recommended
확인

api endpoint:   http://api.paas.sk.com
api version:    2.69.0
```

```sh
$ cf api cf api http://api.paas.sk.com --skip-ssl-validation
```

### cf login

PaaS에 로그인합니다.

#### 사용법

```sh
cf login
```

#### 옵션

- 옵션 없이 사용하면 로그인에 필요한 정보를 입력할 수 있는 필드가 생성됩니다.
- `-a [api end point]` 접속하려는 PaaS API End point 정보
- `-u [username]` 접속하려는 PaaS 아이디
- `-p [password]` 접속하려는 PaaS 비밀번호
- `-o [org]` 접속하려는 Org 정보
- `-s [space]` 접속하려는 Space 정보
- `--skip-ssl-validation` 비보안 End point 사용하기 위한 옵션

#### 예시

```sh
$ cf login

API 엔드포인트: http://api.paas.sk.com
경고: 비보안 http API 엔드포인트 발견: 보안 https API 엔드포인트를 사용하는 것이 좋습니다.


Email> sya@sk.com

Password>
인증 중...
확인

조직 선택(또는 Enter를 눌러 건너뜀):
1. cloudlab
2. dtlab

Org>
...
```

```sh
$ cf login -a https://api.dev.ghama.io
```

### cf logout

PaaS를 로그아웃합니다.

#### 사용법

```sh
cf logout
```

#### 예시

```sh
$ cf logout

로그아웃 중...
확인
```

### cf target

현재 접속되어 있는 API End Point , Org, Space 정보 표시 및 Org, Space를 변경합니다.

#### 사용법

```sh
cf target
```

#### 옵션

- 옵션 없이 실행하면 현재 접속되어 있는 API EndPoint , Org, Space 정보가 표시됩니다.
- `-o [org]` 변경 접속하려는 Org 정보
- `-s [space]` 변경 접속하려는 Space 정보

#### 예시

```sh
$ cf target
api endpoint:   http://api...
api version:    2.69.0
user:           sya@sk.com
org:            dtlab
space:          prod
```

```sh
$ cf target -o cloudlab -s dev

api endpoint:   http://api...
api version:    2.69.0
user:           sya@sk.com
org:            cloudlab
space:          dev
```

## Apps

### cf apps

현재 접속되어 있는 Org와 Space의 모든 Application 목록을 보여줍니다.

#### 사용법

```sh
cf apps
```

#### 예시

```sh
$ cf apps
sya@sk.com(으)로 dtlab 조직/prod 영역의 앱 가져오는 중...
확인

이름                               요청된 상태   인스턴스   메모리   디스크   URL
dtlabs-admin-bff-service           started       1/1        256M     256M     dtlabs-admin-bff-service.paas.sk.com
dtlabs-apigateway-service          started       1/1        1G       1G       dtlabs-apigateway-service.paas.sk.com
```

### cf push

PaaS에 애플리케이션을 배포합니다.

#### 사용법

```sh
cf push

cf push -f [manifest.yml file path]
```

#### 옵션

- `[manifest.yml file path]` manifest yml 경로
- manifest.yml을 이용하지 않고 옵션을 통해서 애플리케이션을 배포하시려면 옵션의 자세한 내용은 [CF Push 가이드](http://cli.cloudfoundry.org/en-US/cf/push.html)를 참고하세요.
- 옵션을 이용하지 않고 **manifest.yml을 통한 배포를 추천**합니다. 명령 프롬프트에서 프로젝트 경로로 이동 후, `cf push` 명령어를 실행하거나,옵션으로 manifest.yml 경로를 지정해서 애플리케이션을 배포합니다.
- manifest.yml 파일이 개발과 운영환경에서 다른 정보를 가진다면 manifest-dev.yml, manifest-prod.yml 등으로 **설정 분리** 후, manifest 파일을 명시해서 배포합니다.
- manifest.yml 설정 정보는 [CF 공식가이드 문서의 manifest.yml 가이드 문서](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)를 참고하세요.

#### 예시

- 옵션 없이 사용하는 경우

```sh
$ ls
log		mvnw		pom.xml		target
manifest.yml	mvnw.cmd	src
$ cf push

Manifest 파일 /Users/seoyoungahn/git/spring-boot-tutorial/cloud-movie/manifest.yml 사용

sya@sk.com(으)로 dtlab 조직/dev 영역에서 cloud-movie 앱 업데이트 중...
확인

cloud-movie 업로드 중...
업로드 중인 앱 파일 원본 위치: /var/folders/mc/0_v3hb1j2j71t_g53qg8qjg40000gn/T/unzipped-app239365053
443.5K, 105 파일 업로드
Done uploading
확인


sya@sk.com(으)로 dtlab 조직/dev 영역에서 cloud-movie 앱 시작 중...
Downloading java_buildpack...
Downloaded java_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (16.6M)
Staging...
-----> Java Buildpack Version: v3.11 | https://github.com/cloudfoundry/java-buildpack.git#eba4df6
-----> Downloading Open Jdk JRE 1.8.0_111 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_111.tar.gz (10.6s)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.3s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (0.0s)
       Memory Settings: -Xmx681574K -XX:MaxMetaspaceSize=104857K -XX:MetaspaceSize=104857K -Xms681574K -Xss349K
-----> Downloading Spring Auto Reconfiguration 1.12.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.12.0_RELEASE.jar (0.0s)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (44.9M)
Uploaded droplet (61.8M)
Uploading complete
Destroying container
Successfully destroyed container

0 / 1 인스턴스 실행 중, 1 시작 중
1 / 1 인스턴스 실행 중

앱 시작됨


확인

`CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher` 명령을 사용하여 cloud-movie 앱이 시작되었습니다.

sya@sk.com(으)로 dtlab 조직/dev 영역에서 cloud-movie 앱의 상태 표시 중...
확인

요청된 상태: started
인스턴스: 1/1
사용법: 1G x 1 인스턴스
URL: cloud-movie-leathern-estrin.paas.sk.com
마지막으로 업로드함: Fri Feb 23 01:02:09 UTC 2018
스택: cflinuxfs2
빌드팩: java_buildpack

     상태      이후                     CPU    메모리   디스크   세부사항
#0   실행 중   2018-02-23 10:07:53 AM   0.0%   0 / 1G   0 / 1G
```

- 옵션 사용 시

```sh
$ cf push -f C:/Users/cf-sample-app-nodejs-master/manifest-dev.yml
```

### cf delete

애플리케이션을 삭제합니다.

#### 사용법

```sh
cf delete [app name]
```

`[app name]` 삭제할 애플리케이션명

#### 옵션

- `-f` 재확인 없이 애플리케이션 삭제합니다.
- `-r` 관련된 모든 라우트 정보에 해당하는 애플리케이션 삭제합니다.

#### 예시

```sh
$ cf delete cna-sample
```

```sh
$ cf delete cna-sample -r
```

### cf logs

애플리케이션 로그를 출력합니다.

#### 사용법

```sh
cf logs [app name]
```

`[app name]` 로그를 출력할 애플리케이션명

#### 예시

```sh
$ cf logs cna-sample

Retrieving logs for app cna-sample in org dtlab / space prod as sya@sk.com...

```

### cf start

애플리케이션 실행을 시작합니다.

#### 사용법

```sh
cf start [app name]
```

`[app name]` 시작시킬 애플리케이션명

#### 예시

```sh
$ cf start cna-sample

aiting for app to start...

이름:                cna-sample
요청된 상태:           started
인스턴스:              1/1
사용법:               512M x 1 instances
routes:             cna-sample.paas.sk.com
마지막으로 업로드함:   Mon 12 Feb 18:56:13 KST 2018
스택:                  cflinuxfs2
빌드팩:                java_buildpack
start command:         CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE
                       -memorySizes=metaspace:64m..,stack:228k..
                       -memoryWeights=heap:65,metaspace:10,native:15,stack:10
                       -memoryInitials=heap:100%,metaspace:100%
                       -stackThreads=300 -totMemory=$MEMORY_LIMIT) &&
                       JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR
                       -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh
                       $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec
                       $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp
                       $PWD/. org.springframework.boot.loader.JarLauncher

     상태      이후                   CPU    메모리           디스크         세부사항
#0   실행 중   2018-02-22T09:50:13Z   0.0%   318.1M of 512M   143.8M of 1G 
```

### cf stop

애플리케이션을 중지시킵니다.

#### 사용법

```sh
cf stop [app name]
```

`[app name]` 중지할 애플리케이션명

#### 예시

```sh
$ cf stop cna-sample

sya@sk.com(으)로 dtlab 조직/dev 영역에서 cna-sample앱 중지 중...
확인
```

### cf restart

애플리케이션을 재실행합니다.

#### 사용법

```sh
cf restart [app name]
```

`[app name]` 재시작할 애플리케이션명

#### 예시

```sh
$ cf restart cna-sample

Restarting app cna-sample in org dtlab / space dev as sya@sk.com...

Stopping app...

Waiting for app to start...

이름:                  cna-sample
요청된 상태:           started
인스턴스:              1/1
사용법:                512M x 1 instances
routes:               cna-sample.paas.sk.com
마지막으로 업로드함:   Mon 12 Feb 18:56:13 KST 2018
스택:                  cflinuxfs2
빌드팩:                java_buildpack
start command:         CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE
                       -memorySizes=metaspace:64m..,stack:228k..
                       -memoryWeights=heap:65,metaspace:10,native:15,stack:10
                       -memoryInitials=heap:100%,metaspace:100%
                       -stackThreads=300 -totMemory=$MEMORY_LIMIT) &&
                       JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR
                       -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh
                       $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec
                       $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp
                       $PWD/. org.springframework.boot.loader.JarLauncher

     상태      이후                   CPU    메모리           디스크         세부사항
#0   실행 중   2018-02-22T09:53:32Z   0.0%   163.1M of 512M   143.8M of 1G 
```

## Services

### cf services

현재 접속되어 있는 Org과 Space에서 사용 가능한 모든 Service의 정보를 표시합니다.

#### 사용법

```sh
cf services
```

#### 예시

```sh
$ cf services

sya@sk.com(으)로 dtlab 조직/dev 영역의 서비스를 가져오는 중...
확인

이름                        서비스           플랜                   바인딩된 앱                                                                                                                                                마지막 조작
amqp-service                RabbitMQ         standard                                                                                                                                                                          create 성공
cloud-movie-apigateway      사용자 제공
cloud-movie-config-server   사용자 제공
cloud-movie-discovery       사용자 제공
cloud-movie-redis           Redis            shared-vm
```

### cf marketplace

마켓플레이스에 등록되어 애플리케이션에서 사용 가능한 서비스의 정보를 보여줍니다.

#### 사용법

```sh
cf marketplace
```

#### 옵션

`-s [service name]` [service name]에 해당하는 서비스의 자세한 정책을 보여줍니다.

#### 예시

```sh
$ cf marketplace

sya@sk.com(으)로 dtlab 조직/dev 영역에서 서비스를 가져오는 중...
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

```sh
$ cf marketplace -s Redis

sya@sk.com(으)로 Redis 서비스의 서비스 플랜 정보를 가져오는 중...
확인

서비스 플랜   설명                                                                                                                무료 또는 유료
shared-vm     This plan provides a single Redis process on a shared VM, which is suitable for development and testing workloads   free
```

### cf create-service

서비스를 생성합니다.

#### 사용법

```sh
cf create-service [service name] [service plan] [service instance]
```

- 애플리케이션에서 marketplace에 등록되어 있는 서비스를 사용하기 위해 생성합니다. 생성하려는 서비스의 정보는 `cf marketplace` 명령어를 통해 알 수있습니다.
- `[service name]` 생성하려는 service name
- `[service plan]` 생성하려는 service plan
- `[service instance]` 임의 지정

#### 옵션

- `-c [parameters as json]` 해당 서비스에 전달할 parameter 를 JSON 으로 작성합니다.
- OS 별로 파라미터 전달하는 방식은 [CF 공식문서](http://cli.cloudfoundry.org/en-US/cf/create-service.html)를 참고하세요.

#### 예시

```sh
$ cf create-service Redis shared-vm cna-sample-redis-service

sya@sk.com(으)로 dtlab 조직/dev 영역에 서비스 인스턴스 cna-sample-redis-service 작성 중...
확인
```

### cf delete-service

서비스 인스턴스를 삭제합니다.

#### 사용법

```sh
cf delete-service [service instance]
```

`[service instance]` 삭제할 서비스명. 현재 space에 등록되어 있어야 합니다.

#### 옵션
`-f` 재확인 없이 애플리케이션을 삭제합니다.

#### 예시

```sh
$ cf delete-service cna-sample-redis-service

서비스 cna-sample-redis-service을(를) 삭제하시겠습니까?> y
sya@sk.com(으)로 dtlab 조직/dev 영역에서 cna-sample-redis-service 서비스 삭제 중...
확인
```

```sh
$ cf delete-service cna-sample-redis-service -f

sya@sk.com(으)로 dtlab 조직/dev 영역에서 cna-sample-redis-service 서비스 삭제 중...
확인
```

### cf bind-service

애플리케이션에 서비스를 연동합니다.

#### 사용법

```sh
cf bind-service [app name] [service instance]
```

- `[app name]` 대상 애플리케이션명
- `[service instance]` 연동할 service명
- `cf services` 명령어를 통해 해당 space에서 사용할 수 있는 [service instance]를 확인할 수 있습니다.

#### 옵션

`-c [parameters as json]` 해당 서비스에 전달할 parameter(JSON으로 작성)

OS 별로 파라미터 전달하는 방식은 [CF 공식홈페이지의 bind service 가이드](http://cli.cloudfoundry.org/en-US/cf/bind-service.html)를 참고하세요.

#### 예시

```sh
$ cf bind-service cna-sample cna-sample-redis-service

sya@sk.com(으)로 dtlab 조직/dev 영역의 cna-sample 앱에 cna-sample-redis-service 서비스 바인드 중...
확인
팁: 환경 변수 변경사항을 적용하려면 'cf restage cna-sample'을(를) 사용하십시오.
```

### cf unbind-service

애플리케이션에 연동한 서비스 연동을 해제합니다.

#### 사용법

```sh
cf unbind-service [app name] [service instance]
```

- `[app name]` 대상 애플리케이션명
- `[service instance]` 연동을 해제할 서비스명

#### 예시

```sh
$ cf unbind-service cna-sample cna-sample-mysql-service

sya@sk.com(으)로 dtlab 조직/dev 영역의 cna-sample-redis-service 서비스에서 cna-sample 앱 바인드 해제 중...
확인
```
