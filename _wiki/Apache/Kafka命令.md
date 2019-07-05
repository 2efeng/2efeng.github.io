---
layout: wiki
title: Apache Kafka命令
categories: Apache
description: 
keywords: Apache，Kafka
---



# Kafka日常操作命令

下面命令要在kafka根目录下执行

## Start

```shell
# 先启动zook
./bin/windows/zookeeper-server-start.bat ./config/zookeeper.properties 
# 启动kafka
./bin/windows/kafka-server-start.bat ./config/server.properties   
```



## Create Topic

```shell
# kafka默认端口是2181 
# topicName
./bin/windows/kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic topicName
```



## Topic List

```shell
./bin/windows/kafka-topics.bat --list --zookeeper localhost:2181
```



## Conusmer

```shell
# Conusmer和Producer用到的端口是9092
./bin/windows/kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic topicName --from-beginning
```



## Producer

```shell
./bin/windows/kafka-console-producer.bat --broker-list localhost:9092 --topic topicName
```



## Topic Describe

```shell
./bin/windows/kafka-topics.bat --zookeeper localhost:2181 --topic topicName --describe
```
