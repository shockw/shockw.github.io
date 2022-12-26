---
layout: post
title:  "数据平台浅谈"
categories: 数据平台
tags:  数据平台 数据中台 数据仓库 数据集市 数据工厂
---

* content
{:toc}

## 概念解析
什么是数据平台？和数据工厂、数据中台、数据仓库、数据集市、数据工厂有什么区别。我们来看下相关文章里对上述概念的理解。
### 数据工厂
数据工厂通过实时多源数据集成，柔性数据治理工艺、智能数据加工、综合数据服务，帮助客户降本增效、高效利用数据、充分挖掘数据资产价值，实现企业智能化生产、监管、运营。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20220306/2ACD33B3B9C7017A025F914E47AAE5F4.jpg)
### 数据中台
数据中台本身没有严格的学术定义，其具体特征包括支持多个前台业务且具备业务属性的共性数据能力体系。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20220306/1AD9007AFDDD2A09CB1D45C5219635FA.jpg)
### 数据仓库
数据仓库是一个面向主题的、集成的、随时间变化的、但信息本身相对稳定的数据集合，用于对管理决策过程的支持。数据仓库具备以下4个特征。
+ 面向主题：数据仓库都是基于某个明确主题，仅需要与该主题相关的数据，其他的无关细节数据将被排除掉
+ 集成的：从不同的数据源采集数据到同一个数据源，此过程会有一些ETL操作
+ 随时间变化：关键数据隐式或显式的基于时间变化
+ 数据仓库的数据是不可更新的：数据装入以后一般只进行查询操作，没有传统数据库的增删改操作。数据仓库的数据反映的是一段相当长的时间内历史数据的内容，是不同时点的数据库快照的集合，以及基于这些快照进行统计、综合和重组的导出数据，而不是联机处理的数据。

![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20220306/A0585B4602A91FC3FE7927780E28A65A.jpg)
### 数据集市
数据集市（Data Mart），也叫数据市场，为满足特定的部门或者用户需求，按照多维的方式进行存储，包括定义维度、需要计算的指标、维度的层次等，生成面向决策分析需求的数据立方体。   
数据集市，迎合了专业用户群体的特殊需求，包括分析、内容、表现，以及易用性方面。  
数据集市，是企业级数据仓库的一个子集，主要面向部门级业务，只面向某个特定的主题。  
数据集市数据来源于企业范围的数据库、专业的数据仓库。  
数据集市的特征：规模小；特定的应用；面向部门；由业务部门定义、设计和开发；业务部门管理和维护；快速实现；购买较便宜；投资快速回收；工具集的紧密集成；提供更详细的、预先存在的、数据仓库的摘要子集；可升级到完整的数据仓库。  
与数据仓库的区别：

|  指标   | 数据仓库  |  数据集市  | 
|  ----  | ----  |----  |
| 数据来源  |遗留系统、外部数据 | 数据仓库 |
| 范围  | 企业级 |部门级或工作组级 |
| 主题  | 企业主题	 |部门或特殊的分析主题 |
| 数据粒度  | 最细的粒度 |较粗的粒度 |
| 数据结构  | 规范化结构、星型模型、雪花模型	 |星型模型、雪花模型 |
| 历史数据  | 大量的历史数据 |适度的历史数据 |
| 优化  | 处理海量数据/数据探索 |便于访问和分析/快速查询 |
| 索引  | 高度索引 |高度索引 |

### 相关关系分析
数据平台更多强调的是IT技术，而非业务或数据。中台包括技术、数据和业务。数据仓库和集市更多强调的是数据。数据平台和数据工厂的概念相似，更多的是为数据开发人员、管理人员、数据应用人员提供一站式的工具链，快速完成数据沉淀和对外能力透出的流水线工作，实现数据智能化生产运营。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20220306/E4E5FF1DBBF1133B38354DBF5D83BEDD.jpg)
## 平台架构
数据平台提供一系列数据集成、数据治理、数据加工、数据共享的工具组件，打造核心数据体系，为上层业务中心和应用提供二次建模的智慧运营数据，驱动业务生产。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20220306/967D9FC134B65FD55FFF35FEBC1A1C3C.jpg)