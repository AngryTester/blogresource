---
title: TCPClient模式输出精准测试覆盖率
date: 2019-07-29 17:05:15
tags: [jacoco,精准测试]
---

## 1. JaCoCo agent运行数据的4种输出模式

参考JaCoCo官方文档：

> The JaCoCo agent collects execution information and dumps it on request or when the JVM exits. There are three different modes for execution data output:
<br>
> File System: At JVM termination execution data is written to a local file.
TCP Socket Server: External tools can connect to the JVM and retrieve execution data over the socket connection. Optional execution data reset and execution data dump on VM exit is possible.
TCP Socket Client: At startup the JaCoCo agent connects to a given TCP endpoint. Execution data is written to the socket connection on request. Optional execution data reset and execution data dump on VM exit is possible.

一共有三种模式可以输出运行数据，分别是：
- 文件模式：JVM终止时会将运行数据写入一个本地文件。
- TCP Socket Server模式：可以通过外部工具Socket连接到JVM获取实时运行数据。JVM终止时运行数据重置并导出。
- TCP Socket Client模式：启动时agent会连接到提供的一个TCP终端。运行数据会持续往这个socket连接写入。JVM终止时运行数据重置并导出。

## 2. 各种模式的使用场景

三种模式并无本质区别，可根据实际使用场景选择。

比较常见的是使用TCPServer模式，在JVM启动命令中增加如下配置：

```
-javaagent:/opt/pmo/jacocoagent.jar=includes=*,output=tcpserver,port=2014,address=*
```

即在JVM启动同时在`2014`端口启动另外一个TCPServer，然后使用官方的cli工具dump即可将运行数据导出到本地文件。

以上模式在传统部署模式下比较方便，但是在容器化部署环境中，由于TCPServer需要单独占用一个端口，对于一些实用`ingress`的方案中，只能通过域名方式将端口映射到宿主机上，但是dump工具不支持连接ingress暴露的域名，因此这种方式不适合，因此需要考虑采用其他模式。

文件模式会将文件导出在JVM所在服务器，即处于容器内部，这种显然也不符合实际需求，因此考虑采用TCPClient模式。

## 3. TCPClient模式配置

### 3.1 启动[TCPServer](https://www.jacoco.org/jacoco/trunk/doc/examples/java/ExecutionDataServer.java):

核心代码片段如下：
```
        // 导出文件绝对路径
	private static final String DESTFILE = "jacoco-server.exec";
        // 启动的服务器IP，注意实际使用中修改为IP，不要用localhost
	private static final String ADDRESS = "localhost";
        // 启动的服务器端口
	private static final int PORT = 6300;
```

### 3.2 增加启动配置

```
-javaagent:/opt/pmo/jacocoagent.jar=includes=*,output=tcpclient,port=6300,address=10.25.74.82
```

在JVM停止时就会自动将运行数据导出到第一步中指定的路径。
