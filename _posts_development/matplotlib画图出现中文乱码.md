---
title: matplotlib画图出现中文乱码
tags:
  - matplotlib
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-12 22:32:27
---

# 1 问题描述

使用matplotlib画图，其中x和y轴标题中出现中文乱码。

# 2 解决

在代码前面加入：

```python
from pylab import *
mpl.rcParams['font.sans-serif'] = ['SimHei']
mpl.rcParams['axes.unicode_minus'] = False
```

# 3 参考

* [python中画图显示中文乱码](https://blog.csdn.net/xjh163/article/details/101076551)

 
