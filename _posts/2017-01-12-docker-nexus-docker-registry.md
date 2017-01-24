---
layout: post
title:  "Docker 로 Nexus 설치 후 Private Docker Registry 구축하기"
date:   2017-01-12 14:58:00 +0900
tags: [인프라, docker, ubuntu, nexus, docker-registry]
---

이번 포스팅에서는 Docker에 Nexus를 설치하고, Nexus를 활용하여 Private Docker Registry 를 구축하는 방법을 설명한다.

# 설명

## Docker

![Docker]({{ site.url }}/assets/images/20170112-1/docker.png){:.center}

 Docker는 Docker Inc에서 출시한 오픈 소스 컨테이너 프로젝트 이다. 전 세계적으로 Cloud Platform과 DevOps 확산으로 널리 쓰이고 있다. 기존에 VM 등으로 만든 가상화 서버에서 새로 가상화 서버를 만들 때 마다 각종 패키지를 설치하는 방식을, Docker Image를 활용하여 컴퓨터에 CD 교체 하듯이 필요한 패키지, 소프트웨어만 설치하여 활용 할 수 있다.

 더 자세한 내용은 [이재홍 님의 가장 빨리 만나는 Docker](http://www.pyrasis.com/docker.html)에서 확인한다.

## Nexus

 Nexus 는 Sonatype Inc에서 만든 Maven Repository 를 관리하기 위한 프로젝트 이다. 기존 Nexus 2 버전까지는 Maven Repository를 사설로 구축하여 공통 라이브러리 배포 용도나 다른 Maven Repository를 cache 하여 빠르게 다운로드 받을 수 있게 하는 기능이었다. Nexus 3 버전부터는 npm private repository, docker private repository 까지 구축 할 수 있다.

## Docker Registry

 Docker Image를 관리하는 프로젝트로 Docker Hub 가 오피셜 이미지를 관리한다면, Docker Registry 는 Docker Hub 를 사설로 구축 할 수 있게 해준다. 현재 Docker Inc에서 Docker Store 라는 기업을 위한 Repository 서비스를 오픈 했다.

# 설치

 설치는 **Ubuntu 14.04** 기준으로 작성했다.

## Docker

Docker 를 설치하는 방법은 [Ubuntu에서 Docker 설치하기]({{ site.baseurl }}{% link _posts/2017-01-16-docker-install.md %}) 를 확인한다.

## Nexus 설치

 설치한 Docker 에 Nexus 컨테이너를 올린다.

### nexus에서 사용할 data 폴더 생성 및 권한 변경

 nexus official image 에서 user 를 200 으로 사용한다. 만약 호스트에서 따로 데이터를 관리하지 않을 경우에는 아래 부분을 하지 않아도 된다.

```shell
mkdir ~/nexus-data && sudo chown -R 200 ~/nexus-data
```

### 위에 생성한 폴더로 nexus 실행

위 내용을 하지 않은 경우 -v /home/ubuntu/nexus-data:/nexus-data 없이 시작한다.

```shell
docker run -d -p 8081:8081 -p 12000:12000 --name nexus -v /home/ubuntu/nexus-data:/nexus-data sonatype/nexus3
```

![Nexus Image 실행 결과]({{ site.url }}/assets/images/20170112-1/docker-nexus-install.png){:.center}

### 접속

* 주소 : http://ip:8081
* 관리자 유저 ID 및 password : admin // admin123

![Nexus 접속]({{ site.url }}/assets/images/20170112-1/nexus-connect.png){:.center}

## Nexus 에 Docker Registry 설정

 Nexus 를 활용해서 Docker Registry를 서비스 하기 위해, 관리자로 로그인 후 설정창에 들어간다.

![Nexus 설정]({{ site.url }}/assets/images/20170112-1/nexus-settings.png){:.center}

 Repositories 를 클릭 후, Create repository 를 클릭한다. 그리고 docker (hosted) 를 선택한다. docker 부분에 group, hosted, proxy 가 있는데 각각의 역활은 아래와 같다.

 * group : 여러개의 repository 를 묶음
 * hosted : docker Registry
 * proxy : docker hub 또는 사설 docker registry 를 캐쉬해서 빠른 속도로 사용할 수 있게 해준다.

![Docker Registry 만들기]({{ site.url }}/assets/images/20170112-1/create-docker-registry.png){:.center}

 * Name : 이해하기 쉽게 만들면 된다.
 * HTTP : Repository로 사용할 포트 입력
 * Enable Docker V1 API : 체크

### <a id="dockerConfig"></a>Docker 설정 변경
 Docker V1 API 는 Docker Registry V1 을 의미한다. 현재 사용되고 있는 버전은 V2 로 https 만 지원한다. https 로 하려면 추가로 인증서 작업이 필요해서 해당 글에서는 Docker Registry V1 을 사용하면서 Http 로 할 수 있도록 설정을 변경한다.

 **[Nexus 서버 IP 또는 도메인] 부분을 변경해서 실행한다.**

```shell
sudo sh -c "echo 'DOCKER_OPTS=\"\$DOCKER_OPTS --insecure-registry=[Nexus 서버 IP 또는 도메인]:12000\"' >> /etc/default/docker"
```

### <a id="dockerRestart"></a>Docker 재시작

Docker 옵션을 변경하고, 서비스를 재시작하면 해당 옵션이 반영된다. Docker 재시작 시 컨테이너가 종료되서, nexus 를 다시 실행 시켜준다.

```shell
sudo service docker restart
docker start nexus
```

## Test

### <a id="dockerRegistryLogin"></a>로그인

```shell
docker login -u admin -p admin123 [Nexus 서버 IP 또는 도메인]:12000
```

### Docker Image Push, Pull Test

```shell
docker pull hello-world
docker tag hello-world [Nexus 서버 IP 또는 도메인]:12000/hello-world
docker push [Nexus 서버 IP 또는 도메인]:12000/hello-world
docker rmi hello-world [Nexus 서버 IP 또는 도메인]:12000/hello-world
docker pull [Nexus 서버 IP 또는 도메인]:12000/hello-world
docker images
```

 위에 명령어가 에러 없이 종료 되면 아래와 같이 나온다.

![Docker Registry Test 확인]({{ site.url }}/assets/images/20170112-1/docker-registry-check.png){:.center}

### 다른 서버에서 확인
 [Docker 설정 변경](#dockerConfig) , [Docker 재시작](#dockerRestart) , [Docker Registry 로그인](#dockerRegistryLogin) 를 수행 후 아래 코드를 실행한다.

 IP 와 Port 가 방화벽 상에서 우선 처리가 되어 있어야 한다.

```shell
docker pull [Nexus 서버 IP 또는 도메인]:12000/hello-world
docker images
```

# 마치며

 Docker 를 활용하여, Nexus 를 설치하고 Nexus 를 Docker Registry 로 활용했다. Docker Registry 이미지와 Nginx 를 사용하는 방법도 있으나, 용도에 따라 Repository 를 만들지 않고, Nexus 를 활용한다면, Maven 를 비롯하여, Docker 및 Npm 의 Repository 로도 활용할 수 있을 것이다.

 그리고 만약 Docker 없이 Nexus 를 설치한다면, 직접 설치 후 Nexus 에 Docker-Registry 설정 부분 부터 하면 된다.

 Docker Registry V2 설정의 경우 기회가 되면 다시 포스팅 할 예정이다.
