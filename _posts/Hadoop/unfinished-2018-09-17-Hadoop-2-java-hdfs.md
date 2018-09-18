layout: post
title: Hadoop（二）—— java连接Hadoop，操作HDFS文件
categories: Hadoop
description: java操作hdfs文件
keywords: Hadoop, java，hdfs

![img](E:\FengWorkSpace\Blog\FengHelloWorld.github.io\images\Hadoop\hadoop.jpg) 

## 一、HDFS文件系统

**HDFS** 全称是Hadoop Distributed System。HDFS是为以流的方式存取大文件而设计的。适用于几百MB，GB以及TB，并写一次读多次的场合。而对于低延时数据访问、大量小文件、同时写和任意的文件修改，则并不是十分适合。



## 二、Java操作HDFS



### 1. 实例化FileSystem

利用FileSystem API 操作HDFS文件系统，所以首先需要获取FileSystem

```java
private static FileSystem getFileSystem() throws IOException {
	String url = "hdfs://master:9000";
	Configuration config = new Configuration();
	config.set(FileSystem.FS_DEFAULT_NAME_KEY, url);
	return FileSystem.get(config);
}
```

### 2. 获取当前工作目录 























**更多HDFS的操作请查看 [HDFS JAVA API](https://hadoop.apache.org/docs/r2.7.4/api/index.html)**



