# 시작하기(Single Host, Single Application)

이 문서는 Docker를 처음 접하는 사용자를 위한 문서입니다.

Docker를 이용하여 애플리케이션을 배포하고 컨테이너를 관리하는 방법에
대한 설명입니다.

-   1
    [Prerequisites](#id-시작하기(SingleHost,SingleApplication)-Prerequisites)
-   2 [애플리케이션
    배포](#id-시작하기(SingleHost,SingleApplication)-애플리케이션배포)
    -   2.1 [앱준비](#id-시작하기(SingleHost,SingleApplication)-앱준비)
    -   2.2 [Dockerfile
        작성](#id-시작하기(SingleHost,SingleApplication)-Dockerfile작성)
    -   2.3 [Docker 이미지
        생성 ](#id-시작하기(SingleHost,SingleApplication)-Docker이미지생성)
    -   2.4 [애플리케이션 실행(이미지실행,
        컨테이너실행)](#id-시작하기(SingleHost,SingleApplication)-애플리케이션실행(이미지실행,컨테이너실행))
-   3 [컨테이너
    관리](#id-시작하기(SingleHost,SingleApplication)-컨테이너관리)
    -   3.1 [컨테이너
        생성](#id-시작하기(SingleHost,SingleApplication)-컨테이너생성)
    -   3.2 [컨테이너
        중지](#id-시작하기(SingleHost,SingleApplication)-컨테이너중지)
    -   3.3 [컨테이너
        삭제 ](#id-시작하기(SingleHost,SingleApplication)-컨테이너삭제)
-   4 [이미지
    관리](#id-시작하기(SingleHost,SingleApplication)-이미지관리)
    -   4.1 [Docker Hub
        활용](#id-시작하기(SingleHost,SingleApplication)-DockerHub활용)
    -   4.2 [Private Registry(Harbor)
        활용](#id-시작하기(SingleHost,SingleApplication)-PrivateRegistry(Harbor)활용)

## Prerequisites

-   JDK 1.8 이상
-   Maven 3.0 이상
-   STS (or IntelliJ IDE)
-   Docker

## 애플리케이션 배포

애플리케이션을 Docker에 배포하는 방법을 설명합니다.

### 앱준비

-   샘플 앱 다운로드  
    -   Spring Boot 공식 가이드에서 제공하는 샘플입니다.
    -   <https://github.com/spring-guides/gs-spring-boot-docker.git> 에
        접속하여 **Download ZIP** 하거나 
    -   git
        clone https://github.com/spring-guides/gs-spring-boot-docker.git 을
        통해 프로젝트를 다운로드 합니다.
    -   **STS Tool 활용**
        -   샘플 프로젝트 로컬 레파지토리에 다운로드
            -   Git Repositories → 화면 중간에 Clone a repository 또는,
                Git Repositories view  상단 아이콘중에 왼쪽에서 세 번째
                 Clone a Git repogitory and add the clone to this view
                선택 → Clone URI 선택
        -   샘플 프로젝트 STS 설정
            -   **Git Repositories
                → gs-spring-boot-docker 프로젝트 Working Tree의 initial
                폴더 마우스 오른쪽 버튼 클릭 → Import Maven Projects**를
                선택
            -   STS의 Package Explorer에 추가한 애플리케이션이
                정상적으로 추가되었는지 확인 
-   샘플 앱 수정 및 빌드
    -   샘플 프로젝트의 initial 폴더로 이동한 뒤
        /src/main/java/hello/Application.java 파일에 아래와 같이 간단한
        API를 추가합니다.

        **Application.java**

        ``` java
        package hello;

        import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.SpringBootApplication;
        import org.springframework.web.bind.annotation.RequestMapping;
        import org.springframework.web.bind.annotation.RestController;

        @SpringBootApplication
        @RestController
        public class Application {
            
            @RequestMapping("/")
            public String home() {
                return "Hello Docker World";
            }
            
            public static void main(String[] args) {
                SpringApplication.run(Application.class, args);
            }

        }
        ```

    -   Maven을 활용하여 애플리케이션을 빌드합니다.
        -   Maven CLI 활용

              

            ``` java
            $ mvn package
            ```

        -   STS Maven 플러그인 활용

            -   빌드 설정 

            1.  1.  Package
                    Explorer에서**** gs-spring-boot-docker****** ****선택
                    → ****마우스 오른쪽 버튼 → Run AS → Run
                    configurations... → Maven build를 더블 클릭**

                2.  **Name: ****gs-spring-boot-docker******
                3.  **Goal : clean install**

                4.  **Apply** 선택해서 설정 정보를 저장 

            -   빌드 

            1.  1.  Package
                    Explorer에서** ****gs-spring-boot-docker** 선택
                    →** 마우스 오른쪽 버튼 → Run AS → Run
                    configurations... → Maven build**를 선택해서 빌드
                    합니다.

                2.  workspace의 샘플 프로젝트 하위에 tartget 폴더에 JAR
                    파일이 생성되었는지  확인

              

### Dockerfile 작성

Dockerfile은 Docker에서 이미지 레이어를 지정하고 컨테이너 내부 환경을
정의하는데 사용되는 간단한 파일 형식입니다.

아래와 같이 Dockerfile을 작성해봅니다.

**Dockerfile**

``` java
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD target/gs-spring-boot-docker-0.1.0.jar app.jar
ENV JAVA_OPTS=""
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

Dockerfile 을 작성하는데 필요한 기본 명령어 입니다. (참조 - [Dockerfile
작성방법](https://docs.docker.com/engine/reference/builder/))

-   FROM : 기반이 되는 이미지를 설정합니다. Dockerfile은 FROM 명령어로
    시작해야 합니다.
-   VOLUME : Docker 컨테이너에 의해 사용되는 데이터를 유지하는데
    사용합니다.   
    해당 실습에서는 Spring Boot  어플리케이션이 기본적으로 톰캣 작업
    디렉토리로 /tmp 폴더를 사용하므로 해당 폴더를 VOLUME으로
    추가합니다.  
    그러면 호스트의 Docker 특정 영역에 임시 파일을 만들고 컨테이너 내부
    디렉토리인 /tmp 에 링크합니다. 

    Volumes vs. Bind mounts

    컨테이너의 데이터를 유지하는 또 다른 방식인 **Bind mounts** 는
    호스트 시스템의 파일이나 디렉토리가 컨테이너에 마운트됩니다. 이
    방식은 Dockerfile에서는 사용할 수 없으며 Docker 명령어\[docker run
    -v ...\]의 옵션을 통해 사용 가능합니다. Bind mounts는 성능이
    뛰어나지만 호스트 시스템의 파일시스템 구조에 의존적이며 Docker CLI로
    관리할 수 없어 Volume 방식에 비해 제한적입니다.

-   ADD : 파일이나 디렉토리 등을 복사하여 이미지 파일시스템에
    추가합니다. 해당 실습에서는 프로젝트 JAR
    파일\[target/gs-spring-boot-docker-0.1.0.jar\]이 컨테이너에
    app.jar로 추가됩니다.

    ADD vs. COPY

    ADD 와 COPY 명령어는 모두 &lt;src&gt; 경로의 파일이나 디렉토리를
    복사하여 &lt;dest&gt; 경로의 이미지 파일시스템에 추가하지만 약간의
    차이점이 있습니다.

    <table style="width:100%;">
    <colgroup>
    <col style="width: 4%" />
    <col style="width: 25%" />
    <col style="width: 70%" />
    </colgroup>
    <thead>
    <tr class="header">
    <th>명령어</th>
    <th>사용법</th>
    <th>설명</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td>ADD</td>
    <td>ADD ... ADD [&quot;&quot;, ... &quot;&quot;]</td>
    <td>는 &quot;docker build&quot; 명령어를 통해 빌드할 컨텍스트 하위의 상대경로로 지정해야 하며 는 상대경로 또는 절대경로 모두 가능합니다. 에 URL 지정도 가능하며 URL을 통해 로 파일을 다운로드합니다.<br />
    URL형식은 적절한 파일 이름을 찾을 수 있는 형태이어야 합니다.<br />
    예) ADD http://example.com/foobar / (o), ADD http://example.com / (x) 압축형식이 tar (tar, gzip, bzip2, xz) 인 압축 파일일 경우 해제하여 복사합니다.</td>
    </tr>
    <tr class="even">
    <td>COPY</td>
    <td>COPY ... COPY [&quot;&quot;, ... &quot;&quot;]</td>
    <td>는 &quot;docker build&quot; 명령어를 통해 빌드할 컨텍스트 하위의 상대경로로 지정해야 하며 는 상대경로 또는 절대경로 모두 가능합니다. 에 URL 지정이 불가합니다. 압축 파일은 압축을 해제하지 않고 그대로 복사합니다.</td>
    </tr>
    </tbody>
    </table>

-   ENV : 환경변수를 설정합니다.
-   RUN : 새로운 레이어에서 명령어를 실행하고 결과를 커밋합니다. 보통
    이미지 위에 다른 패키지(프로그램)을 설치하여 새로운 레이어를 생성할
    때 사용합니다.

    RUN 사용 예제( ubuntu 위에 curl을 설치)

        FROM ubuntu:14.04
        RUN apt-get update
        RUN apt-get install -y curl

-   ENRTYPOINT : 컨테이너를 실행할 때 실행될 명령을 정의합니다.

    ENTRYPOINT 작성에는 아래 두가지 방식이 있습니다. 권장되는 방식은
    exec 형태입니다.

    <table style="width:100%;">
    <colgroup>
    <col style="width: 9%" />
    <col style="width: 34%" />
    <col style="width: 56%" />
    </colgroup>
    <thead>
    <tr class="header">
    <th><br />
    </th>
    <th>Syntax</th>
    <th>예시</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td>exec 방식 (권장)</td>
    <td>ENTRYPOINT [&quot;executable&quot;, &quot;param1&quot;, &quot;param2&quot;]</td>
    <td><pre><code>ENTRYPOINT [&quot;java&quot;,&quot;-Djava.security.egd=file:/dev/./urandom&quot;,&quot;-jar&quot;,&quot;/app.jar&quot;]</code></pre></td>
    </tr>
    <tr class="even">
    <td>shell 방식</td>
    <td>ENTRYPOINT command param1 param2</td>
    <td><pre><code>ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar</code></pre>
    <p>exec로 실행하지 않으면 &quot;docker stop&quot; 명령어로 컨테이너를 종료할 때 Unix 시그널인 SIGTERM을 받을수 없어 timeout 후 SIGKILL( 강제종료)됨. shell 방식에서는 Graceful Shutdown을 위해 exec로 실행해야 함.</p></td>
    </tr>
    </tbody>
    </table>

    ENTRYPOINT vs. CMD

    ENTRYPOINT와 CMD 명령어는 모두 컨테이너를 실행할 때 실행될 명령을
    정의합니다.

    CMD는 docker run 실행 시 명령어를 주지 않았을 때 사용할 기본 명령을
    설정하거나, 컨테이너를 실행할 때 대체(Override)될 수 있기 때문에
    ENTRYPOINT의 기본 파라미터를 설정하는데 사용합니다.

    Dockerfile에 둘 중 하나의 명령어는 반드시 지정되어야 합니다.

<!-- -->

-   EXPOSE : 컨테이너가 런타임 시에 지정된 포트에서 수신 대기하고 있음을
    정의합니다.   
    이 명령어는 어떤 포트가 publish 되어야 하는지에 대해 이미지를 만든
    사람과 컨테이너를 실행할 사람 사이에 일종의 규약처럼 기능합니다.  
    해당 명령어로는 실제로 포트를 publish 할 수 없으며  docker run
    명령어의 -p 또는 -P 플래그를 통해 가능합니다.

### Docker 이미지 생성 

아래와 같이 ***"docker build"*** 명령어를 실행하여 Dockerfile을 통해
이미지를 생성합니다.

-t 옵션을 사용하여 \[이미지명:태그\] 형식의 태그를 지정합니다.

``` java
$ docker build -t gs-spring-boot-docker .

Sending build context to Docker daemon  29.32MB
Step 1/5 : FROM openjdk:8-jdk-alpine
8-jdk-alpine: Pulling from library/openjdk
b56ae66c2937: Pull complete 
2296e775ba08: Pull complete 
6e753bb2ec67: Pull complete 
Digest: sha256:cee76de7d24c94a9453981cc2c95ff3b7e6de71fdb67ffc2390ab8429b886b95
Status: Downloaded newer image for openjdk:8-jdk-alpine
 ---> 3b1fdb34c52a
Step 2/5 : VOLUME /tmp
 ---> Running in 84c7fed2e2d8
 ---> 694f0b4edd9a
Removing intermediate container 84c7fed2e2d8
Step 3/5 : ADD target/gs-spring-boot-docker-0.1.0.jar app.jar
 ---> 5bb45b344163
Step 4/5 : ENV JAVA_OPTS ""
 ---> Running in c4388e5d2cab
 ---> 1a387aa87055
Removing intermediate container c4388e5d2cab
Step 5/5 : ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar
 ---> Running in a1bcf2591634
 ---> fd3cd7c70ff6
Removing intermediate container a1bcf2591634
Successfully built fd3cd7c70ff6
Successfully tagged gs-spring-boot-docker:latest
```

***"docker images"*** 명령어를 통해 이미지가 제대로 생성되었는지
확인합니다.

``` java
$ docker images

REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
gs-spring-boot-docker   latest              fd3cd7c70ff6        2 seconds ago      116MB
```

### 애플리케이션 실행(이미지실행, 컨테이너실행)

***"docker run"*** 명령어를 통해 생성한 이미지를 기반으로 애플리케이션을
실행합니다. 

Docker에서는 애플리케이션이 하나의 컨테이너로 실행됩니다.

-p \[호스트 포트:컨테이너 포트\] 옵션을 통해 컨테이너의 포트를 호스트에
publish하여 호스트에서 접속 가능하도록 실행합니다. 

``` java
$ docker run -p 8080:8080 gs-spring-boot-docker
```

docker run

Dockerfile 명령어 중 4개 (FROM, MAINTAINER, RUN, ADD)는 "***docker
run***" 으로 오버라이드 할 수 없으며 이외의 모든 명령어는 오버라이드가
가능합니다.

"***docker run***" 명령어에는 많은 옵션이 있습니다. 그 중 주요 옵션에
대한 설명입니다.

<table>
<colgroup>
<col style="width: 15%" />
<col style="width: 84%" />
</colgroup>
<thead>
<tr class="header">
<th>옵션</th>
<th>설명</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>--detach, -d</td>
<td><p>컨테이너를 실행할 때 백그라운드(&quot;detached mode&quot;)로 실행할 지에 대한 여부를 설정합니다.</p>
<p>-d 옵션을 사용하면 백그라운드로 실행되며 기본 설정은 포그라운드 모드입니다.</p>
<p>포그라운드 모드로 실행하면 표준 입력, 출력 및 오류가 콘솔에 보여질 수 있습니다.</p></td>
</tr>
<tr class="even">
<td>--env, -e</td>
<td><p>컨테이너를 실행할 때 간단한 환경 변수를 설정합니다. (배열 사용 불가)</p>
<p>Dockerfile에 정의된 변수는 덮어씁니다.</p></td>
</tr>
<tr class="odd">
<td>--link</td>
<td><p>해당 컨테이너로 다른 컨테이너를 연결합니다.</p>
<p>Docker 는 환경 변수나 호스트 파일을 통해 소스 컨테이너의 연결 정보를 해당 컨테이너로 노출합니다.</p>
<p>소스 컨테이너의 환경변수가 공유되기 때문에 보안에 취약하므로 해당 옵션을 통한 연결은 권장되지 않는 방식입니다.</p>
<p>network를 정의하여 사용하기를 권장합니다.</p></td>
</tr>
<tr class="even">
<td>--publish, -p</td>
<td>컨테이너의 포트를 호스트의 특정 포트나 특정 범위를 지정하여 매핑하여 publish합니다.</td>
</tr>
<tr class="odd">
<td>--publish-all, -P</td>
<td>컨테이너 내부의 모든 포트를 호스트의 임시 포트 범위 내의 임의의 포트에 매핑하여 publish합니다.</td>
</tr>
<tr class="even">
<td>--volume, -v</td>
<td>bind 방식으로 호스트 디렉토리에 컨테이너의 데이터를 마운트합니다.</td>
</tr>
<tr class="odd">
<td>--mount</td>
<td><p>이 옵션은 volume, bind, tmpfs 세가지 방식으로 마운트하도록 지원합니다.</p>
<p>--volume 플래그가 지원하는 대부분의 옵션을 지원하지만 문법에 차이가 있습니다.</p>
<p>--volume 플래그보다 --mount 플래그의 사용을 권장합니다.</p></td>
</tr>
<tr class="even">
<td>-it</td>
<td>Docker 는 컨테이너에 bash 쉘을 생성하여 컨테이너의 STDIN에 연결된 가상 TTY을 할당합니다.</td>
</tr>
</tbody>
</table>

  

애플리케이션 확인

아래와 같이 ***"docker ps"*** 명령어를 통해 실행중인 컨테이너 목록을
확인할 수 있습니다.

애플리케이션이 컨테이너로 실행되었음을 확인할 수 있습니다.

``` java
$ docker ps

CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                    NAMES
f4345141af03        gs-spring-boot-docker   "/bin/sh -c 'exec ..."   27 seconds ago      Up 26 seconds       0.0.0.0:8080->8080/tcp   unruffled_gates
```

<http://localhost:8080> 으로 접속하여 확인해봅니다.

![](attachments/39943173/39943171.png)

## 컨테이너 관리

### 컨테이너 생성

컨테이너 생성 및 확인은 위 애플리케이션 실행 및 확인을 참고합니다.

### 컨테이너 중지

아래와 같이 ***"docker stop \[컨테이너ID\]"*** 명령어를 통해 실행중인
컨테이너를 중지합니다.

``` java
$ docker stop f4345141af03

f4345141af03
```

***"docker ps"*** 명령어에 -a 옵션을 통해 전체 컨테이너 목록을
확인해보면 STATUS가 Exited 상태임을 확인할 수 있습니다.

``` java
$ docker ps -a

CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                       PORTS               NAMES
f4345141af03        gs-spring-boot-docker         "/bin/sh -c 'exec ..."   7 minutes ago       Exited (143) 6 minutes ago                       unruffled_gates
```

### 컨테이너 삭제 

아래와 같이 ***"docker rm \[컨테이너ID\]"*** 명령어를 통해 컨테이너를
삭제합니다.

``` java
$ docker rm f4345141af03

f4345141af03
```

삭제하고 난뒤 컨테이너 목록을 보면 해당 컨테이너가 삭제되었음을 확인할
수 있습니다.

``` java
$ docker ps -a

CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                        PORTS               NAMES
```

## 이미지 관리

컨테이너를 운영환경에 배포하는 등 이미지의 이식성을 위해 레지스트리를
통해 이미지를 관리해야 합니다.

레지스트리는 레파지토리의 모음이며  레지스트리에서 하나의 계정으로 여러
레파지토리를 생성할 수 있습니다.

Docker의 공용 퍼블릭 레지스트리인 Docker Hub는 사전에 이미 구성되어 있는
무료 레지스트리이지만 Docker Trusted Registry를 사용하여 private한
레지스트리를  설정할 수 있습니다.

### Docker Hub 활용

Docker  계정이 없다면 [https://hub.docker.com](https://cloud.docker.com)
에서 가입합니다.

***"docker login"*** 명령어를 통해 로컬에서 Docker Hub에 로그인 합니다.

``` java
$ docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: dtlabs
Password: 
Login Succeeded
```

docker login 시 에러가 발생하는 경우

Window PC에서 Docker Quickstart Terminal 이용하시는 경우

docker login 명령어 실행 시 'Cannot perform an interactive login from a
non TTY device' 에러가 발생할 수 있습니다.

그런 경우에는 아래와 같이 winpty 명령어를 앞에 붙여서 실행해 보시기
바랍니다.

$winpty docker login

  

***"docker tag image username/repository:tag"*** 명령어를 통해  이미지를
원하는 곳에 업로드 할 수 있도록 합니다.

로컬 이미지를 레파지토리와 연관시키는 표기법은 username/repository:tag
입니다.

태그는 선택 사항이지만 Docker  이미지의 버전을 제공하는 것이기 때문에
사용하기를 권장합니다.

``` java
$ docker tag gs-spring-boot-docker dtlabs/gs-spring-boot-docker:1.0
```

  

새로 태그 된 이미지를 확인합니다.

``` java
$ docker images

REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
dtlabs/gs-spring-boot-docker     1.0                 fd3cd7c70ff6        About an hour ago   116MB
gs-spring-boot-docker            latest              fd3cd7c70ff6        About an hour ago   116MB
```

  

***"docker push"*** 명령어를 통해 레파지토리에 태그된 이미지를 업로드
합니다.

``` java
$ docker push dtlabs/gs-spring-boot-docker:1.0 

The push refers to a repository [docker.io/dtlabs/gs-spring-boot-docker]
a14546d9cd41: Mounted from blingeee/gs-spring-boot-docker 
9ea2e4869a53: Mounted from blingeee/gs-spring-boot-docker 
eef4e2bfc309: Mounted from blingeee/gs-spring-boot-docker 
2aebd096e0e2: Mounted from blingeee/gs-spring-boot-docker 
1.0: digest: sha256:1c892a91e700c1868242072c32121850fcd1dfd01737066fcd41818381e9cfc3 size: 1159
```

![](attachments/39943173/39943169.png)

  

이제부터는 업로드한 이미지로 모든 컴퓨터에서 애플리케이션을 실행할 수
있습니다.

이미지가 로컬에 없다면 Docker는 Docker Hub 레지스트리의 레파지토리로부터
이미지를 가져옵니다.

``` java
$ docker run -p 8080:8080 dtlabs/gs-spring-boot-docker:1.0 
```

###  Private Registry(Harbor) 활용

 Private Registry로서 활용 가능한 Harbor를 이용해서 이미지를 보호할 수
있습니다.

  

## Attachments:

![](images/icons/bullet_blue.gif){width="8" height="8"}
[image2017-11-3\_11-9-50.png](attachments/39943173/39943169.png)
(image/png)  
![](images/icons/bullet_blue.gif){width="8" height="8"}
[image2017-11-3\_11-7-15.png](attachments/39943173/39943170.png)
(image/png)  
![](images/icons/bullet_blue.gif){width="8" height="8"}
[image2017-11-1\_19-59-54.png](attachments/39943173/39943171.png)
(image/png)  
