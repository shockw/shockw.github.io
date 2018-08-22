---
layout: post
title:  "通过Python实现Kerberos主体管理"
categories: 大数据
tags:  大数据 安全  Kerberos subprocess flask  
---

* content
{:toc}

## 开发环境准备
开发环境使用eclipse进行python代码开发，安装pydev，由于eclipse版本较低，需要选择合适的pydev版本。版本安装后配置python路径，本机使用python2.7。本机python模块安装通过pip在线安装。安装flask-restful时提示要升级pip，先升级下pip。

~~~
sudo pip install flask
sudo pip install --upgrade pip
sudo pip install flask-restful
~~~
通过菜鸟网站进行python知识简单学习。

## 运行环境准备
由于主要运维的环境都是linux主机，而且不能连公网，需要先在主机上通过非root用户配置一套python环境，使用版本为2.7.5。本文python模块全部使用下载包离线安装。
配置python环境请参考https://blog.csdn.net/Dream_angel_Z/article/details/51338546
需要安装的软件主要如下：

~~~
Python-2.7.5.tgz
setuptools-2.0.tar.gz
pip-8.1.1.tar.gz
aniso8601-3.0.2.tar.gz
click-6.7.tar.gz
itsdangerous-0.24.tar.gz
Jinja2-2.10.tar.gz
MarkupSafe-1.0.tar.gz
pytz-2018.5.tar.gz
six-1.11.0.tar.gz
Werkzeug-0.14.1.tar.gz
Flask-1.0.2.tar.gz
Flask-RESTful-0.3.6.tar.gz
~~~

## 程序设计
通过python subprocess模块与kadmin交互，实现用户的管理。通过Flask将其开放成接口。JAVA应用程序调用接口实现Kerberos票据管理。subprocess模块不需要安装，Flask需要安装。

* flask接口编程学习例子：https://www.jianshu.com/p/6ac1cab17929
* subprocess编程学习例子：https://blog.csdn.net/zhouyuanlinli/article/details/78617484

## 增加用户的代码示例

~~~
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import subprocess
from flask import Flask
from flask_restful import reqparse, abort, Api, Resource
app = Flask(__name__)
api = Api(app)

parser = reqparse.RequestParser()
parser.add_argument('name')
parser.add_argument('passwd')

class kpm(Resource):
    
   def post(self):
        args = parser.parse_args()
        name = args['name']
        passwd = args['passwd']
        RunCmd.user_add(name, passwd)
        return "sucess", 201

api.add_resource(kpm, '/kps/create')

class RunCmd(object):
    def __init__(self):
        self.cmd = 'ls'
 
    @staticmethod
    def local_run(cmd):
       print('start executing...')
       print('cmd is -------> %s' % str(cmd))
       s = subprocess.Popen(str(cmd), shell=True, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
       out, err = s.communicate()
       print("outinfo is -------> %s" % out)
       print("errinfo is -------> %s" % err)
       print('finish executing...')
       print('result:------> %s' % s.returncode)
       return s.returncode
    
    @staticmethod
    def user_add(username,passwd):
        cmd = r"""
           expect -c "
           set timeout 1;
           spawn kadmin ;
           expect COM {{ send \"cloudera-scm-eip\r\" }}  ;
           expect * {{ send \"addprinc -pw {password} {principal}@HADOOP.COM\r\" }}  ;
           expect *\r
           expect \r
           expect eof
           "
        """.format(principal=username,password=passwd)
        RunCmd.local_run(cmd)
        
if __name__ == '__main__':
    app.run(host='0.0.0.0', port = 8181, debug=True)  
~~~
## 接口测试
通过http客户端发送post请求，成功创建kerberos用户。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180821/1.png)