---
layout: post
title: Hadoop（三）—— Centos7下Hadoop完全分布式集群安装
categories: Hadoop
description: 在Centos7下Hadoop分布式安装
keywords: Hadoop, Centos7，分布式
---

![1537073767297](/images/Hadoop/hadoop.jpg)

## 一、开始

### 1. 准备工作

Hadoop的安装，伪分布式只需要一台服务器，而完全分布式至少需要三台服务器，最好是`2n+1`台。

- 本次准备三台服务器搭集群。集群各个节点（并在`/etc/host`文件下添加下面ip和）

```
192.168.128.150	master //namenode节点
192.168.128.151	slave1 //datanode节点
192.168.128.152	slave2 //datanode节点
```

- Hadoop安装包（hadoop-2.8.0.tar.gz）

### 2. 配置SSH免密码登录

SSH（Secure Shell的缩写）建立在TCP/TP协议的应用层和传输层基础上的安全协议。SSH保障了远程登录和网络传输服务的安全性，起到防止信息泄露等作用，通过SSH可以对文件进行加密处理，SSH也可以运行于多平台。配置SSH无密码登录的步骤如下所示，以下步骤都是在主节点 **master** 上操作。

```shell
ssh localhost # 生成 ~/.ssh/ 目录
exit #退出ssh								
cd ~/.ssh/ # 若没有该目录，需要再执行一次ssh localhost
ssh-keygen -t rsa # 按三次回车就可以
cat id_rsa.pub >> authorized_keys
chmod 600 ./authorized_keys
```

生成私钥密钥`id_rsa`和公有密钥`id_rsa.pub`两个文件。`ssh-keygen`是用来生成RSA类型的密钥以及管理该密钥，参数`“-t”`用于指定要创建的SSH密钥的类型为RSA。用`ssh-copy-id`将公钥复制到远程机器中(/依次输入yes和root用户的密码)：

```shell
ssh-copy-id -i /root/.ssh/id_rsa.pub master
ssh-copy-id -i /root/.ssh/id_rsa.pub slave1
ssh-copy-id -i /root/.ssh/id_rsa.pub slave2
```

验证SSH是否能够免密钥登录：

```shell
ssh master
ssh slave1
ssh slave2
```

### 3. 配置时间同步服务

NTP服务器是用来使计算机时间同步化的一种协议，它可以使计算机对其服务器或时钟源做同步化，提供高精准度的时间校正。Hadoop集群对时间要求很高，主节点与各从节点的时间都必须要同步。配置时间同步服务主要是为了进行集群间的时间同步。Hadoop集群配置时间同步服务的步骤如下所示。

所有节点都需要安装NTP服务：

```shell
yum install –y ntp
```

以master节点作为NTP服务主节点，设置master时间为北京时间

```shell
ntpdate asia.pool.ntp.org
```

ps：要是不想继续下面同步时间的步骤，直接在每个节点都运行下面这个命令，都设置成北京时间，只要所有节点的时间一致就行。

修改master 的 `/etc/ntp.conf` 文件

```shell
vim /etc/ntp.conf
```

在 `/etc/ntp.conf` 文件注释掉server开头的行，添加下面代码

```
restrict 192.168.0.0 mask 255.255.255.0 nomodify notrap
server 127.127.1.0
fudge 127.127.1.0 stratum 10
```

在slave1,slave2配置NTP，同样修改 `/etc/ntp.conf` 文件，释掉server开头的行，添加下面代码

```
server master
```

启动NTP服务，在mater节点启动NTP服务，并设置开机启动

```shell
systemctl start ntpd & systemctl disable ntpd
```

在slave1，slave2上执行以下命令即可同步时间，并启动NTP服务和设置开机启动

```shell
ntpdate maste
systemctl start ntpd & systemctl disable ntpd
```

## 二、搭建Hadoop完全分布式集群

下面所有安装配置都是在master节点弄好，然后用scp命令同步其他节点。

### 1. 安装Hadoop

把下载好的Hadoop安装包上传到master，解压

```shell
tar –zxf hadoop-2.8.0.tar.gz –C /usr/hadoop
mv /usr/hadoop/hadoop-2.8.0 /usr/hadoop/hadoop
```

### 2. 修改配置文件

搭建Hadoop完全分布式集群安装相比Hadoop伪分布式安装，需要修改的配置文件多一些，其中配置涉及到的文件有7个：**core-site.xml**、**hadoop-env.sh**、**yarn-env.sh**、**mapred-site.xml**、**yarn-site.xml**、**slaves**、**hdfs-site.xml**。

#### core-site.xml

该文件Hadoop的核心配置文件，这里需要配置两个属性，`fs.defaultFS`配置了Hadoop的HDFS系统的命名，位置为主机的8020端口，这里需要注意替换hdfs://master:8020中斜体的master，改名字为NameNode所在机器的机器名；`hadoop.tmp.dir`配置了Hadoop的临时文件的位置。

```xml
<configuration>    
	<property>
		<name>fs.defaultFS</name>  
		<value>hdfs://master:8020</value>  
	</property>    
	<property>
		<name>hadoop.tmp.dir</name>
        	<value>/var/log/hadoop/tmp</value>
	</property>    
</configuration>
```

#### hadoop-env.sh

该文件是Hadoop运行基本环境的配置，需要修改为JDK的实际位置

```sh
export JAVA_HOME=/usr/jvm/jdk1.8
```

#### yarn-env.sh

该文件是YARN框架运行环境的配置，同样需要修改JDK所在位置。

```sh
export JAVA_HOME=/usr/jvm/jdk1.8
```

#### mapred-site.xml

这个是MapReduce的相关配置，由于Hadoop2.x使用了YARN框架，所以必须在`mapreduce.framework.name`属性下配置yarn。`mapreduce.jobhistory.address`和`mapreduce.jobhistoryserver.webapp.address`是配置JobHistoryserver的相关配置，即运行MapReduce任务的日志相关服务，这里同样需要注意修改“master”为实际服务所在机器的机器名。

`mapred-site.xml`文件是从`mapred-site.xml.template`文件复制得到的，执行复制文件命令：

```shell
cp mapred-site.xml.template mapred-site.xml
```

然后修改**mapred-site.xml** 文件内容

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <!-- jobhistory properties -->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>master:10020</value>
    </property>
    <property>
         <name>mapreduce.jobhistory.webapp.address</name>
         <value>master:19888</value>
    </property>
</configuration>
```

#### yarn-site.xml

该文件为YARN框架的配置，在最开始命名了一个`yarn.resourcemanager.hostname`的变量，这样在后面YARN的相关配置中就可以直接引用该变量了，其他配置保持不变即可。

```xml
<configuration>
<!-- Site specific YARN configuration properties -->
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>master</value>
	</property>    
	<property>
        	<name>yarn.resourcemanager.address</name>
       		<value>${yarn.resourcemanager.hostname}:8032</value>
	</property>
	<property>
        	<name>yarn.resourcemanager.scheduler.address</name>
        	<value>${yarn.resourcemanager.hostname}:8030</value>
	</property>
	<property>
        	<name>yarn.resourcemanager.webapp.address</name>
        	<value>${yarn.resourcemanager.hostname}:8088</value>
	</property>
	<property>
        	<name>yarn.resourcemanager.webapp.https.address</name>
        	<value>${yarn.resourcemanager.hostname}:8090</value>
	</property>
	<property>
        	<name>yarn.resourcemanager.resource-tracker.address</name>
        	<value>${yarn.resourcemanager.hostname}:8031</value>
	</property>
	<property>
        	<name>yarn.resourcemanager.admin.address</name>
        	<value>${yarn.resourcemanager.hostname}:8033</value>
	</property>
	<property>
        	<name>yarn.nodemanager.local-dirs</name>
        	<value>/data/hadoop/yarn/local</value>
	</property>
	<property>
        	<name>yarn.log-aggregation-enable</name>
        	<value>true</value>
	</property>
	<property>
        	<name>yarn.nodemanager.remote-app-log-dir</name>
        	<value>/data/tmp/logs</value>
	</property>
	<property> 
		<name>yarn.log.server.url</name> 
		<value>http://master:19888/jobhistory/logs/</value>
		<description>URL for job history server</description>
    	</property>
    	<property>
		<name>yarn.nodemanager.vmem-check-enabled</name>
        	<value>false</value>
	</property>
	<property>
        	<name>yarn.nodemanager.aux-services</name>
        	<value>mapreduce_shuffle</value>
	</property>
	<property>
        	<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
		<value>org.apache.hadoop.mapred.ShuffleHandler</value>
	</property>
</configuration>
```

#### slaves

该文件里面保存有slave节点的信息，该文件的节点将作为DataNode节点。

```
slave1
slave2
```

#### hdfs-site.xml

这个是HDFS相关的配置文件，`dfs.namenode.name.dir`和`dfs.datanode.data.dir`分别指定了NameNode元数据和DataNode数据存储位置。`dfs.namenode.secondary.http-address`配置的是SecondaryNameNode的地址，同样需要注意修改“master”为实际SecondaryNameNode地址。`dfs.replication`配置了文件块的副本数，默认就是3个，所以这里也可以不配置。

```xml
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///data/hadoop/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///data/hadoop/hdfs/data</value>
    </property>
    <property>
         <name>dfs.namenode.secondary.http-address</name>
         <value>master:50090</value>
    </property>
    <property>
         <name>dfs.replication</name>
         <value>2</value>
    </property>
</configuration>
```

### 3. 启动关闭集群

#### Hadoop同步到其他节点

如果slave1和slave2没有 `/usr/hadoop` 目录就先用mkdir命令创建目录，然后在master执行以下命令将hadoop同步其他节点

```shell
scp -r /usr/hadoop/hadoop root@slave1:/usr/hadoop & scp -r /usr/hadoop/hadoop root@slave2:/usr/hadoop
```

#### 设置Hadoop环境变量

修改所有节点的 `/etc/profile` 文件，在其文件末尾添加

```
export HADOOP_HOME=/usr/local/hadoop-2.6.4
export PATH=$HADOOP_HOME/bin:$PATH
```

使其生效

```shell
source /etc/profile
```

#### 格式化NameNode

```shell
cd $HADOOP_HOME/bin
hadoop namenode -format
```

若出现 *Storage directory /data/hadoop/hdfs/name has been successfully formatted* 和*status 0* 的提示，则格式化成功。

#### 启动Hadoop

启动集群只需要在master节点

```shell
cd $HADOOP_HOME/sbin
//启动HDFS相关服务
start-dfs.sh  
//启动YARN相关服务
start-yarn.sh    
```

或者

```shell
cd $HADOOP_HOME/sbin
start-all.sh
```

集群启动之后，在主节点master，子节点slave1，slave2分别执行jps，出现如下图所示的信息，说明集群启动没问题。

![1541003810556](/images/Hadoop/3/jps.png)

相关命令

```shell
//关闭hadoop
stop-all.sh
//关闭YARN相关服务
stop-yarn.sh
//关闭HDFS相关服务
stop-dfs.sh    
//关闭日志相关服务
mr-jobhistory-daemon.sh  stop  historyserver 
```

#### 监控集群

服务|Web接口|默认端口
:---|:---|:--:
NameNode|http://NamenodehHost:port|50070
ResourceManager|http://ResourcemanagerHost:port|8088
MapReduce JobHistory Server| http://JobhistoryserverHost:port |19888

由于这些节点都是在master，所以上面的web接口都是 *http://master:port* 。

​	
