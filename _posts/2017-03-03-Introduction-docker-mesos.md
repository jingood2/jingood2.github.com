#Docker

## Hypervisor vs Docker

### 목적 
독립된 실행환경, 어플리케이션의 완전 격리상에서 서비스 구동


![vm vs docker](http://www.unixarena.com/wp-content/uploads/2016/06/hypervisor-Vmware-vs-Docker.jpg)

### 가상화
### 1. 장점
자원이 허용하는 한, 여러 독립된 운영체제가 작동하는 격리된 환경 제공
### 2. 단점
- 가상머신이 실제 물리적인 하드웨어를 제어해야 하기 때문에 게스트 운영체제가 필요함
- 따라서 Guest OS를 위한 CPU,Memory가 필요함.(**자원의 낭비**)
- Guest OS가 필요하기 때문에 **배포시 느림**.(클라우드 환경에서는 문제임)

### Docker
어플리케이션이 독립적으로 구동 가능한 격리된 환경(컨테이너)에서 서비스를 실행
![컨테이너 개념](https://docs.google.com/drawings/d/19Ud5YeRJ1nA3Rd8ussD-T7-e_vRo4uBRxivEm9nxnek/pub?w=578&h=335)

### 1. 장점
- 리눅스 컨테이너 기술을 이용하여 모든 프로세스가 격리된 환경에서 호스트 운영체제에서 바로 실행 (**가상머신없이 호스트 OS가 사용하는 리소스를 분리하여 격리된 여러 환경을 만드는 기술**)
- 게스트 운영체제가 필요없기 때문에 컨테이너는 밀리세컨드 단위로 실행됨
- 컨테이너는 실행하기 전에는 자원을 소비하지 않음
- 실제 성능차가 거의 없음

### 2. 단점
- 스토리지 구성의 어려움
- 네트웍 구성의 어려움 ( 호스트 운영체제 레벨에서 네트웍이 한번 더 추상화 필요)
## images, container의 이해

### image
![container on base image](https://docs.docker.com/engine/userguide/storagedriver/images/sharing-layers.jpg)

![update base image](https://docs.docker.com/engine/userguide/storagedriver/images/saving-space.jpg)

### container
![container](https://docs.docker.com/engine/userguide/storagedriver/images/container-layers-cas.jpg)

![data volumes](https://docs.docker.com/engine/userguide/storagedriver/images/shared-volume.jpg)

# Apache Mesos
Cloud Infrastructure 및 Computing Engine들의 자원을 통합적으로 관리 할수 있도록 만든 자원관리 프로젝트

## Overview
![mesosphere](http://media.bestofmicro.com/U/N/440447/original/Mesosphere-Image.jpg)

![Node abstraction in mesos](https://opensource.com/sites/default/files/images/business-uploads/mesos1.png)

![](https://image.slidesharecdn.com/mesospherewebcast-140805130801-phpapp02/95/apache-mesos-and-mesosphere-live-webcast-by-ceo-and-cofounder-florian-leibert-35-638.jpg?cb=1442180394)

![](https://image.slidesharecdn.com/mesospherewebcast-140805130801-phpapp02/95/apache-mesos-and-mesosphere-live-webcast-by-ceo-and-cofounder-florian-leibert-36-638.jpg?cb=1442180394)

![Resource sharing across the cluster increases throughput and utilization](https://image.slidesharecdn.com/mesospherewebcast-140805130801-phpapp02/95/apache-mesos-and-mesosphere-live-webcast-by-ceo-and-cofounder-florian-leibert-37-638.jpg?cb=1442180394)


## Mesos Architecture
![Mesos Architecture](https://image.slidesharecdn.com/pxnmesos-140410175005-phpapp02/95/datacenter-computing-with-apache-mesos-bigdata-dc-28-638.jpg?cb=1397574488)

![](http://datastrophic.io/content/images/2015/09/Mesos-Overview.png)



# DCOS - Data Center Operating System
![](http://image.slidesharecdn.com/sergiuszurbaniakandstefanschimanskikubernetesontopofmesosontopofdcos-160113054143/95/kubernetes-on-top-of-mesos-on-top-of-dcos-8-638.jpg?cb=1452663790)

## Frameworks

### Marathon

![marathon](https://mesosphere.com/wp-content/uploads/2016/04/dcos1_7_services_marathon_open.png)

### Chronos

![chronos](https://clusterhq.com/assets/images/blog/chronosadd.png)

### Ochestration Tool

#### Kubernate

#### Docker Swarm


