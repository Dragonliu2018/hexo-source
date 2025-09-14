---
title: Python-字符串-去除首尾的指定字符
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-09 22:43:17
---

# 1 前言

最近在用到防止字符串转义函数——`repr`，但是他生成的字符串首尾会自动加上单引号，详见[博客](https://dragonliu.tk/2022/02/09/python%E4%B8%AD%E9%98%B2%E6%AD%A2%E5%AD%97%E7%AC%A6%E4%B8%B2%E8%BD%AC%E4%B9%89/)，可以使用`strip`函数进行去除。 

# 2 实践

```python
# strip不传参数默认去掉空格
test_str = '  string  '
# 去掉首尾空格
print(test_str.strip())  # string
# 去掉首空格
print(test_str.lstrip())  # string  
# 去掉尾空格
print(test_str.rstrip())  #   string

# 指定字符
str1 = "123\tabc"
str2 = repr(str1)
print(str1)  # 123	abc
print(str2)  # '123\tabc'
print(str2.strip("\'"))  # 123\tabc
```
