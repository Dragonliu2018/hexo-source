---
title: Python-字典-判断dict或list中是否含有某个元素或key
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-09 22:49:35
---

# 1 自带函数has_key()

在python的字典的属性方法里面有一个`has_key()`方法，这个方法使用起来非常简单。

Python 3.x不再支持 `has_key()` 函数，而被`__contains__()`所替代。

```python
d = {'name': {}, 'age': {}, 'sex': {}}
# print(d.has_key('name'))
print(d.__contains__('name'))  # True
```

# 2 in 方法

```python
# 字典是否含有key
d = {'name': {}, 'age': {}, 'sex': {}}
print("name" in d.keys())  # True
print("name" not in d.keys())  # False

# 列表是否含有value
l = ['name', 'age', 'sex']
print("name" in l)  # True
print("name" not in l)  # False
```

# X 参考

* [判断python字典或者列表中是否包含某个元素或者key](https://blog.csdn.net/linjpg/article/details/82190565)
