---
title: 通过noVNC远程监控自动化测试执行过程
date: 2019-08-16 11:52:26
tags: [自动化测试]
---

## why

自动化测试追求的是无人值守，因此执行过程的监控就是鸡肋，但是在遇到以下场景时情况可能就有所不同：

- 除了实时日志外希望输出更丰富的执行过程内容

- 需要给领导汇报自动化测试工作

其中第二点尤为关键，对于不熟悉自动化测试的领导来说，可视化的执行视图对于领导来说会显得更友好且直观。

## 难点

对于Chrome和Firefox这类可运行在Linux环境下的前端自动化测试来说，已有前人种树，直接乘凉即可。可直接使用[docker-selenium](https://github.com/elgalu/docker-selenium)，因此难点主要在于只能运行于Windows环境下的IE浏览器。

## 方案

整体方案包括`Selenium Grid+UltraVNC+noVNC+websockify`。操作步骤如下：

- Selenium Grid的环境配置无需赘述。

- 安装UltraVNC。

- 安装NodeJS。

- 安装相关依赖
```
npm install ws
npm install optimist
npm install @novnc/novnc
```

- 下载[noVNC](http://github.com/kanaka/noVNC/zipball/master),解压至`node_modules`文件夹。

- 下载[Wesockify](https://github.com/novnc/websockify-js/archive/master.zip),解压至`node_modules\noVNC`文件夹。

- 配置UltraVNC，去掉密码要求。进入UltraVNC安装目录找到配置文件，在配置文件中增加配置`AuthRequired=0`，然后将UltraVNC密码设置为空。

- 修改`vnc.html`，在body最后增加如下内容：
```
 <script>
       window.onload=function(){
           var timer = setTimeout(() => {
            clearTimeout(timer)
            document.getElementById("noVNC_connect_button").click();
           }, 500);
       }
 </script>
```

- 运行如下命令:
```
node C:\Users\Administrator\node_modules\noVNC\websockify-js-master\websockify\websockify.js --web C:\Users\Administrator\node_modules\noVNC 9000 localhost:5900
```

## 效果

通过http://服务器IP:9000即可浏览器访问服务器实时页面效果。
