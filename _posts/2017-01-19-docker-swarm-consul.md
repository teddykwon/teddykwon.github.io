---
layout: post
title:  "Docker Swarm과 Consul로 Docker 클러스터링 하기"
tags: ["인프라", "ubuntu", "docker", "swarm", "consul"]
---

여러 호스트에 Docker Swarm 과 Consul 를 설치하여 인프라를 구축 하는 방법을 설명하려고 한다.

기본적으로 아래 환경을 구성하기 위해 **2대 이상** 의 호스트가 필요하다. 필요한 호스트 서버는 AWS 등 클라우드 서비스나 VM 또는 Docker machine 등을 활용하여 구성 후 진행한다.

# 설명

Docker Swarm은 여러대의 호스트 및 컨테이너를 관리하는 Docker 클러스터링 도구이다. 1000개의 노드(호스트) 와 50,000개의 컨테이너까지 성능 저하 없이 확장 할 수 있다고 한다. Docker Swarm을 활용하게 되면 여러대의 호스트에 설치되어 있는 Docker를 하나의 Docker처리 사용 할 수 있게 해준다.

Docker Swarm은 **Docker 1.12 버전 이상** 부터 사용 가능하다.

Consul 에 대한 내용은 [여러 호스트에 Consul 설치 후 연동하기]({{ site.baseurl }}{% link _posts/2017-01-18-consul-install.md %}) 에서 확인한다.

# <a id="DockerSwarmInstall"></a>설치

설치는 **Ubuntu 14.04** 으로 진행한다.

Docker Swarm 을 설치하려면 기본적으로 Docker를 먼저 설치해야 한다. Docker 를 설치하는 방법은 [Ubuntu에서 Docker 설치하기]({{ site.baseurl }}{% link _posts/2017-01-16-docker-install.md %}) 를 확인한다.

Consul 설치는 [여러 호스트에 Consul 설치 후 연동하기]({{ site.baseurl }}{% link _posts/2017-01-18-consul-install.md %})를 먼저 수행 후 작업한다.

1. go 설치 및 환경변수 설정

   ``` shell
   sudo add-apt-repository ppa:ubuntu-lxc/lxd-stable
   sudo apt-get update
   sudo apt-get install golang

   sudo mkdir /tmp/goworkspace && sudo chown -R ubuntu:ubuntu /tmp/goworkspace
   echo 'export GO15VENDOREXPERIMENT=1' >> ~/.bash_profile
   echo 'export GOPATH=/tmp/goworkspace' >> ~/.bash_profile
   source ~/.bash_profile
   ```

2. swarm 빌드

   ``` shell
   go get -d -u github.com/docker/swarm
   cd $GOPATH/src/github.com/docker/swarm
   git checkout v1.2.5
   go install

   sudo cp $GOPATH/bin/swarm /usr/local/bin
   sudo cp $GOPATH/bin/swarm /usr/bin
   ```

## 설치 확인

``` shell
swarm -v
```

아래의 이미지 같이 나오면 된다.

![Swarm Version 확인]({{ site.url }}/assets/images/docker-swarm-consul/swarm-version.png){:.center}

# 설정 및 연동

처음 글에도 설명했듯이 Swarm과 Consul을 설정하기 위해서는 최소한 두 대 이상의 호스트가 필요하다.

설치는 2대의 호스트로 진행할 예정이어서 다른 호스트에 설치를 진행하지 않았다면 [Docker Swarm 설치](#DockerSwarmInstall)를 다른 호스트에서도 수행한다.

## 방화벽 설정

Swarm manager와 node 끼리 통신을 위하여 아래 방화벽들을 오픈해준다. 방화벽 오픈은 내부 네트워크 안에서 통신할 수 있도록 한다.

1. Swarm manager

   |Port|Protocol|비고|
   |---|---|---|
   |2375|TCP||
   |3375|TCP||
   |4789|UDP||

2. Swarm nodes

   |Port|Protocol|비고|
   |---|---|---|
   |2375|TCP||
   |4789|UDP||

## Swarm manager

Swarm manager는 Swarm 클러스터에서 메인이 되는 호스트이다. Consul server에서 진행한다.

직접 실행하여 테스트 완료 후 스크립트를 작성한다.

1. 실행

   Swarm manage를 실행한다.

   [Consul server IP] 부분에 Swarm manager에서 자신의 호스트가 Consul과 통신 가능한 IP를 적는다.

   localhost가 아닌 네트워크 환경에서 접속 가능한 IP를 적는다.(**Consul server 설치 시 -bind 옵션에 적은 IP**)

   ``` shell
   nohup swarm manage -H :8999 --advertise :2375 consul://[Consul server IP]:8500/swarmnodes &
   ```

2. 확인

   Swarm manage에 접속해서 실행 여부를 확인한다.

   ``` shell
   docker -H :8999 info
   ```

   로그를 확인할 수도 있다.

   ``` shell
   tail -f nohup.out
   ```

## Swarm node

Swarm node는 Swarm manage에서 관리할 Docker Engine 들이다. Consul agent에서 진행한다.

직접 실행하여 테스트 완료 후 스크립트를 작성한다.

1. <a id="DockerConfig"></a>Docker 설정 변경

   Swarm 내에서 통신하기 위해 Docker를 외부에서도 접근할 수 있도록 수정한다.

   [Swarm node IP]를 Swarm node의 호스트로 다른 호스트가 접근 가능한 IP를 입력한다.

   ``` shell
   sudo sh -c "echo 'DOCKER_OPTS=\"\$DOCKER_OPTS -H unix:///var/run/docker.sock -H tcp://[Swarm Node IP] --cluster-advertise [Swarm Node IP]:2375 --cluster-store consul://0.0.0.0:8500/swarmnodes \"' >> /etc/default/docker"
   ```

   그리고 Docker를 재시작한다.

   ``` shell
   sudo service restart docker
   ```

2. 실행

   Swarm node를 실행한다.

   [Swarm Node IP] 부분을 [Docker 설정 변경](#DockerConfig)에서 진행한 IP와 동일하게 적는다.

   ``` shell
   nohup swarm join --addr=[Swarm Node IP]:2375 consul://0.0.0.0:8500/swarmnodes &
   ```

## 확인

1. Swarm 확인

   Swarm manager에서 Swarm nodes가 정상적으로 연결되는지 확인한다.

   ``` shell
   docker -H :8999 info
   ```

   ![Swarm info]({{ site.url }}/assets/images/docker-swarm-consul/swarm-info.png)

2. Docker 테스트

   Swarm manager에서 Swarm nodes에 Docker 컨테이너를 실행하여 본다.

   [Swarm node 호스트이름] 부분을 테스트 할 Swarm node의 호스트 이름을 적는다.

   ``` shell
   docker -H :8999 run -i --name=node1 -e="constraint:node==[Swarm node 호스트이름]" hello-world
   ```

   Swarm manager에서 실행하였지만, Swarm nodes에서 컨테이너가 실행된 것을 확인할 수 있다.

   ![Container 시작]({{ site.url }}/assets/images/docker-swarm-consul/swarm-start-container.png)

   Swarm node에서 아래 명령어를 실행하여 컨테이너가 정상적으로 올라왔는지도 확인할 수 있다.

   ``` shell
   docker ps -a
   ```

   ![Container 시작 2]({{ site.url }}/assets/images/docker-swarm-consul/swarm-start-container-test.png)

## 환경변수 설정 및 서비스 스크립트 작성

### Swarm manager

1. DOCKER_HOST 환경변수를 추가한다.

   ``` shell
   echo "export DOCKER_HOST=:8999" >> ~/.bash_profile
   source ~/.bash_profile
   ```

2. Swarm 서비스 스크립트 작성

   서비스 스크립트로 Swarm을 자동으로 실행하기 위해 작성한다.

   ``` shell
   sudo vi /etc/init/swarm.conf
   ```

   swarm.conf 파일에 아래 내용을 추가한다.

   [Consul server IP] 부분에 Swarm manager에서 자신의 호스트가 Consul과 통신 가능한 IP를 적는다.

   ``` shell
   description "Docker Swarm manager process"

   start on local-filesystems and net-device-up IFACE=eth0
   stop on runlevel [!12345]

   respawn

   setuid ubuntu
   setgid ubuntu

   exec swarm manage -H :8999 --advertise :2375 consul://[Consul server IP]:8500/swarmnodes
   ```

3. Swarm 서비스 스크립트 실행 및 종료

   만약 위에서 nohup 으로 실행시킨 swarm이 있다면, kill 명령어로 종료 후 실행 한다.

   **실행**

   ``` shell
   sudo start swarm
   ```

   **종료**

   ``` shell
   sudo stop swarm
   ```

4. 로그 확인

   로그는 아래 명령어로 확인 할 수 있다.

   ``` shell
   sudo tail -f /var/log/upstart/swarm.log
   ```

### Swarm node

1. Swarm 서비스 스크립트 작성

   서비스 스크립트로 Swarm을 자동으로 실행하기 위해 작성한다.

   ``` shell
   sudo vi /etc/init/swarm.conf
   ```

   swarm.conf 파일에 내용은 아래와 같다.

   [Swarm Node IP] 부분을 [Docker 설정 변경](#DockerConfig)에서 진행한 IP와 동일하게 적는다.

   ``` shell
   description "Docker Swarm node process"

   start on local-filesystems and net-device-up IFACE=eth0
   stop on runlevel [!12345]

   respawn

   setuid ubuntu
   setgid ubuntu

   exec swarm join --addr=[Swarm Node IP]:2375 consul://0.0.0.0:8500/swarmnodes
   ```

2. Swarm 서비스 스크립트 실행 및 종료

   만약 위에서 nohup 으로 실행시킨 swarm이 있다면, kill 명령어로 종료 후 실행 한다.

   **실행**

   ``` shell
   sudo start swarm
   ```

   **종료**

   ``` shell
   sudo stop swarm
   ```

3. 로그 확인

   로그는 아래 명령어로 확인 할 수 있다.

   ``` shell
   sudo tail -f /var/log/upstart/swarm.log
   ```

# 마치며
DevOps 인프라를 구축하기 위해 사용하는 방법 중에서 Docker, Swarm, Consul을 활용하여 간단하게 구축하였다. 안전성 및 실제로 위 환경들을 구축하기 위해서는 Swarm manager와 Consul server를 이중화 이상으로 구축하여야 한다.

Docker 사이트에서는 위 방법을 Docker image로 구축하는 예제를 만들었지만, 실제 환경에서는 Docker machine을 사용하지 않고, 실제 EC2나 물리 서버 자체에서 구축하기 때문에, 해당 방법을 정리하였다.

다음에는 Consul registrator, docker compose, [docker registry]({{ site.baseurl }}{% link _posts/2017-01-12-docker-nexus-docker-registry.md %}), docker overlay network를 활용하여 한 곳에 마스터에서 여러 원격 서버에 컨테이너를 올리고, 서로 다른 원격 서버에 있는 컨테이너끼리 통신하는 방법을 포스팅할 예정이다.
(~~글을 나눠서 포스팅 해야할지도...~~)
