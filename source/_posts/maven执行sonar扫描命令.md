---
title: Maven执行Sonar扫描命令
date: 2017-11-02 14:57:24
tags: [Maven，Sonar]
---

需要通过jacoco统计单元测试覆盖率，不需要对项目做配置，直接运行如下命令即可：

```shell
mvn clean org.jacoco:jacoco-maven-plugin:0.7.4.201502262128:prepare-agent  install sonar:sonar -Dmaven.test.failure.ignore=true -Dsonar.host.url=http://localhost:9000
```
