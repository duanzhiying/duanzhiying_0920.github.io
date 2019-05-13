---
title: 'Scrapy爬虫实例（rbc.cn节目单）|保存在MySQL数据库|Windows下定时任务'
date: 2016-03-28 10:48:53
tags: Scrapy
categories: Python
toc: true
---
[最爱视听网](http://www.zuiaishiting.com/)提供了几乎国内大部分广播电台的在线直播，如进到“北京新闻广播”直播收听页面，其“节目表”的显示是以链接的形式，即链接跳转到[北京广播网](http://audio.rbc.cn/calendar.form)节目单页面。那何不自己写一个爬虫程序爬取北京台所有广播的节目单并每天自动更新呢？

<!--more-->

## 参考链接
1. 实例：[Scrapy安装、爬虫入门教程、爬虫实例（豆瓣电影爬虫）](http://www.cnblogs.com/Shirlies/p/4536880.html)
2. MySQL保存：[自定义Pipeline将scrapy采集的数据保存到mysql数据库](http://www.sharejs.com/codes/python/8392)、[scrapy爬虫成长日记之将抓取内容写入mysql数据库](http://www.w2bc.com/Article/44862)

## 环境
本爬虫程序是在Wiondows环境下，基于Python的Web抓取框架Scrapy。参考[Scrapy入门文档](http://scrapy-chs.readthedocs.org)。

## 主要类及代码分析 
创建一个Scrapy项目后，其目录显示如下图的形式。

![scrapy项目结构](/images/posts/2016.3.29/project.png)

其中：items文件以结构的形式保存你的定制爬虫爬下来的内容。pipelines文件设定爬虫内容的保存形式（文本还是数据库等）。`spiders`文件夹是放置spider代码的目录，我们的爬虫主类需要新建并添加在该目录下。

### 实体定义
我们所要爬取的节目单页面显示如下，当然这只是一个台某一天的节目，而我们的目的是**得到北京广播网所有电台当天的节目单**。

![北京新闻广播节目单](/images/posts/2016.3.29/playlist.png)

items.py文件格式设定如下，我们主要需要得到的就是电台名、节目播放时间、节目名。

```python
# -*- coding: utf-8 -*-
import scrapy
class PlaylistCrawlerItem(scrapy.Item):
    radio_name = scrapy.Field() # 频道名
    program = scrapy.Field() # 节目名
    time = scrapy.Field() # 播放时间
```

### 爬虫主程序
在`spiders`文件夹下建立一个py文件：`playlist_spider.py`，这个爬虫程序的关键。直接贴代码分析：


```python
# coding=utf-8
# 在这里省略了部分引用的显示
from TimeProcess.timeprocess import *
from playlist_crawler.items import PlaylistCrawlerItem

class PlaylistSpider(scrapy.Spider):
    name = "playlist"
    allowed_domains = ["audio.rbc.cn"]
    start_urls = []
    item = PlaylistCrawlerItem()
    # 保存电台页面对应的id，这里我们选了这几个电台来进行爬取，用list保存
    radio_name = ['1', '2', '3', '4', '5', '6', '7', '8', '11', '12', '14', '17', '18', '19', '21']

    def start_requests(self):
        # 电台节目单网址规则
        url_pre = "http://audio.rbc.cn/calendar.form?radio_id="
        middle_week = "&week="
        middle_day = "&day="
        for id in self.radio_name:
            # 拼接网址，需要用到时间处理，得到周第一天的时间戳和当天的时间戳
            url = url_pre + id + middle_week + get_week_begin() + middle_day + get_day_begin()
            self.start_urls.append(url)

        for url in self.start_urls:
            yield self.make_requests_from_url(url)

    def parse(self, response):
        name = response.xpath('//div[@id="m_nav"]/a[@class="here"]/em/text()').extract()[0]
        for sel in response.xpath('//div[@class="p2_tr"]/div/table/tbody/tr'):
            self.item['radio_name'] = name
            self.item['time'] = sel.xpath('td[1]/text()').extract()[0]
            self.item['program'] = sel.xpath('td[2]/a/text()').extract()[0]
            yield self.item
```

该类的主要方法介绍：
**start_requests方法**：初始化要爬取的URL，并保存在`start_urls`列表中。

代码分析：对于要爬取的北京广播网的节目单，首先要分析页面URL的规则，发现其URL的组成形式如下：
http://audio.rbc.cn/calendar.form?radio_id=1&week=1459094400000&day=1459180800000

radio_id后的id对应每个台，如新闻广播radio_id = 1，交通广播radio_id = 6，而后面拼接的week和day，则指本周第一天凌晨的时间戳，和当天凌晨的时间戳。熟悉了该规则，便可以将我们需要的URL添加到`start_urls`列表中。即代码19-22行的for循环。其中对时间的处理，可参考[链接](http://www.thinksaas.cn/group/topic/91972/)，写的非常仔细。

**make_requests_from_url方法**：转化URL为Request，默认情况下，`parse`是其回调函数，处理response。

**parse方法**：处理网页内容，必须返回一个包含Request或者Item的可迭代对象(*需要了解yield相关知识*）。

代码分析：首先我们需要了解待爬取的内容的HTML源代码结构，这里配合火狐浏览器的FireBug组件使用，如下图。可见我们需要的节目播出时间和节目名，是tr中的两列。从div class = 'p2_tr'开始分析，tr属于该div的子div的table中的tbody中。得到该行，便可拿到其中的两列td值，即得到节目名和播出时间。
![待爬取内容页面结构](/images/posts/2016.3.29/firebug.png)

对于频道名，再次分析页面结构。频道名是显示在具有class = here的a标签内具有em标记的text。且该a标签在id为m_nav的div下。即对应的XPATH为：
**name = response.xpath('//div[@id="m_nav"]/a[@class="here"]/em/text()')**

![广播电台名称页面结构](/images/posts/2016.3.29/radioname.png)

关于**XPath表达式**，提供一种简单快速的生成方式。**在FireFox浏览器下安装XPath Check组件**，即可在你需要获取的内容上右键点击“view xpath”，自动获取其XPath地址。
如还是刚才的频道名，我们通过该组件得到XPath路径。

![第一步](/images/posts/2016.3.29/xpath1.png) ![第二步](/images/posts/2016.3.29/xpath.png)

对于该表达式，可以在Scrapy提供的Shell中进行验证。在当前工程路径下，输入如下命令启动Shell：
```dos
>scrapy shell "http://a
udio.rbc.cn/calendar.form?radio_id=6&week=1459094400000&day=1459180800000"
```

shell载入后，我们便可以输入类似于response.xpath()来对表达式进行查询或者验证。验证结果如下图，可以看出使用`XPath Check`组件生成的表达式和本程序中提供的表达式得到的结果是相同的。这种方式对于不想深入了解XPath具体知识的情况下非常有效 *（但要注意把XPath Check生成的XPath中的x都删除掉）*。

![XpathCheck生成的xpath在shell中的显示](/images/posts/2016.3.29/shell.png) 

### 保存到json or Mysql
如何将上一步收集的item保存，需要使用到item pipeline组件。该组件的功能是对结果进行清理、验证、查重或者保存。
将爬取的内容保存在文本中，这里提供的方式是首先保存在json文件中，然后写了一个小程序来将该文本中的内容写入MySQL。
虽然pipelines支持直接写入MySQL（具体可参考本文第一部分给出的参考链接），**但经过自己的测试，写入的MySQL文件内容极其个别的几个节目的播出时间竟然是重复的**，即不能保证结果的完全正确。目前还没有去研究为什么这样，但是我觉得比较保险的方式是先保存在文件（经检查没有重复或错误），然后自行写入数据库。

保存在json文本需要两步：
1. 在pipelines中定义类。本程序的代码如下。pipeline必须要实现的方法包括process_item。

    ```python
    # -*- coding: utf-8 -*-
    import codecs
    import json

    import sys
    reload(sys)
    sys.setdefaultencoding("utf-8")

    class JsonPlaylistCrawlerPipeline(object):
        def __init__(self):
            import datetime
            today = datetime.date.today()
            # 节目单保存文本命名规则：yyyy_MM_dd_play_list_data.json
            file_name = today.strftime("%Y_%m_%d") + '_play_list_data.json'
            self.file = codecs.open('data/' + file_name, 'w', encoding='utf-8')

        def process_item(self, item, spider):
            line = json.dumps(dict(item), ensure_ascii=False) + "\n"
            self.file.write(line)
            return item

        def spider_closed(self, spider):
            self.file.close()
    ```


2. 启动我们写好的pipieline：在settings.py中添加配置。
```python
ITEM_PIPELINES = {'playlist_crawler.pipelines.JsonPlaylistCrawlerPipeline':300}
```

经过上述两步操作后，节目单便被保存在了按照我们自己制定规则的json中，截图如下。这个时候再将其保存在MySQL数据库中就很好操作了，具体步骤就略过了。

![爬取后的节目单文件](/images/posts/2016.3.29/data.png) 

### Windows下定时任务：每天凌晨自动爬取节目单并保存在MySQL据库

现在爬虫也写好了，推送到MySQL数据库也搞定了，下一步就是让这些操作自动化，定时执行。
在Linux下可以使用crontab定时任务，那在windows下呢？
实现方案：所有程序>附件>系统工具>任务计划程序。打开任务计划程序，常见基本任务，如下图，在这里便可以设置我们要定时运行的程序，定时规则。

![Windows下任务计划程序](/images/posts/2016.3.29/schedule.png) 

我将启动scrapy爬虫程序和保存到MySQL程序命令写在了一个bat文件中，然后让这个计划任务去定时调用运行该bat文件：

![执行bat文件](/images/posts/2016.3.29/bat.png) 

bat文件中的具体内容如下，如有类似需求的同学可以参考。
```dos
@echo off 
H: 
cd 000_radio_program\rbc_playlist_scrapy\playlist_crawler
scrapy crawl playlist
cd data
start pythonw play_list_data_to_mysql.py
exit
```


## 那些事后觉得完全可以避免的“坑”
1. 每行读取txt文本解析时句尾多余的空格，导致页面URL拼接失败。
2. unicode编码的处理无时不刻的存在并难为着初学的你。

## 其他需要关注的知识点(待完善)
1. yield
2. xpath

----
总结：这是第一个完全自己写的Scrapy爬虫程序，耗时一天，并没有用到Scrapy支持的Rules以及Select选择器等等，程序相对简单，但比较完整，更像一个入门的实例。切记一定要细心，出现错误首先要做的是耐下心来看log错误日志，这才是问题的根源所在。
