---
title: Python-字符串-截取特定部分
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-03 14:34:17
---

# 1 引入

截取字符串中特定部分的方法：可以使用 `str[beginIndex:endPosition]`，其中`str`为需要截取的字符串，`beginIndex`为需要截取的第一个字符的下标，`endPosition`为截取字符最后一个字符的下标。

该部分和 `Python-字符串-分割字符串` 有点类似。

**参考**：[python如何截取字符串中特定部分](https://www.php.cn/python-tutorials-465121.html)

# 2 解决

## 2.1 切片

`str[l,r]`：字符串可以理解成一个下标从0开始的数组，这里表示`[l,r)`的部分；`l`不写默认0，`r`不写默认最后一个；`l`和`r`也可以是负数，表示截取方向从右到左。

```python
test_str = "hello.python"
print(test_str[1:4])  # ell
print(test_str[:4])  # hell
print(test_str[1:])  # ello.python
print(test_str[-3:-1])  # ho
print(test_str[-3:])  # hon
```

## 2.2 index+切片

先获取字符的下标位置，再通过以上的方法截取。

`a.index(b, begin, end)`：`a`为需要校验字符串，`b`为字符串，`begin` 为开始截取的字符的下标（默认为`0`），`end`结束字符下标（默认为字符长度），截取字符串`[begin,end)`

```python
test_str = "hello.python"
ch = '.'

print(test_str.index(ch))  # 5
print(test_str[:test_str.index(ch)])  # Hello
print(test_str[test_str.index(ch):])  # .python
```

