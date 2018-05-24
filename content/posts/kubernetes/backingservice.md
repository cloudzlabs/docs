---
date: "2018-05-08T15:23:03+09:00"
title: "[Kubernetes 활용(4/8)] Mysql DB 연동하기"
authors: ["blingeeeee"]
categories:
  - posts
tags:
  - kubernetes
  - container
  - container orchestration
  - 서비스 연동
  - DB 연동
  - service discovery
  - k8s
description: ""
draft: false
---
지난 챕터에서는 사용한 리소스를 기반으로 애플리케이션의 수를 자동으로 조절할 수 있는 HPA 라는 기능을 적용해 보았습니다. <br/>그렇다면 애플리케이션에 Mysql 또는 Redis 등과 같은 서비스를 연동하고 싶을 땐 어떻게 해야할까요?<br/>
Kubernetes에서는 애플리케이션에서 필요한 서비스를 Docker Image를 사용하여 바로 구성할 수 있습니다.<br/> 물론 대부분이 오픈소스 솔루션에 대한 서비스겠지요.<br/>
Legacy에 있는 서비스들 역시 연동이 가능하긴 하지만, 여기서는 Docker Image 를 통해 Mysql DB를 구성하고 애플리케이션에 연동해보도록 하겠습니다. 

**샘플 애플리케이션에 대한 자세한 설명은 Spring의 [Accessing data with
MySQL](https://spring.io/guides/gs/accessing-data-mysql/) 문서를
참고하시기 바랍니다.**

## 서비스 사용 워크플로우
------------------------------------------------------------------------

1.  Kubernetes에서 사용할 수 있는 서비스(Mysql, Redis 등등)는 [Docker
    Store](https://store.docker.com/)에서 공식 Image를 Pull받아 구성할
    수 있습니다.
2.  사용할 서비스의 Image를 선택한 후, 서비스를 생성합니다.
3.  생성한 서비스를 사용하기를 원하는 애플리케이션에 서비스 연결 정보를
    입력한 후, 동일한 Namespace에 애플리케이션을 배포하면 됩니다.
4.  해당 Namespace에서 더 이상 서비스 인스턴스를 사용하지 않는다면
    서비스를 삭제합니다.

아래에서 서비스 워크플로우 단계별로 자세히 설명합니다. 


## 서비스 사용방법
------------------------------------------------------------------------

### 서비스 생성

  Kubernetes에서 서비스를 사용하기 위해 Docker Image를 통해 서비스를 생성합니다.

  1.  Kubernetes 환경에서 사용할 Mysql DB 이미지를 검색합니다.
  2.  **Deployment YAML 파일** 을 작성합니다.(spec.image 항목에 검색한
        Mysql DB Image 명을 적용)

      ```yml
      apiVersion: apps/v1beta2
      kind: Deployment
      metadata:
        name: gs-mysql-data-deployment
        labels:
          app: gs-mysql-data
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: gs-mysql-data
        template:
          metadata:
            labels:
              app: gs-mysql-data
          spec:
            containers:
            - env:
              - name: MYSQL_DATABASE
                value: db_example
              - name: MYSQL_PASSWORD
                value: ThePassword
              - name: MYSQL_ROOT_PASSWORD
                value: root
              - name: MYSQL_USER
                value: springuser
              image: mysql
              imagePullPolicy: IfNotPresent 
              name: gs-mysql-data-container
              ports:
              - containerPort: 3306
            restartPolicy: Always
      status: {}
      ```

  3.  **Service YAML 파일**을 작성합니다.

      ```yml
      apiVersion: v1
      kind: Service
      metadata:
        name: gs-mysql-data
      spec:
        ports:
        - name: "3306"
          port: 3306
          targetPort: 3306
        selector:
          app: gs-mysql-data
        type: NodePort
      ```

  4.  Deployment와 Service Object를 생성합니다.

      ``` bash
      $ kubectl apply -f gs-mysql-data-deployment.yaml -f gs-mysql-data-service.yaml
      deployment "gs-mysql-data" created
      service "gs-mysql-data" created
      ```

      ``` bash
      $ kubectl get pod,svc,deployment
      NAME                                    READY     STATUS    RESTARTS   AGE
      po/gs-mysql-data-1903746765-26mlf       1/1       Running   0          8s

      NAME                        TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)             AGE
      svc/gs-mysql-data           NodePort       10.0.0.19    <none>        3306:<DB_PORT>/TCP  8s

      NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
      deploy/gs-mysql-data       1         1         1            1           8s
      ```

  5.  `kubectl exec` 명령어를 입력해 생성된 MySQL에
      springuser/ThePassword로 접속하여 DB가 정상적으로 동작하는지
      확인합니다.

      ``` bash
      $ kubectl exec -it gs-mysql-data-1903746765-26mlf bash 
      root@gs-mysql-data-1903746765-26mlf:/# mysql -u springuser -p
      Enter password: 
      Welcome to the MariaDB monitor.  Commands end with ; or \g.
      Your MySQL connection id is 3
      Server version: 5.7.20 MySQL Community Server (GPL)


      Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

      Oracle is a registered trademark of Oracle Corporation and/or its
      affiliates. Other names may be trademarks of their respective
      owners.

      Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

      mysql> \u db_example
      Database changed
      mysql>
      ```

### 서비스 연결

애플리케이션에서 생성한 서비스를 사용하는 방법에 대해서 설명합니다. 생성한 서비스 명으로 애플리케이션과 연동합니다.<br/>
아래는 두가지의 서비스 디스커버리 방식에 대한 설명입니다.
해당 샘플에서는 환경변수 방식을 통해 연동해 보도록 하겠습니다.


##### 서비스 디스커버리 방식
+ 환경 변수 방식
  - Pod 생성 시 컨테이너 내부에 모든 서비스들의 HOST, PORT 정보가 환경변수로 주입됨
  - 컨테이너에서 환경변수를 통해 다른 서비스에 대한 접속 정보를 알수 있음
  - redis-master 서비스가 미리 생성되어 있는 경우 , 애플리케이션을  Pod로 띄우면 내부에 아래와 같은 네이밍 룰에 따라 환경 변수가 주입됨
  ![](redis-master.png)
+ DNS 서버 방식
  - Kubernetes에 DNS 서비스를 구성하고 서비스명으로 접근 
  - 예를 들면 호출 url이 redis-master:6379 와 같은 형태
  - **다른 네임스페이스에 있는 서비스도 접근 가능함**

1. 애플리케이션의 설정파일에 서비스 정보 작성 
  * 아래 예제의 경우 Spring Boot 애플리케이션에서 gs-mysql-data 서비스를 사용하기 위해 **application.properties** 설정 파일에 작성한 정보입니다.   

    ``` text
    spring.jpa.hibernate.ddl-auto=create
    spring.datasource.url=jdbc:mysql://${GS_MYSQL_DATA_SERVICE_HOST}:${GS_MYSQL_DATA_SERVICE_PORT}/${MYSQL_DATABASE}
    spring.datasource.username=${MYSQL_USER}
    spring.datasource.password=${MYSQL_PASSWORD}
    ```
  * 연결할 서비스의 접속정보는 **\[서비스명\]\_SERVICE\_HOST** , **\[서비스명\]\_SERVICE\_PORT** 형태로 기입합니다. <br/>
  * 그외의 나머지 Credential 정보는 Container의 환경변수로 설정하고,애플리케이션에서는 주입받아 사용할 수 있도록 합니다.

2. Image 빌드(애플리케이션 컨테이너화)
  * 이미지를 정의하는 Dockerfile을 작성합니다.

    ``` bash
    FROM openjdk:8-jdk-alpine
    ADD target/gs-mysql-data-0.1.0.jar app.jar
    EXPOSE 8080 
    ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ```

    위치는 다음과 같습니다.

    ``` bash
    /gs-accessing-data-mysql/complete $ tree  
    .
    ├── Dockerfile
    ├── build.gradle
    ├── docker-compose.yml
    ├── gradle
    ├── gradlew
    ├── gradlew.bat
    ├── mvnw
    ├── mvnw.cmd
    ├── pom.xml
    ├── src
    └── target
    ```

  * `docker build` 명령어를 사용하여, 애플리케이션에 대한 Image를 구성합니다.

    ``` bash
    /gs-accessing-data-mysql/complete $ docker build -t dtlabs/gs-mysql-data .
    Sending build context to Docker daemon  28.26MB
    Step 1/4 : FROM openjdk:8-jdk-alpine
    8-jdk-alpine: Pulling from library/openjdk
    Digest: sha256:388566cc682f59a0019004c2d343dd6c69b83914dc5c458be959271af2761795
    Status: Downloaded newer image for openjdk:8-jdk-alpine
      ---> 3642e636096d
    Step 2/4 : ADD target/gs-mysql-data-0.1.0.jar app.jar
      ---> 2419541c05c4
    Step 3/4 : EXPOSE 8080
      ---> Running in 4b6ef032a843
      ---> e866f2e8876e
    Removing intermediate container 4b6ef032a843
    Step 4/4 : ENTRYPOINT java -Djava.security.egd=file:/dev/./urandom -jar /app.jar
      ---> Running in 824af86ada81
      ---> 7bc04f36cf6a
    Removing intermediate container 824af86ada81
    Successfully built 7bc04f36cf6a
    Successfully tagged dtlabs/gs-mysql-data:latest
    ```
  
3. Image를 Registry로 업로드
  * 만들어진 Image를  `docker push` 명령어로 Registry에 Push합니다. 이전에 Push된 Image가 있을 경우, 변경된 Layer만 Push됩니다.           

    ``` bash
    /gs-accessing-data-mysql/complete $ docker push dtlabs/gs-mysql-dataㅁ
    The push refers to a repository [docker.io/dtlabs/gs-mysql-data]
    44630b8580cc: Pushed 
    69cc5717c281: Layer already exists 
    5b1e27e74327: Layer already exists 
    04a094fe844e: Layer already exists 
    latest: digest: sha256:c664a46b2341683aebcb8da9ea31e93df3c2225552dc38f494d664cfa39582d1 size: 1159
    ```

4. Kubernetes 환경에 애플리케이션 배포(컨테이너)
  * 임의의 폴더를 생성하고 애플리케이션의 Deployment, Service Object 배포를 위한 YAML 파일들을 작성합니다.

    **gs-mysql-data-app-deployment.yaml**

    ``` yml
    apiVersion: apps/v1beta2
    kind: Deployment
    metadata:
      name: gs-mysql-data-app-deployment
      labels:
        app: gs-mysql-data-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: gs-mysql-data-app
      template:
        metadata:
          labels:
            app: gs-mysql-data-app
        spec:
          containers:
          - env:
            - name: MYSQL_DATABASE
              value: db_example
            - name: MYSQL_PASSWORD
              value: ThePassword        
            - name: MYSQL_USER
              value: springuser
            image: dtlabs/gs-mysql-data
            imagePullPolicy: IfNotPresent
            name: gs-mysql-data-app-container
            ports:
            - containerPort: 8080
          restartPolicy: Always
    status: {}
    ```

    **env** 항목에 gs-mysql-data의 연결 정보를 설정합니다.

    **gs-mysql-data-app-service.yaml**

    ``` yml
    apiVersion: v1
    kind: Service
    metadata:
      name: gs-mysql-data-app
    spec:
      ports:
      - name: "8080"
        port: 8080
        targetPort: 8080
      selector:
        app: gs-mysql-data-app
      type: NodePort
    ```

    위치는 다음과 같습니다.

    ``` bash
    /gs-accessing-data-mysql/complete $ tree  
    .
    ├── Dockerfile
    ├── build.gradle
    ├── docker-compose.yml
    ├── gradle
    ├── gradlew
    ├── gradlew.bat
    ├── kubernetes
    │   ├── gs-mysql-data-app-deployment.yaml
    │   ├── gs-mysql-data-app-service.yaml
    ├── mvnw
    ├── mvnw.cmd
    ├── pom.xml
    ├── src
    └── target1
    ```

* kubernetes 폴더로 이동하여, `kubectl apply` 명령어로 Kubernetes 환경에 애플리케이션을 배포합니다.

    ``` bash
    /gs-accessing-data-mysql/complete/kubernetes $ kubectl apply -f gs-mysql-data-app-deployment.yaml -f gs-mysql-data-app-service.yaml
    deployment "gs-mysql-data-app" created
    service "gs-mysql-data-app" created
    ```

* kubectl get 명령어를 입력해 Pod, Service, Deployment가
    제대로 생성되었는지 확인합니다.

    ``` bash
    /gs-accessing-data-mysql/complete/kubernetes $ kubectl get po,svc,deploy
    NAME                                    READY     STATUS    RESTARTS   AGE
    po/gs-mysql-data-1903746765-26mlf       1/1       Running   0          22m
    po/gs-mysql-data-app-1364076206-0x3lp   1/1       Running   0          12m

    NAME                        TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
    svc/gs-mysql-data           NodePort       10.0.0.19    <none>        3306:<DB_PORT>/TCP       22m
    svc/gs-mysql-data-app       NodePort       10.0.0.25    <none>        8080:<EXTERNAL_PORT>/TCP  12m

    NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    deploy/gs-mysql-data       1         1         1            1           22m
    deploy/gs-mysql-data-app   1         1         1            1           12m
    ```

### 서비스 연결 확인

컨테이너로 배포된 애플리케이션이 정상적으로 연결되는지 확인해보겠습니다.

  1. 먼저 애플리케이션의 Pod 이름을 확인하고, `kubectl describe` 명령어로 생성된 Pod의 상세정보를 확인합니다. 

    ``` bash
    /gs-accessing-data-mysql/complete/kubernetes $ kubectl get pod
    NAME                                 READY     STATUS    RESTARTS   AGE
    gs-mysql-data-1903746765-26mlf       1/1       Running   0          25m
    gs-mysql-data-app-1364076206-0x3lp   1/1       Running   0          15m 

    /gs-accessing-data-mysql/complete/kubernetes $ kubectl describe pod gs-mysql-data-app-1364076206-0x3lp
    Name:           gs-mysql-data-app-1364076206-0x3lp
    Namespace:      default
    Node:           poc.k8s-worker02.cloudz.co.kr/<EXTERNAL_HOST>

    ... 이하 생략 ...
    ```


  2. `kubectl get` 명령어로 Service 목록을 확인합니다. PORT(s) 항목에서 gs-mysql-data-app의 내부 Port는 8080, 외부 Port는 **EXTERNAL\_PORT**로 설정되어 있는 것을 확인할 수 있습니다.

    ``` bash
    /gs-accessing-data-mysql/complete/kubernetes $ kubectl get svc
    NAME                    TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
    gs-mysql-data           NodePort       10.0.0.19    <none>        3306:<DB_PORT>/TCP       30m
    gs-mysql-data-app       NodePort       10.0.0.25    <none>        8080:<EXTERNAL_PORT>/TCP  20m
    ```

  3.  `curl` 명령어로 **gs-mysql-data-app** 애플리케이션이 MySQL 서비스와 연동하여 정상적으로 동작하는지 확인합니다.

    ``` bash
    /gs-accessing-data-mysql/complete/kubernetes $ curl 'http://<EXTERNAL_HOST>:<EXTERNAL_PORT>/demo/all'
    []

    /gs-accessing-data-mysql/complete/kubernetes $ curl 'http://<EXTERNAL_HOST>:<EXTERNAL_PORT>/demo/add?name=First&email=someemail@someemailprovider.com' 
    Saved

    /gs-accessing-data-mysql/complete/kubernetes $ curl 'http://<EXTERNAL_HOST>:<EXTERNAL_PORT>/demo/all'                                                 
    [{"id":1,"name":"First","email":"someemail@someemailprovider.com"}]
    ```

### 서비스 삭제

더 이상 서비스를 사용하지 않는다면 Deployment와 Service를 삭제합니다.

* Kubernetes 설정 파일이 있는 폴더로 이동하여, `kubectl delete` 명령어로 Deployment와 Service를 삭제합니다.

    ``` bash
    $ kubectl delete -f gs-mysql-data-deployment.yaml -f gs-mysql-data-service.yaml
    deployment "gs-mysql-data" deleted
    service "gs-mysql-data" deleted
    ```
