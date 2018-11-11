---
layout: post
title:  "Docker 기본 개념 설명"
tags: [docker]
---

이미 Docker에 대해서는 많이 설명하고 있기 때문에, 자세한 설명은 다른 링크나 책을 확인하도록 한다.

# Docker란 ?
Docker는 일종에 VMware나 VirtualBox 같은 가상머신으로 이해하면 쉽다.
 
다른 점은 하드웨어 가상화를 통해서 Host OS 위에 각각 Guest OS를 설치해서 사용하는게 아니라 OS를 가상화해서 유저 별로 home을 각각 만들어서 사용하는 개념이다.

## Docker Hub
Docker에서 사용하는 Docker Image를 저장하는 저장소이다. docker에서 운영하는 docker hub에는 official image라고 불리는 모든 이미지에 base가 되는 image들을 받아서 사용할 수 있다. (예: ubuntu 등)

Docker에서 제공하는 Docker registry를 사용하면 개인/회사 별로 private repository를 구성할 수 있다.

## Docker Image
Dockerfile 이름을 가지고 build하면 Image가 만들어진다. Docker Image에서 사용하는 한 줄, 한 줄을 레이어 형식으로 구성하여, 기본적으로 다음 build 시 변경된 부분부터만 build하고 변경한다.

Docker Image를 과거에 가상 디스크(ISO) 파일로 생각하고, Docker를 가상 드라이브(CD스페이스, 데몬) 개념으로 생각하면 이해하기 쉽다.

Docker Image 내부에 저장되는 방식은 git repository에서 commit tree로 생각하면 쉽다.

---

* 가장 많이 사용하는 명령어
```
docker build --tag teddy:latest .
docker run -d --name app teddy:latest
docker exec -it app bash
docker pull teddy
docker push teddy
```

> docker run 에서 -d 옵션은 background 실행 명령으로 없이 실행할 경우 세션 종료 시 자동 종료된다.
> docker exec 에서 -it 명령은 STDIN / pesudo-TTY로 가상 터미널로 접속하겠다는 의미다.

# Docker Compose
Docker Compose는 Docker Container를 손쉽게 실행할 수 있게 도와준다.

Docker Container를 실행할 때 필요한 설정을 template으로 만들어서 실행 설정을 저장해놓고 사용한다.

* 가장 많이 사용하는 명령어
```
docker-compose up -d
docker-compose down
```
> -d 옵션은 background 실행 명령으로 없이 실행할 경우 세션 종료 시 자동 종료된다.