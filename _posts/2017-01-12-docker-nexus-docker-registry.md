---
layout: post
title:  "Docker 로 Nexus 설치 후 docker-registry 구성하기"
date:   2017-01-12 14:58:00 +0900
---
# Docker, Nexus, Docker-Registry 란 무엇인가

## Docker

![Docker]({{ site.url }}/assets/images/20170112-1/docker.png){:.center}

 Docker는 Docker Inc에서 출시한 오픈 소스 컨테이너 프로젝트 입니다. 전 세계적으로 Cloud Platform과 DevOps 확산으로 널리 쓰이고 있습니다. 기존에 VM 등으로 만든 가상화 서버에서 새로 가상화 서버를 만들 때 마다 각종 패키지를 설치하는 방식을, Docker Image를 활용하여 컴퓨터에 CD 교체 하듯이 필요한 패키지, 소프트웨어만 설치하여 활용 할 수 있습니다.

 더 자세한 내용은 [이재홍 님의 가장 빨리 만나는 Docker](http://www.pyrasis.com/docker.html)에서 확인바랍니다.

## Nexus

 Nexus 는 Sonatype Inc에서 만든 Maven Repository 를 관리하기 위한 프로젝트 입니다. 기존 Nexus 2 버전까지는 Maven Repository를 사내 환경에서 구축하여 사용하는 용도 였으나, 현재 Nexus 3 버전부터는 npm private repository, docker private repository 를 생성할 수 있습니다.
 
## Docker-Registry

 Docker Image를 관리하는 프로젝트로 Docker Hub 가 오피셜 이미지를 관리한다면, Docker-Registry 는 Docker Hub 를 사설로 구축 할 수 있게 해준다. 현재 Docker Inc에서 Docker Store 라는 기업을 위한 Repository 서비스를 오픈 했다.

# 설치

 설치는 *Ubuntu 14.04* 기준으로 작성했다.

## Docker

{% highlight bash %}
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
sudo mkdir -p /etc/apt/sources.list.d
sudo sh -c "echo 'deb https://apt.dockerproject.org/repo ubuntu-trusty main' > /etc/apt/sources.list.d/docker.list"
sudo apt-get install linux-image-extra-$(uname -r)
sudo apt-get install apparmor
sudo apt-get update
sudo apt-get install docker-engine
sudo usermod -aG docker ubuntu
{% endhighlight %}

마지막 명령어 이후에 다시 로그인 한다.

### 설치 확인
{% highlight bash %}
docker info
{% endhighlight %}

![Docker]({{ site.url }}/assets/images/20170112-1/docker-install-check.png){:.center}

## Nexus 설치

 설치한 Docker 에 Nexus 컨테이너를 올린다.

### nexus-data 폴더 생성 및 권한 변경

 nexus image 가 user 200을 사용한다.

{% highlight bash %}
mkdir ~/nexus-data && sudo chown -R 200 ~/nexus-data 
{% endhighlight %}

### 위에 생성한 폴더로 nexus 실행

{% highlight bash %}
docker run -d -p 8081:8081 --name nexus -v /home/ubuntu/nexus-data:/nexus-data sonatype/nexus3
{% endhighlight %}

![Nexus Image 실행 결과]({{ site.url }}/assets/images/20170112-1/docker-nexus-install.png){:.center}

### 접속

* 주소 : http://ip:8081
* 유저 ID 및 password : admin // admin123


-- 작성중