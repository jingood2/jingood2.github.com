---
layout: post
title: "Docker 기반으로 Web Application 환경 구성하기"
description: ""
category: docker
tags: [docker,strongloop]

---
{% include JB/setup %}

## 목적
사내 프로젝트로 개발한 웹 어플리케이션을 서비스하기 위해 Ubuntu가 설치된 서버에 환경을 구성중입니다.
이 포스트는 docker를 기반으로 웹 서비스 환경을 구성하기 위한 절차를 정리하기 위해 작성되었습니다.
목표는 node.js + strongloop 로 개발된 REST API와 연동하는 간단한 웹 서비스를 개발하는 것입니다. 이 모든 것을 docker로 환경을 구성하는 것이 목표입니다.사용된 기술은 다를 수 있지만 docker를 이용하여 웹 서비스 환경을 구성하실 때 참조하시면 좋을 것 같습니다.


## 사용된 기술
웹 서비스 개발에 사용된 기술은 아래와 같습니다.

- Ubuntu 14.04 - Docker Host OS
- Docker - lightweight container virtualization
- Docker compose - orchestration for my Docker containers
- Docker machine
- nginx - reverse proxy
- mongodb - database for persistence
- strongloop - REST API Backend App build, deploy, monitor framework
- strong-pm - app scaleout, deploy component
- AngularJS 1.4

## Docker Host 설치
docker로 환경을 구성할때 가장 큰 장점은 서버는 Docker 컨테이너를 위한 Docker host 용도로 사용되기 때문에 최소 설치 및 설정 정도만 요구됩니다.  [docker](https://docs.docker.com/installation/ubuntulinux/)를 참조하여 docker를 지원하는 OS를 설치하세요. 이 포스트는 Docker host 용도로 사용된 OS는 Ubuntu 14.04입니다. 서버에 Ubuntu server를 설치합니다.

### Secure SSH 설정
ssh의 기본 포트를 변경합니다. `변경한 포트를 잃어버리지 않도록 기록해두세요!`

~~~bash
    # What ports, IPs and protocols we listen for
    Port 3322
~~~

root로 login 을 차단합니다.

~~~bash
	AuthenticationLoginGraceTime 120
	PermitRootLogin no
	StrictMode yes
~~~

서버에 등록한 유저만 접속할 수 있도록 파일의 마지막 라인에 아래 내용을 추가하고 저장합니다
~~~vim
    AllowUsers player
~~~
변경한 내용을 적용하기 위해 sshd 를 재기동하고, 변경한 포트가 적용되었는지 ssh로 login 해봅니다. 

~~~bash
	# sudo service ssh restart
    # ssh player@192.168.100.14
    ssh: connect to host 192.168.100.14 port 22: Connection refused
    player@RNNPDEVTEST:~$ ssh player@192.168.100.14 -p 3322
	The authenticity of host '[192.168.100.14]:3322 ([192.168.100.14]:3322)' can't be established.
	ECDSA key fingerprint is 92:7a:0b:1e:5b:e7:be:cb:f8:43:12:f3:6e:1a:00:aa.
	Are you sure you want to continue connecting (yes/no)?
~~~

### Ubuntu firewall 설정
SSH 접속 포트를 변경했으니 호스트 머신의 firewall 설정을 합니다. SSH port 3322만 호스트 머신에 통과될 수 있도록 합니다. iptables 대신 좀 더 사용하기 쉬운 `ufw(uncomplicated firewall)`를 사용하여 방화벽 설정을 합니다.

ufw가 설치됐는지 확인합니다. 설치가 안되있다면 `#sudo apt-get install ufw`로 설치합니다.
기본은 아래와 같이 disable 상태입니다.

~~~bash
$ sudo ufw status
Status: inactive
~~~

변경한 ssh port를 열어줍니다. 그리고 기본적으로 모든 incoming 요청은 막고, outgoing 요청은 허용하도록 변경합니다.

```bash
$ sudo ufw allow 3322/tcp
$ sudo ufw deny incoming
$ sudo ufw allow outgoing
```

기본 방화벽 설정이 완료되었으면 ufw를 활성화 시킵니다.

```bash
$ sudo ufw enable
$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
3322/tcp                   ALLOW       Anywhere
3322/tcp (v6)              ALLOW       Anywhere (v6)
```

### Docker 및 Docker compose 설치
Docker Host 설치가 완료되었으면 이제 Docker 및 Docker compose를 설치합니다.
[docker](https://docs.docker.com/installation/ubuntulinux/) 링크를 참조하여 해당하는 OS에 해당하는 설치방법에 따라 docker 및 compose를 설치합니다.

docker를 설치합니다.
```bash
$ wget -qO- https://get.docker.com/ | sh
```
docker가 설치되었으면 이제 여러 컨테이너들을 한번에 구성해서 설치하는데 편한 [docker-compose](https://docs.docker.com/compose/install/)를 설치합니다.

```bash
$ curl -L https://github.com/docker/compose/releases/download/1.4.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

$ chmod +x /usr/local/bin/docker-compose

$ docker-compose --version
docker-compose version: 1.4.2
```

$ docker를 사용자 그룹계정에 추가합니다.

```bash
$ sudo usermod -aG docker ubuntu

```

로그 아웃한 후에 docker 설치가 제대로 됐는지 확인합니다. 

```bash
$ docker run hello-world
player@RNNPDEVTEST:~$ docker run hello-world

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
```



## Docker machine을 이용하여 Cloud에 구성하기





