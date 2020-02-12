---
title: 我所经历的以及我所理解的敏捷-七-Backlogs
date: 2019-06-03 08:40:04
tags: [敏捷]
---

## 前言

`Backlog`,中文通常译作`待办列表`，在Scrum中，又细分为`Product Backlog`(产品待办列表)和`Sprint Backlog`(迭代代办列表)。`Sprint Backlog`通常作为`Product Backlog`的子集。

## User Stroy

产品待办列表和迭代待办列表中的待办项在Scrum中称呼为`User Stroy`(用户故事)，用户故事的格式通常如下：

`As A`...,`I want to`...,`so that`...，作为一个`角色`，我想要`功能`，以便于`商业价值`。

用户故事的描述的格式体现了从用户价值出发，从非技术的角度描述了故事最终体现的商业价值。

## Product Backlog的基本特征

`Product Backlog`可以理解为产品想做的所有事。具体表现形式为一组排序的features列表，可能在敏捷实施过程中动态调整（包括调整优先级或者增删修改），且永远不会完成（若`Product Backlog`为空了则代表产品生命周期已完结）

其中`排序`作为`Product Backlog`的重要特征，可以根据价值高低进行调整。`Product Owner`承担了创建待办项和排序，澄清的职责。

待办列表内容描述应该尽量从非技术性的角度出发，且应该保证`Product Backlog`中的各个条目的相互独立性。总结来说，Scrum中的`Product Backlog`中的条目特征可以总结为`INVEST`:

- Independent，独立性确保了排序的可行性

- Negotiable，可协商体现从用户价值出发，待办列表不等同于合同，根据用户需求的关注点可动态变化

- Valuable，有价值是待办项的基本属性

- Estimate-able，可估算代表团队已经清楚了工作内容和工作量大小

- Small，足够小确保待办列表能够在一个迭代中完成

- Testable，可测试代表团队已经清楚如何对待办项进行验收，即清楚了验收标准

`Product Backlog`的粒度会根据不同的类型而不同，其中越靠近当前迭代的用户故事粒度会小一些，越远的用户故事可能为`Epic`(史诗级故事)，在后续再进行细化。

## Product Backlog梳理

Product Backlog梳理的主要工作内容包括增加新的条目和细化条目，为持续性活动，不超过10%的开发时间。

## Sprint Backlog

`Sprint Backlog`通常作为`Product Backlog`的子集。基本特征与`Product Backlog`基本一致，其他附加特征如下：

- `Sprint Backlog`由开发团队拥有，`Product Backlog`由`Product Owner`拥有。即开发团队可决定 `Sprint Backlog`的内容，`Product Owner`可决定`Product Backlog`的内容。

- 责任分摊，即`Sprint Backlog`的责任由开发团队公共分摊。

- 团队负责测量Sprint绩效。