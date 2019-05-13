---
title: hexo github windows  博客搭建过程记录
date: 2016-03-21 20:14:46
tags: [Hexo,部署,记录]
categories: 技术
toc: true
---
去年6月份各种冲动买的域名kidzying.com今天提醒我该续费了，原来这么快就过了一年了，怀着好奇心（不死心）搜了自己的全拼，竟然不再被占用，于是决定买下并重头开始。经过对Pelican、Jekyll和Hexo的犹豫，最终选择了Hexo，因为某些低级错误（拼写）折腾了很久，必须记下某些过程。如下仅记录遇到的问题。
<!--more-->

### Hexo生成博客与github的绑定
1. github建立仓库，若建立个人博客，仓库名必须为*username.github.io*,其中username必须和github账号相同。一个github账号有唯一的个人博客。
2. hexo项目目录*_config.yml*文件修改，绑定github地址（在此处耗费了1个小时，原因仓库的单词*repository*拼错了）。
```dos
deploy:  
    type: git  //此处要填为git，区别于hexo版本2.0
    repository: git@github.com:duanzhiying/duanzhiying.github.io.git  
    branch: master
```
    注意：在*hexo c*、*hexo g*和*hexo d*之前要运行：
    ```dos
    npm install hexo-deployer-git --save
    ```
### github绑定二级域名 (如blog.username.com)
上次我用github绑定的域名是全域名www，这次想绑定二级域名，仅作为域名下的博客内容链接。在网上也搜了一些时间，如下方式成功：

1. 在dnspod上对二级域名进行解析：

    ![dnspod](../images/posts/dnspod.png)

    **主机记录**填写二级域名的名字，**记录类型**选择*CNAME*，**记录值**填写为github账号对应的仓库地址：*username.github.io*。

2. 将godday上我的域名更改dns服务器地址为dnspod的地址：
 
    ![goddaynameserver](../images/posts/nameserver.png)

    上图的*Nameservers*地址和1中该域名“记录类型”为*NS*的记录值相对应。

3. 在github项目中添加CNAME完成域名绑定。
    
    需要在项目主页中添加文件，命名为**CNAME**，内容填写要绑定的域名值，如我的域名“blog.duanzhiying.com”，不需要加入http这些头。
    因直接在github仓库中新建该文件，会在hexo deploy时被删除，导致域名需要重新绑定，所以解决方法是：在hexo博客目录的**source**文件夹下添加该文件。

经过这3个步骤，即实现hexo、github、二级域名三者之间的绑定。

### 使用maupassant主题

经过一番寻觅，主要参考主题链接：[github/hexo/themes](https://github.com/hexojs/hexo/wiki/Themes) 和 [hexo.io/themes](https://hexo.io/themes/)，前者以列表的方式，后者提供缩略图预览，都提供链接Demo。一眼就被[屠城](https://www.haomwei.com/) 博客的干净所吸引，虽然有些地方还是和自己的期待的差那么一点点，所以希望将来强大了再自己修改吧，现在算刚入门。
参考[中文教程](https://www.haomwei.com/technology/maupassant-hexo.html)进行安装，其中使用npm安装的两条命令需要采用**cnpm**进行替换安装。
```dos
$ npm install hexo-renderer-jade --save
$ npm install hexo-renderer-sass --save
```
cnpm的安装方法如下：
```dos
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 添加“多说”评论并设置跳转

之前的博客就夭折在评论添加上，所以这次必须加上。maupassant默认支持disqus和多说评论，添加方法非常简单。

1. 修改主题文件夹下的_config.yml文件：

    ```dos
    themes/maupassant/_config.yml ##修改该文件
    fancybox: true ## If you want to use fancybox please set the value to true.
    duoshuo: duanzhiying ## Your duoshuo_shortname, e.g. username
    disqus: ## Your disqus_shortname, e.g. username
    ```

    如上所示，在“duoshuo”标签后写下你的多说short_name。如何获得该name，初次搭建博客的同学需要进入[http://duoshuo.com ](http://duoshuo.com )网页，点击“我要安装”按钮，添加站点信息，获取多说域名（如duanzhiying.duoshuo.com），去掉“.duoshuo.com”，就是你的short_name。

2. 上述操作完毕，即可在博客页面看到评论框，若有人评论，博文标题下面便会显示“XX条评论”：

    ![comments](/images/posts/comments.png)

    但是现在点击这条评论还无法跳转到页面底部的评论框，原因是没有设置*section id*。需要修改如下文件：

    ```dos
    if theme.duoshuo
    #comments  //默认文档没有这句话，需要添加上去。
        .ds-thread(data-thread-key=page.path, data-title=page.title, data-url=page.permalink, data-author-key='1')
    ```

    添加上去的“#comments”对应html中的内容如下，这样便可以在点击上图中的“1条回复”跳转到页面底部相应区块了。

    ```css
    <div id = "comments">
    </div>
    ```

经过近一天的折腾，博客已然可用，没有太多兴奋，因为目的不是搭建，而是使用和坚持，技术点滴的记录积累，犯过的错误要避免出现，要细心，静心。



