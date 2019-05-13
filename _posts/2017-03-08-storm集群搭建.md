---
title: storm集群搭建
date: 2017-03-08 14:37:20
tags:
---

在windows环境下尝试了单机版的storm环境部署，需要准备JDK，安装Zookeeper和Storm并启动，浏览器localhost:8080便可以看到storm UI，但本文的重点是linux服务器部署storm集群并提交workdCount拓扑。

<!--more-->

### 环境
1. Linux服务器的版本信息如下：
    ![版本信息](/images/posts/2017.3.8/Linux服务器版本信息.png)
2. 已经配置好了Java环境，JDK信息如下：
    ![JDK](/images/posts/2017.3.8/JDK.png)
3. 准备好storm和zookeeper的安装包：
    ![安装包](/images/posts/2017.3.8/安装包.png)

### 部署步骤
1. 解压storm和zookeeper安装包，并配置软链接
```doc 
tar xzvf repository/zookeeper-3.4.9.tar.gz
ln -s repository/zookeeper-3.4.9/* zookeeper
```
2. 拷贝zoo_sample.cfg在相同文件夹下命名为zoo.conf并修改
```
cp zoo_sample.cfg zoo.conf
```
修改内容如下：
    ![配置信息](/images/posts/2017.3.8/zooconf.png)
在该配置文件中指定的dataDir下新建myid文件，只有一行数字，对不同的server主机，是对应的server.ID的这个ID。

3. 配置storm和zookeeper环境变量，加入path路径，如下：
```doc
export ZK_HOME_DZY=/export/duanzhiying/app/zookeeper
export PATH=$ZK_HOME_DZY/bin:$PATH
export STORM_HOME_DZY=/export/duanzhiying/app/storm
export PATH=$STORM_HOME_DZY/bin:$PATH
```
