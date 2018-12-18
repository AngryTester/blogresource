---
title: 管道中通过shell验证测试环境是否可用
date: 2017-03-14 17:50:19
tags: [Pipeline,Shell]
---

## 坑

因为要将传统的持续交付管道改造成基于Docker的，所以需要切换原有的环境部署方式，相应的环境验证方式也需要更改。之前的方式Jenkins会把部署脚本和环境验证脚本当成一个shell脚本执行，做了切换之后，需要将之前的环境验证脚本改成执行通过shell执行，即只需要用linux环境就可以运行。由于对linux的shell不熟，才遇到这样一个大坑，特此记录。


参考脚本如下:

```
sh " while [ `curl -o /dev/null -s -w %{http_code} http://xxx:xxx/test.html` != 200 ];do sleep 1;done;"
```

## 注意点

while里面的判断左右的中括号都需要有一个空格，否则判断语句无效，因为这个问题纠结一下午...