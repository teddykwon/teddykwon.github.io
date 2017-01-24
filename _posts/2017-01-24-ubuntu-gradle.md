---
layout: post
title:  "Ubuntu에 Gradle 3.3 설치하기"
date:   2017-01-24 19:10:00 +0900
tags: [gradle, ubuntu]
---

Gradle은 Java진형에서 예전부터 사용하던 Ant와 Maven같은 빌드 툴이다.

Ant와 같이 자유롭게 Build 및 Deploy에 자유도를 주면서, Maven의 Repository와 Dependency를 같이 취하고 있다.

이 포스팅에서는 Ubuntu에서 Gradle을 설치하는 방법을 설명한다.

# 스크립트

1. 공식 사이트에서 다운로드 및 압축 해제

   ``` shell
   sudo apt-get install -y unzip
   cd /tmp
   wget https://services.gradle.org/distributions/gradle-3.3-bin.zip
   unzip gradle-3.3-bin.zip
   ```

2. 실제 사용 폴더로 옮기고, Symbolic link 설정

   ``` shell
   sudo chown -R ubuntu:ubuntu gradle-3.3
   sudo mv gradle-3.3 /usr/local/gradle-3.3
   sudo ln -s /usr/local/gradle-3.3 /usr/local/gradle
   ```

3. 실행파일 PATH로 Symbolic link 설정

   ``` shell
   sudo ln -s /usr/local/gradle/bin/gradle /usr/local/bin
   sudo ln -s /usr/local/gradle/bin/gradle /usr/bin
   ```

4. 환경변수 설정

   ``` shell
   echo 'GRADLE_HOME=/usr/local/gradle' >> ~/.bash_profile
   source ~/.bash_profile
   ```

5. 실행 확인

   ``` shell
   gradle -v
   ```

   위 명령어를 실행하면, Gradle 3.3이 설치되어있는 것을 확인할 수 있다.

# 마치며

설치 완료 후 gradle 실행이 안될때, 보통 JAVA_HOME이 설정되어 있지 않은 경우임으로, [Ubuntu에 Java8 설치하기]({{ site.baseurl }}{% link _posts/2017-01-24-ubuntu-java.md %})를 진행하고, 확인하면 된다.
