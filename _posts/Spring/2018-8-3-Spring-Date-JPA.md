---
layout: post
title: Spring Data JPA + MySQL / SQLserver
categories: Spring
description: JPA 
keywords: JPA, MySQL, SQLserver
---

本文主要介绍Spring Data JPA 的用法



## 说明
简单了解一下JPA
> **JPA : ** Java Persistence API的简称，Java持久层API，是一种ORM规范。

什么是Spring Data JPA，来看看官网的定义

> **Spring Data JPA ：**part of the larger Spring Data family, makes it easy to easily implement JPA based repositories. This module deals with enhanced support for JPA based data access layers. It makes it easier to build Spring-powered applications that use data access technologies.

简单来说就是Spring框架对JPA的整合,简化 CURL操作，有兴趣的可以看看  [Spring Data JPA 源码](https://github.com/spring-projects/spring-data-jpa)	



## 开始
##### 1. 在Idea中创建maven项目 
步骤......（此处省略很多字很多图）最终项目结构如图

![1536117297115](..\..\images\Spring\Spring-Data-JPA\jpa-1-1.png)



##### 2. 创建好项目之后，我们需要在pom.xml文件添加spring-data-jpa的依赖

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
  
    <groupId>com.hzf</groupId>
    <artifactId>jpa</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>

        <!-- 下面两个是必须的 -->
        <!-- Spring data jpa -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
            <version>2.0.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.3.3.Final</version>
        </dependency>

        <!-- 日志-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.2</version>
        </dependency>

        <!-- bean的get、set方法注解 -->
        <!-- 如果用到lombok，idea需要添加lombok插件才能使用 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.0</version>
            <scope>provided</scope>
        </dependency>

        <!-- mysql和sqlserver 二选一 -->
        <!-- mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.30</version>
        </dependency>

        <!-- sql server -->
        <dependency>
            <groupId>net.sourceforge.jtds</groupId>
            <artifactId>jtds</artifactId>
            <version>1.3.1</version>
        </dependency>
    </dependencies>
</project>
```



##### 3. 在entity包下，新建Persion.java  

```java
package com.hzf.jpa.entity;

import lombok.Data;

import javax.persistence.*;
import java.util.Date;

/**
 * Created by zf.huang on 2018.8.3
 */

@Data
@Entity
@Table(name = "tb_person")
public class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;

    @Column(name = "name")
    private String name;

    //length默认是255
    @Column(name = "gender", length = 2)
    private String gender;

    //updatable=false 字段不更新
    @Column(name = "birthday", updatable = false)
    private Date birthday;

    @Column(name = "remark" ,length = Integer.MAX_VALUE)
    private String remark;
}
```

- 简单解释一下上面用到的注解，其实还有许多，有兴趣的可以去查查
  - **@Data** lombok，这个可以省略get，set和toString方法，如果没有这个注解，就老老实实自动生成吧。
  - **@Entity**  说明这个class是实体类，并且使用默认的orm规则 
  - **@Table** 
    - *name：* 数据库对应的表名
  - **@Id** 设置该字段为主键，这没什么好讲的，但是如果是有联合主键的话，就不是在第二个字段再加个@Id这么简单了，具体的下次补充。
  - **@GeneratedValue** 
    - *AUTO：* 自动选择一个最适合底层数据库的主键生成策略。如MySQL会自动对应auto increment。这个是默认选项，可以直接这样写@GeneratedValue
    - *IDENTITY：* 表自增长字段，Oracle不支持这种方式。
    - *SEQUENCE：*通过序列产生主键，MySQL不支持这种方式。
    - *TABLE：*通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。 不同的JPA实现商生成的表名是不同的，如 OpenJPA生成openjpa_sequence_table表，Hibernate生成一个hibernate_sequences表， 而TopLink则生成sequence表。这些表都具有一个序列名和对应值两个字段，如SEQ_NAME和SEQ_COUNT。
    - 如果需要将主键设置成uuid，可以@GenericGenerator(name = "system-uuid", strategy = "uuid")
  - **@Column**
    - *name：* 数据库对应的字段名
    - *updatable：* 执行update时，如果是false，该就不会更新该字段，不写默认是true
    - *length：* 字段的长度，默认是255，如果不需要限制该字段，mysql的就直接Integer.MAX_VALUE。对于sqlserver的话，就不用设置length，直接在字段上添加**@Lob**，就是把该字段类型设置成text，相当于varchar(max)。



##### 4. 在dao包下，创建PersonDao.java，这一步是最简单的

```java
package com.hzf.jpa.dao;

import com.hzf.jpa.entity.Person;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.PagingAndSortingRepository;

import java.util.List;

/**
 * Created by zf.huang on 2018.8.3
 */

public interface PersonDao extends PagingAndSortingRepository<Person, Integer>, JpaSpecificationExecutor<Person> {

}
```



这样就可以，是不是很简单，什么都不用写，这个接口也不需要去实现里面的方法，直接就能使用下面这些方法。

![1536119806597](..\..\images\Spring\Spring-Data-JPA\jpa-4-1.png)



如果上面这些方法不能够满足你的需求时，你可以手写sql，如下所示

```java
public interface PersonDao extends PagingAndSortingRepository<Person, Integer>, JpaSpecificationExecutor<Person> {

    @Query(value = "from Person person where person.name = ?1")
    List<Person> findByName(String name);
    
}
```

当然，如果你需要联表查询的话，上面这肯定是不行的，所以还有一个高级一点的写法  (**Specification** ↓↓↓↓↓↓↓)



##### 5. 在service包下创建PersonService.java

```java
package com.hzf.jpa.service;

import com.hzf.jpa.dao.PersonDao;
import com.hzf.jpa.entity.Person;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Sort;

import javax.persistence.criteria.Predicate;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

/**
 * Created by zf.huang on 2018.8.3
 */
public class PersonService {
    
    @Autowired
    private PersonDao dao;

    public Iterable<Person> findAll() {
        return dao.findAll();
    }

    public Person find(int id) {
        Optional<Person> person = dao.findById(id);
        return person.orElse(null);
    }

    public boolean updateUser(Person... persons) {
        for (Person person : persons) {
            dao.save(person);
        }
        return true;
    }

    public boolean deleteUserById(int... ids) {
        for (int id : ids) {
            dao.deleteById(id);
        }
        return true;
    }

    //Specification 拼接查询语句
    public void find() {
        List<Person> personList = dao.findAll((root, criteriaQuery, criteriaBuilder) -> {
            List<Predicate> predicatesList = new ArrayList<>();
            predicatesList.add(criteriaBuilder.and(criteriaBuilder.equal(
                    root.get("name"), "小明")));
            predicatesList.add(criteriaBuilder.and(criteriaBuilder.like(
                    root.get("remark"), "%呵呵%")));
            return criteriaBuilder.and(predicatesList.toArray(new Predicate[0]));
        }, Sort.by(Sort.Direction.DESC, "gender"));
        personList.forEach(it -> System.out.println(it.toString()));
    }
}
```



##### 6. 配置文件

- applicationContext.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:tx="http://www.springframework.org/schema/tx"
         xmlns:jpa="http://www.springframework.org/schema/data/jpa"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
                             http://www.springframework.org/schema/beans/spring-beans.xsd
                             http://www.springframework.org/schema/context
                             http://www.springframework.org/schema/context/spring-context.xsd
                             http://www.springframework.org/schema/tx
                             http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
                             http://www.springframework.org/schema/data/jpa
                             http://www.springframework.org/schema/data/jpa/spring-jpa-1.0.xsd">
  
      <context:component-scan base-package="com.hzf.jpa"/>
  
       <!--加载config.properties配置文件，然后下面可以直接使用-->
      <context:property-placeholder location="classpath:config.properties"/>
  
      <!--sql server数据源配置-->
      <bean id="dataSource"
            class="org.springframework.jdbc.datasource.DriverManagerDataSource">
          <property name="driverClassName" value="${db.driverClass}"/>
          <property name="url" value="${db.url}"/>
          <property name="username" value="${db.username}"/>
          <property name="password" value="${db.password}"/>
      </bean>
  
      <tx:annotation-driven/>
      <bean id="personService" class="com.hzf.jpa.service.PersonService"/>
  
      <jpa:repositories base-package="com.hzf.jpa.dao" 
                        repository-impl-postfix="Impl"
                        entity-manager-factory-ref="entityManagerFactory" 
                        transaction-manager-ref="transactionManager"/>
  
      <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
          <property name="entityManagerFactory" ref="entityManagerFactory"/>
      </bean>
  
      <bean id="entityManagerFactory"
            class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
          <property name="dataSource" ref="dataSource"/>
          <property name="persistenceUnitName" value="persistenceUnit"/>
          <property name="jpaVendorAdapter">
              <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                  <property name="showSql" value="false"/>
              </bean>
          </property>
      </bean>
  
  </beans>
  ```

  

- META-INF\persistence.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
      <persistence-unit name="persistenceUnit" transaction-type="RESOURCE_LOCAL">
          <properties>
              <!--mysql-->
              <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
              <property name="hibernate.show_sql" value="false"/>
              <property name="hibernate.format_sql" value="true"/>
              <property name="hibernate.use_sql_comments" value="true"/>
              <property name="hibernate.jdbc.use_getGeneratedKeys" value="false" /> 
              <property name="hibernate.hbm2ddl.auto" value="update"/>
              <!-- 批量数据操作，累计到n条SQL后再批量提交 -->
              <property name="hibernate.jdbc.batch_size" value="10" /> 
          </properties>
      </persistence-unit>
  </persistence>
  ```

  

  - **hibernate.dialect**

    如果是SQLserver时，可换成：

    ```xml
    <property name="hibernate.dialect"  value="org.hibernate.dialect.SQLServerDialect"/>
    ```

    

  - **hibernate.show_sql** 

    false：不在显示sql语句 true：当执行数据库操作时，会在显示其对应的sql。

    **hibernate.format_sql**和**hibernate.use_sql_comments**，是格式化显示的sql，如果hibernate.show_sql为false，这两条可以省略

    

  - **hibernate.hbm2ddl.auto**

    - update   ： 第一次加载时会建表，之后只更新表结构，不建表 （一般都是用这个）
    - create  ：每次都会删除之前表结构，包括数据，然后重新建表（不要再生产环境上用）
    - create-drop  ：加载hibernate时创建，退出时删除表结构 
    - validate  ：加载hibernate时，验证创建数据库表结构，这样 spring在加载之初，如果model层和     数据库表结构不同，就会报错，这样有助于技术运维预先发现问题。 



- config.properties

  ```properties
  #数据库 二选一
  #MySQL
  db.driverClass=com.mysql.jdbc.Driver
  db.url=jdbc:mysql://localhost:3306/jpstest
  db.username=yourname
  db.password=yourpwd
  
  #SQL Server
  #db.driverClass=net.sourceforge.jtds.jdbc.Driver
  #db.url=jdbc:jtds:sqlserver://localhost:1433/jpatest
  #db.username=yourname
  #db.password=yourpwd
  
  #也可以在这添加其他config数据
  ```

  

- log4j.properties

  ```properties
  log4j.rootLogger=Info,stdout
  log4j.appender.stdout.Threshold=INFO
  log4j.appender.stdout=org.apache.log4j.ConsoleAppender
  log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
  log4j.appender.stdout.layout.ConversionPattern=[%p] %d{yyyy-MM-dd HH:mm:ss,SSS} [%l]%m%n
  
  log4j.logger.org=ERROR
  log4j.logger.org.hibernate=ERROR
  log4j.logger.com.mchange=ERROR
  
  #日志配置只是简单显示在控制台，具体的配置可自行查询
  ```

  

##### 7.代码已经写完了，建个App.java来测一下吧

```java
package com.hzf.jpa;

import com.hzf.jpa.service.PersonService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Created by zf.huang on 2018.8.3
 */
public class App {

    public static void main(String[] args) {
        ApplicationContext context = new
            ClassPathXmlApplicationContext("applicationContext.xml");
        PersonService dao = (PersonService) context.getBean("personService");
        dao.findAll().forEach(it -> System.out.println(it.toString()));
    }
}

```

> **提示：** 好像App.java文件一定要建在第一层包base-package下，在这个demo里，App.java要建com.hzf.jpa下面。



加载ApplicationContext还可以用绝对路径加载

```java
String applicationContext = System.getProperty("user.dir") +
						  File.separator + "src" +
						  File.separator + "main" +
						  File.separator + "resources" +
						  File.separator + "applicationContext.xml";
ApplicationContext context = new FileSystemXmlApplicationContext(applicationContext);
```



---------

MySQL 和 SQLserver  切换时需要修改的几个地方

- pom.xml 的依赖
- persistence.xml
- log4j.properties

---------



最后附上源码 [戳这里](https://github.com/FengHelloWorld/JavaDemo/tree/master/JpaDemo)	