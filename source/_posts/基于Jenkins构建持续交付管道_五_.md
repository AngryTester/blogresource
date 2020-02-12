---
title: 基于Jenkins构建持续交付管道（五）
date: 2017-03-09 17:06:05
tags: [Jenkins,Pipeline,Docker,持续交付]
---

## 前言

上一章中介绍了目前基于Jenkins的Pipeline As Code，解决了之前基于Build Pipeline的一系列问题，本章中我们将介绍引入docker来优化我们的持续交付管道。

## Docker是什么

简单来说，Docker是区别于传统虚拟化的另外一种虚拟方式，更轻便。

具体可以参考：https://yeasy.gitbooks.io/docker_practice/content/introduction/what.html

## Docker可以解决什么问题

之前有提到过Jenkins完成构建任务主要是通过插件，插件则依赖持续集成服务器上的工具。比如如果我需要完成一个Maven工程的编译打包，首先得需要持续集成服务器上得有Maven工具，才能在管道中调用服务器里的Maven工具完成打包。这样带来的直接问题就是：不同构建任务如果需要不同构建工具，则需要在服务器上配置各种构建工具，这对于持续集成服务器的维护工作是个挑战。

<!-- more -->
除了持续集成服务器的维护工作，其实Docker技术的初衷是解决环境一致性的问题。众所周知，生产环境和测试环境或者开发环境的一致性，一直是交付团队苦恼的问题，特别在一些底层环境非常复杂的系统中尤为明显。Docker可以实现将应用包和底层依赖环境打包成一个完成镜像，这个镜像在各种环境下启动的表现完全一致，前提是各种环境需要支持Docker。这里所说的依赖包括JDK、中间件等等。

## Docker在Jenkins持续交付管道中的应用

管道代码片段:

```
docker.image('registry.cae.com/angrytester/maven:3-jdk-7').inside('-v /root/.m2/repository:/root/.m2/repository){
	stage 'Build'
	sh "mvn clean install -Dmaven.test.skip=true"	  
}
```

以上代码实现了一个stage，名为Build。以下是具体执行逻辑：

>首先从镜像库中根据镜像tag=hub.com/angrytester/maven:3-jdk-7拉一个镜像到持续集成服务器并启动一个容器，将容器里的root/.m2/repository目录挂载到持续集成服务器的root/.m2/repository目录。前面为持续集成服务器的路径，后面为容器内的路径。
	
>然后在容器内执行shell命令： mvn clean install -Dmaven.test.skip=true

>从以上就可以看出来，不需要构建服务器上有Maven工具，如果需要执行Maven构建，只需要拉一个Maven镜像并起一个容器，然后在容器内就可以直接执行Maven的相关构建命令了。


## Docker in Docker

在持续交付管道中引入Docker之后，由于后续涉及打镜像，所以肯定需要用到Docker的相关服务，即需要在Docker容器里再起Docker服务，这就涉及到Docker in Docker的知识了。

官方其实是支持Docker in Docker的，具体可参考：https://hub.docker.com/_/docker/

即需要先起一个Docker容器，然后再起另外一个容器link到这个容器，然后在新起的这个容器里就可以执行Docker相关命令了。

下面的脚本示例片段：

```
docker.image('docker:1.13-dind').run("--privileged --name docker-dind")
docker.image('docker:1.13').withRun("--link docker-dind:docker"){
      sh "docker images"
    }
```

需要注意，在启动dind容器时，必须使用特权即privileged模式。

## 引入Docker之后持续交付管道的一些变化

- 构建环境自定制而非依赖构建服务器

- 构建产物为镜像而非应用包

- 应用通过容器启动而非应用服务器启动











