---
title: Python-列表-将list中的值拼接成字符串
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-04 10:50:45
---

# 1 前言

在Python中，有时需要将list以字符串的形式输出。如`list = ['a', 'b', 'c']`，要求输出`abc`。

**参考**：[python技巧——将list中的每个int元素转换成str](https://cloud.tencent.com/developer/article/1389881)

# 2 实现

## 2.1 元素是字符或字符串

```python
test_list = ["h", "e", "l", "l", "o"]

print("".join(test_list))  # hello
print(",".join(test_list))  # h,e,l,l,o
```

## 2.2 元素中含有数字

### 2.2.1 新建list

```python
test_list = ["h", "e", "l", "l", "o", 6]
test_list_new = [str(x) for x in test_list]

print("".join(test_list_new))  # hello6
print(",".join(test_list_new))  # h,e,l,l,o,6
```

### 2.2.2 lambda表达式

```python
test_list = ["h", "e", "l", "l", "o", 6]
test_list_new = map(lambda x: str(x), test_list)

print("".join(test_list_new))  # hello6
```

