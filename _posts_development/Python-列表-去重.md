---
title: Python-列表-去重
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-06 20:37:31
---

# 1 前言

最近在使用python操作mysql，涉及列表操作比较多，整理一下列表去重。

主要copy于[Python 列表去重的4种方式及性能对比](https://zhuanlan.zhihu.com/p/364610029)

# 2 辅助数组

新建一个数组，遍历原数组，如果值不在新数组里便加入到新数组中：

```python
item_list = ['a', 'b', 'c', 'd', 'a']

unique = []
for item in item_list:
    if item not in unique:
        unique.append(item)

print(unique)  # ['a', 'b', 'c', 'd']
```

上面使用了数组作为存储对象，实际上如果我们改成集合存储去重后的结果，性能会快不少：

```python
item_list = ['a', 'b', 'c', 'd', 'a']

unique = set()
for item in item_list:
    if item not in unique:
        unique.add(item)

print(unique)  # {'b', 'a', 'd', 'c'}
```

因为集合和数组的内在数据结构完全不同，集合使用了哈希表，因此速度会比列表快许多，但缺点在于无序。

# 3 数组->集合->数组

直接对数组进行集合转化，然后再转回数组：

```python
item_list = ['a', 'b', 'c', 'd', 'a']

unique = list(set(item_list))

print(unique)  # ['d', 'a', 'b', 'c']
```

**注意**：这种去重方式是最快的，但正如前面所说，集合是无序的，将数组转为集合后再转为列表，就失去了原有列表的顺序。

# 4 保留原有数组顺序的去重（dict.fromkeys()）

Python3.6+使用`dict.fromkeys()`函数，可以保留原有数组的顺序并去重：

```python
item_list = ['a', 'b', 'c', 'd', 'a']

unique = list(dict.fromkeys(item_list))

print(unique)  # ['a', 'b', 'c', 'd']
```

当然，它会比单纯用集合进行去重的方式耗时稍微久一点；`dict.fromkeys()`仅在Python3.6及以上才支持。

***

在Python3.6以下，其实也存在fromkeys函数，只不过它由collections提供：

```python
from collections import OrderedDict

item_list = ['a', 'b', 'c', 'd', 'a']
unique = list(OrderedDict.fromkeys(item_list))
print(unique)  # ['a', 'b', 'c', 'd']
```

比 Python3.6 的内置`dict.fromkeys()`慢一些，这是因为`OrderedDict`是用纯Python实现的。

# X 参考

* 
