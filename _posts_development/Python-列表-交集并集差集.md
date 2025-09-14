---
title: Python-列表-交集并集差集
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-03 13:45:23
---

# 1 list的集合运算

## 1.1 交集

**方法1**：朴素的遍历；如果列表出现重复元素，该方法会出现问题，**重复元素可能会保留**。

```python
# 无重复元素
a = [2, 3, 4, 5]
b = [2, 5, 8]
tmp = [val for val in a if val in b]
print(tmp)  # [2, 5]

# 有重复元素 
a = [2, 2, 3, 4, 5]
b = [2, 5, 8]
tmp = [val for val in a if val in b]
print(tmp)  # [2, 2, 5]
```

**方法2**：转化为set操作，**速度快**；**重复元素不会保留**。

```python
# 无重复元素
a = [2, 3, 4, 5]
b = [2, 5, 8]
print(list(set(a).intersection(set(b))))  # [2, 5]

# 有重复元素
a = [2, 2, 3, 4, 5]
b = [2, 5, 8]
print(list(set(a).intersection(set(b))))  # [2, 2, 5]
```

## 1.2 并集

**方法1**：

```python
a = [2, 3, 4, 5]
b = [2, 5, 8]
print(list(set(a+b)))  # [2, 3, 4, 5, 8]
```

**方法2**：

```python
a = [2, 3, 4, 5]
b = [2, 5, 8]
print(list(set(a).union(set(b))))  # [2, 3, 4, 5, 8]
```

## 1.3 差集

**方法1**：朴素的遍历；如果列表出现重复元素，该方法会出现问题，**重复元素可能会保留**。

```python
# 无重复元素
a = [2, 3, 4, 5]
b = [2, 5, 8]
tmp = [val for val in b if val not in a]  # b中有而a中没有的
print(tmp)  # [8]

# 有重复元素 
a = [2, 3, 4, 5]
b = [2, 5, 8, 8]
tmp = [val for val in b if val not in a]  # b中有而a中没有的
print(tmp)  # [8, 8]
```

**方法2**：转化为set操作，**速度快**；**重复元素不会保留**。

```python
# 无重复元素
a = [2, 3, 4, 5]
b = [2, 5, 8]
print(list(set(b).difference(set(a))))  # [8]

# 有重复元素 
a = [2, 3, 4, 5]
b = [2, 5, 8, 8]
print(list(set(b).difference(set(a))))  # [8]
```

# 2 set的集合运算

```python
s = set([3, 5, 9, 10, 20, 40])  # 创建一个数值集合
t = set([3, 5, 9, 1, 7, 29, 81])  # 创建一个数值集合

# t 和 s的并集 ,等价于t.union(s)
print(t | s)  # {1, 3, 5, 7, 40, 9, 10, 81, 20, 29}
# t 和 s的交集 ,等价于t.intersection(s)
print(t & s)  # {9, 3, 5}
# 求差集（项在t中，但不在s中）  ,等价于t.difference(s)
print(t - s)  # {81, 1, 29, 7}
# 对称差集（项在t或s中，但不会同时出现在二者中）,等价于t.symmetric_difference(s)
print(t ^ s)  # {1, 7, 40, 10, 81, 20, 29}
```

