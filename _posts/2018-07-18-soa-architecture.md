---
layout: post
title:  "浅谈从单体应用到业务中台架构的演变"
categories: 应用开发
tags:  SOA 微服务  
---

* content
{:toc}

根据笔者见过的项目情况，就企业应用的技术架构进行一些粗浅分析。业务中台的理念来源于SOA思想，经过阿里等互联网公司发扬光大，形成实际可行的技术方案。事实上行业软件里早就有了SOA架构的实践，不过从实际上来看，笔者所在的运营商行业其实发展并不好，经常是投资大，收益小。业务的设计和IT人员的水平关系很大，大多设计一个集中式的ESB管理所有服务，但服务很少是从业务角度考虑，大多只是一个技术接口，没有体现出SOA的本质，更谈不上其效果。

## 科学的方法

不管是什么技术方案，都不是统一天下的方案，不同的场景总要有适合的方案。我们总是要寻找切实高效的方案，而不是人云亦云，领导说好就是好，客户说好就是好。事实上我们需要有自己独立的思考，在业务较简单时，很明显不适宜上复杂的方案。

## 单体应用

企业单体应用一般就是一个web程序和一个数据库。比较常见的技术方案如下：

![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180618/20180718001.png)


## 业务中台架构

当业务变化很快，且系统复杂后，就需要将系统拆分成多个业务中心，拆分的目的就是抽象业务，形成高内聚低耦合的服务中心，这样服务支撑会更迅速，开发迭代的速度会提高，另外每个中心的开发规模会变小，大家会更专业。每个服务中心都会需要架构师、开发人员、ued人员、dba与运维等。复杂的系统的响应总是很慢，由于业务之间的耦合，导致新增一个功能漫长且容易出错。只有最熟悉企业业务的人参与，才可能从业务角度对系统进行架构。公共技术的服务剥离实际上并没有什么成效，现在很多企业IT的现状是懂技术的人不懂业务，懂业务的人不懂技术，这会严重影响企业的架构水平。下面是著名的阿里巴巴中台战略思想与架构实战中的共享服务架构：

![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180618/20180718002.png)

## 技术栈

|层级\架构   |业务架构                     |技术架构                    |
|:----------|:--------------------------|:--------------------------|
|应用层|天猫、淘宝、聚划算、手机淘宝、口碑等     |Html5/Js、移动应用、表单、流程    |
|服务层|用户中心、商品中心、交易中心、评价中心等     |微服务(spring boot)、负载均衡(haproxy/nginx)、弹性伸缩(docker)、服务熔断(Hystrix)、链式跟踪(pinpoint)    |
|数据层|用户模型、商品模型、交易模型  |Oracle、MySQL、分布式数据库(Drds/Mycat)   |

## 业务中台架构的重要特征

* 服务显性化管理，服务一旦部署，所有人可见服务并且能够看到服务API并进行测试，比如swaggerui即可显性化服务接口的调用协议。所有人都可以看到服务列表。
* 服务的全面监控，通过统一的日志拦截，记录所有服务的关联并跟踪其成功失败、耗时等情况，实现链式监控。
* 服务有个统一的入口，系统间均可以通过这个入口统一调用所有服务。
* 业务封装在服务中心，服务的抽象需要有原子性。应用重点关注服务的使用和前端的交互，重点通过服务编排实现应用功能。
* 业务服务可以实现弹性伸缩，比如瞬间增加N个节点以提高其性能。
* 数据层需要打好基础，一般需要有统一的分布式文件系统、统一的分布式数据库。文件系统里有分针对大文件，比如hdfs，也有针对小文件，比如TFS。
* 系统对外提供能力，可以通过API网关，重点实现权限的控制。