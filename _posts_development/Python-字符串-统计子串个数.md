---
title: Python-字符串-统计子串个数
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-04 11:11:56
---

# 1 前言

**参考**：[Python里统计一个字符串中子字符串的个数](https://blog.csdn.net/u014675548/article/details/46332843)

母串`str1 = 'ababa'`，子串`str2='aba'`，统计子串有两种情况：

1. 重复统计：被统计过的字符还可以参与统计，所以子串数量为3；
2. 不重复统计：被统计过的字符不再参与统计，所以子串数量为2。

# 2 实现

## 2.1 重复统计

```python
import re

s = "abababa"
reg = re.compile("(?=aba)")
length = len(reg.findall(s))
print(length)  # 3
```

## 2.2 不重复统计

```python
s = "abababa"
print(s.count('aba'))  # 2
```



