---
layout: post
title:  "通过kettle实现文件到数据库转换的ETL流程实践"
categories: 数据平台
tags:  大数据 预处理 数据采集  转换   
---

* content
{:toc}

## kettle介绍
Pentaho Data Integration又称kettle，提供数据抽取、转换、加载的功能，纯java编写，有开源版本，纯绿色安装。
本文使用的是PDI7.1社区版本试验。pdi介绍请参见[一篇博文](http://www.kettle.net.cn/1579.html)。
版本自带详细的教程和案例，如果英文水平不错的话可以直接看原文，或者到网上一些中文社区中找资料学习。版本依赖于jdk1.8。

### 社区版与商业版区别
PDI包含spoon、pan、kitchen、carte四部份，spoon通过图形接口，用于编辑作业和转换的桌面应用；Pan是一个独立的命令行程序，用于执行由Spoon编辑的转换和作业；Kitchen是一个独立的命令行程序，用于执行由Spoon编辑的作业；Carte是一个轻量级的Web容器，用于建立专用、远程的ETL Server。

商业版本多的是管理调度、监控、协作开发、权限控制、集群管理等功能。

### 核心概念
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180919/7D2B1BD7-3B51-427F-9D01-34FAAE08F83D.png)
Steps（步骤）是转换的建筑模块，比如一个文本文件输入或者一个表输出就是一个步骤。在PDI中有140多个步骤，它们按不同功能进行分类，比如输入类、输出类、脚本类等。每个步骤用于完成某种特定的功能，通过配置一系列的步骤就可以完成你所需要完成的任务。Jobs（工作）是基于工作流模型的，协调数据源、执行过程和相关依赖性的ETL活动。

## 入门实例
官网有一个入门案例,简书上有中文翻译。[PDI7.1入门实例](https://www.jianshu.com/p/901bf932b614)。

## 资源库使用与监控
有两种方式，一种是xml配置文件，一种是使用数据库。配置与运行完全分开，通过spoon可以将任务执行在单节点，或是集群上面。默认情况下运行节点不需要连接资源库，运行的任务都是由spoon推送给运行节点执行。没有统一的日志监控，每个节点会记录日志，如果在集群模式下需要统一执行。

## carte使用
直接使用kitchen后台执行作业，作业里设置好调度策略。此种方式对于很多作业来说，不方便管理。在实际使用时，使用carte集群可以实现作业分节点运行，前提是针对流程内的环节进行集群设置。

## 常见使用方式
将作业和转换编排好，在linux上通过crontab进行作业调度，通过后台的日志查看运行是否正常。

## 优缺点分析
缺少统一的通过web浏览器的方式进行作业管理与监控，缺少任务自动切割的功能，虽然也能配置某个环节使用集群，但实际上处于单点故障的风险之中。不能作为企业级数据集成平台，缺少统一管理作业、转换的功能，也不方便进行集中的监控。数据转换的功能非常强大。