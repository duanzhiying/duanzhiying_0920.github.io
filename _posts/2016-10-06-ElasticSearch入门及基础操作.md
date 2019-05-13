---
title: ElasticSearch入门及基础操作
date: 2016-10-06 23:53:09
tags: 入门
categories: ElasticSearch
toc: true
---
1. 从elasticsearch官网下载ES：http://www.elasticsearch.org/download。我下载的版本是2.4.1.
2. 下载后解压，仅如bin文件夹，双击elasticsearch.bat即可启动ES。在web浏览器输入127.0.0.1：9200 便可查看ES服务是否就绪。浏览器显示内容如下：![127.0.0.1:9200](/images/posts/2016.10.6/127.0.0.1.png)
3. 安装head插件工具，在没有安装前，输入网址是空白的：![head](/images/posts/2016.10.6/head.png)
安装命令如下：
![head_plugin](/images/posts/2016.10.6/head_plugin.png)
安装完毕后，便可以使用该插件，如下:
![head_html](/images/posts/2016.10.6/head_html.png)
4. 在Head插件工具上执行复合查询，写入数据(PUT)：![put](/images/posts/2016.10.6/put.png)
5. 查询刚才写入的内容：![get](/images/posts/2016.10.6/get.png)
6. POST方式写入，随机生成id：![post](/images/posts/2016.10.6/post.png)

