---
title: Python-取整
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-11 16:25:36
---

# 1 向下取整——int()

```python
f1 = 1.23
print(int(f1))  # 1
f2 = 1.73
print(int(f2))  # 1
```

# 2 向上取整——ceil()

```python
import math

f1 = 1.23
print(math.ceil(f1))  # 2
f2 = 1.73
print(math.ceil(f2))  # 2
```

# 3 四舍五入——round()

```python
f1 = 1.23
print(round(f1))  # 1
f2 = 1.73
print(round(f2))  # 2
```

# 4 分别取整数和小数部分

```python
f1 = 1.23
print(math.modf(f1))  # (0.22999999999999998, 1.0)
```

至于小数部分不准确：涉及**浮点数在计算机中的表示**。在计算机中是无法精确的表示小数的，至少目前的计算机做不到这一点。Python 和 C 一样, 采用 `IEEE 754` 规范来存储浮点数。

# X 参考

* [python中取整数的四种方法](https://zhuanlan.zhihu.com/p/365406577)
