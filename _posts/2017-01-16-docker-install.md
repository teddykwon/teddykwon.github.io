---
layout: post
title:  "Ubuntu에서 Docker 설치하기"
date:   2017-01-16 18:40:00 +0900
tags: [인프라, docker, ubuntu]
---

이번 포스팅에서는 Ubuntu 에 Docker 를 설치하는 스크립트를 설명한다.

# Docker 란

![Docker]({{ site.url }}/assets/images/20170112-1/docker.png){:.center}

 Docker는 Docker Inc에서 출시한 오픈 소스 컨테이너 프로젝트 이다. 전 세계적으로 Cloud Platform과 DevOps 확산으로 널리 쓰이고 있다. 기존에 VM 등으로 만든 가상화 서버에서 새로 가상화 서버를 만들 때 마다 각종 패키지를 설치하는 방식을, Docker Image를 활용하여 컴퓨터에 CD 교체 하듯이 필요한 패키지, 소프트웨어만 설치하여 활용 할 수 있다.

 더 자세한 내용은 [이재홍 님의 가장 빨리 만나는 Docker](http://www.pyrasis.com/docker.html)에서 확인한다.

# 설치

 설치는 **Ubuntu 14.04** 기준으로 작성했다.

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
