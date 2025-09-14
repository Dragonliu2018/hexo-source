---
title: Python-列表-遍历
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-08 15:03:09
---

# 1 前言

最近在使用python操作mysql，涉及列表操作比较多，整理一下列表遍历。

主要copy于[python3 遍历列表list 四种方法](https://blog.csdn.net/whatday/article/details/100557888)

# 2 方法

## 2.1 in

```python
item_list = ['a', 'b', 'c', 'd', 'a']

for item in item_list:
    print(item_list.index(item), item)

'''
0 a
1 b
2 c
3 d
0 a
'''
```

**注意**：按顺序输出值；如果列表中有重复值，下标会返回第一个值的下标，如上面的第二个`a`，但是这种bug只需要添加一个计数变量就解决了。

## 2.2 enumerate()

`enumerate(sequence, [start=0])`，返回枚举对象(元组)

参数：

- `sequence`：一个序列、迭代器或其他支持迭代对象。
- `start`：下标起始位置。

 用法实例：

```python
item_list = ['a', 'b', 'c', 'd', 'a']

for item in enumerate(item_list):
    print(item)

'''
(0, 'a')
(1, 'b')
(2, 'c')
(3, 'd')
(4, 'a')
'''

for index, val in enumerate(item_list):
    print(index, val)

'''
0 a
1 b
2 c
3 d
4 a
'''
```

**注意**：按顺序输出值；下标会返回对应值的下标，解决了`2.1`的问题。

## 2.3 iter()迭代器

 `iter(object, [sentinel]) `函数用来生成迭代器，返回迭代对象。

 参数：

- `object`：支持迭代的集合对象。
- `sentinel`：如果传递了第二个参数，则参数 object 必须是一个可调用的对象（如，函数），此时，iter 创建了一个迭代器对象，每次调用这个迭代器对象的__next__()方法时，都会调用 object。

用法实例：

```python
item_list = ['a', 'b', 'c', 'd', 'a']

for item in iter(item_list):
    print(item)

'''
a
b
c
d
a
'''
```

**注意**：按顺序输出。

## 2.4 range()函数

`range(start, end, [step])` 函数返回类型是`ndarray`，可用`list()`返回一个整数列表，一般用在 `for` 循环中。

参数：

- `start`：计数从 `start` 开始，默认是从 `0` 开始，例如`range(5)`等价于`range(0,5)`；
- `end`：计数到 `end` 结束，但不包括 `end`。例如：`range(0, 5)`是 `[0, 1, 2, 3, 4]` 没有 `5`
- `step`：步长，默认为`1`。例如：`range(0, 5)` 等价于 `range(0, 5, 1)`

用法实例：

```python
item_list = ['a', 'b', 'c', 'd', 'a']

for i in range(len(item_list)):
    print(item_list[i])

'''
a
b
c
d
a
'''
```

**注意**：按顺序输出。

## 2.5 倒序遍历

### 2.5.1 reversed函数

```python
item_list = ['a', 'b', 'c', 'd', 'a']

for item in reversed(item_list):
    print(item)

'''
a
d
c
b
a
'''
```

### 2.5.2 range()函数

```python
item_list = ['a', 'b', 'c', 'd', 'a']

for i in range(len(item_list)-1, -1, -1):
    print(item_list[i])

'''
a
d
c
b
a
'''
```

### 2.5.3 切片

```python
item_list = ['a', 'b', 'c', 'd', 'a']

for item in item_list[::-1]:
    print(item)

'''
a
d
c
b
a
'''
```

# X 参考

* [Python 列表(List) 的三种遍历(序号和值)方法](https://www.cnblogs.com/pizitai/p/6398276.html)
* [python列表遍历（包括倒序遍历的三种方法）](https://www.cnblogs.com/GumpYan/p/12334839.html)
