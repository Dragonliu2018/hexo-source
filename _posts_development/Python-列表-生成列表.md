---
title: Python-列表-生成列表
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-12 22:39:10
---

# 1 问题

需求：生成1-10的列表；生成10个0的列表

# 2 解决

```python
# list方法
init_list = list(range(1, 10+1))  # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# []方法
init_list = [i for i in range(1, 10+1)]  # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 循环方法
init_list = []
for i in range(1, 10+1):
    init_list.append(i)
print(init_list)  # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 乘法
init_list = [0] * 10  # [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```

# X 参考

* [python------用多种方法生成1到100的列表并打印](https://blog.csdn.net/QQ18180564/article/details/105839040)
