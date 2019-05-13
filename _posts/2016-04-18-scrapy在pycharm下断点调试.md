---
title: scrapy在pycharm下断点调试
date: 2016-04-18 22:36:02
tags: Scrapy
categories: Python
---
最近用scrapy爬取了北京台的节目单，现在的进阶需求是读取节目单中各个节目的播放页面，得到下载连接并下载节目。期间遇到了一些错误，非常需要断点调试。

<!--more-->

## 解决方法

1. 在PyCharm中，项目结构如下图所示，在和`scrapy.cfg`并列的一级，添加main方法：

    ![添加方法在pycharm调试scrapy](/images/posts/2016.4.18/main.png)

2. main方法中的内容如下：

    ```python
    from scrapy import cmdline
    cmdline.execute("scrapy crawl program".split())   #其中的program是我定义的spider名
    ```


3. 在main.py文件下右键单击debug即可在自己所设的断点处进行单步调试：

    ![debug](/images/posts/2016.4.18/main_debug.png)


----
总结：工具是让人使用的，可是不仅要会使用，而且要懂得内部的原理，我还没有到这一步。
