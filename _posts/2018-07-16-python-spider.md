---
layout: post
title:  "使用pyspider实现抓取合肥房产局数据Demo"
categories: 大数据
tags:  python 爬虫 
---

* content
{:toc}

# 使用pyspider实现抓取合肥房产局数据Demo

源码在github上，有个中文学习网站http://www.pyspider.cn/详细介绍其使用。需要简单学习下Python脚本语言、Pyquery语法、phantomjs等。

##安装pyspide

http://www.pyspider.cn/book/pyspider/centos-install-pyspider-7.html
使用root用户，操作系统为centos7.2最小版本，主要步骤：

* 依赖安装
* 升级pip：pip install --upgrade pip
* 在线安装：pip install pyspider
* 启动： pyspider

操作过程中的问题：
本地无法打开界面，因为服务器上有防火墙限制，无法访问，下面centos7关闭防火墙操作。
查看防火墙状态：firewall-cmd    --state
关闭防火墙：systemctl  stop   firewalld.service
开启防火墙：systemctl  start   firewalld.service
禁止开机启动启动防火墙：systemctl   disable   firewalld.service

##安装phantomjs

http://www.pyspider.cn/book/pyspider/phantomjs-install-10.html
有很多页面无法直接抓取url，需要调用js函数才能钻取到下一级页面，需要按照上述方法安装phantomjs，用于在pyside里执行js代码。

操作过程的问题：由于centos连接github网站很慢，需要设置git代理才能进行代码下载与操作，这块耗时很多。
另外编译安装时间比较长，需要耐心等待。
安装后需要把phantomjs执行命令放到环境变量，再执行pyspider all，否则找不到phantomjs。

##demo例子

重点参考网站的各个教程，里面从简单到复杂讲了不少例子。另外官网有很多例子可以直接参考
http://demo.pyspider.org/
采集合肥市房产项目，统计下每年的住宅开发面积。合肥社区实际参保人员200万左右。150万套住房需求差不多。
http://real.hffd.gov.cn/?servicetype=%25E4%25BD%258F%25E5%25AE%2585

采集数据例子：http://real.hffd.gov.cn/details/H3AWIn9dZliabVfzcupM0YyLw2BH-eKjtE5Nw27sV1UzaBFbbEreCTCkyNUp9AfYK2z5SVvClRpIdRv0Z-5Kcx6k7_8hES2n9maMr_L5Wgb25X81hgGCUGuTEYxCuzn-iaQJ0OhwmkgTh4cHEhfpsAmtsGPjueNZ_epkV6zm24k=

采集字段：许可证号，套数、预售许可面积、网上销售面积、用途、开盘日期

##数据分析
目前合肥社保统计约有150万人在市区工作，另外加上公务员、事业单位职工约200万人，则房屋需求应该在150万左右。实际上很多人在合肥工作但没有交社保，这块有很大遗漏。

根据数据统计，合肥目前登记的住宅或公寓约146万套，总面积约1亿3千万平米，每套面积约90平米。

看下每年的开盘套数，2000年以前几乎没有，2018年数据不全，主要还是最近十几年：

|2000 |2001 |2002 |2003 |2004 |2005 |2006 |2007 |2008 |2009 |2010 |2011 |2012 |2013  |2014  |2015  |2016  | 2017 | 2018|
|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:---- |:---- |:---- |:---- |:---- |:----|
|435  |641  |1459 |5667 |12148|40801|64382|76618|89192|95065|86978|80384|97671|138949|143498|160623|169633|128277|69107|


##代码

```
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Created on 2018-07-13 16:24:24
# Project: houseCrawl

from pyspider.libs.base_handler import *
import re
from pyquery import PyQuery as pq

class Handler(BaseHandler):
    crawl_config = {
    }

    @every(minutes=24 * 60)
    def on_start(self):
        self.crawl('http://real.hffd.gov.cn/?servicetype=%25E4%25BD%258F%25E5%25AE%2585',
                   fetch_type='js', js_script="""
                   function() {
                     setTimeout("$('a').click()", 1000);
                   }""", callback=self.index_page)

    @config(age=10 * 24 * 60 * 60)
    def index_page(self, response):
        for each in response.doc('a[href^="http"]').items():
            if re.match("http://real.hffd.gov.cn/item/\w+", each.attr.href, re.U):
                self.crawl(each.attr.href, callback=self.list_page)
    
    @config(age=10*24*60*60, priority=2)
    def list_page(self, response):
        for each in response.doc('a[href^="http"]').items():
            if re.match("http://real.hffd.gov.cn/details/\w+", each.attr.href, re.U):
                self.crawl(each.attr.href, priority=9, callback=self.detail_page)
            
    @config(priority=3)
    def detail_page(self, response):
        return {
            "url":response.url,
            "许可证号":response.doc('HTML>BODY>DIV.content.mt20>DIV.content_nav1>DIV.content_xiangx>UL>LI.rbg_1>P>SPAN').text().split()[0],
            "套数":response.doc('HTML>BODY>DIV.content.mt20>DIV.content_nav1>DIV.content_xiangx>UL>LI>P>SPAN').text().split()[4],
            "预售许可面积":response.doc('HTML>BODY>DIV.content.mt20>DIV.content_nav1>DIV.content_xiangx>UL>LI.rbg_1>P>SPAN').text().split()[4],
            "网上销售面积":response.doc('HTML>BODY>DIV.content.mt20>DIV.content_nav1>DIV.content_xiangx>UL>LI>P>SPAN').text().split()[9],
            "用途":response.doc('HTML>BODY>DIV.content.mt20>DIV.content_nav1>DIV.content_xiangx>UL>LI.rbg_1>P>SPAN').text().split()[6],
            "开盘日期":response.doc('HTML>BODY>DIV.content.mt20>DIV.content_nav1>DIV.content_xiangx>UL>LI.rbg_1>P>SPAN').text().split()[7],
        }
```