---
layout: post
title: Swish的App开发策略：选择托管服务还是自构框架？
author: Mike
keyword: Developement Methodology
tags: #database #数据库 #back-end #后台 #后端 #development_strategy #开发策略
excerpt: App开发策略的选择与其优劣
postid: 3
---
本文整理翻译自『App Development Magazine』对Swish App CTO Jeff Whelpley的采访：

**原文地址**：[App development strategies from Swish's CTO Jeff Whelpley](https://appdevelopermagazine.com/6087/2018/5/30/app-development-strategies-from-swish's-cto-jeff-whelpley/)

#### 一、托管服务 还是 自构框架：
对于一个App开发团队来讲，初期最好尽可能多的选择托管服务(Managed Services)，因为还没有成功之前，项目可能随时会进行不下去，而被扔进垃圾桶。早期需要考虑的是产品特性(Feature)和用户体验(User experience)，等到逐渐成熟的时候再考虑升级托管服务(Premium)或重构内部架构(Internal infrastructure)。

主要是以下两方面的权衡：一是雇佣协调托管服务的工程师的价格，二是重建内部架构时对整个项目生产力的影响。

另外一点值得考量 *[译者按:需要Bare in mind]* 的是**转换的代价**(Switch cost)是否相对较低，比较简单的检验方法是，是否有信心在一个月内将现有的项目从托管服务转换到自构框架。

当然对于一个较大的团队来讲，构建自己的研发-运维团队(DevOps team)非常有价值，但是对于一些需要快速实验的想法或项目，托管服务仍然是较好的选择，正如上文所说，项目可能随时流产令之前的努力付之一炬。


#### 使用开源技术：
开源技术可以在使用较低预算的同时提升团队效率，Jeff提到他们的Swish App中较多依赖的开源项目：
**Angular**: 构建现代化的Web App
**Ionic**:      快速搭建Mobile App
#### 二、后端相关：
Swish使用的**后端技术**(Back-end technologies)和**数据库引擎**(Database engine)：
**MongoDB**: Our system of record 作为记录档案的系统
初期使用MongoDB，并决定尽可能的多使用JavaScript，MongoDB的接口对于JavaScript比较友好，随后开始用以下两种技术解决一些MongoDB不能很好解决的问题：_(PS: Studio 3T - IDE* for MongoDB with GUI**)_
**ElasticSearch**: For search queries 文字的搜索查询
**Firebase**: For real-time data sync 实时数据同步


#### 选择数据库的一些建议：
- **性能表现**(Performance)
- **可靠性**(Reliability)
- **良好的技术支持**(A great support of staff)

如上文所言，选择数据库服务需要考虑转换的代价，Whelpley的经验是，试用一个**数据库即服务**(Database-as-a-service)供应商一个礼拜，然后转换到另外一个服务供应商，用这种方式来检验数据库是否拥有较低的**转换代价**(Switch cost)。


#### 三、在项目开发中学到的一些教训/经验：

###### 1. 把程序分割成模块，并始终令它们容易被替换或舍弃
在开头构建架构的时候，可能会过度思考，模块可能会随着项目更新迭代而被认为不合适。因此在建立模块的时候，始终把它想象为**临时的**，并且在不久的将来**随时可能被替换或者取缔**。但是要**保证编码的高标准**(High quality standard)，尽量**减少内部相关性**(Internal dependency)，让未来去耦(Decouple)的工作更加容易进行。

###### 2. 精益数据分析方法 [Lean analytics methodology](http://leananalyticsbook.com)
Jeff的团队在开发过程中遵循精益数据分析方法，这种方法中一个最重要的原则是，**避免过度开发**(Avoid building software as much as possible)。
每当提出一项能够提升产品的假设时，**先**对这项假设**进行检验**(Hypothesis testing)，然后**才投入生产**。
这样一来，每次**对代码的改变**，都是**基于目标** _[译者按：这个目标就是通过检验的假设]_ 产生的，在**改变过后**对其进行**测量**(Measure)和**评估**(Evaluate)，在缩短迭代周期的同时，提高团队效率。
从实践层面来讲，Jeff的团队每天推出测试版本(Daily beta releases )，供一部分用户使用并获取反馈，来检验这些测试的项目是否正确、以及是否为用户真正所需。

###### 3. 转用自建架构改善后台数据处理的压力 ***此段翻译可能有争议

Jeff提到Swish App的大多数数据处理压力都集中在后台，遵循上文(第一部分)提到的关于托管服务与自建架构的分析逻辑，近期在着力从托管服务提供商转换利用Firebase搭建自用的系统以改善后台数据的运算。

----------
**字母缩写：**
\* IDE - Independent Development Environment 集成开发环境
\** GUI - Graphic User Interface 图形用户界面