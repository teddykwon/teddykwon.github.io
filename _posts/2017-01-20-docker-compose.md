---
layout: post
title:  "Docker Compose 메뉴얼 설치"
tags: ["인프라", "ubuntu", "docker", "docker-compose"]
---

Docker Compose는 Docker 컨테이너를 손쉽게 실행할 수 있게 도와준다.

Docker 컨테이너를 실행할 때 필요한 설정을 template으로 만들어서 자동으로 처리해주고, scale up도 지원한다.

# 설치

설치는 **Ubuntu 14.04** 버전으로 진행한다.

1. 파이썬 패키지 관리 도구 설치

   파이썬 패키지 관리 도구를 설치한다.

   ``` shell
   sudo apt-get install -y python-pip
   ```

2. Docker Compose 설치

   pip로 Docker compose를 설치한다.

   ``` shell
   sudo pip install docker-compose
   ```

3. 설치 확인

   ``` shell
   docker-compose version
   ```

   아래와 같이 나오면 정상적으로 설치 완료되었다.

   ![Docker compose 버전]({{ site.url }}/assets/images/docker-compose/docker-compose-version.png)

# 마치며
사용 예제는 향후에 포스팅할 수 있도록 한다.
