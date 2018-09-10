---
layout: post
title: ActiveMQ + Spring Camel
categories: Java
description: ActiveMQ和Spring Camel
keywords: ActiveMQ, MQ, Camel
---

## 准备

### 1. ActiveMQ
[ActiveMQ](http://www.kiangkiangkiang.cn/wiki/Apache/ActiveMQ安装/)

### 2. Spring  Camel

[SpringCamel](http://camel.apache.org/spring.html)



## 代码

- **pom.xml**

```xml
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-camel</artifactId>
    <version>${activemq-version}</version>
</dependency>

<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-core</artifactId>
    <version>${camel-version}</version>
</dependency>

<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-jms</artifactId>
    <version>${camel-version}</version>
</dependency>

<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-spring</artifactId>
    <version>${camel-version}</version>
</dependency>
```

- **spring.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:camel="http://camel.apache.org/schema/spring"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
       http://camel.apache.org/schema/spring
       http://camel.apache.org/schema/spring/camel-spring.xsd">
  
    <!-- 接收消息的bean -->
    <bean id="jmsInEndpoint" class="com.hzf.mq.App"/>
  
    <!-- 创建ActiveMQ连接 -->
    <bean id="jms" class="org.apache.activemq.camel.component.ActiveMQComponent">
        <property name="brokerURL" value="failover:(tcp://localhost:61616)"/>
    </bean>
  
    <camel:camelContext xmlns="http://camel.apache.org/schema/spring">
        <!-- 用来实例化ProducerTemplate，发送消息 -->
        <camel:template id="camelTemplate"/>
        <!-- 接收MQ消息 eg => jms:queue/topic:msgName-->
        <camel:route>
            <camel:from uri="jms:queue:hzf.queue"/>
            <!--<camel:from uri="jms:topic:hzf.queue"/>-->
            <camel:to uri="bean:jmsInEndpoint?method=getMsg"/>
        </camel:route>
    </camel:camelContext>
    
</beans>

```

- **App.java**

```java
package com.hzf.mq;

import org.apache.camel.ProducerTemplate;
import org.apache.xbean.spring.context.ClassPathXmlApplicationContext;
import org.springframework.context.ConfigurableApplicationContext;

/**
 * Created by zf.huang on 2018.7.17
 */
public class App {

    public static void main(String[] args) {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("spring.xml");
        ProducerTemplate producerTemplate = (ProducerTemplate) ctx.getBean("camelTemplate");
        String mqName = "jms:queue:hzf.queue";//"jms:topic:hzf.topic"
        //发送消息
        producerTemplate.asyncSendBody(mqName, "some message......");
    }

    //接收消息
    public void getMsg(String msg) {
        System.out.println(msg);
    }

}
```

一言不合就发源码[戳这里](https://github.com/FengHelloWorld/JavaDemo/tree/master/ActiveMQ)