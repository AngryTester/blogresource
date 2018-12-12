---
title: shell判断sonar的质量门结果
date: 2017-10-12 15:08:03
tags: [sonar,quliaty gate]
---

## 使用场景

Jenkins虽然有质量门插件,但是存在的问题是通过命令行触发静态扫描完成后,会将扫描结果发送至Sonarqube进行解析,解析需要一定时间,因此如果在解析未完成的时候获取质量门结果,有可能获取到上一次的扫描结果.

## 脚本内容

```
#!/bin/bash

#默认后面跟的第一个参数就是sonar服务地址,第二个参数为项目代码
 sonar_url=$1
 CI_PROJECT_NAME=$2

#执行sonar
<!-- more -->
mvn sonar:sonar -Dsonar.host.url=$sonar_url

#获取projectKey

projectKey=com.travelsky.$(echo $CI_PROJECT_NAME | tr '[A-Z]' '[a-z]'):$(echo $CI_PROJECT_NAME | tr '[A-Z]' '[a-z]')

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

#获取qualitygateStatus
curl -s $sonar_url/api/qualitygates/project_status?projectKey=$projectKey > qualitygate.json

qualitygateStatus=$(jq .projectStatus.status qualitygate.json | sed 's/\"//g')

#判断如果qualitygateStatus不为OK，则终止
if [[ $qualitygateStatus != OK ]]
then
	echo "质量门不通过，请查看Sonar结果详情！$sonar_url/dashboard?id=$projectKey"
	exit 1
else
	echo "质量门通过"
fi
```

## 使用方法和注意事项

### 使用方法

```
chkgate http://localhost:9000 Demo
```
### 注意事项

- 1.不同版本sonar可能api有所区别,需要做调整.本脚本基于sonarqube6.4版本.

- 2.需要用到jq工具解析json串.需要安装.