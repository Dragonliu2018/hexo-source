---
title: Python-列表-排序
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-08 15:02:59
---

# 1 前言

最近在使用python操作mysql，涉及列表操作比较多，整理一下列表排序。

主要copy于 [菜鸟教程——Python List sort()方法](https://www.runoob.com/python/att-list-sort.html)

# 2 sort()方法

`sort()` 函数用于对原列表进行排序，如果指定参数，则使用比较函数指定的比较函数。

`sort()`方法语法：

```python
list.sort(cmp=None, key=None, reverse=False)
```

**参数**：

- `cmp` -- 可选参数, 如果指定了该参数会使用该参数的方法进行排序。
- `key` -- 主要是用来进行比较的元素，只有一个参数，具体的函数的参数就是取自于可迭代对象中，指定可迭代对象中的一个元素来进行排序。
- `reverse` -- 排序规则，**reverse = True** 降序， **reverse = False** 升序（默认）。

**返回值**：

该方法没有返回值，但是会对列表的对象进行排序。

以下实例展示了 sort() 函数的使用方法：

```python
item_list = ['a', 'b', 'c', 'd', 'a']

print(item_list)  # ['a', 'b', 'c', 'd', 'a']
item_list.sort()
print(item_list)  # ['a', 'a', 'b', 'c', 'd']
```

关于`key`参数的使用：

```python
# 获取列表的第二个元素
def takeSecond(elem):
    return elem[1]


# 列表
random = [(2, 2), (3, 4), (4, 1), (1, 3)]

# 指定第二个元素排序
random.sort(key=takeSecond)

# 输出类别
print(random)  # [(4, 1), (2, 2), (1, 3), (3, 4)]
```

# X 参考

* 
