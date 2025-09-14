---
title: Python-字符串-统计词频
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-09 18:44:46
---

# 1 问题引入

最近在做恶意样本的标签信息提取、投票表决方式确定样本的最终最终标签信息，恰好用到了词频统计，或者说查找数组中出现次数最多的元素。

# 2 词频统计

```python
txt = '''in a job interview or in a situation'''

# 分解并提取英文文章的单词
for s in ',.\n ':
    txt = txt.replace(s, ' ')
txt = txt.lower()  # 将字母变成小写，排除原文大小写差异对词频统计的干扰
word_list = txt.split()
print(word_list)  # ['in', 'a', 'job', 'interview', 'or', 'in', 'a', 'situation']

# 对每个单词进行计数
count = dict()
for i in word_list:
    count[i] = count.get(i, 0) + 1
print(count)  # {'in': 2, 'a': 2, 'job': 1, 'interview': 1, 'or': 1, 'situation': 1}
# 对单词的统计值从高到低进行排序
sorted_word = sorted(count.items(), key=lambda item: item[1], reverse=True)
print(sorted_word)  # [('in', 2), ('a', 2), ('job', 1), ('interview', 1), ('or', 1), ('situation', 1)]
```

# 2 查找数组(list)中出现次数最多的元素

## 2.1 np.argmax(np.bincount())

```python
import numpy as np

array = [0, 1, 2, 2, 3, 4, 4, 4, 5, 6]
print(np.bincount(array))  # [1 1 2 1 3 1 1]
print(np.argmax(np.bincount(array)))  # 4
```

`np.argmax`：就是返回数组中最大值对应的下标，
`np.bincount`：首先找到数组最大值max，然后返回0～max的各个数字出现的次数，<font color=red>只能处理不含负数的集合</font>

## 2.2 Counter().most_common

```python
from collections import Counter

array = [0, 1, 2, 2, 3, 4, 4, 4, 5, 6, 'aswd']
print(Counter(array))  # Counter({4: 3, 2: 2, 0: 1, 1: 1, 3: 1, 5: 1, 6: 1, 'aswd': 1})
print(Counter(array).most_common(1)[0][0])  # 4
```

Counter用来对数组中元素出现次数进行统计，然后通过most_common函数找到出现次数最多的元素。这种方法对于数组就没有过多限制，甚至是各种类型元素混合的数组也可以。<font color=red>数组只能是array，不能是ndarray。</font>

## 2.3 自己实现

```python
appear_times = {}
for label in [1, 1, 2, 3, 4, 5, 5, 5]:
    if label in appear_times:
        appear_times[label] += 1
    else:
        appear_times[label] = 1

most_common = max(appear_times, key=lambda x: appear_times[x])
print(appear_times)  # {1: 2, 2: 1, 3: 1, 4: 1, 5: 3}
print(most_common)  # 5
```

用max求最大值默认情况返回value值（出现次数）最大的key值（元素），而不是value值。

# X 参考

* [python之统计文本中出现最多的单词](https://blog.csdn.net/qq_43527713/article/details/114482509?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3.pc_relevant_default&utm_relevant_index=6)
* [python统计数组中出现次数最多的元素](https://blog.csdn.net/aofavx/article/details/103097217)
