---
layout: post
title:  "하둡2 설치 'OS X El Capitan'"
date:   2017-07-23 23:45:22 +0900
categories: psyoblade update
---
## 환경설정
```bash
    sudo scutil --set ComputerName psyoblade-mac
    sudo scutil --set LocalHostName psyoblade-mac
    sudo scutil --set HostName psyoblade-mac
    127.0.0.1 psyoblade-mac # /etc/hosts
```

### 하둡 초기화
```bash
    ./bin/hdfs namenode -format
```

### 서비스 시작
```bash
    ./sbin/start-all.sh
    ./sbin/mr-jobhistory-daemon.sh start historyserver
    ./sbin/yarn-daemon.sh start proxyserver
```
### 프로세스 확인
```bash
    $ jps -ml 
    2261 org.apache.hadoop.hdfs.server.namenode.NameNode
    2463 org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode
    2833 org.apache.hadoop.mapreduce.v2.hs.JobHistoryServer
    2754 org.apache.hadoop.yarn.server.webproxy.WebAppProxyServer
    2684 org.apache.hadoop.yarn.server.nodemanager.NodeManager
    2586 org.apache.hadoop.yarn.server.resourcemanager.ResourceManager
    2351 org.apache.hadoop.hdfs.server.datanode.DataNode
    2861 sun.tools.jps.Jps -ml
```

### 서비스 종료 
```bash
    ./sbin/yarn-daemon.sh stop proxyserver
    ./sbin/mr-jobhistory-daemon.sh stop historyserver
    ./sbin/stop-yarn.sh
    ./sbin/stop-dfs.sh
```

### 하둡2 웹 인터페이스
* [psyoblade-mac:50070](http://psyoblade-mac:50070) : 하둡 웹 인터페이스
* [psyoblade-mac:8088](http://psyoblade-mac:8088) : 얀 웹 인터페이스
* [psyoblade-mac:19888](http://psyoblade-mac:19888) : 하둡 잡 히스토리

### 워드 카운팅
```bash
    bin/hdfs dfs -mkdir -p /user/psyoblade/conf
    bin/hdfs dfs -put etc/hadoop/hadoop-env.sh conf/
    bin/yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount conf output 
```
