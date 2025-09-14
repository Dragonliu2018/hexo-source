---
title: Python-time模块-测量程序运行时间
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-09 16:48:03
---

# 1 问题引入

最近做到恶意样本可视化，跑脚本需要测量程序运行时间，下面是几种测量方法：

# 2 datetime.datetime.now()

```python
import datetime

starttime = datetime.datetime.now()
#long running
#do something other
endtime = datetime.datetime.now()
print((endtime - starttime).seconds)  # 秒 
```

`datetime.datetime.now()`获取的是当前日期，在程序执行结束之后，这个方式获得的时间值为**程序执行的时间**。

# 3 time.time()

```python
import time

start = time.time()
#long running
#do something other
end = time.time()
print(end - start)  # 秒 
```

`time.time()`获取自纪元以来的当前时间（以秒为单位）。如果系统时钟提供它们，则可能存在秒的分数。所以这个地方返回的是一个浮点型类型。这里获取的也是**程序的执行时间**。

# 4 time.clock() 或 time.perf_counter()

```python
# start = time.clock()
start = time.perf_counter()
#long running
#do something other
i = 0
while i < 1e8:
    i += 1
# end = time.clock()
end = time.perf_counter()
print(end - start)  #秒
```

`time.clock()`返回程序开始或第一次被调用`clock()`以来的CPU时间。 这具有与系统记录一样多的精度。返回的也是一个浮点类型。这里获得的是**CPU的执行时间**。

python3.8+ 不支持 clock 了，替换成 `time.perf_counter()` 就可以了

<font color=red>**注意**</font>：程序执行时间=cpu时间 + io时间 + 休眠或者等待时间

# X 参考

* [几种Python执行时间的计算方法](https://blog.csdn.net/wangshuang1631/article/details/54286551)

