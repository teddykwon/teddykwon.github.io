---
layout: post
title:  "여러 호스트에 Consul 설치 후 연동하기"
tags: [인프라, docker, ubuntu, consul]
---

여러 호스트에 Consul를 설치하여 인프라를 구축하는 방법을 설명하려고 한다. 향후 포스팅에서는 Consul 환경에서 Swarm을 사용하는 방법을 포스팅 할 예정이다.

기본적으로 아래 환경을 구성하기 위해 **2대 이상** 의 호스트가 필요하다. 필요한 호스트 서버는 AWS 등 클라우드 서비스나 VM 또는 Docker machine 등을 활용하여 구성 후 진행한다.

# 설명

[Consul]('https://www.consul.io')은 HashiCorp 에서 만든 인프라에서 서비스를 발견하고 설정하는 도구이다. 제공하는 서비스로는 아래와 같다.

* 서비스 발견
* 상태 확인
* Key/Value 저장소
* Multi Datacenter

# <a id="ConsulInstall"></a>설치

설치는 **Ubuntu 14.04** 으로 진행한다.

wget 을 이용해서 다운로드 받은 후 실행경로로 이동한다.

{% highlight bash %}
wget https://releases.hashicorp.com/consul/0.7.2/consul_0.7.2_linux_amd64.zip -O /tmp/consul.zip
sudo apt-get install -y unzip
unzip /tmp/consul.zip
sudo cp ./consul /usr/local/bin
sudo mv ./consul /usr/bin
{% endhighlight %}

## 설치 확인

{% highlight bash %}
consul version
{% endhighlight %}

아래의 이미지와 같이 나오면 된다.

![Consul Version 확인]({{ site.url }}/assets/images/docker-swarm-consul/consul-version.png){:.center}

# 설정 및 연동

최소한 2대 이상의 호스트가 필요함으로, 다른 호스트에 설치하지 않았다면 [Consul 설치](#ConsulInstall) 후 아래 설정을 진행한다.

Consul 에서 리더 서버, 즉 관리 서버를 Consul server 라고 지칭하고, 노드 서버들을 Consul agent 라고 지칭한다.

Consul 에서는 기본적으로 Consul server를 데이터 손실 등을 방지하기 위한 목적으로 **3~5대** 의 호스트를 클러스터로 구성하기를 추천한다. Web UI 서버, agent 까지 합치면 최소한 5대 이상으로 구성하여야 한다.

여기서는 Consul Server 와 Web UI를 1대에 설치하고, agent 테스트를 위해 1대를 구성하여 총 **2대** 로 진행한다.

## 방화벽 설정

Consul 에서 사용하는 포트끼리 통신할 수 있도록 방화벽 설정을 진행한다.

Consul 에서 사용하는 포트는 아래와 같으며, 8500 포트는 Consul server에만 오픈한다.

|Port|Protocol|비고|
|8300|TCP||
|8301|TCP & UDP||
|8500|TCP|UI 접속용|
|8600|TCP & UDP||

## Consul server 설정

1. **폴더 생성**

    Consul 데이터 폴더, Web UI 소스가 위치할 폴더, 설정파일이 위치할 폴더를 생성한다.

    ```shell
    sudo mkdir -p /data/consul
    sudo mkdir -p /data/consul-ui
    sudo mkdir -p /etc/consul.d/server
    sudo chown -R ubuntu:ubuntu /data/consul /data/consul-ui /etc/consul.d
    ```

2. **Web UI 설치**

    Web UI를 다운로드 받고, 압축을 푼다.

    ```shell
    cd /data/consul-ui
    wget https://releases.hashicorp.com/consul/0.7.2/consul_0.7.2_web_ui.zip
    unzip consul_0.7.2_web_ui.zip
    rm consul_0.7.2_web_ui.zip
    ```

3. **키생성**

    암호화를 위해 키를 생성한다. 해당 키는 Consul 전체에서 사용된다.

    ```shell
    consul keygen
    ```

4. **Config**

    마지막으로 Consul server에 config 파일을 생성한다.

    ```shell
    vi /etc/consul.d/server/config.json
    ```

    위에서 [key 생성 내용]을 consul keygen으로 생성한 키로 변경하고 [Consul 서버 IP]를 Consul server를 설치한 서버 IP로 변경한다.

    **Consul server IP는 서버끼리 통신이 가능하여야 한다.**

    ```json
    {
        "bootstrap": true,
        "server": true,
        "datacenter": "test",
        "ui_dir": "/data/consul-ui",
        "data_dir": "/data/consul",
        "encrypt": "[key 생성 내용]",
        "addresses": {
            "http": "[Consul 서버 IP]"
        }
    }
    ```

    위에서 맞게 변경하면 아래와 같이 변경된다.

    ```json
    {
        "bootstrap": true,
        "server": true,
        "datacenter": "test",
        "ui_dir": "/data/consul-ui",
        "data_dir": "/data/consul",
        "encrypt": "7mASsjRU1X0ORmQyymo9Hw==",
        "addresses": {
            "http": "10.2.1.4"
        }
    }
    ```

5. **Consul 임시 실행 및 확인**

    consul 실행 후 UI 접속을 확인한다.

    [Consul 서버 IP]를 Consul server 를 설치한 서버 IP로 변경한다.

    ```bash
    nohup consul agent --server -config-dir=/etc/consul.d/server -bind=[Consul 서버 IP] &
    ```

    consul 실행 후 http://[Consul 서버 IP]:8500 로 접속하면 아래와 같이 나온다.

    ![Consul UI 화면]({{ site.url }}/assets/images/docker-swarm-consul/consul-ui.png)

    확인 후 ps 로 consul 을 찾아서 kill 명령어로 서비스를 종료한다.

## Consul server Service 스크립트 만들기

1. **만들기**

    ```bash
    sudo vi /etc/init/consul.conf
    ```

2. **스크립트 내용**

    [Consul 서버 IP]를 Consul server 를 설치한 서버 IP로 변경한다.

    ```bash
    description "Consul server process"

    start on local-filesystems and net-device-up IFACE=eth0
    stop on runlevel [!12345]

    respawn

    setuid ubuntu
    setgid ubuntu

    exec consul agent --server -config-dir=/etc/consul.d/server -bind=[Consul 서버 IP]
    ```

3. **스크립트 실행**

    ```bash
    sudo start consul
    ```

    실행 후 로그는 아래 명령어로 확인 할 수 있다.

    ```bash
    sudo tail -f /var/log/upstart/consul.log
    ```

4. **종료**

    ```bash
    sudo stop consul
    ```

## Consul agent 설정

Consul agent 설정임으로, Consul agent가 될 호스트에서 작업한다.

1. **폴더 생성**

    agent에서 데이터를 관리할 폴더를 생성한다.

    ```bash
    sudo mkdir -p /data/consul
    sudo mkdir -p /etc/consul.d/agent
    sudo chown -R ubuntu:ubuntu /data/consul /etc/consul.d
    ```

2. **Config**

    agent 에 대한 설정을 추가한다.

    ```bash
    vi /etc/consul.d/agent/config.json
    ```

    encrypt 부분은 처음 서버에서 생성한 키를 입력하고, start_join 부분에는 Consul server ip를 적어준다.

    ```json
    {
        "server": false,
        "datacenter": "test",
        "data_dir": "/data/consul",
        "encrypt": "[key 생성 내용]",
        "start_join": ["[Consul 서버 IP]"]
    }
    ```

    [key 생성 내용] 및 [Consul 서버 IP] 를 맞게 변경하면 아래와 같이 변경된다.

    ```json
    {
        "server": false,
        "datacenter": "test",
        "data_dir": "/data/consul",
        "encrypt": "7mASsjRU1X0ORmQyymo9Hw==",
        "start_join": ["10.2.1.4"]
    }
    ```

3. **Consul 임시 실행**

    Consul 임시로 실행한다.

    [Consul agent IP]를 Consul agent 를 설치한 서버 IP로 변경한다.

    ```bash
    nohup consul agent -config-dir /etc/consul.d/agent -bind=[Consul agent IP] &
    ```

    consul 실행 후 http://[Consul 서버 IP]:8500 로 접속해서 Nodes 로 이동하면 agent 노드가 추가되어 있다.

    ![Consul UI에서 agent 추가 확인]({{ site.url }}/assets/images/docker-swarm-consul/consul-ui-agent-add.png)

    Consul server 에서 아래 명령어로도 확인할 수 있다.

    ```bash
    consul members
    ```

    ![Consul members]({{ site.url }}/assets/images/docker-swarm-consul/consul-members.png)

    확인 후 ps 로 consul 을 찾아서 kill 명령어로 서비스를 종료한다.

## Consul agent service 스크립트 작성

1. **만들기**

    ```bash
    sudo vi /etc/init/consul.conf
    ```

2. **스크립트 내용**

    [Consul agent IP]를 Consul agent 를 설치한 서버 IP로 변경한다.

    ```bash
    description "Consul agent process"

    start on local-filesystems and net-device-up IFACE=eth0
    stop on runlevel [!12345]

    respawn

    setuid ubuntu
    setgid ubuntu

    exec consul agent -config-dir=/etc/consul.d/agent -bind=[Consul agent IP]
    ```

3. **스크립트 실행**

    ```bash
    sudo start consul
    ```

    실행 후 로그는 아래 명령어로 확인 할 수 있다.

    ```bash
    sudo tail -f /var/log/upstart/consul.log
    ```

4. **종료**

    ```bash
    sudo stop consul
    ```
