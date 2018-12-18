---
title: 低侵入配置Gradle项目获取单元测试结果
date: 2018-12-17 18:47:41
tags: [Gradle]
---

## 写在前面

目前公司CI采用的是基于`Drone CI`的方案，Pipeline的各个Job主要依靠各种Docker镜像支撑。例如拉取代码使用的是官方的插件镜像，编译代码采用在官方镜像基础上的定制镜像等等。随着集团对于代码质量的要求越来越高，对于早该纳入持续集成体系的单元测试工作终于提上了议程。

### 工作目标

第一步的工作目标是在`尽量少侵入各产品线当前正常研发工作的前提下`将单元测试的Job集成进当前的持续集成体系中，并能成功收集当前所有后端系统的单元测试现状，后续再根据现状制定相关指标提升计划。

### 工作难点

公司后端系统大都采用的Java，构建工具统一使用Gradle。市场上对于配置Gradle获取单元测试相关结果的方案已经很成熟了，因此配置并不是这项工作的难点。难点在于工作前提-`尽量少侵入各产品线当前正常研发工作的前提下`。现有方案大都需要修改项目的`build.gradle`文件以添加各种插件依赖和task，虽然知道其中原理的我们清楚这并不会对项目自身产生任何影响，但是无奈要求来自于高层，作为严格遵循BDD(Boss Driven Development)的我们来说，必须考虑实现。

## 工作思路

<!--more-->

### 主体方案

方案仍然选择通用的Jacoco+SonarQube，其中Jacoco用于收集单元测试的代码覆盖率，SonarQube用于展现单元测试相关结果，包括单元测试个数，成功率，覆盖率等。

### 核心思路

不对项目代码有任何侵入，唯一可入手的地方就是持续集成编排文件(.drone.yml)和CI中用到的各种镜像了。考虑易用性，最佳入手点当然就是镜像了。理想效果是在单元测试Job中引用我提供的镜像，执行一行命令即可完成单元测试执行和单元测试结果收集的工作。

镜像需要完成的工作包括处理Jacoco和SonarQube的Task所需要的依赖以及Task的相关配置，有一种比较笨的方案是在Job执行到单元测试之后，通过封装的shell脚本修改项目的`build.gradle`文件，手动加入相关依赖和Task配置，执行结束之后再恢复。这种方案的处理相对复杂，且如果操作不当，很有可能影响到后续Job的执行。因此我们采取了另外一种方案，通过父子项目传递依赖。通过创建一个虚拟的父项目，在父项目的配置文件中加入相关依赖和Task配置，然后将相关配置传递到作为子项目的真正项目。

### 核心操作

我们以`gradle`的镜像作为基底镜像，创建了一个虚拟的gradle项目-cov，这个项目只包含两个文件：`build.gradle`和`settings.gradle`。

`build.gradle`的内容如下：

```xml
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:2.6.2"
    }
}
subprojects{

  apply plugin: 'java'
  apply plugin: 'jacoco'
  apply plugin: 'org.sonarqube'

  jacoco {
    toolVersion = "0.7.9"
  }

  jacocoTestReport {
      reports {
          xml.enabled false
          html.enabled true
      }
  }

  check.dependsOn jacocoTestReport
}
```

`settings.gradle`的内容如下：

```xml
rootProject.name = 'cov'
include 'unit-test-sample'
```

然后再封装shell脚本实现自动执行单元测试和收集单元测试结果。

shell脚本内容如下：

```shell
#!/bin/bash

# 退回上一级目录方便做mv
cd ..

# 将当前项目移动至/cov
mkdir -p /cov/${DRONE_REPO_NAME}
cp -r ${DRONE_REPO_NAME}/* /cov/${DRONE_REPO_NAME}

# 替换处理cov下settings.gradle
sed -i "s/unit-test-sample/${DRONE_REPO_NAME}/g" /cov/settings.gradle

sonar_url=http://localhost:9000/sonar
projectName=${DRONE_REPO_NAME}
projectKey=${DRONE_REPO//\//\:}

# cd到cov目录做后续操作
cd /cov

# 单元测试失败暂时不做拦截，单独执行
gradle test || true

# 单元测试执行单独执行sonar，否则由于sonar依赖test，test失败无法执行
gradle sonarqube -x test -Dsonar.host.url=$sonar_url -Dsonar.projectKey=$projectKey -Dsonar.projectName=$projectName

#获取componentId
curl -s $sonar_url/api/components/show?key=$projectKey > component.json
componentId=$(jq .component.id component.json | sed 's/\"//g')

#等待当前项目无in progress任务
while [[ $QUEUE != [] ]]
do
	curl -s $sonar_url/api/ce/component?componentId=$componentId > task.json
	QUEUE=$(jq .queue task.json)
	echo "等待Sonar解析任务完成..."
done

# 保存覆盖率数据
curl -i -H 'Content-type:application/json' -X POST -d "{\"drone_repo\":\"${DRONE_REPO}\",\"drone_commit\":\"${DRONE_COMMIT}\",\"drone_tag\":\"${DRONE_TAG}\",\"component_key\":\"$projectKey\"}" http://localhost:8080/codequalitydata

#获取qualitygateStatus
curl -s $sonar_url/api/qualitygates/project_status?projectKey=$projectKey > qualitygate.json
qualitygateStatus=$(jq .projectStatus.status qualitygate.json | sed 's/\"//g')

#判断如果qualitygateStatus不为OK，则终止
if [[ $qualitygateStatus != OK ]]
then
	echo "${projectName}单元测试不达标，请查看Sonar结果详情！$sonar_url/dashboard?id=$projectKey"
	# 暂时不拦截，直接通过，拦截则退出码改成1
	exit 0
else
	echo "质量门通过"
fi

```

这个脚本主要完成了以下几项工作：

- 将当前项目复制到我们创建的虚拟父项目cov下。（不能剪切，这里与drone的执行机制有关，所有Job操作的是同一个工作空间，如果剪切则下一个Job将访问不到任何项目内容）

- 替换父项目中的`settings.gradle`文件中的include的内容，将当前项目设置为子项目。

- 通过drone内置的环境变量设置`projectKey`和`projectName`，其中`projectKey`采用包含group的仓库完整路径，斜杠用冒号替换。

- 在虚拟父项目cov下执行单元测试，由于第一阶段目标不对CI流程做阻断，添加设置`|| true`继续永远往下执行。

- 执行完单元测试后执行`sonarqube`，要注意`sonarqube`默认依赖`test`，因此需要通过`-x test`排除，并将上面设置的`projectKey`和`projectName`传入命令行。

- 由于sonar解析单元测试结果需要一定时间，为了获取准确结果，通过等待不存在解析中任务来判断sonar执行完成，再获取质量门状态。

- 由于sonar上不会按照CI的执行保存每次的单元测试结果，为了满足这一需求，我们自定义了一个外部存储接口，在sonar执行完成后自动收集单元测试相关结果入数据库。


### 镜像制作

镜像制作相对简单，基底镜像采用`gradle`,将虚拟父项目`cov`和shell脚本加入镜像中。

`Dockerfile`内容参考如下：

```
FROM gradle:4.7

MAINTAINER AngryTester <thx_phila@yahoo.com>

USER root

COPY check.sh /

COPY jq /

WORKDIR /

RUN mv check.sh /usr/local/bin/check \
&& mv jq /usr/local/bin/jq \
&& chmod +x /usr/local/bin/check \
&& chmod +x /usr/local/bin/jq

COPY cov /cov
```

***因为shell脚本中用到jq工具，因此还需要将jq工具加入镜像中。***

### 使用方式

拉取镜像并执行封装的shell即可。

## 写在后面

该方案虽然解决了侵入性的问题，同时也带来局限性，例如对于自身为多模块的项目无法级联传递。仅供参考。