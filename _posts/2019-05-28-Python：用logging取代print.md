---
layout: post
title: 用logging取代print
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
### 1. 基础版本

```python
import logging
logging.warning('Watch out!')  # will print a message to the console
logging.info('I told you so')  # will not print anything

> 输出：
WARNING:root:Watch out!
```

### 2. 设置日志格式和级别

```python
# 设置默认的level为DEBUG
# 设置log的格式
logging.basicConfig(
    level=logging.DEBUG,
    format="[%(asctime)s] %(name)s:%(levelname)s: %(message)s"  # 参数设置参考：https://docs.python.org/zh-cn/3.6/library/logging.html#logrecord-attributes
)
logging.warning('Warning')

> 输出：
[2019-05-30 08:57:05,543] root:WARNING: Warning
```

### 3. 设置写入日志到文件

```python
logging.basicConfig(filename='demo.log',
                level=logging.DEBUG,
                filemodel='w', # 设置追加写入文件
                format="[%(asctime)s] %(name)s:%(levelname)s: %(message)s")
logging.debug('Debug')
logging.info('Info')
logging.warning('Warning')

> 输出（在demo.log文件）：
[2019-05-30 09:13:27,284] root:DEBUG: Debug
[2019-05-30 09:13:27,284] root:INFO: Info
[2019-05-30 09:13:27,284] root:WARNING: Warning
```

### 4. Logger类的使用

#### 关键点：
1. `Logger`：主要包括三大类的方法和配置：
    * Logger.setLevel()
    * Logger.addHandler() / Logger.removeHandler()
    * Logger.addFilter() / Logger.removeFilter()

2. `Handlers`：负责将适当的日志消息分派给处理程序的指定目标。更一般的场景：将所有类型的日志消息写入日志文件，将错误或更高级别的日志消息打印到控制台，将关键错误或异常信息发送到邮件，这就需要单独的Handles来处理对应的事情。其方法和配置主要包括：
    * setLevel()
    * setFormatter()
    * addFilter() / removeFilter()

3. `Formatters`：格式化

#### 创建方法示例：
1. 通过Python代码显式创建loggers, handlers, formatters

    ```python
    # create logger
    logger = logging.getLogger('demo')
    logger.setLevel(logging.DEBUG)

    # create console handler and set level to debug
    ch = logging.StreamHandler()
    ch.setLevel(logging.DEBUG)

    # create formatter
    formatter = logging.Formatter('[%(asctime)s] %(name)s:%(levelname)s: %(message)s')

    # add formatter to ch
    ch.setFormatter(formatter)

    # add ch to logger
    logger.addHandler(ch)

    > 执行：
    logger.debug('debug message')
    logger.info('info message')
    logger.warning('warn message')
    logger.error('error message')
    logger.critical('critical message')

    > 输出：
    [2019-05-30 09:34:51,856] demo:DEBUG: debug message
    [2019-05-30 09:34:51,856] demo:INFO: info message
    [2019-05-30 09:34:51,856] demo:WARNING: warn message
    [2019-05-30 09:34:51,856] demo:ERROR: error message
    [2019-05-30 09:34:51,856] demo:CRITICAL: critical message
    ```
2. 通过配置文件

    **conf文件**

    调用fileConfig()函数设置
    ```
    [loggers]
    keys=root,simpleExample

    [handlers]
    keys=consoleHandler

    [formatters]
    keys=simpleFormatter

    [logger_root]
    level=DEBUG
    handlers=consoleHandler

    [logger_simpleExample]
    level=DEBUG
    handlers=consoleHandler
    qualname=simpleExample
    propagate=0

    [handler_consoleHandler]
    class=StreamHandler
    level=DEBUG
    formatter=simpleFormatter
    args=(sys.stdout,)

    [formatter_simpleFormatter]
    format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
    ```

    运行示例：

    ```python
    logging.config.fileConfig('logging.conf')
    logger = logging.getLogger("simpleExample")

    logger.debug('debug message')
    logger.info('info message')
    logger.warning('warn message')
    logger.error('error message')
    logger.critical('critical message')

    > 输出：
    2019-05-30 12:33:22,806 - simpleExample - DEBUG - debug message
    2019-05-30 12:33:22,806 - simpleExample - INFO - info message
    2019-05-30 12:33:22,806 - simpleExample - WARNING - warn message
    2019-05-30 12:33:22,806 - simpleExample - ERROR - error message
    2019-05-30 12:33:22,806 - simpleExample - CRITICAL - critical message
    ```
    **字典文件**

    python 3.2之后支持使用字典来保存配置信息的日志记录方法。可以使用不同的方式填充字典：如json or yaml等。如下是采用YAML格式的配置文件：
    ```yaml
    version: 1
    formatters:
    simple:
        format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    handlers:
    console:
        class: logging.StreamHandler
        level: DEBUG
        formatter: simple
        stream: ext://sys.stdout
    loggers:
    simpleExample:
        level: DEBUG
        handlers: [console]
        propagate: no
    root:
        level: DEBUG
        handlers: [console]
    ```

    运行示例：
    ```python
    with open('logging.yaml', 'r') as f:
        config = yaml.safe_load(f.read())
        logging.config.dictConfig(config)

    logger = logging.getLogger("simpleExample")

    logger.debug('debug message')
    logger.info('info message')
    logger.warning('warn message')
    logger.error('error message')
    logger.critical('critical message')

    > 输出：
    [2019-05-30 12:36:31,864] simpleExample:DEBUG: debug message
    [2019-05-30 12:36:31,864] simpleExample:INFO: info message
    [2019-05-30 12:36:31,864] simpleExample:WARNING: warn message
    [2019-05-30 12:36:31,865] simpleExample:ERROR: error message
    [2019-05-30 12:36:31,865] simpleExample:CRITICAL: critical message
    ```

### 5. 参考文档
1. [Logging HOWTO](https://docs.python.org/3.6/howto/logging.html)
2. [替换你的print（logging模块超简明指南）](https://www.zlovezl.cn/articles/replacing-print-simple-introduction-to-logging/)

