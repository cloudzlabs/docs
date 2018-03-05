---
date: "2018-02-23T16:42:27+09:00"
title: "K8S App deploy script"
authors: ["bckim0620"]
categories:
  - posts
tags:
  - Kuberetes
  - Docker
  - Deploy
draft: false
---

최근 Container 기술이 각광을 받고 있습니다. 그래서 저도 Kubernets 교육을 수강하고 있는데, Kubernets에 배포 한번 하기 참 힘드네요.
Project build, Docker image build, Docker push, K8s deploy 총 4개의 과정을 거쳐야 애플리케이션 배포가 끝이 납니다.
도저히 이 과정을 참을 수 없어서 배포 스크립트를 만들었습니다. 사용방법은 아래 설명하였으니, 잘 활용하시기 바랍니다.

### 사용법

#### 1. deploy.sh 저장
페이지 하단 deploy.sh 파일을 개발중인 프로젝트 루트에 저장합니다.

#### 2. 설정
deploy.sh 파일의 상단 설정을 입력합니다.

  ```sh
  #!/bin/sh

  #작성중인 Project 종류
  #maven | node
  PROJECT_TYPE="maven"

  #Maven build 과정 중 테스트 수행 여부
  MAVEN_TEST_SKIP=true;
  NODE_NPM_INSTALL=false;

  #기존에 생성된 Docker Image, Kubernetes deploy, service 삭제여부
  #기존에 생성된 Kubernetes deploy, service는 kubectl apply로 덮어쓰기 불가, 삭제 후 재생성 필요 
  DELETE_PREVIOUS_DOCKER_IMAGE_AND_KUBERNETES_DEPLOYMENT=true

  #Docker Repository 정보
  #Dockerhub registry: docker.io
  #Harbor registry: harbor1.ghama.io
  #Docker hub는 DOCKER_REPOSITORY_PROJECT와 DOCKER_REPOSITORY_USER 동일

  DOCKER_REPOSITORY_URL="docker.io"
  DOCKER_REPOSITORY_PROJECT=""
  DOCKER_REPOSITORY_USER=""
  DOCKER_REPOSITORY_PASSWORD=""
  DOCKER_IMAGE_NAME=""


  #Kuberetes deployment, service/Dockerfile 경로
  #파일이 존재하는 폴더까지의 경로
  KUBERNETES_DEPLOYMENT_PATH="./k8s/"
  DOCKER_FILE_PATH="./"

  #Kuberetes Cluster 정보
  #ICP의 경우 포탈에서 Configur client 코드를 복사하여 showlog "Login kubernetes" 밑에 덮어쓰기
  KUBERNETES_CLUSTER_NAME="mycluster.icp"
  KUBERNETES_CLUSTER_URL="https://169.56.113.156:8001"
  KUBERNETES_CLUSTER_USERNAME="admin"
  KUBERNETES_CLUSTER_TOKEN="eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsImF0X2hhc2giOiIzT2lwNy01VzZaNnRfVGxDQmVoR1pnIiwiaXNzIjoiaHR0cHM6Ly9teWNsdXN0ZXIuaWNwOjk0NDMvb2lkYy9lbmRwb2ludC9PUCIsImF1ZCI6ImRmODJmYjY3YWEyMWZjZTc5N2M1ODBjNDI5MTEyMmQxIiwiZXhwIjoxNTE3NTAyNDc4LCJpYXQiOjE1MTc0NTkyNzh9.cPG-_QtnLMqM1gV6TREurD8ToUM2c2GbrjcZJfXIvRZEgiNeyhqUp2HzBncNJIcx3LKWmNlNInTz-HTR1spt9j-EyTtetTQTspfW1GmuA4FX207yLwHz7M9veRkQx7uOMekUomfUYnZYyuXEUnicthrilhqldz6gIS-xtd9cnSU-d0qIOR3Syl1O20z_ektN5GL0KqoFaC_QHY2in8h94E_jSnuXU3WPCRxOaet31r0LquXHaIFgdRZAMbKqlr5j15YTSpnctOzq-ipbacFKE-UlPaep5bEm-UMqTbrGW4a318x8TXRjmNdbYqQaQ--tBKhmPhPc_8xnWXgcMPfSFQ"
  KUBERNETES_CLUSTER_NAMESPACE="dtlabs08"
  KUBERNETES_CONTEXT_NAME="mycluster.icp-context"
  ```

#### 3. deploy.sh 실행
deploy.sh를 실행합니다. 단, Windows docker toolbox를 사용할 경우 docker terminal에서 실행해야합니다.

```sh
$ ./deploy.sh
```

### deploy.sh 코드
```sh
#!/bin/sh

#작성중인 Project 종류
#maven | node
PROJECT_TYPE="maven"

#Maven build 과정 중 테스트 수행 여부
MAVEN_TEST_SKIP=true;
NODE_NPM_INSTALL=false;

#기존에 생성된 Docker Image, Kubernetes deploy, service 삭제여부
#기존에 생성된 Kubernetes deploy, service는 kubectl apply로 덮어쓰기 불가, 삭제 후 재생성 필요 
DELETE_PREVIOUS_DOCKER_IMAGE_AND_KUBERNETES_DEPLOYMENT=true

#Docker Repository 정보
#Dockerhub registry: docker.io
#Harbor registry: harbor1.ghama.io
#Docker hub는 DOCKER_REPOSITORY_PROJECT와 DOCKER_REPOSITORY_USER 동일

DOCKER_REPOSITORY_URL="harbor1.ghama.io"
DOCKER_REPOSITORY_PROJECT="test"
DOCKER_REPOSITORY_USER="bckim0620"
DOCKER_REPOSITORY_PASSWORD="Skcc1234"
DOCKER_IMAGE_NAME="test-app"


#Kuberetes deployment, service/Dockerfile 경로
#파일이 존재하는 폴더까지의 경로
KUBERNETES_DEPLOYMENT_PATH="./k8s/"
DOCKER_FILE_PATH="./"

#Kuberetes Cluster 정보
#ICP의 경우 포탈에서 Configur client 코드를 복사하여 showlog "Login kubernetes" 밑에 덮어쓰기
KUBERNETES_CLUSTER_NAME="mycluster.icp"
KUBERNETES_CLUSTER_URL="https://169.56.113.156:8001"
KUBERNETES_CLUSTER_USERNAME="admin"
KUBERNETES_CLUSTER_TOKEN="eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsImF0X2hhc2giOiJiekJud0JVcmVGQmRrVVBSSE5DTl9BIiwiaXNzIjoiaHR0cHM6Ly9teWNsdXN0ZXIuaWNwOjk0NDMvb2lkYy9lbmRwb2ludC9PUCIsImF1ZCI6ImRmODJmYjY3YWEyMWZjZTc5N2M1ODBjNDI5MTEyMmQxIiwiZXhwIjoxNTE3NjAzMTk5LCJpYXQiOjE1MTc1NTk5OTl9.sNL3jmBt1844D3gJkiVSC3NUhYzd3auL0HKLWIPfJ_qLEadLSBoOl8qk__Cjd26eAPSaMcR9TDE2TgleeWKASzM_E4tVuuzo_U0hqr8xQApvdYXnsrzr-US0PtVQusIv9NLGi7NxYIMdcLVWUAPuCJAsyK4IxxtjZ6hlTDpuaqz4JEbWvjE6yWmtPxXSBnVvWtT8RkjXsmUdDoVD2UdSSOIASr24oc8xXmkNiyV5xKL3XjsuM1MU5VNkL9wv6c2iB5EEDFrEPsEMturxoTVEQqUVJ0kq0PTK_uVIEBJxXbNnUICFgbVzlzqXY56t-cfkITNwiAJ72P_O2pGpc4niQQ"
KUBERNETES_CLUSTER_NAMESPACE="dtlabs11"
KUBERNETES_CONTEXT_NAME="mycluster.icp-context"



function showlog() {
	echo ""
	echo ""
	echo ""
	echo "--------------------------- $1 ---------------------------"
}

showlog "deploy.sh"

showlog "Packaging"

if [ "${PROJECT_TYPE}" == "maven" ]; then
	showlog "Maven packaging"
	if ./mvnw clean install -Dmaven.test.skip=${MAVEN_TEST_SKIP}; then
		echo ""
	else
		exit 1
	fi
else
	showlog "Node packaging"
	if ${NODE_NPM_INSTALL}; then
		npm install
	fi
fi

if ${DELETE_PREVIOUS_DOCKER_IMAGE_AND_KUBERNETES_DEPLOYMENT}; then
	showlog "Delete docker image, kubernetes deployment"
	kubectl delete -f ${KUBERNETES_DEPLOYMENT_PATH}
	docker rmi ${DOCKER_REPOSITORY_URL}/${DOCKER_REPOSITORY_PROJECT}/${DOCKER_IMAGE_NAME}
	docker rmi ${DOCKER_REPOSITORY_PROJECT}/${DOCKER_IMAGE_NAME}
fi

showlog "Build/Tag docker image" 
docker build ${DOCKER_FILE_PATH} -t ${DOCKER_REPOSITORY_URL}/${DOCKER_REPOSITORY_PROJECT}/${DOCKER_IMAGE_NAME}

showlog "Login docker repository"
docker login ${DOCKER_REPOSITORY_URL} -u ${DOCKER_REPOSITORY_USER} -p ${DOCKER_REPOSITORY_PASSWORD}

showlog "Push Docker image"
docker push ${DOCKER_REPOSITORY_URL}/${DOCKER_REPOSITORY_PROJECT}/${DOCKER_IMAGE_NAME}

showlog "Login kubernetes"
kubectl config set-cluster ${KUBERNETES_CLUSTER_NAME} --server=${KUBERNETES_CLUSTER_URL} --insecure-skip-tls-verify=true
kubectl config set-context ${KUBERNETES_CONTEXT_NAME} --cluster=${KUBERNETES_CLUSTER_NAME} 
kubectl config set-credentials ${KUBERNETES_CLUSTER_USERNAME} --token=${KUBERNETES_CLUSTER_TOKEN}
kubectl config set-context ${KUBERNETES_CONTEXT_NAME} --user=${KUBERNETES_CLUSTER_USERNAME} --namespace=${KUBERNETES_CLUSTER_NAMESPACE}
kubectl config use-context ${KUBERNETES_CONTEXT_NAME}

showlog "Create kuberetes deployment"
kubectl apply -f ${KUBERNETES_DEPLOYMENT_PATH}

showlog "Pod infomation"
kubectl get pod

showlog "Service infomation"
kubectl get service
```
