---
title: Python-字符串-字典与字符串互转
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-28 22:15:59
---

# 1 字符串转为字典

 如果要将字符串转为字典，则可以使用 eval，但是原来双引号会变为单引号。

```python
dict_str = """
{
    "key_a": 1,
    "key_b": 2,
}
"""
dict_new = eval(dict_str)
print(type(dict_new)) # <class 'dict'>
print(dict_new) # {'key_a': 1, 'key_b': 2}
```

# 2 字典转字符串

将字典转为字符串的方式，可以使用 str，但是原来双引号会变为单引号。

```python
dict_test = {
    "key_a": 1,
    "key_b": 2,
}
dict_str = str(dict_test)
print(type(dict_str)) # <class 'str'>
print(dict_str) # {'key_a': 1, 'key_b': 2}
```

使用 json.dumps，就可以保住双引号；如果不用考虑双引号，则使用 str，毕竟比 json.dumps 快了一个数量级。

```python
import json

dict_test = {
    "key_a": 1,
    "key_b": 2,
}
dict_str = json.dumps(dict_test)
print(type(dict_str)) # <class 'str'>
print(dict_str) # {"key_a": 1, "key_b": 2}
```

# X 参考

* [python字典转字符串](https://blog.csdn.net/Kester_/article/details/113132295)
