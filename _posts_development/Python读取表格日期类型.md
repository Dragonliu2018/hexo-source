---
title: Python读取表格日期类型
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-07-20 23:57:48
---

# 1 问题

读取excel表格，但是日期类型的cell取出来是个数字，需要改成日期类型。

# 2 解决

```python
# 转化为元组形式
xlrd.xldate_as_tuple(table.cell(2,2).value, 0)
(2014, 7, 8, 0, 0, 0)

# 直接转化为datetime对象
xlrd.xldate.xldate_as_datetime(table.cell(2,2).value, 1)
datetime.datetime(2018, 7, 9, 0, 0)

# 没有转化
table.cell(2,2).value
41828.0
```

# X 参考

* [[Python]xlrd 读取excel 日期类型2种方式](https://blog.csdn.net/orangleliu/article/details/38476881)
