---
title: Python-全局变量-list
date: 2021-06-27 09:44:22
tags: 
categories:
  - [Python, 基础]
# keywords:
description:
top: #True #顶置
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
cover: https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/ddddd.jpg
toc:  #是否显示toc （除非特定文章设置，可以不写）
toc_number: #是否显示toc数字 （除非特定文章设置，可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不写）
---

# 1 List

**python中list作为全局变量无需global声明的原因**：

> https://www.cnblogs.com/yym2013/p/5915980.html

python中list变量作为全局变量时，在函数中可以直接修改。

而普通变量则需要先在函数中global声明，否则会报错。

例如：

```python
a = 1
def fun():
　　global a
　　a = 2
    
b = [1,2]
def fun():
　　b[0] = 2
```

在函数中直接修改list则是可以的。

原因是：

普通变量如果在函数中赋值

`a = 2`

会有歧义。因为它既可以是表示引用全局变量a，也可以是创建一个新的局部变量，所以在python中，默认它的行为是创建局部变量，除非显式声明global。

而对列表list变量进行赋值

`b[0] = 2`

则不会有歧义。它是“明确的”，因为如果把b当作是局部变量的话，它会报KeyError，所以它只能是引用全局的b,故不需要多此一举显式声明global。
