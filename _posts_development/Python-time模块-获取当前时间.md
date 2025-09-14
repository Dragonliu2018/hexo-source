---
title: Python-time模块-获取当前时间
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-28 21:01:50
---

# 1 time.time()

- 返回当前时间戳，值为按秒计算的浮点数
- 表示从1970年1月1日0点0分开始，到当前时间，一共经历了多少秒

 ```python
 import time
 
 test_time = time.time()
 print(type(test_time))  # <class 'float'>
 print(test_time)  # 1646054545.0786133
 ```

# 2 time.localtime()

- 格式化时间戳为本地的时间，年月日，时分秒等信息
- 若未输入参数，默认当前时间

```python
import time

test_time = time.localtime()
print(type(test_time)) # <class 'time.struct_time'>
print(test_time) # time.struct_time(tm_year=2022, tm_mon=2, tm_mday=28, tm_hour=21, tm_min=29, tm_sec=32, tm_wday=0, tm_yday=59, tm_isdst=0)
print(test_time.tm_year) # 年：2022
print(test_time.tm_mon) # 月：2
print(test_time.tm_mday) # 日：28
print(test_time.tm_hour) # 时：21
print(test_time.tm_min) # 分：29
print(test_time.tm_sec) # 秒：32

t2 = time.time()
print(time.localtime(t2)) # time.struct_time(tm_year=2022, tm_mon=2, tm_mday=28, tm_hour=21, tm_min=32, tm_sec=44, tm_wday=0, tm_yday=59, tm_isdst=0)
```

# 3 time.asctime()

- 格式化时间
- 若未输入参数，默认当前时间

```python
import time

t1 = time.asctime()
print(type(t1)) # <class 'str'>
print(t1) # Mon Feb 28 21:37:53 2022

t2 = time.localtime(time.time())
print(time.asctime(t2)) # Mon Feb 28 21:37:53 2022

t3 = time.asctime(time.localtime())
print(t3) # Mon Feb 28 21:37:53 2022
```

# 4 time.strftime()

- 格式化时间

```python
import time

t1 = time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())
print(type(t1)) # <class 'str'>
print(t1) # 2022-02-28 21:39:55

t2 = time.strftime('%a %b %d %H:%M:%S %Y', time.localtime())
print(type(t2)) # <class 'str'>
print(t2) # Mon Feb 28 21:39:55 2022
```

# X 参考

* [Python获取当前时间（time模块）](https://blog.csdn.net/qq_36512295/article/details/99694528)
