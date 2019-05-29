---
layout: post
title: 互信息概念和应用举例
date: 2019-05-28 09:16:00
tags: 日志
categories: python
toc: true
---

最近在review组内的代码，发现一些模块中使用了大量的print来记录信息，一般来讲我们都推崇使用logging模块，无论是在Java语言下的log4j，还是C#语言下的Log4Net，那logging模块相比于print到底有什么优势呢？

## Print or Logging
代码中的print语句举例：

```python
print("0. Loading words need to be filtered...")
...
print("Done")
print("1. Reading ocr result...")
...
print("Done.")
...
print("2. Output finally result...")
...
print("Done"
print("All Done!")
```

乍一看没有什么问题，就是想记录并打印出当前流程的进展。无论是测试环境还是正式环境，我都需要这段信息打印，那么为什么非要改成`logging`模块来进行日志打印呢？

的确，在这个例子中无法体现`logging`模块的优势，其实在写一些小程序时，`print`已经够用了，但是当我们在做一个较多模块的项目时，`logging`模块的优势就逐渐体现出来了：

**使用logging模块会更加灵活**，项目的实施过程可能需要经过反复的开发、调试、测试和上线，我们在调试的过程中可能会打印出一些中间结果，包括代码段的运行时间等，这些在最终上线后是不需要的，如果是用`print`，可能需要注释或者删除掉，尤其是删除掉后再次调试又需要在加上，而注释掉更是降低了代码的简洁性。这个时候用`logging`模块，可以给你带来更多的助力：
1. 灵活配置信息层级（如DEBUG、INFO等），仅记录需要的信息。
2. 灵活配置日志记录的位置，在控制台打印 or 写入文件。
3. 灵活配置并统一日志格式。
4. 方便知晓日志输出来自哪个模块。
5. 能够统一配置触发某种层级的信息（如ERROR）后，发送邮件or短信通知报警等。

## Logging介绍
logging是python内置的日志模块，其默认的Level包括如下几项：
1. `DEBUG`：调试问题时常用。
2. `INFO`：一般信息，记录程序运行时常用
3. `WARNING`：警告，程序仍可正常运行，但可能有潜在问题。*是默认的level*
4. `ERROR`：错误
5. `CRITICAL`：严重错误

## Logging使用
1. 基础版本

    ```python
    import logging
    logging.warning('Watch out!')  # will print a message to the console
    logging.info('I told you so')  # will not print anything

    > 输出：
    WARNING:root:Watch out!
    ```

2. 在基础版本的基础上设置日志格式
3. 在基础版本的基础上设置写入日志到文件

4. logger的使用