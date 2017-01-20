---
layout: post
title:  "Registrator 실행 및 테스트"
tags: ["인프라", "ubuntu", "consul", "registrator"]
date:   2017-01-20 19:50:00 +0900
---

Registrator는 [Gliderlabs](http://gliderlabs.com/registrator/latest/)에서 MIT 라이센스로 만든 Docker container를 자동으로 Consul, etcd, SkyDNS2에 등록해주고 제거해주는 어플리케이션이다.

# 들어가기전에

기존에 포스팅한 Docker, Swarm, Consul로 클러스터링된 환경에서 진행한다.

기존 환경세팅을 하지 않은 사람은 이전 포스트로 클러스터링 환경을 구성하고 진행한다.

* [Docker Swarm과 Consul로 Docker 클러스터링 하기]({{ site.baseurl }}{% link _posts/2017-01-19-docker-swarm-consul.md %})

# <a id="Run"></a>실행

[Consul 서버 IP]부분에 Web UI가 설치된 호스트라면 외부와 통신 가능한 IP, consul agent는 0.0.0.0을 적으면 된다.

``` shell
docker -H unix:///var/run/docker.sock run -d \
    --name=registrator \
    --net=host \
    --volume=/var/run/docker.sock:/tmp/docker.sock \
    gliderlabs/registrator:latest \
      consul://[Consul 서버 IP]:8500
```

# 테스트

``` shell
docker -H unix:///var/run/docker.sock run -d -P --name=redis redis
```

실행 한 이후에 컨테이너를 실행하면 Consul Web UI에서 실행된 컨테이너를 확인할 수 있다.

![Consul ui 화면]({{ site.url }}/assets/images/docker-swarm-consul/consul-registrator.png)

실행 화면에서 redis 서비스가 추가된 것을 볼 수 있다.

# 서비스 스크립트 작성

Registrator가 서버 부팅 시 자동으로 올라올 수 있게, 서비스를 추가한다.

``` shell
sudo vi /etc/init/registrator.conf
```

스크립트 내용은 아래와 같다.

``` shell
description "Registrator process"

start on docker and consul
stop on runlevel [!12345]

respawn

setuid ubuntu
setgid ubuntu

exec docker -H unix:///var/run/docker.sock run --rm --name=registrator --net=host --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://[Consul 서버 IP]:8500
```

# 서비스 실행 및 종료

[실행](#Run)에서 docker run을 수행한 경우, 종료 및 삭제처리 후 실행한다.

``` shell
docker -H unix:///var/run/docker.sock stop registrator && docker -H unix:///var/run/docker.sock rm registrator
```

1. 실행

   ``` shell
   sudo start registrator
   ```

2. 종료

   ``` shell
   sudo sop registrator
   ```
