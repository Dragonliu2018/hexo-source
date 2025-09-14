---
title: Python-文件-读取整理
date: 2022-01-17 09:44:22
tags: 
  - 待补充
categories:
  - [Python, 基础]
# keywords:
description:
top: #True #顶置
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
cover: https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/ddddd.jpg
toc: true #是否显示toc （除非特定文章设置，可以不写）
toc_number: #是否显示toc数字 （除非特定文章设置，可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不写）
---

# 1 按行读取文本文件

```python
file_path = 'xxx'
with open(file_path) as f:
    for line in f.readlines():
        print(line)
```



[open详解](https://www.runoob.com/python/python-func-open.html)

