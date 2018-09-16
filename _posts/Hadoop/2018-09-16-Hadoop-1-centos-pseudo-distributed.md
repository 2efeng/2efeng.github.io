---
layout: post
title: Hadoop（一）—— Centos7下Hadoop伪分布式安装
categories: Hadoop
description: 在Centos7下Hadoop伪分布式安装
keywords: Hadoop, Centos7
---

![1537073767297](/images/Hadoop/1/000.png)

# 一、准备

- 工具
  - VMware
  - Xshell
  - Xftp

- 系统和软件
  - Centos7 64位 （Minimal）
  - hadoop-2.7.4.tar 
  - jdk-8u131-linux-x64.tar

# 二、环境安装

打开xftp，将要安装的安装包上传到Liunx，这里只需要要用到hadoop和jdk。

![1537065481507](/images/Hadoop/1/1_01.png)

## 1. Liunx安装Java环境

### 检测历时安装

查看是否安装过jdk

```shell
rpm -qa | grep java
```
如果显示

```shell
java-x.x.x-gcj-compat-x.x.x.x-xxjpp.xxx
java-x.x.x-openjdk-x.x.x.x-x.x.bxx.exx
```

卸载（有几个卸载几个）

```shell
rpm -e --nodeps java-x.x.x-gcj-compat-x.x.x.x-xxjpp.xxx
rpm -e --nodeps java-x.x.x-openjdk-x.x.x.x-x.x.bxx.exx
```

因为我是使用最小安装，并没有安装java版本，如下图所示，所以就跳过这个流程。

![1537066072293](/images/Hadoop/1/2_1_01.png)

### 解压安装

进入JDK安装包的目录`/usr/local/temp` ,然后将安装包解压到`/usr/jvm` ，并修改解压后文件夹的文件名。

```shell
cd /usr/local/temp/
tar zxvf jdk-8u131-linux-x64.tar.gz /usr/jvm
mv /usr/jvm/jdk1.8.0_131/ /usr/jvm/jdk1.8
```

### 配置环境变量

编辑`/etc/profile`文件：

```shell
vim /etc/profile
```

在文件尾部添加如下配置：

```shell
export JAVA_HOME=/usr/jvm/jdk1.8
export JRE_HOME=$JAVA_HOME/jre
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
```

重新加载`/etc/profile`文件，使其生效

```shell
source /etc/profile
```

### 检查

```she
java
javac
```



## 2. 安装SSH、配置SSH无密码登陆

集群、单节点模式都需要用到 SSH 登陆（类似于远程登陆，你可以登录某台 Linux 主机，并且在上面运行命令），一般情况下，CentOS 默认已安装了 SSH client、SSH server,执行如下命令进行检验。

```shell
rpm -qa | grep ssh
```

若需要安装，则可以通过 yum 进行安装

```shell
yum install openssh-clients -y
yum install openssh-server -y
```

执行如下命令测试一下 SSH 是否可用：

```shell
ssh localhost
```

此时会有如下提示(SSH首次登陆提示)，输入 yes 。然后按提示输入密码，这样就登陆到本机了。

![1537068555607](/images/Hadoop/1/2_2_01.png)

首先输入 `exit` 退出刚才的 ssh，就回到了我们原先的终端窗口，然后利用 ssh-keygen 生成密钥，并将密钥加入到授权中：

```shell
exit								# 退出刚才的 ssh localhost
cd ~/.ssh/							# 若没有该目录，需要再执行一次ssh localhost
ssh-keygen -t rsa					# 按三次回车就可以
cat id_rsa.pub >> authorized_keys	# 加入授权
chmod 600 ./authorized_keys			# 修改文件权限
```

此时再用 `ssh localhost` 命令，无需输入密码就可以直接登陆了，如下图所示。

![1537068698561](/images/Hadoop/1/2_2_02.png)



# 三、Hadoop安装

## 1. 安装 Hadoop

将 Hadoop 安装至 `/usr/hadoop`中

```shell
cd /usr/local/temp/
tar -zxf hadoop-2.7.4.tar.gz -C /usr/hadoop
mv /usr/hadoop/hadoop-2.7.4/ /usr/hadoop/hadoop
```

Hadoop 解压后即可使用，进入bin目录下输入如下命令来检查 Hadoop 是否可用，成功则会显示 Hadoop 版本信息：

```shell
cd /usr/hadoop/hadoop/bin/
./hadoop version
```

![1537069514636](/images/Hadoop/1/2_2_03.png)



## 2. Hadoop伪分布式配置

Hadoop 可以在单节点上以伪分布式的方式运行，Hadoop 进程以分离的 Java 进程来运行，节点既作为 NameNode 也作为 DataNode，同时，读取的是 HDFS 中的文件。

### 配置Hadoop环境变量

首先，需要进入`~/.bashrc`设置Hadoop环境变量

```shell
vim ~/.bashrc
```

在文件尾部添加如下配置：

```shell
export JAVA_HOME=/usr/jvm/jdk1.8
export HADOOP_HOME=/usr/hadoop/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
```

保存后，重新加载`~/.bashrc`，使其生效

```shell
source ~/.bashrc
```



### 伪分布式配置

Hadoop 的配置文件位于 `/usr/hadoop/hadoop/etc/hadoop/` 中，伪分布式需要修改2个配置文件 **core-site.xml** 和 **hdfs-site.xml** 。

进入配置文件目录

```shell
cd /usr/hadoop/hadoop/etc/hadoop/
```

修改配置文件 **core-site.xml** 

```shell
vim core-site.xml
```

修改后：

```shell
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/hadoop/hadoop/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.128.150:9000</value>
    </property>
</configuration>
```

修改配置文件 **hdfs-site.xml**

```shell
vim hdfs-site.xml
```

修改后：

```shell
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/hadoop/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/hadoop/hadoop/tmp/dfs/data</value>
    </property>
</configuration>
```

配置完成后，执行 NameNode 的格式化:

```shell
cd /usr/hadoop/hadoop/
./bin/hdfs namenode -format		
```

成功的话，在最后会看到 “successfully formatted” 和 “Exitting with status 0” 的提示，



# 四、测试

hadoop启动和停止的命令

```shell
cd $HADOOP_HOME
./sbin/start-all.sh
./sbin/stop-all.sh
./sbin/restart-all.sh
```

启动后，`jps`查看进程是否启动成功，下面的进程都在，则说明hadoop启动成功

```shell
[root@master hadoop]# jps
4436 NameNode
4985 NodeManager
4875 ResourceManager
4558 DataNode
4718 SecondaryNameNode
5294 Jps
```

打开浏览器，http://192.168.128.150:50070 （我虚拟机配置的ip是192.168.128.150），出现下面的界面

![1537071953065](/images/Hadoop/1/4_01.png)

如果打不开这个页面，可能防火墙未开放端口

由于在虚拟机，为了方便我直接把防火墙关了，并禁止开机启动

```shell
systemctl stop firewalld
systemctl disable firewalld
```

如果是在生产环境中，只需要开放所需的端口就行

```shell
firewall-cmd --zone=public --add-port=50070/tcp --permanent
firewall-cmd --reload
```

