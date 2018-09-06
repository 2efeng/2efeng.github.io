---
layout: post
title: Spring任务调度--Spring task
categories: Spring
description: Spring 定时器
keywords: Spring , timer，task
---

Spring task是Spring3.0以后自主开发的定时任务工具，轻量级的Quartz ，使用简单。支持配置文件和注解两种形式，下面将分别介绍这两种方式。 

## 一、基于xml配置

### 1. 第一种

这个例子是在公司的一个项目看到，这种所需的依赖较多和xml内容也多一点，推荐使用下面的基于配置文件的第二个例子，这里只是记录一下有这个用法。

- **pom.xml**

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>3.0.6.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>3.0.6.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>3.0.6.RELEASE</version>
</dependency>

<dependency>
    <groupId>commons-collections</groupId>
    <artifactId>commons-collections</artifactId>
    <version>3.2</version>
</dependency>

<dependency>
    <groupId>org.opensymphony.quartz</groupId>
    <artifactId>quartz-all</artifactId>
    <version>1.6.1</version>
</dependency>
```

- **spring.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="job" class="com.hzf.task.TimerTask"/>

    <bean id="timerTask" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
        <property name="targetObject">
            <ref local="job"/>
        </property>
        <property name="targetMethod" value="task"/>
    </bean>

    <bean id="taskCron" class="org.springframework.scheduling.quartz.CronTriggerBean">
        <property name="jobDetail" ref="timerTask"/>
        <property name="cronExpression" value="0/1 * * * * ? "/>
    </bean>

    <bean id="taskTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerBean">
        <property name="jobDetail" ref="timerTask"/>
        <property name="startDelay" value="10000"/>
        <property name="repeatInterval" value="10800000"/>
    </bean>

    <bean autowire="no"
          class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
            <list>
                <ref local="taskCron"/>
            </list>
        </property>
    </bean>
    
</beans>
```

- **TimerTask.java**

```java
package com.hzf.task.task;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * Created by zf.huang on 2018.9.6
 */
public class TimerTask {

    private SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    private int time = 0;

    public void task() {
        System.out.print(format.format(new Date()) + " ...... start task-" + ++time);
        System.out.println(" ...... end task-" + time);
    }

}
```



### 2 . 第二种

这个例子相对于，第一种来说，简洁很多。

- **pom.xml**

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context-support</artifactId>
	<version>3.0.6.RELEASE</version>
</dependency>
```

- **spring.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:task="http://www.springframework.org/schema/task"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/task
       http://www.springframework.org/schema/task/spring-task-3.0.xsd
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.hzf.task"/>

    <task:scheduled-tasks>
        <task:scheduled ref="timerTask" method="task" cron="0/1 * * * * ?"/>
    </task:scheduled-tasks>

</beans>
```

- **TimerTask.java**

比较上一个例子，这个类需要添加@Service。个人感觉就是配置文件和注解方式的结合。

```java
package com.hzf.task.task;

import org.springframework.stereotype.Service;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * Created by zf.huang on 2018.9.6
 */

@Service
public class TimerTask {

    private static SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    public void task() {
        System.out.print(format.format(new Date()) + " ...... start task-" + ++time);
        System.out.println(" ...... end task-" + time);
    }
   
}
```

## 二、基于注解

- **pom.xml**

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context-support</artifactId>
	<version>3.0.6.RELEASE</version>
</dependency>
```

- **spring.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:task="http://www.springframework.org/schema/task"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation=" http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/task
       http://www.springframework.org/schema/task/spring-task-3.0.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.hzf.task"/>

    <!--定时任务开关-->
    <task:annotation-driven/>

</beans>
```

- **TimerTask.java**

```java
package com.hzf.task.task;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * Created by zf.huang on 2018.9.6
 */

@Component
public class TimerTask {

    private SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    private int time = 0;

    @Scheduled(cron = "0/1 * * * * ?")
    public void task() {
        System.out.print(format.format(new Date()) + " ...... start task-" + ++time);
        System.out.println(" ...... end task-" + time);
    }
}
```



## 运行结果

```java
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Created by zf.huang on 2018.9.6
 */
public class App {

    public static void main(String[] args) {
        new ClassPathXmlApplicationContext("taskContext.xml");
    }
}
```

![1536212811470](/images/Spring/Spring-Task/task-result.png)