---
title: Jenkins环境部署结束自动验证
date: 2017-02-28 10:49:49
tags: [Shell]
---

## 遇到问题

前面有讲到通过Jenkins完成环境部署的步骤，在实际使用过程中遇到一个问题，因为在war包传输结束后执行完shell，部分容器可能启动时间需要较长时间，可能造成shell执行结束，直接触发了构建成功的邮件，测试人员收到邮件之后却发现环境还没有起来，对我们造成一定困扰。

## 解决方案

在shell最后加以下这段代码：

```shell
STATUS_CODE=0
echo $STATUS_CODE
while [[ $STATUS_CODE != 200 ]]
do
    STATUS_CODE=`curl -o /dev/null -s -w %{http_code} http://localhost:8080/test.html`
done
```
<!-- more -->
其中http://localhost:8080/test.html即为你需要验证的测试环境地址，这样一直到环境访问返回200，否则shell会一直执行等待。