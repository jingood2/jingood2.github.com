-doop Pseudo Distributed Mode 설치 

하둡 의사 분산 모드 설치는 단일 서버에 모든 데몬이 구동된다는 점을 제외하고는 멀티 분산 하둡 클러스터와 동일하게 동작한다. 의사 분산 모드 설치에 필요한 과정을 간단히 정리하고자 한다. 

## Overview
------
의사 분산 모드( Pseudo Distributed Mode) 설치 과정은 다음과 같다. 

- JAVA 및 Hadoop 설치 및 환경 설정
- conf/hadoop-env.sh 수정
- SSH로 패스워드 없이 로그인 가능하도록 공개키와 개인키 생성 및 등록
- Hadoop 환경 설정 파일 수정
- - mapred-site.xml
- - hdfs-site.xml
- - core-site.xml
- HDFS 포맷
- 데몬 실행 
- 설치 확인

## Hadoop 설치 
--------------------------
하둡을 구동할 Hadoop 계정을 생성한다. 

    sudo addgroup hadoop
    sudo adduser --ingroup hadoop hadoop 

하둡 홈페이지에서 안정화 버젼 Hadoop-1.0.4.tar.gz 을 download 하고, 압축을 푼다. 

    cd ~/
    pwd .
    /home/hadoop
    wget http://archive.apache.org/dist/hadoop/core/hadoop-1.0.4/hadoop-1.0.4.tar.gz
    tar xvfz hadoop-1.0.4.tar.gz
    
HADOOP_HOME 및 JAVA_HOME 환경변수를 등록한다. 

    export HADOOP_HOME=/home/hadoop/hadoop-1.0.4
    
    
Hadoop을 실행할 때 환경과 관련된 변수들을 설정하는 conf/hadoop-env.sh 을 수정한다. 

    # The java implementation to use. Required. 
    export jAVA_HOME=/usr/lib/jvm/java-1.6.0-openjdk-amd64

원격 서버에 로그인 없이 SSH를 사용하기 위해 Hadoop 유저를 위한 SSH 설정을 하고 제대로 설정되었는지 확인한다. 

    ssh-keygen -t dsa -P "" -f ~/.ssh/id_dsa
    cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_key
    ssh localhost 
    
### Configuration 설정 ###
의사 분산 모드를 위한 3개의 Configuration 파일을 편집한다. 
    
#### conf/core-site.xml
이 파일은 네임노드 위치 지정에 사용된다. 네임노드가 실행될 때 이 파일을 참조하여 네임노드를 같은 서버의 9000번 포트에 실행시키다. 데이터 노드들은 이를 참조하여 네임노드의 위치를 알아낸다. 그리고 하둡 namenode 및 datanode 설치 위치시 사용되는 hadoop.tmp.dir 를 설정한다. ( 설정하지 않으면 namenode, datanode 가 /tmp/hadoop-{username}/dfs/name,data 위치에 설치된다. ) 

    <configuration>
      <property>
        <name>hadoop.tmp.dir</name>
        <value>/mnt/data/tmp/hadoop-hadoop</value>
      </property>
      <property>
        <name>fs.default.name</name>
        <value>hdfs://localhost:9000</value>
      </property>
    </configuration>

#### conf/mapred-site.xml
이 파일은 MapReduce Famework와 관련된 내용을 편집하는데 사용된다. 
하둡 홈디렉토리의 conf/mapred-site.xml을 다음과 같이 수정한다. 

    <configuration>
      <property>
        <name>mapred.job.tracker</name>
        <value>localhost:9001</value>
      </property>
      <property>
            <name>mapred.system.dir</name>
            <value>${hadoop.tmp.dir}/mapred/system</value>
        </property>
        <property>
            <name>mapred.local.dir</name>
            <value>${hadoop.tmp.dir}/mapred/local</value>
        </property>
    </configuration>

#### conf/hdfs-site.xml
이 파일은 HDFS 설정과 관련된 내용을 편집하는데 사용된다. 데이터노드가 어차피 한 개이기 때문에 Replication 수는 당연히 1개이다. dfs.name.dir 는 namespace가 저장되는 NameNode의 path이고, dfs.data.dir은 block이 저장되는 DataNode의 local filesystem의 path를 설정하지 않으면 %{hadoop.tmp.dir}/dfs/name,data 위치에 설치된다. 

    <configuration>
      <property>
        <name>dfs.replication</name>
        <value>1</value>
      </property>
    </configuration>


    
### HDFS Format ###
HDFS을 포맷해야 하는데 이는 데이터 노드에 필요한 작업이다. hdfs-site.xml의 dfs.data.dir 파라미터에 디렉토리를 설정할 수 있는데 이 파라미터를 설정하지 않으면 기본적으로 /tmp/hadoop-{Username}/dfs/data 디렉토리를 사용한다. 

**note :** 서버가 재부팅되면 /tmp 디렉토리가 정리될 수 있기 때문에 오래 사용하고 싶다면 hdfs-site.xml의 dfs.name.dir에 dfs.data.dir path를 설정하거나 core-site.xml 에 hadoop.tmp.dir를 설정한다. ( 여기서는 hadoop.tmp.dir를 /mnt/data/tmp/hadoop-hadoop 으로 설정했다.) 주의할 것은 ${hadoop.tmp.dir} 위치에 /dfs/name, /dfs/node 를 미리 만들어놓지 말아야 한다. 이것때문에 namenode가 포멧이 않되서 namenode가 구동되지 않아 엄청 삽질했다.)
    
이제 준비가 되었다면 HDFS를 포맷한다. 

    hadoop namenode -format 
    
결과를 확인한다. 

    hadoop@jingood2-N40L:~/hadoop-1.0.4/conf$ hadoop namenode -format
    13/06/03 23:30:36 INFO namenode.NameNode: STARTUP_MSG:
    /************************************************************
    STARTUP_MSG: Starting NameNode
    STARTUP_MSG:   host = jingood2-N40L/127.0.1.1
    STARTUP_MSG:   args = [-format]
    STARTUP_MSG:   version = 1.0.4
    STARTUP_MSG:   build = https://svn.apache.org/repos/asf/hadoop/common/branches/ branch-1.0 -r 1393290; compiled by 'hortonfo' on Wed Oct  3 05:13:58 UTC 2012
    ************************************************************/
    Re-format filesystem in /mnt/data/hadoop/dfs/name ? (Y or N) y
    Format aborted in /mnt/data/hadoop/dfs/name
    13/06/03 23:30:41 INFO namenode.NameNode: SHUTDOWN_MSG:
    /************************************************************
    SHUTDOWN_MSG: Shutting down NameNode at jingood2-N40L/127.0.1.
    
## 구동 하기 ##
------
구동 스크립트를 실행하여 데몬을 구동한다. 

    bin/start-all.sh
    
데몬이 정상적으로 구동되었는지 확인한다. 
    
    hadoop@jingood2-N40L:~$ jps
    20835 JobTracker
    21130 Jps
    20755 SecondaryNameNode
    20218 NameNode
    21067 TaskTracker
    
## 설치 확인 ##
-----

### 웹 인터페이스로 확인 ###

웹 인터페이스로 namenode와 Jobtracker에 접속해본다. 

    잡 트래커 웹 인터페이스: http://잡트래커이름:50030/
    네임노드 웹 인터페이스: http://네임노드이름:50070/

![잡 트래커 웹 인터페이스](http://{{ site.url }}/assets/jpg/jobtracker_interface.png)

![네임노드 웹 인터페이스](http://{{ site.url }}/assets/jpg/namenode_interface.png)

### HDFS 확인 ###
HDFS의 루트를 확인한다. 

    hadoop fs -ls /
    
    hadoop@jingood2-N40L:~/hadoop-1.0.4/conf$ hadoop fs -ls /
    Found 1 items
    drwxr-xr-x   - hadoop supergroup          0 2013-06-03 22:21 /tmp

## Wordcount 예제 실행 ##
--------
Hadoop 설치시에 포함되어 있는 wordcount를 실행해본다. 

하둡 홈 디렉토리로 이동한다. 

    cd $HADOOP_HOME
    
HDFS의 /input 디렉토리를 만들고 하둡의 README.txt 파일을 복사한 후, wordcount를 실행한다. 

    hadoop@jingood2-N40L:~/hadoop-1.0.4$ hadoop fs -copyFromLocal README.txt /input
    hadoop@jingood2-N40L:~/hadoop-1.0.4$
    hadoop@jingood2-N40L:~/hadoop-1.0.4$
    hadoop@jingood2-N40L:~/hadoop-1.0.4$ hadoop fs -ls /input
    Found 1 items
    -rw-r--r--   1 hadoop supergroup       1366 2013-06-04 00:09 /input/README.txt
    hadoop@jingood2-N40L:~/hadoop-1.0.4$
    hadoop@jingood2-N40L:~/hadoop-1.0.4$
    hadoop@jingood2-N40L:~/hadoop-1.0.4$ hadoop jar hadoop-examples-1.0.4.jar wordcount /input/README.txt /output
    13/06/04 00:11:05 INFO input.FileInputFormat: Total input paths to process : 1
    13/06/04 00:11:06 INFO util.NativeCodeLoader: Loaded the native-hadoop library
    13/06/04 00:11:06 WARN snappy.LoadSnappy: Snappy native library not loaded
    13/06/04 00:11:06 INFO mapred.JobClient: Running job: job_201306040005_0001
    13/06/04 00:11:07 INFO mapred.JobClient:  map 0% reduce 0%
    13/06/04 00:11:23 INFO mapred.JobClient:  map 100% reduce 0%
    13/06/04 00:11:35 INFO mapred.JobClient:  map 100% reduce 100%
    13/06/04 00:11:40 INFO mapred.JobClient: Job complete: job_201306040005_0001
    13/06/04 00:11:40 INFO mapred.JobClient: Counters: 29
    13/06/04 00:11:40 INFO mapred.JobClient:   Job Counters
    13/06/04 00:11:40 INFO mapred.JobClient:     Launched reduce tasks=1
    13/06/04 00:11:40 INFO mapred.JobClient:     SLOTS_MILLIS_MAPS=14766
    13/06/04 00:11:40 INFO mapred.JobClient:     Total time spent by all reduces waiting after reserving slots (ms)=0
    13/06/04 00:11:40 INFO mapred.JobClient:     Total time spent by all maps waiting after reserving slots (ms)=0
    13/06/04 00:11:40 INFO mapred.JobClient:     Launched map tasks=1
    13/06/04 00:11:40 INFO mapred.JobClient:     Data-local map tasks=1
    13/06/04 00:11:40 INFO mapred.JobClient:     SLOTS_MILLIS_REDUCES=11031

    13/06/04 00:11:40 INFO mapred.JobClient:     Map output records=179--
layout: post
title: "Hadoop Pseudo Distributed Mode 설치"
description: ""
category: Hadoop Ecosystem
tags: [hadoop,pseudo,installation]
---
{% include JB/setup %}


