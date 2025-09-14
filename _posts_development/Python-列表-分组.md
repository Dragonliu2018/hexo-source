---
title: Python-列表-分组
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-10 10:20:19
---

# 1 前言

最近在整python多线程，需要对文件列表进行分组，下面整理下有关list分组的内容。

# 2 分块

```python
num = 3
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
group_data = [data[i:i + num] for i in range(0, len(data), num)]
print(group_data)  # [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10]]
```

# 3 groupby分组

```python
from itertools import groupby

user_list = [
    {"uid": 1, "sex": "男", "age": 10},
    {"uid": 3, "sex": "男", "age": 20},
    {"uid": 4, "sex": "女", "age": 20},
    {"uid": 4, "sex": "女", "age": 31},
    {"uid": 2, "sex": "男", "age": 10}
]
# 多字段排序
user_sort = sorted(user_list, key=lambda x: (x["sex"], x["age"]))
# 多字段分组
user_group = groupby(user_sort, key=lambda x: (x["sex"], x["age"]))
for key, group in user_group:
    print(key, list(group))
"""
('女', 20) [{'uid': 4, 'sex': '女', 'age': 20}]
('女', 31) [{'uid': 4, 'sex': '女', 'age': 31}]
('男', 10) [{'uid': 1, 'sex': '男', 'age': 10}, {'uid': 2, 'sex': '男', 'age': 10}]
('男', 20) [{'uid': 3, 'sex': '男', 'age': 20}]
"""
```

<font color=red>注意分组之前应先使用分组字段排好序。</font>

# X 参考

* [python list按规定的范围分组](https://blog.csdn.net/qq_21570029/article/details/84062979)
* [Python list列表groupby分组用法](https://blog.csdn.net/xiaoc100200/article/details/111402566)
