---
layout: wiki
title: Apache ActiveMQ安装
categories: Apache
description: 
keywords: Apache，ActiveMQ, MQ
---

## ActiveMQ介绍
> **Apache ActiveMQ **™ is the most popular and powerful open source messaging and Integration Patterns server.
> **Apache ActiveMQ ** is fast, supports many Cross Language Clients and Protocols, comes with easy to use Enterprise Integration Patterns and many advanced features while fully supporting JMS 1.1 and J2EE 1.4.
> **Apache ActiveMQ** is released under the Apache 2.0 License

MQ是消息中间件，是一种在分布式系统中应用程序借以传递消息的媒介，常用的有ActiveMQ，RabbitMQ，kafka。ActiveMQ是Apache下的开源项目，完全支持JMS1.1和J2EE1.4规范的JMS Provider实现。 

特点：  

- 支持多种语言编写客户端  
- 对spring的支持，很容易和spring整合  
- 支持多种传输协议：TCP,SSL,NIO,UDP等  
- 支持AJAX  消息形式：  
- 点对点（queue） 
- 一对多（topic）  



## 安装

### 1. 下载ActiveMQ

- [下载地址](http://activemq.apache.org/download.html)

- 选择最新版本

![1536559525598](/images/wiki/apache/ActiveMQ/downloadPage1.png)

- 里面有linux和windows的压缩包，选择需要的下载就行

![1536560134163](/images/wiki/apache/ActiveMQ/downloadPage2.png)



### 2. Liunx下安装

- 将文件下载好，上传到linux系统
- 解压： tar -zxvf apache-activemq-5.15.5-bin.tar.gz 
- 然后就可以了，进入bin目录
  - 启动：./activemq start
  - 重启：./activemq restart
  - 关闭：./activemq stop
  - 状态：./activemq status
- 进入后台
  - 链接：http://localhost:8681/admin
  - 用户名：admin
  - 密码：admin

### 3. Windows下安装

windows下的安装跟liunx的一模一样

- 下载文件
- 解压
- 进入安装目录的bin目录
  - 启动：./activemq start
  - 重启：./activemq restart
  - 关闭：./activemq stop
  - 状态：./activemq status
- 进入后台
  - 链接：http://localhost:8681/admin
  - 用户名：admin
  - 密码：admin

### 安装成功
![1536560660865](/images/wiki/apache/ActiveMQ/activemq-indexpage.png)

