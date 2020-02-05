

Docker와 Consul 설치를 위한 필수 package 설치

``` shell
sudo apt-get update
sudo apt-get install -y \
   unzip \
   apt-transport-https \
   ca-certificates \
   curl \
   software-properties-common
```

Consul을 다운로드 하고, /usr/local/bin 과 /usr/bin 으로 이동한다.

``` shell
wget https://releases.hashicorp.com/consul/0.8.4/consul_0.8.4_linux_amd64.zip -O /tmp/consul.zip
unzip /tmp/consul.zip
sudo cp ./consul /usr/local/bin
sudo mv ./consul /usr/bin
```

docker 설치

``` shell
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-xenial main'
sudo apt-get update
sudo apt-get install docker-engine=17.05.0~ce-0~ubuntu-xenial
sudo usermod -aG docker ubuntu
```

gradle

``` shell
cd /tmp
wget https://services.gradle.org/distributions/gradle-3.5-bin.zip
unzip gradle-3.5-bin.zip
sudo chown -R ubuntu:ubuntu gradle-3.5
sudo mv gradle-3.5 /usr/local/gradle-3.5
sudo ln -s /usr/local/gradle-3.5 /usr/local/gradle
sudo ln -s /usr/local/gradle/bin/gradle /usr/local/bin
sudo ln -s /usr/local/gradle/bin/gradle /usr/bin
echo 'GRADLE_HOME=/usr/local/gradle' >> ~/.bashrc
source ~/.bashrc
```


java

``` shell
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install -y oracle-java8-installer

echo 'JAVA_HOME=/usr/lib/jvm/java-8-oracle' >> ~/.bashrc
source ~/.bashrc
```

self-signed certificates

``` shell
mkdir -p certs
openssl req \
  -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
  -x509 -days 365 -out certs/domain.crt

sudo mkdir -p /etc/docker/certs.d/myregistrydomain.com:5000
sudo cp certs/domain.crt /etc/docker/certs.d/myregistrydomain.com:5000/ca.crt

sudo cp certs/domain.crt /usr/local/share/ca-certificates/myregistrydomain.com.crt
sudo update-ca-certificates
```

docker-compose

``` shell
curl -L https://github.com/docker/compose/releases/download/1.13.0/docker-compose-`uname -s`-`uname -m` > /tmp/docker-compose
chmod +x /tmp/docker-compose
sudo cp docker-compose /usr/local/bin
sudo mv docker-compose /usr/bin
```

# consul ui
``` shell
cd /data/consul-ui
wget https://releases.hashicorp.com/consul/0.8.4/consul_0.8.4_web_ui.zip
unzip consul_0.8.4_web_ui.zip
rm consul_0.8.4_web_ui.zip
```

aws elb create-load-balancer-policy --load-balancer-name bel-nginx-lb --policy-name EnableProxyProtocol --policy-type-name ProxyProtocolPolicyType --policy-attributes AttributeName=ProxyProtocol,AttributeValue=True

aws elb set-load-balancer-policies-for-backend-server --load-balancer-name bel-nginx-lb --instance-port 80 --policy-names EnableProxyProtocol
