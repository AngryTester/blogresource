---
title: 使用maven原型快速搭建自动化测试项目框架
date: 2017-11-09 14:58:16
tags: [maven,archetype]
---

## 使用场景

在搭建自动化测试项目框架时，需要对一些配置文件进行特殊配置，例如特殊依赖，配置文件中的配置值，以前这种工作只能通过人工完成．依赖maven原型，可以快速搭建起全配置的自动化测试项目框架．

## 如何生成原型

- 新建maven工程，将自动化测试项目需要的配置都准备好．

- 进行上面新建的工程目录，执行如何命令：

```shell
mvn archetype:create-from-project
```

<!-- more -->
就会在工程目录下的target/generated-sources/archetype生成原型文件．

-　修改上一步生成的原型文件中的`pom.xml`，可修改原型名称，加入私服地址等，然后执行如下命令将原型工程打包上传至私服：

```shell
mvn deploy -Dmaven.test.skip=true
```

原型就可以提供给私服的用户使用了．

##　如何通过原型生成自动化测试项目框架

### 通过eclipse客户端

- 首先新建一个Maven项目

- 不勾选创建简单项目点击下一步

- 点击Add Archetype(如果原型是SNAPSHOT版本，还得选择包含SNAPSHOT原型)

- 输入原型的groupId,artifactId,Version以及对应私服地址，这些信息都可以在生成原型的`pom.xml`中找到．

### 通过命令行

```shell
mvn archetype:generate -DarchetypeGroupId=com.angrytest -DarchetypeArtifactId=speedy-archetype -DremoteRepositories=http://localhost:8080/content/repositories/snapshots/ -DarchetypeVersion=0.0.1-SNAPSHOT -DgroupId=com.angrytest -DartifactId=demo -DinteractiveMode=false
```