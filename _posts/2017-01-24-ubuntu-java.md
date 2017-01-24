---
layout: post
title:  "Ubuntu에 Java8 설치하기"
date:   2017-01-24 18:43:00 +0900
tags: [java, ubuntu]
---

기존에 설치하던 방법으로는 Ubuntu에서 Java8을 설치할 수 없다.

이 포스팅은 Ubuntu 14.04 기준으로 Java8을 설치하는 스크립트에 대해 설명한다.

# 설치

1. 스크립트

   ``` shell
   sudo add-apt-repository ppa:webupd8team/java
   sudo apt-get update
   sudo apt-get install -y oracle-java8-installer
   ```

   위의 스크립트로 java8을 설치할 수 있다.

   설치 시 GUI화면이 나오는데, 라이센스 부분에서 Accept하면 설치가 진행된다.

2. 환경변수 설정

   기본적으로 /usr/bin 폴더에 java가 등록되어 있지만, 추가적으로 JAVA_HOME을 설정해 준다.

   ``` shell
   echo 'JAVA_HOME=/usr/lib/jvm/java-8-oracle' >> ~/.bash_profile
   source ~/.bash_profile
   ```

# 확인

``` shell
java -version
```

위 명령어로 확인할 수 있다.
