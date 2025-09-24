---
title: '[PL][Python] 常见 bug 汇总'
categories:
  - [PL,Python]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-24 23:47:16
tags:
---

# sort() & sorted()

对比两个列表是否含有相同的元素，不需要顺序一样。sort() 函数返回 None，因此不能直接使用 method-1 进行判断。 

```python
# method-1 error
if l1.sort() == l2.sort():
  xxx

# method-2 OK
if sorted(l1) == sorted(l2):
  xxx
```
