---
title: Python-集合-操作教程
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-10 17:22:22
---

# 1 引言

集合（set）是一个无序的不重复元素序列。

主要copy于[菜鸟教程](https://www.runoob.com/python3/python3-set.html)

# 2 使用

## 2.1 创建

可以使用大括号 **{ }** 或者 **set()** 函数创建集合：

```python
set1 = {1, 2, 3}
print(set1)  # {1, 2, 3}

set2 = {'pear', 'apple'}
print(set2)  # {'apple', 'pear'}

set3 = set("apple")
print(set3)  # {'l', 'p', 'e', 'a'}

set4 = set({'pear', 'apple'})
print(set4)  # {'apple', 'pear'}

set5 = set(('pear', 'apple'))
print(set5)  # {'pear', 'apple'}
```

<font color=red>注意：创建一个空集合必须用 **set()** 而不是 **{ }**，因为 **{ }** 是用来创建一个空字典。</font>

## 2.2 集合运算

```python
>>> # 下面展示两个集合间的运算.
...
>>> a = set('abracadabra')
>>> b = set('alacazam')
>>> a                                  
{'a', 'r', 'b', 'c', 'd'}
>>> a - b                              # 集合a中包含而集合b中不包含的元素
{'r', 'd', 'b'}
>>> a | b                              # 集合a或b中包含的所有元素
{'a', 'c', 'r', 'd', 'b', 'm', 'z', 'l'}
>>> a & b                              # 集合a和b中都包含了的元素
{'a', 'c'}
>>> a ^ b                              # 不同时包含于a和b的元素
{'r', 'd', 'b', 'm', 'z', 'l'}
```

## 2.3 基本操作

```python
test_set = {1, 2}
# 添加元素——add(x)
##将元素 x 添加到集合 s 中，如果元素已存在，则不进行任何操作。
test_set.add(2)  # {1, 2}
test_set.add(3)  # {1, 2, 3}

# 添加元素——update(x)
##参数是列表，元组，字典等，int会报错
##x 可以有多个，用逗号分开。
test_set.update(4)  # TypeError: 'int' object is not iterable
test_set.update({4, 5})  # {1, 2, 3, 4, 5}
test_set.update([4, 5], [5, 6])  # {1, 2, 3, 4, 5, 6}
test_set.update((6, 7))  # {1, 2, 3, 4, 5, 6, 7}

# 移除元素——remove(x)
##将元素 x 从集合 s 中移除，如果元素不存在，则会发生错误。
test_set.remove(8)  # KeyError: 8
test_set.remove(7)  # {1, 2, 3, 4, 5, 6}

# 移除元素——discard(x)
##移除集合中的元素，且如果元素不存在，不会发生错误
test_set.discard(7)  # 不报错 {1, 2, 3, 4, 5, 6}
test_set.discard(6)  # {1, 2, 3, 4, 5}

# 移除元素——pop()
##随机删除集合中的一个元素
##对集合进行无序的排列，然后将这个无序排列集合的左面第一个元素进行删除。
test_set.pop()  # {2, 3, 4, 5}

# 计算元素个数——len
len(test_set)  # 4

# 判断元素是否存在于集合中——in
print(2 in test_set)  # True
print(1 in test_set)  # False
print(1 not in test_set)  # True

# 清空集合——clear
test_set.clear()
print(test_set)  # set()
```

