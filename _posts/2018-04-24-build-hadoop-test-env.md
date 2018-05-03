---
layout: post
title:  "手工搭建hadoop3节点测试环境"
data:   2017-09-06 14:31:48
categories: hadoop
tags: hadoop 大数据 安装部署 flume sqoop2 hbase hive scala spark zookeeper
---


# Hadoop环境搭建
## Hadoop资源地址

```
# sqoop官网
# 各版本sqoop软件下载，源码位置，官方文档等
http://sqoop.apache.org/docs/1.99.3/index.html
# maven仓库，可以找到各版本的jar包
https://mvnrepository.com/artifact/org.apache.sqoop/sqoop-client/1.99.2
```
## 安装URL汇总
```
# hadoop状态查看
http://192.168.56.101:50070/dfshealth.html#tab-overview
http://192.168.56.101:8088/cluster/apps
```

## 环境介绍

```
虚拟化环境：virtualBOX
虚拟机版本：centos7
JDK:jdk1.7.0_79
hadoop:hadoop-2.6.0-cdh5.5.2
flume:apache-flume-1.6.0-cdh5.5.2-bin
sqoop2:sqoop-1.99.7-bin-hadoop200
hbase:hbase-1.0.0-cdh5.5.2.tar
hive:hive-1.1.0-cdh5.5.2
JDBC:mysql-connector-java-5.1.32-bin.jar
scala：scala-2.9.3
spark:spark-1.3.0-bin-hadoop2.4
zookeeper:zookeeper-3.4.5-cdh5.5.2
```

## 配置基础运行环境
每台机器都需要配置的东西
### 配置主机名

```
# 配置主机名
vi /etc/hostname
h101
```

### 配置网卡

```
# 这里ifcfg-enp0s3配置文件可能不同，根据
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-enp0s3 
# 配置网卡信息参考
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=enp0s3
UUID=285643fb-10b2-4ea5-8f65-79b2d6b8f200
DEVICE=enp0s3
ONBOOT=yes
IPADDR=192.168.56.101
NETMASK=255.255.255.0
GATEWAY=192.168.56.1
DNS1=114.114.114.114

```

### 配置Hosts

```java
# 编辑hosts文件
[root@h101 ~]# sudo vi /etc/hosts
# 在hosts文件后面追加
192.168.56.101   h101
192.168.56.102   h102
192.168.56.103   h103


# 提示：如果是102就不是这么配置了
192.168.56.101   h101
192.168.56.102   h102
192.168.56.103   h103
```

### 创建hadoop用户

```
[root@h101 ~]# useradd hadoop
[root@h101 ~]# passwd hadoop
# 这里的密码我输入qwer123$
# 密码要大于8位，不能太简单，最好有特殊字符
Changing password for user hadoop.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
Sorry, passwords do not match.
New password: 
BAD PASSWORD: The password fails the dictionary check - it is based on a dictionary word
Retype new password: 
passwd: all authentication tokens updated successfully.
# 验证
[root@h101 ~]# su hadoop
[hadoop@h101 root]$ 

```

### 安装JDK

```
# 将jdk安装文件复制到/usr/local/java/下并解压
cp jdk7u79linuxx64.tar.gz /usr/local/java/
cd /usr/local/java/
tar -zxvf jdk7u79linuxx64.tar.gz 
# 在配置文件中配置环境变量
vi /etc/profile

export JAVA_HOME=/usr/local/java/jdk1.7.0_79
export JAVA_BIN=/usr/local/java/jdk1.7.0_79/bin
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME JAVA_BIN PATH CLASSPATH

# 重启验证
[root@h101 ~]# reboot
[root@h101 ~]# java -version
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
```
### 安装ssh 证书

```
# hadoop账号创建ssh-keygen，一般直接回车就行
[hadoop@h101 root]$ ssh-keygen -t rsa
[hadoop@h102 root]$ ssh-keygen -t rsa
[hadoop@h103 root]$ ssh-keygen -t rsa

# 过程参考
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa): 
Created directory '/home/hadoop/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/hadoop/.ssh/id_rsa.
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
The key fingerprint is:
18:38:c1:4e:32:40:18:87:48:a1:e4:46:ef:ef:9a:00 hadoop@h101
The key's randomart image is:
+--[ RSA 2048]----+
|BXo..            |
|O.+ oo           |
|.o *o .          |
|. . .. o         |
|E  .  . S        |
|.   .            |
| .   .           |
|  . o            |
|   o..           |
+-----------------+


```
### 配置SSH免密登录

```
# 使用hadoop用户，使用上面输入的密码qwer123$
# 101主节点
[hadoop@h101 java]$  ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub h101
[hadoop@h101 java]$  ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub h102
[hadoop@h101 java]$  ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub h103

# 102从节点
[hadoop@h102 root]$  ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub h101
[hadoop@h102 root]$  ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub h102
[hadoop@h102 root]$  ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub h103

# 103从节点
[hadoop@h103 root]$ ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub h101
[hadoop@h103 root]$ ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub h102
[hadoop@h103 root]$ ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub h103
```
**验证**

```
# 使用Hostname或者IP可以免密登录
[hadoop@h101 java]$ ssh h102
Last failed login: Mon Apr 23 14:15:54 EDT 2018 from h101 on ssh:notty
There were 2 failed login attempts since the last successful login.
Last login: Mon Apr 23 14:07:58 2018
[hadoop@h102 ~]$ ssh 192.168.56.101
Last login: Mon Apr 23 14:15:16 2018
```

### 关闭防火墙、SELINUX

```
vi /etc/selinux/config
SELINUX=disabled
```

### 配置NTP时钟同步

```

```
## Hadoop配置
### 配置hadoop环境变量

```
# 解压
tar -zxvf hadoop-2.6.0-cdh5.5.2.tar.gz 
# 赋权
chown -R hadoop.hadoop hadoop-2.6.0-cdh5.5.2
# 修改环境变量配置文件
vi /etc/profile
# 配置参考
HADOOP_HOME=/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2
HADOOP_BIN=/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/bin
HADOOP_SBIN=/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/sbin
HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
PATH=$HADOOP_HOME/bin:$PATH
export HADOOP_HOME HADOOP_CONF_DIR PATH
# 配置生效
[root@h101 hadoop]# source /etc/profile
# 验证
[root@h101 hadoop]# echo ${HADOOP_HOME}
/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2
```
### 配置文件修改
#### hadoop-env.sh
```
# 文档位置
/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/etc/hadoop/hadoop-env.sh
[hadoop@h101 hadoop-2.6.0-cdh5.5.2]$ vi etc/hadoop/hadoop-env.sh 
# 修改配置文件
export JAVA_HOME=/usr/local/java/jdk1.7.0_79
```

#### core-site.xml

```
# 文档位置
/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/etc/hadoop/core-site.xml

# 修改配置文件
<configuration>
  <property>
   <name>fs.defaultFS</name>
   <value>hdfs://h101:9000</value>        <!--主机名-->
   <description>NameNode URI.</description>
 </property>

 <property>
   <name>io.file.buffer.size</name>
   <value>131072</value>
   <description>Size of read/write buffer used inSequenceFiles.</description>
 </property>
</configuration>
```
#### hdfs-site.xml
```
# 文档路径
/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/etc/hadoop/hdfs-site.xml
# 这里路径要注意下file:///usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/
<configuration>
<property>
   <name>dfs.namenode.secondary.http-address</name>
   <value>h201:50090</value>
   <description>The secondary namenode http server address andport.</description>
 </property>

 <property>
   <name>dfs.namenode.name.dir</name>
   <value>file:///usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/dfs/name</value>
   <description>Path on the local filesystem where the NameNodestores the namespace and transactions logs persistently.</description>
 </property>

 <property>
   <name>dfs.datanode.data.dir</name>
   <value>file:///usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/dfs/data</value>
   <description>Comma separated list of paths on the local filesystemof a DataNode where it should store its blocks.</description>
 </property>

 <property>
   <name>dfs.namenode.checkpoint.dir</name>
   <value>file:///usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/dfs/namesecondary</value>
   <description>Determines where on the local filesystem the DFSsecondary name node should store the temporary images to merge. If this is acomma-delimited list of directories then the image is replicated in all of thedirectories for redundancy.</description>
 </property>

<property>
    <name>dfs.replication</name>
    <value>2</value>
</property>
</configuration>
```
#### mapred-site.xml

```
# 文档路径
/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/etc/hadoop/mapred-site.xml
# 复制创建mapred-site.xml配置文件
[hadoop@h101 hadoop]$ cp mapred-site.xml.template mapred-site.xml
[hadoop@h101 hadoop]$ vi mapred-site.xml
# 配置文件参考
<configuration>
<property>
   <name>mapreduce.framework.name</name>
   <value>yarn</value>
   <description>Theruntime framework for executing MapReduce jobs. Can be one of local, classic or yar
n.</description>
  </property>

  <property>
   <name>mapreduce.jobhistory.address</name>
    <value>h101:10020</value>
    <description>MapReduce JobHistoryServer IPC host:port</description>
  </property>

  <property>
   <name>mapreduce.jobhistory.webapp.address</name>
    <value>h101:19888</value>
    <description>MapReduce JobHistoryServer Web UI host:port</description>
  </property>
</configuration>
```
#### yarn-site.xml

```
# 文档路径
/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/etc/hadoop/yarn-site.xml
[hadoop@h201 hadoop]$ vi yarn-site.xml
# 配置文件参考
<configuration>
<property>
   <name>yarn.resourcemanager.hostname</name>
  <value>h101</value>
  <description>The hostname of theRM.</description>
</property>

 <property>
   <name>yarn.nodemanager.aux-services</name>
   <value>mapreduce_shuffle</value>
   <description>Shuffle service that needs to be set for Map Reduceapplications.</description>
 </property>
</configuration>
```
#### slaves 
配置从节点
```
[hadoop@h101 hadoop]$ vi slaves

h102
h103
```

### hodoop执行运行

```
# 在h102,h103上创建hadoop目录
[root@h102 ~]# cd /usr/local/
[root@h102 local]# mkdir hadoop

# 格式化hadoop看有没有错误
[hadoop@h101 hadoop-2.6.0-cdh5.5.2]$ bin/hadoop namenode -format
# 启动hadoop
[hadoop@h101 hadoop-2.6.0-cdh5.5.2]$  sbin/start-all.sh
# ps:停止hadoop
[hadoop@h101 hadoop-2.6.0-cdh5.5.2]$
sbin/stop-all.sh 
# 验证
# 主节点4个服务
[hadoop@h101 hadoop-2.6.0-cdh5.5.2]$ jps
5022 NameNode
5160 SecondaryNameNode
5559 Jps
5304 ResourceManager
# 从节点3个服务
[hadoop@h102 hadoop-2.6.0-cdh5.5.2]$ jps
3866 NodeManager
3774 DataNode
4010 Jps

# 测试是否可用
[hadoop@h101 hadoop-2.6.0-cdh5.5.2]$ bin/hadoop fs -mkdir /aaa
[hadoop@h101 hadoop-2.6.0-cdh5.5.2]$ bin/hadoop fs -ls /
# 以下是执行结果
[hadoop@h101 hadoop-2.6.0-cdh5.5.2]$ bin/hadoop fs -mkdir /aaa
18/04/23 20:49:52 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
[hadoop@h101 hadoop-2.6.0-cdh5.5.2]$ bin/hadoop fs -ls /
18/04/23 20:50:04 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2018-04-23 20:49 /aaa
```

## Flume
### 架构参考
flume 是一个数据抽取工具
![image](http://dl2.iteye.com/upload/attachment/0113/4940/3027c672-7305-3e09-a350-d018307412df.png)
1. Source负责日志流入，比如从文件、网络、Kafka等数据源流入数据，数据流入的方式有两种轮训拉取和事件驱动；

2. Channel负责数据聚合/暂存，比如暂存到内存、本地文件、数据库、Kafka等，日志数据不会在管道停留很长时间，很快会被Sink消费掉；

3. Sink负责数据转移到存储，比如从Channel拿到日志后直接存储到HDFS、HBase、Kafka、ElasticSearch等，然后再有如Hadoop、Storm、ElasticSearch之类的进行数据分析或查询。
### 配置环境变量

```
# 这里配置flume的环境变量
[hadoop@h101 apache-flume-1.6.0-cdh5.5.2-bin]$ vi /etc/profile
# 环境变量参考配置
export FLUME_HOME=/usr/local/hadoop/hadoop2file/apache-flume-1.6.0-cdh5.5.2-bin
export FLUME_CONF_DIR=$FLUME_HOME/conf
export FLUME_HOME FLUME_CONF_DIR
```

### 配置测试环境
```
# 复制并解压flume
cp flume-ng-1.6.0-cdh5.5.2.tar.gz /usr/local/hadoop/
tar -zxvf flume-ng-1.6.0-cdh5.5.2.tar.gz 
cd apache-flume-1.6.0-cdh5.5.2-bin/conf/
# 复制配置文件
cp flume-conf.properties.template flume-conf.properties
# 编辑配置文件
vi flume-conf.properties
```
#### 测试配置文件参考
```
#名字
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
#模式网络模式
a1.sources.r1.type = netcat
#捕获那个机器
a1.sources.r1.bind = h101
#端口
a1.sources.r1.port = 44444

# Describe the sink
#投到了当前模式显示logger日志
a1.sinks.k1.type = logger
# Use a channel which buffers events in memory
#通道类型：内存
a1.channels.c1.type = memory
#通道总量1000字节可以设大
a1.channels.c1.capacity = 1000
#每个事物达到100，进行投递
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

#### 测试环境验证

```
# 运行测试flume

[hadoop@h101 apache-flume-1.6.0-cdh5.5.2-bin]$ bin/flume-ng agent --conf /usr/local/hadoop/hadoop2file/apache-flume-1.6.0-cdh5.5.2-bin/conf/ --conf-file conf/flume-conf.properties --name a1 -Dflume.root.logger=INFO,console

# 打开另一个窗口执行telnet命令测试fllum
[root@h101 ~]# telnet h101 44444
Trying 192.168.56.101...
Connected to h101.
Escape character is '^]'.
hello word;
OK

# flume后台
(SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:70)] Event: { headers:{} body: 68 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D          hello world!. }

```
### 配置集群环境
#### 启动命令解释
命令解释

```
# 使用flume目录下flume-ng命令
# --conf  这里是你flume的地址
# --conf-file 这里是你的配置文件名称
# agent1 这个是你启动JVM这个进程的名称，在配置文件中如:agent1.channels = ch1
bin/flume-ng agent --conf /usr/local/hadoop/hadoop2file/apache-flume-1.6.0-cdh5.5.2-bin/conf/ --conf-file conf/aa.conf --name agent1 -Dflume.root.logger=INFO,console
```
#### 测试文件配置参考
aa.conf配置文件参考

```
真正生产环境中的配置文件后续详解
# 创建配置文件 aa.conf
touch aa.conf
# 集群环境配置文件参考

agent1.channels = ch1
agent1.sources = avro-source1
agent1.sinks = log-sink1

# Define a memory channel called ch1 on agent1
#通道类型：内存
agent1.channels.ch1.type = memory

#缓存的最大容量
agent1.channels.ch1.capacity = 10000

#每事务的最大容量
agent1.channels.ch1.transactionCapacity = 10000
#保持连接的秒数30秒
agent1.channels.ch1.keep-alive = 30


#define source monitor a file
#执行模式
agent1.sources.avro-source1.type = exec
#用的shell格式是bash格式
agent1.sources.avro-source1.shell = /bin/bash -c
#抽取那个文件/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/logs/hadoop-hadoop-namenode-h101.log
agent1.sources.avro-source1.command = tail -n +0 -F /usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/logs/hadoop-hadoop-namenode-h101.log  
agent1.sources.avro-source1.channels = ch1
#应该是连接数是5
agent1.sources.avro-source1.threads = 5

# Define a logger sink that simply logs all events it receives
# and connect it to the other end of the same channel.
#投递
agent1.sinks.log-sink1.channel = ch1
#头带到哪hdfs
agent1.sinks.log-sink1.type = hdfs
#投递路径
agent1.sinks.log-sink1.hdfs.path = hdfs://192.168.56.101:9000/usr/local/hadoop/hadoop2file/apache-flume-1.6.0-cdh5.5.2-bin/flumeTest
#写类型text文本
agent1.sinks.log-sink1.hdfs.writeFormat = Text
#文件类型是数据流
agent1.sinks.log-sink1.hdfs.fileType = DataStream

#hdfs创建多长时间新建文件，0不基于时间
agent1.sinks.log-sink1.hdfs.rollInterval = 0

#基于数据大小  1000字节
agent1.sinks.log-sink1.hdfs.rollSize = 10000

#hdfs有多少条消息时新建文件，0不基于消息个数
agent1.sinks.log-sink1.hdfs.rollCount = 0

#匹配的一个大小
agent1.sinks.log-sink1.hdfs.batchSize = 1000
#事件的最大数有多少个
agent1.sinks.log-sink1.hdfs.txnEventMax = 1000
#调用的超时多少秒
agent1.sinks.log-sink1.hdfs.callTimeout = 60000
#追加的timeout时间多少秒
agent1.sinks.log-sink1.hdfs.appendTimeout = 60000

```
## Sqoop2搭建
sqoop就是转换工具
- 写的很好：https://blog.csdn.net/u014729236/article/details/46876651
### sqoop1和sqoop2架构及区别
这两个版本是完全不兼容的，其具体的版本号区别为1.4.x为sqoop1，1.99x为sqoop2。sqoop1和sqoop2在架构和用法上已经完全不同
![image](http://s3.51cto.com/wyfs02/M02/37/8B/wKiom1OtGw-xcD4sAAFDETuQy_c685.jpg)
![image](http://s3.51cto.com/wyfs02/M00/37/8A/wKioL1OtGuHAOCISAAGbmEFRBzQ551.jpg)

### 安装及配置

```
# 解压并移动
tar -zxvf sqoop-1.99.7-bin-hadoop200.tar.gz 
mv sqoop-1.99.7-bin-hadoop200 /usr/local/hadoop/sqoop
# 配置环境变量
[root@h101 hadoop2file]# vi /etc/profile
[root@h101 hadoop2file]# source /etc/profile

#sqoop配置文件参考
export SQOOP_HOME=/usr/local/hadoop/sqoop
export PATH=$SQOOP_HOME/bin:$PATH
export CATALINA_BASE=$SQOOP_HOME/server
export LOGDIR=$SQOOP_HOME/logs
```


```
common.loader=/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/common/*.jar,/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/common/lib/*.jar,/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/hdfs/*.jar,/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/hdfs/lib/*.jar,/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/mapreduce/*.jar,/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/mapreduce/lib/*.jar,/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/tools/*.jar,/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/tools/lib/*.jar,/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/yarn/*.jar,/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/yarn/lib/*.jar,/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/httpfs/tomcat/lib/*.jar,
```


```
[hadoop@h101 hadoop2file]$ cp mysql-connector-java-5.1.32-bin.jar /usr/local/hadoop/sqoop/server/lib/
```

### 这里记录与参考链接的不同点
```
# 因为版本是1.99.7没有ID 所以后面创建 Link选择类型的时候需要使用名称
sqoop:000> show connector
+------------------------+---------+------------------------------------------------------------+----------------------+
|          Name          | Version |                           Class                            | Supported Directions |
+------------------------+---------+------------------------------------------------------------+----------------------+
| oracle-jdbc-connector  | 1.99.7  | org.apache.sqoop.connector.jdbc.oracle.OracleJdbcConnector | FROM/TO              |
| sftp-connector         | 1.99.7  | org.apache.sqoop.connector.sftp.SftpConnector              | TO                   |
| kafka-connector        | 1.99.7  | org.apache.sqoop.connector.kafka.KafkaConnector            | TO                   |
| kite-connector         | 1.99.7  | org.apache.sqoop.connector.kite.KiteConnector              | FROM/TO              |
| ftp-connector          | 1.99.7  | org.apache.sqoop.connector.ftp.FtpConnector                | TO                   |
| hdfs-connector         | 1.99.7  | org.apache.sqoop.connector.hdfs.HdfsConnector              | FROM/TO              |
| generic-jdbc-connector | 1.99.7  | org.apache.sqoop.connector.jdbc.GenericJdbcConnector       | FROM/TO              |
+------------------------+---------+------------------------------------------------------------+----------------------+
sqoop:000> create link -c generic-jdbc-connector
Creating link for connector with name generic-jdbc-connector
Please fill following values to create new link object
Name: mysql-docker-link

Database connection

Driver class: com.mysql.jdbc.Driver
Connection String: jdbc:mysql://192.168.56.1:3306/keenpower_corpus
Username: root
Password: ******
Fetch Size: 
Connection Properties: 
There are currently 0 values in the map:
entry# protocol=tcp
There are currently 1 values in the map:
protocol = tcp
entry# protocol=tcp
There are currently 1 values in the map:
protocol = tcp
entry# 

SQL Dialect

Identifier enclose: 
New link was successfully created with validation status OK and name mysql-docker-link

sqoop:000> show link
+-------------------+------------------------+---------+
|       Name        |     Connector Name     | Enabled |
+-------------------+------------------------+---------+
| mysql-docker-link | generic-jdbc-connector | true    |
+-------------------+------------------------+---------+
```

## zookeeper安装
### 安装配置过程参考
```
# 解压zookeeper
  454  tar -zxvf zookeeper-3.4.5-cdh5.5.2.tar.gz 
# 移动zookeeper并重命名
  456  mv zookeeper-3.4.5-cdh5.5.2 /usr/local/hadoop/zookeeper
# 创建配置文件zoo.cfg
  461  cd conf/
  465  cp zoo_sample.cfg zoo.cfg
# 创建zookeeper目录下创建目录data,log
  472  mkdir data
  473  mkdir log
# 编辑配置文件conf/zoo.cfg 
  478  vi zoo.cfg 
  
```
#### zoo.cfg配置文件参考
编辑配置文件conf/zoo.cfg 参考

```
tickTime=2000    
initLimit=10   
syncLimit=5
clientPort=2181
dataDir=/usr/local/hadoop/zookeeper/data
dataLogDir=/usr/local/hadoop/zookeeper/log
server.1=192.168.56.101:2888:3888
server.2=192.168.56.102:2888:3888
server.3=192.168.56.103:2888:3888

***2888端口号是zookeeper服务之间通信的端口，而3888是zookeeper与其他应用程序通信的端口

```
#### 远程复制文件
将文档复制到102，103两个从节点
```
# 将文档复制到102，103两个节点
scp -r zookeeper h102:/usr/local/hadoop/
scp -r zookeeper h103:/usr/local/hadoop/
```
#### 查看权限
复制过后要看看权限是否正确

```
[root@h102 hadoop]# ll
total 8
drwxr-xr-x. 16 hadoop hadoop 4096 Apr 23 20:35 hadoop-2.6.0-cdh5.5.2
drwxr-xr-x. 16 hadoop hadoop 4096 Apr 25 14:58 zookeeper
```
#### myid设置
分别设置zookeeper的ID
```
# 没有myid文件需要创建
[hadoop@h101 hadoop]$ vi zookeeper/data/myid
[hadoop@h102 hadoop]$ vi zookeeper/data/myid
[root@h103 hadoop]# vi zookeeper/data/myid
```
#### 启动zookeeper

```
# h101

[hadoop@h101 bin]$ ./zkServer.sh start
JMX enabled by default
Using config: /usr/local/hadoop/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[hadoop@h101 bin]$ ./zkServer.sh status
JMX enabled by default
Using config: /usr/local/hadoop/zookeeper/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.

# h102
[hadoop@h102 zookeeper]$ cd bin/
[hadoop@h102 bin]$  ./zkServer.sh start
JMX enabled by default
Using config: /usr/local/hadoop/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[hadoop@h102 bin]$  ./zkServer.sh status
JMX enabled by default
Using config: /usr/local/hadoop/zookeeper/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.

# h103
[root@h103 bin]#  ./zkServer.sh start
JMX enabled by default
Using config: /usr/local/hadoop/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@h103 bin]#  ./zkServer.sh status
JMX enabled by default
Using config: /usr/local/hadoop/zookeeper/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.

# 最后成功的状态,一个主节点，2个从节点
[hadoop@h101 bin]$ ./zkServer.sh status
JMX enabled by default
Using config: /usr/local/hadoop/zookeeper/bin/../conf/zoo.cfg
Mode: follower
[hadoop@h102 bin]$  ./zkServer.sh status
JMX enabled by default
Using config: /usr/local/hadoop/zookeeper/bin/../conf/zoo.cfg
Mode: follower
[root@h103 bin]#  ./zkServer.sh status
JMX enabled by default
Using config: /usr/local/hadoop/zookeeper/bin/../conf/zoo.cfg
Mode: leader
```
### 调试

```
# 这里如果报错 可以参考下面的命令进行调试
[hadoop@h101 bin]$ ./zkServer.sh start-foreground  

# 参考文档
https://www.jianshu.com/p/18be6672d2a5  //这里的错误是常规的错误，很有意思的是我这里没有这么报错，所以有些时候还是需要调试的方式来解决，定位问题
http://zookeeper.apache.org/doc/r3.3.3/zookeeperAdmin.html#sc_zkMulitServerSetup //这个是官方文档
# 他这里的默认配置是
tickTime=2000
dataDir=/var/zookeeper/
clientPort=2181  //我最开始并没有配置客户端端口
initLimit=5
syncLimit=2
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
```

## Hbase
### 安装与配置
```
[hadoop@h101 hadoop]$ tar -zxvf hbase-1.0.0-cdh5.5.2.tar.gz 
[hadoop@h101 hadoop2file]$ mv hbase-1.0.0-cdh5.5.2 /usr/local/hadoop/hbase
```
修改hbase-env.sh配置文件

```
# 编辑hbase-env.sh配置文件
[hadoop@h101 hbase]$ vi conf/hbase-env.sh 
# 查看JAVA_HOME
[root@h101 ~]# echo ${JAVA_HOME}
/usr/local/java/jdk1.7.0_79

# 配置JAVA_HOME
# export JAVA_HOME=/usr/java/jdk1.6.0/
export JAVA_HOME=/usr/local/java/jdk1.7.0_79/
# 不使用自带的zookeeper
# export HBASE_MANAGES_ZK=true
export HBASE_MANAGES_ZK=false
```

修改hbase-site.xml配置文件
```
# 参考文档
https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.4/bk_security/content/kerb-config-hbase-site.html
https://github.com/apache/hbase/blob/master/hbase-common/src/main/resources/hbase-default.xml

[hadoop@h101 hbase]$ vi conf/hbase-site.xml 
```

```
<configuration>
<property>
                <name>hbase.rootdir</name>
                <value>hdfs://h101:9000/hbase</value>
        </property>
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
        </property>
        <property>
                <name>hbase.zookeeper.quorum</name>
                <value>h101,h102,h103</value>
        </property>
        <property>
                <name>hbase.zookeeper.property.dataDir</name> <value>/usr/local/hadoop/zookeeper/data</value> 
        </property>
        <property>
                <name>hbase.tmp.dir</name>
                <value>/usr/local/hadoop/hbase/tmp</value>
        </property>
        <property>
                <name>hbase.master.maxclockskew</name>
                <value>150000</value>
        </property>
</configuration>
```
创建目录
```
[hadoop@h101 hbase]$ mkdir data
[hadoop@h101 hbase]$ mkdir tmp
```
编辑regionservers配置文件
```
# 编辑注册服务
[hadoop@h101 hbase]$ vi conf/regionservers
h102
h103
```
启动服务
```
[hadoop@h101 hbase]$ scp -r hbase h102:/usr/local/hadoop/
[hadoop@h101 hbase]$ scp -r hbase h103:/usr/local/hadoop/
# 
[hadoop@h101 hbase]$ bin/start-hbase.sh 
```
验证

```
[hadoop@h101 hbase]$ jps
29128 Jps
20502 SqoopJettyServer
25348 ResourceManager
25201 SecondaryNameNode
29016 HMaster       //主服务
20362 JobHistoryServer
25056 NameNode
28378 QuorumPeerMain

[hadoop@h102 hadoop]$ jps
9101 QuorumPeerMain
7498 DataNode
7593 NodeManager
9542 Jps
9358 HRegionServer   //从服务

[hadoop@h103 bin]$ jps
8963 Jps
7187 DataNode
7283 NodeManager
8746 HRegionServer   //从服务
```
验证
```
[hadoop@h101 bin]$ ./hbase shell
2018-04-25 21:12:52,864 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hadoop/hbase/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
2018-04-25 21:12:56,490 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.0.0-cdh5.5.2, rUnknown, Mon Jan 25 16:33:02 PST 2016

[hadoop@h101 bin]$ ./hbase shell
```
java 操作
```
https://www.jianshu.com/p/ce26f0674078
```
### 配置WEB UI

```
# 在添加如下配置
[hadoop@h101 conf]$ vi hbase-site.xml 

<property>
<name>hbase.master.info.port</name>
<value>60010</value>
</property>

# 登录地址
http://192.168.56.101:60010/master-status
```

## HIVE
### 安装和配置

```
# 解压并移动hive
# 将JDBC包拷贝到hive/lib目录下
cp mysql-connector-java-5.1.32-bin.jar /usr/local/hadoop/hive/lib/

# 编辑文件hive-site.xml
[hadoop@h101 conf]$ pwd
/usr/local/hadoop/hive/conf
vi hive-site.xml
```
#### hivie-site.xml 参考配置
```
# 主节点参考
<configuration>
        <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://192.168.56.1:3306/hive?createDatabaseIfNotExist=true</value>
        <description>JDBC connect string for a JDBC metastore</description>
        </property>
        <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
        <description>Driver class name for a JDBC metastore</description>
        </property>

        <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
        <description>username to use against metastore database</description>
        </property>
        <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
        <description>password to use against metastore database</description>
        </property>
</configuration>
```
### 启动

```
[hadoop@h101 conf]$ hive --service metastore &
```
### 验证

```
[hadoop@h101 conf]$ jps
20502 SqoopJettyServer
25348 ResourceManager
25201 SecondaryNameNode
20362 JobHistoryServer
25056 NameNode
28378 QuorumPeerMain
3350 Jps
3272 RunJar
2110 HMaster

[hadoop@h101 conf]$ hive
which: no hbase in (/usr/local/hadoop/sqoop/bin:/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/java/jdk1.7.0_79/bin:/usr/local/hadoop/hive/bin:/root/bin)

Logging initialized using configuration in jar:file:/usr/local/hadoop/hive/lib/hive-common-1.1.0-cdh5.5.2.jar!/hive-log4j.properties
WARNING: Hive CLI is deprecated and migration to Beeline is recommended.
hive> 

```
## spark
### 参考地址

```
https://www.jianshu.com/p/09143312dd94
```

### scala安装

```
# 解压并移动scala
tar -zxvf scala-2.9.3.tgz 
mv scala-2.9.3 /usr/local/hadoop/scala

vi /etc/profile
# 配置环境变量
#scala
export SCALA_HOME=/usr/local/hadoop/scala
export PATH=$PATH:$SCALA_HOME/bin
export SCALA_HOME PATH

# 验证
scala -version
```
### spark 安装
#### 解压并安装
```
# 解压并移动sprak
tar -zxvf spark-1.3.0-bin-hadoop2.4.tgz 
mv spark-1.3.0-bin-hadoop2.4 /usr/local/hadoop/spark
# 配置配置文件
pwd
cd /usr/local/hadoop/spark/conf/
cp spark-env.sh.template spark-env.sh
vi spark-env.sh
cp slaves.template slaves
vi slaves

# spark环境变量配置
export SPARK_HOME=/usr/local/hadoop/spark
export PATH=$PATH:$SPARK_HOME/bin
export SPARK_HOME PATH
```
#### spark-env.sh配置参考

```
export JAVA_HOME=/usr/local/java/jdk1.7.0_79
export HADOOP_HOME=/usr/local/hadoop/hadoop-2.6.0-cdh5.5.2
export SCALA_HOME=/usr/local/hadoop/scala
# spark
export SPARK_MASTER=192.168.56.101
export SPARK_WORKER_MEMORY=1g
```
#### slaves配置参考

```
h101
h102
h103
```


### 启动
```
[hadoop@h101 spark]$ sbin/start-all.sh 
rsync from 192.168.56.101
/usr/local/hadoop/spark/sbin/spark-daemon.sh: line 141: rsync: command not found
starting org.apache.spark.deploy.master.Master, logging to /usr/local/hadoop/spark/sbin/../logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-h101.out
h102: rsync from 192.168.56.101
h102: /usr/local/hadoop/spark/sbin/spark-daemon.sh: line 141: rsync: command not found
h102: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/hadoop/spark/sbin/../logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-h102.out
h103: rsync from 192.168.56.101
h103: /usr/local/hadoop/spark/sbin/spark-daemon.sh: line 141: rsync: command not found
h103: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/hadoop/spark/sbin/../logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-h103.out
h101: rsync from 192.168.56.101
h101: /usr/local/hadoop/spark/sbin/spark-daemon.sh: line 141: rsync: command not found
h101: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/hadoop/spark/sbin/../logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-h101.out
```

### 验证

```
http://192.168.56.101:8080/
```


## 注意事项
1. 请注意区分用户，root用户和hadoop用户执行命令产生的效果是不同的
2. 

## 问题汇总
### 使用root用户创建ssh-key链接

```
[root@h101 java]#  ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub h101
The authenticity of host 'h101 (127.0.0.1)' can't be established.
ECDSA key fingerprint is 73:f6:0a:0b:a7:ff:63:03:74:72:87:35:78:8f:ab:14.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@h101's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'h101'"
and check to make sure that only the key(s) you wanted were added.

```
