---
title: Xshell连接Linux执行自己写的Hadoop MapReduce WordCount.jar
date: 2016-03-25 22:31:45
tags: [Hadoop,MapReduce]
categories: Hadoop
toc: true
---
今天翻了《Hadoop海量数据处理技术详解与项目实战》这本书。又看到了熟悉的Hello WordCount程序，之前曾经自己搭过单机和伪分布式的Hadoop平台，也运行过hadoop自带的示例。想在windows平台创建maven项目在intelliJ IDEA中重新手写并且运行无果，折腾很久还是生成jar包然后通过Xshell跑在虚拟机里。

<!--more-->

## 环境
1. Windows下IntelliJ IDEA Maven项目WordCount，生成jar包
2. Xshell连接已经搭好的Ubuntu Hadoop服务器

## 开始就遇到的问题
1. Not a valid JAR

    ![error1](/images/posts/2016.3.25/error1.png)

    乍看到这个错误，我以为是在windows下生成的jar包哪里不对，因为不熟练所以不自信。于是就准备下载hadoop-mapreduce-examples看看他们的程序和我的哪里不同。
    但是后面才意识到，这个错误的明显性，root目录下没有这个jar包的，要在这个jar包的存放路径下执行该语句，即应该写绝对路径就不会出现该问题了。

2. org.apache.hadoop.mapreduce.lib.input.InvalidInputException: Input path does not exist

    ![error2](/images/posts/2016.3.25/error2.png)

    该错误是因为向集群提交任务时文件的输入路径必须是HDFS上的路径，输出同样。因为基础知识的缺乏所以才会出现如此低级错误吧。

    解决：
    ![solve](/images/posts/2016.3.25/solve.png)

    hadoop fs 通过put参数实现从本地系统拷贝文件到DFS。

## 第一个MapReduce程序
上述问题都解决后重新运行程序，如下，终于正常执行。

![job](/images/posts/2016.3.25/job.png)

浏览器查看运行结果：

![success](/images/posts/2016.3.25/success.png)

----
总结：因为没有对MapReduce/HDFS文件系统/命令有基础的理论知识了解，导致在实验过程中才晕头转向，且并不是很想去主动询问同学，似乎那样就是太偷懒了，还是也许自己的心态不对。
