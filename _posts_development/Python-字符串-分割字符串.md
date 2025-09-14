---
title: Python-字符串-分割字符串
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-03 13:13:34
---

# 1 前言

工作过程中，会有分割字符串的需求，有可能是单个分隔符，也有可能是多个分隔符。

**参考**：[python split使用多个分隔符分割字符串](https://blog.csdn.net/zhuzuwei/article/details/78886662)

# 2 实现

## 2.1 split()

**参考**：[Python split()方法](https://www.runoob.com/python/att-string-split.html)

**缺点**：python内建的`split()`函数只能使用**单个分隔符**，如果需要多个分隔符，需要使用下面的`re.split()`。

**描述**：通过指定分隔符对字符串进行切片，如果参数 `num` 有指定值，则分隔 `num+1` 个子字符串。

**语法**：`str.split(str="", num=string.count(str))`

**参数**：

* `str`：分隔符，默认为所有的空字符，包括空格、换行(\n)、制表符(\t)等。
* `num`：分割次数。默认为 -1, 即分隔所有。

**返回值**：返回分割后的字符串列表。

```python
txt = "phx what\tphx\nai";
print(txt.split())  # ['phx', 'what', 'phx', 'ai']
print(txt.split(' ', 1))  # ['phx', 'what\tphx\nai']
print(txt.split('phx'))  # ['', ' what\t', '\nai']
```

## 2.2 re.split()

**优点**：`re`模块的`split()`函数可以使用多个分隔符对句子进行分割，其中不同的分隔符要用 `|` 隔开。

```python
import re

txt = "phx what\tphx\nai";
print(re.split(" |\t", txt))  # ['phx', 'what', 'phx\nai']
```

