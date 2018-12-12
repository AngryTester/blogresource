---
title: RobotFramework接口测试设置代理
date: 2018-12-12 19:08:10
tags: [RobotFramework,自动化,代理]
---

## 使用场景

使用RobotFramework+Selenium2Library进行UI自动化测试时，启动的浏览器实例会直接使用在IE中设置的代理，但是接口测试时调用的`robotframework-requests`不会直接调用浏览器代理，因此需要在测试脚本中设置。

## 设置方法

脚本片段如下:


| ${proxies} | Create Dictionary | http=http://172.26.1.1:8080 | https=http://172.26.1.1:8080 |
| ------ | ------ | ------ | ------ |
| Create Session | api | ${domain} | cookies=${cookies} | proxies=${proxies}  |

## 代理需认证的处理
<!-- more -->
遇到代理服务器需要认证的情况，需要在代理地址中加上用户名密码，例如用户名为`user`,密码为`pass`，则代理地址为：`http://user:pass@172.26.1.1:8080`。

### 特殊情况：用户名密码中包含特殊字符

例如密码包含`!@#`等特殊时，则需要将特殊字符转化为ASCII码，对应表为：

```
~ : 0x7E,         ! : 0x21    
@ : 0x40,         # : 0x23  
$ : 0x24,         % : 0x25  
^ : 0x5E,         & : 0x26  
* : 0x2A,         ? : 0x3F   
```
若密码为`pass！@#`,则输入时应该调整为`pass%21%40%23`,即`0x`用`%`代替。