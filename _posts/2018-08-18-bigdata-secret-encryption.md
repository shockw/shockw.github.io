---
layout: post
title:  "Hadoop平台中的透明数据加密"
categories: IT杂谈
tags:  大数据 安全  TDE 透明数据加密  
---

* content
{:toc}

## 什么是透明数据加密
TransparentData Encryption用来加密数据文件里的数据，保护从底层对数据的安全访问。所谓透明是指对使用者来说是未知的，当使用者在写入数据时，系统会自动加密，当使用数据时系统会自动解密。Oracle、SQLServer很早就支持了透明数据加密的特性，下面说一下针对HDFS文件和HBase的透明数据加密过程。

## HDFS KMS透明加密配置及测试
HDFS Encryption zone加密空间是一种end-to-end(端到端)的加密模式.其中的加/解密过程对于客户端来说是完全透明的.数据在客户端读操作的时候被解密,当数据被客户端写的时候被加密,所以HDFS本身并不是一个主要的参与者,形象的说,在HDFS中,你看到的只是一堆加密的数据流.

### Encryption zone原理介绍
1. 每个encryption zone 会与每个encryption zone key相关联,而这个key就是会在创建encryption zone的时候同时被指定。
2. 每个encryption zone中的文件会有其唯一的data encryption key数据加密key,简称就是DEK。
3. DEK不会被HDFS直接处理,取而代之的是,HDFS只处理经过加密的DEK, 就是encrypted data encryption key,缩写就是EDEK。
4. 客户端询问KMS服务去解密EDEK,然后利用解密后得到的DEK去读/写数据。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/D20BF070-AE64-4106-9F7B-525E8A0F83B5.png)
Key Provider可以理解为是一个key store的保存库,其中KMS是其中的一个实现。

### 环境准备
本文中的大数据环境通过CloudManager5.12进行安装，Hadoop版本为hadoop 2.6.0-cdh5.12.1。
通过CM直接安装Java KeyStore KMS服务。
按照默认选项可完成安装，KMS服务使用Kerberos进行认证。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/8B8B9EE0-6E97-4146-9A3D-20DB12A42C8D.png)
管理员用户名设置为kms，组名设置为kms，此用户可以管理所有KMS密钥。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/E00EB5D3-3DF9-4259-9694-7B166A27566D.png)
安装KMS后，CM自动KMS服务作用于Hadoop，相关引用KMS服务的组件均需要重启。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/9C3AB2B0-256D-451A-A5EB-D59E2EA228A7.png)

### Kerberos上增加用户kms，kms用户具有管理密钥的权利
登陆kerberos服务器，用管理员用户进入命令行控制台增加用户，密码设置为kms

```
[root@node181 ~]# kadmin.local
Authenticating as principal hbase/admin@HADOOP.COM with password.
kadmin.local:  add_principal kms@HADOOP.COM
WARNING: no policy specified for kms@HADOOP.COM; defaulting to no policy
Enter password for principal "kms@HADOOP.COM": 
Re-enter password for principal "kms@HADOOP.COM": 
Principal "kms@HADOOP.COM" created.
```
在HDFS客户端登陆kerberos用户kms，创建密钥

```
[root@node86 ~]# kinit kms
Password for kms@HADOOP.COM: 
kinit: Password incorrect while getting initial credentials
[root@node86 ~]# kinit kms
Password for kms@HADOOP.COM: 
[root@node86 ~]# hadoop key create testkey1
testkey1 has been successfully created with options Options{cipher='AES/CTR/NoPadding', bitLength=128, description='null', attributes=null}.
KMSClientProvider[http://node183:16000/kms/v1/] has been updated.
[root@node86 ~]# hadoop key list
Listing keys for KeyProvider: KMSClientProvider[http://node183:16000/kms/v1/]
testkey
testkey1
```
在HDFS客户端登陆Kerberos用户hdfs，创建文件夹/zwang/secret，并设置此文件夹为加密专区

```
[root@node86 ~]# kinit hdfs
Password for hdfs@HADOOP.COM: 
[root@node86 ~]# hadoop fs -mkdir /zwang/secret
[root@node86 ~]# hdfs  crypto -createZone  -keyName  testkey1  -path  /zwang/secret
Added encryption zone /zwang/secret
[root@node86 ~]# hdfs crypto -listZones
/user/hdfs/.Trash/Current/zwang/secret  testkey1 
/zwang/secret                           testkey1 
```
在kms-acl.xml里增加属性配置，cm里通过界面配置完成

```
<property> <name>key.acl.testkey1.DECRYPT_EEK</name> <value>zwang</value> <description> ACL for decryptEncryptedKey operations. </description> </property>
```
重启kms服务

### 测试加密文件
上传测试文件至/zwang/secret下面，报错。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/6057F617-B4A6-48A7-8BD3-23CF931E2A43.png)
赋权zwang用户对secret的读写权限

```
[root@node86 ~]# hdfs dfs -setfacl -m user:zwang:rwx /zwang/secret
[root@node86 ~]# hdfs dfs -getfacl /zwang/secret
# file: /zwang/secret
# owner: hdfs
# group: supergroup
user::rwx
user:zwang:rwx
group::r-x
mask::rwx
other::r-x
```
登陆kerberos用户zwang，上传文件到/zwang/secret下面

```
[root@node86 ~]# kinit zwang
Password for zwang@HADOOP.COM: 
[root@node86 ~]# hadoop fs -put 1.txt /zwang/secret
[root@node86 ~]# hadoop fs -cat /zwang/secret/1.txt
1,jack,18
2,green,15
```
除zwang用户外，换其他用户则无法查看文本内容,通过管理员用户hdfs查看相关文件，比较加密空间与非加密空间的数据备份即可看到加密与非加密的区别。
hdfs dfs -cat /.reserved/raw/zwang/secret/1.txt
hdfs dfs -cat /.reserved/raw/zwang/tmp/1.txt
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/4906E97B-4DD9-489B-AEDA-5497C04A4915.png)

### hive测试
通过hive建外部表，关联/zwang/tmp和/zwang/secret发现，加密空间对应的表无查询数据。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/147033F0-7875-42AB-9EBF-6D672A3DCB26.png)
通过cm界面增加hive使用testkey1密钥的权利，重启kms服务
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/9074EB05-2C94-4B75-8F21-F48B38F1A4CC.png)
再次查询test1表，则数据可以正常显示。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/C98F68EA-3F3F-45DC-8967-DE0F15D4E9FE.png)

## HBase透明服务器端加密
此功能提供透明加密，用于保护静态的HFile和WAL数据，使用双层密钥架构进行灵活且非侵入式的密钥轮换。首先需要修改HBase集群配置使其支持透明服务端加密，本文针对一个表中不同列簇设置不同的策略，以检验其加密的效果。具体过程请参考官网资料：http://archive.cloudera.com/cdh5/cdh/5/hbase-0.98.6-cdh5.3.3/book/hbase.encryption.server.html

为AES创建适当长度的密钥，alias设置为hbase，密码设置为ustcinfo

```
$ keytool -keystore /path/to/hbase/conf/hbase.jks \
  -storetype jceks -storepass <密码> \
  -genseckey -keyalg AES -keysize 128 \
  -alias <alias>
```

将生成的hbase.jks文件拷贝到所有hbase节点的/etc/conf/hbase目录下。
在CM控制台修改hbase-site.xml 的 HBase 服务高级配置代码段（安全阀），找到此配置
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/31DF2FC5-A7EF-4C14-841B-68AB14BD660D.png)
添加安全配置如下：

```
<property><name>hbase.crypto.keyprovider</name><value>org.apache.hadoop.hbase.io.crypto.KeyStoreKeyProvider</value></property><property><name>hbase.crypto.keyprovider.parameters</name><value>jceks:///etc/hbase/conf/hbase.jks?password=ustcinfo</value></property><property><name>hbase.crypto.master.key.name</name><value>hbase</value></property><property><name>hfile.format.version</name><value>3</value></property><property><name>hbase.regionserver.hlog.reader.impl</name><value>org.apache.hadoop.hbase.regionserver.wal.SecureProtobufLogReader</value></property><property><name>hbase.regionserver.hlog.writer.impl</name><value>org.apache.hadoop.hbase.regionserver.wal.SecureProtobufLogWriter</value></property><property><name>hbase.regionserver.wal.encryption</name><value>true</value></property>
```
重启HBase服务。
通过hbase用户登陆hbase shell，在建表时设置某个列簇加密

```
hbase(main):003:0> create 'zwang_foo',{ NAME=> 'f1',ENCRYPTION => 'AES'},{ NAME=> 'f2'}
0 row(s) in 2.2560 seconds

=> Hbase::Table - zwang_foo

hbase(main):006:0> put 'zwang_foo','1','f1:address','wenqulu355 gaoxinqu hefeishi anhui china'
0 row(s) in 0.0080 seconds

hbase(main):010:0> put 'zwang_foo','1','f2:address','wenqulu355 gaoxinqu hefeishi anhui china'
0 row(s) in 0.0160 seconds
```
退出hbase shell ，查看hbase表对应的hfds文件，一开始看不到文件，可以重启下hbase，就可以到看到此表对应不同列簇的数据文件，一个处于AES加密状态，一个可以明显看到数据内容。
未加密列簇f2对应的hdfs文件：
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/588EE4E7-AB0A-47DF-A9FF-D7EA4421EB4E.png)
加密列簇f1对应的hdfs文件：
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20180818/6BB9BF93-67B6-447C-8A30-0CCB9108AC4E.png)