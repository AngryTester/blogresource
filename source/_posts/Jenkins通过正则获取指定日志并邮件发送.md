---
title: Jenkins通过正则获取指定日志并邮件发送
date: 2017-02-28 10:31:39
tags: [Jenkins]
---

## 使用场景

Jenkins没有数据库，因此构建产出的内容都是放在文件中，如果要获取构建日志中的指定内容会非常麻烦。例如有一些用于造测试数据的自动化工具，需要从构建日志中获取到指定的测试数据，则需要借助正则解析构建日志。

## Email-ext plugin

Email-ext plugin插件提供了通过正则解析构建日志，提取指定构建日志内容的功能。$BUILD_LOG_MULTILINE_REGEX

## 使用实例

在邮件content中配置如下内容：

${BUILD_LOG_MULTILINE_REGEX,regex="测试数据：.*",maxMatches=0,showTruncatedLines=false,escapeHtml=false,matchedSegmentHtmlStyle="color:blue"}

<!-- more -->
则可以获取到以“测试数据开头”的内容，并通过邮件发送。





