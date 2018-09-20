---
layout: post
title: Hadoop（二）—— java连接Hadoop，操作HDFS文件
categories: Hadoop
description: java操作hdfs文件
keywords: Hadoop, java，hdfs
---

![img](/images/Hadoop/hadoop.jpg) 

## 一、HDFS文件系统

**HDFS** 全称是Hadoop Distributed System。HDFS是为以流的方式存取大文件而设计的。适用于几百MB，GB以及TB，并写一次读多次的场合。而对于低延时数据访问、大量小文件、同时写和任意的文件修改，则并不是十分适合。

本章节不详细介绍，我认识的也不全面，以后有时间再缩，这里主要讲解HDFS的Java API，对HDFS文件系统进行常用的操作。

**更多HDFS的操作请查看 [HDFS JAVA API](https://hadoop.apache.org/docs/r2.7.4/api/index.html)**

## 二、Java操作HDFS

编写HDFSUtil工具类，直接上代码

### 1. pom.xml

```xml
<properties>
    <hadoop.version>2.7.4</hadoop.version>
</properties>

<dependencies>
    
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>${hadoop.version}</version>
    </dependency>
    
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>${hadoop.version}</version>
    </dependency>
    
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>${hadoop.version}</version>
    </dependency>
    
</dependencies>
```

### 2. HDFSUtil.java

> **提示：** 以下所有方法，只要是HDFS文件路径的，都只能使用`/` ，不能使用`\\`，并且需要在路径开头要带`/`，例如`String hdfs= "/path/filename.txt"`。

#### ① 实例化FileSystem

利用FileSystem API 操作HDFS文件系统，所以首先需要获取FileSystem。

```java
private static FileSystem getFileSystem() throws IOException {
	String url = "hdfs://ip:9000";
	Configuration config = new Configuration();
	config.set(FileSystem.FS_DEFAULT_NAME_KEY, url);
	return FileSystem.get(config);
}
```

`url` 是core-site.xml 中 fs.defaultFS所配置的值。

#### ② 获取当前工作目录 

`getHomeDirectory()` 和`getWorkingDirectory()`都能够获取到当前工作目录。

```java
public static Path getHomeDir() throws IOException {
    try (FileSystem fs = getFileSystem()) {
        System.out.println(fs.getHomeDirectory());
        //System.out.println(fs.getWorkingDirectory());
    }
}
```

两个方法都会输出：`hdfs://ip:9000/user/username`

username就是主机名，要是想指定名字，可以在获取前，添加下面这句代码。

```java
System.setProperty("HADOOP_USER_NAME", "feng");
```

一般可以加在`main`函数前面或者加在实例化FileSystem的方法`getFileSystem()`里。

则会输出`hdfs://ip:9000/user/feng`

#### ③ 创建文件夹

```java
public static boolean createDir(String newDir) throws IOException {
    try (FileSystem fs = getFileSystem()) {
        Path path1 = new Path(fs.getHomeDirectory().toString() + newDir);
        //Path path2 = new Path(fs.getHomeDirectory().toString(), newDir);
        return fs.mkdirs(path1);
    }
}
```

上面方法中`path1` 和 `path2` 生成路径的区别：

- **new Path(fs.getHomeDirectory().toString() + newDir)  => **  hdfs://ip:9000/user/username/newDir
- **new Path(fs.getHomeDirectory().toString(), newDir) => ** hdfs://ip:9000/newDir

在接下来的方法里，我都会用第一种方法来组合hdfs路径。

#### ④ 上传文件

上传文件到HDFS有三种写法

```java
//第一种
public static void uploadFile2hdfs1(String localFilePath, String hdfsPath) 
    throws IOException {
    try (FileSystem fs = getFileSystem()) {
        Path local = new Path(localFilePath);
        Path hdfs = new Path(fs.getHomeDirectory().toString() + hdfsPath);  
        fs.copyFromLocalFile(local, hdfs);
    }
    //也可以直接用下面方法中的其中一种
    //uploadFile2hdfs(new FileInputStream(localFilePath), hdfsPath);
    //uploadFile2hdfs2(new FileInputStream(localFilePath), hdfsPath);
}

//第二种
public static void uploadFile2hdfs2(InputStream localFileStream, String hdfsPath) 
    throws IOException {
    FSDataOutputStream FSos = null;
    try (FileSystem fs = getFileSystem()) {
        byte[] content = new byte[localFileStream.available()];
        int len = localFileStream.read(content);
        Path path = new Path(fs.getHomeDirectory() + hdfsPath);
        FSos = fs.create(path);
        FSos.write(content, 0, len);
    } finally {
        if (FSos != null) FSos.close();
        if (localFileStream != null) localFileStream.close();
    }
}

//第三种
public static void uploadFile2hdfs3(InputStream localFileStream, String hdfsPath) 
    throws IOException {
    FSDataOutputStream FSos = null;
    try (FileSystem fs = getFileSystem()) {
        Path path = new Path(fs.getHomeDirectory() + hdfsPath);
        FSos = fs.create(path);
        IOUtils.copyBytes(localFileStream, FSos, 1024 * 2, true);
    } finally {
        if (FSos != null) FSos.close();
        if (localFileStream != null) localFileStream.close();
    }
}
```

还有一种创建文件的情况，将一串字符串保存到HDFS上，其实原理跟上面第二种和第三种上传文件利用流的方法上传一样。

```java
public static void createHdfsFile(String context, String hdfsPath) 
    throws IOException {
    try (FileSystem fs = getFileSystem()) {
        FSDataOutputStream FSos = null;
        try {
            byte[] content = context.getBytes();
            Path path = new Path(fs.getHomeDirectory() + hdfsPath);
            FSos = fs.create(path);
            FSos.write(content, 0, content.length);
        } finally {
            if (FSos != null) FSos.close();
        }
    }
}
```

#### ⑤ 下载文件

下载文件到本地，直接调用FileSystem的API`copyToLocalFile`

```java
public static void downloadHdfsFile2local(String hdfsFilePath, String localFilePath) 
    throws IOException {
    try (FileSystem fs = getFileSystem()) {
        Path hdfs = new Path(fs.getHomeDirectory() + hdfsFilePath);
        Path local = new Path(localFilePath);
        fs.copyToLocalFile(hdfs, local);
    }
}
```

如果只是想要获取HDFS文件的流或者文件的内容，可以使用下面方法

```java
//获取HDFS文件的ByteArrayOutputStream
public static ByteArrayOutputStream getHDFSFileStream(String hdfsFilePath) 
    throws IOException {
    FSDataInputStream in = null;
    try (FileSystem fs = getFileSystem()) {
        Path path = new Path(fs.getHomeDirectory() + hdfsFilePath);
        in = fs.open(path);
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        IOUtils.copyBytes(in, bos, 1024 * 1024, false);
        //in需要在fs前close(),不能再finally里关
        in.close();
        return bos;
     }
 }

//获取HDFS文件的内容，主要用于获取文件内容是字符串的
public static String getHDFSFileContent1(String hdfsFilePath) 
    throws IOException {
    try (ByteArrayOutputStream bos = getHDFSFileStream(hdfsFilePath)) {
        return bos.toString("UTF-8");
    }
}
```

#### ⑥ 删除文件和文件夹

该方法可以用来删除文件或者是文件夹

```java
public static void deleteHdfsFile(String hdfsPath) throws IOException {
	try (FileSystem fs = getFileSystem()) {
        Path path = new Path(fs.getHomeDirectory() + hdfsPath);
        if (fs.exists(path)) {
            fs.delete(path);
            System.out.println(path + "del succ!");
        } else {
            System.out.println(path + ": the file/dir isn't exists");
		}
    }
}
```

#### ⑦ 遍历HDFS文件目录

```java
//hdfsPath => 要遍历的路径
//isRecursion => 是否递归遍历  
public static LinkedList<String> listFile(String hdfsPath, boolean isRecursion)
        throws IOException {
    try (FileSystem fs = getFileSystem()) {
        LinkedList<String> fileNameList = new LinkedList<>();
        getAllFile(fs, fileNameList, new Path(fs.getHomeDirectory() + hdfsPath), isRecursion);
        return fileNameList;
    }
}

private static void getAllFile(FileSystem fs, LinkedList<String> fileNameList, Path hdfsPath, boolean isRecursion) throws IOException {
    FileStatus[] fList = fs.listStatus(hdfsPath);
    for (FileStatus f : fList) {
        if (null != f) {
            if (f.isDirectory() && isRecursion) {
                getAllFile(fs, fileNameList, f.getPath(), isRecursion);
            } else {
                fileNameList.add(f.getPath().toString());
            }
        }
    }
}
```

#### ⑧ 检查文件是否存在

```java
public static boolean isExist(String hdfsPath) throws IOException {
    try (FileSystem fs = getFileSystem()) {
        Path hdfs = new Path(fs.getHomeDirectory() + hdfsPath);
        return fs.exists(hdfs);
    }
}
```

## 三、异常处理

记录在使用过程中出现过的异常，已经解决方法

