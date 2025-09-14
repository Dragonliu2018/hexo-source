---
title: Python-多线程&多进程-编程教程
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-11 14:05:20
---

# 1 引言

最近在做恶意样本行为可视化，样本数量2w+，执行时间>24h，所以需要多任务处理来提高效率。

> 参考：[Python多进程和多线程（跑满CPU）](https://blog.csdn.net/qq_40317897/article/details/89921083)

任务可以理解为进程（process），如打开一个word就是启动一个word进程。在一个word进程之中不只是进行打字输入，还需要拼写检查、打印等子任务，我们可以把进程中的这些子任务称为线程（thread）。

同时执行多个任务通常各个任务之间并不是没有关联的，而是需要相互通信和协调，有时，任务1必须暂停等待任务2完成后才能继续执行，有时，任务3和任务4又不能同时执行，所以，多进程和多线程的程序的复杂度要远远高于我们前面写的单进程单线程的程序。

# 2 多任务实现的3种方式

## 2.1 多进程模式

启动多个进程，每个进程虽然只有一个线程，但多个进程可以一块执行多个任务。<font color=red>多个CPU核心可以一起做多个任务。</font>

进程执行带有参数的任务：

| 参数名 | 说明                                               |
| ------ | -------------------------------------------------- |
| args   | 以元祖的方式给任务传递参数, 顺序要正确             |
| kwrags | 以字典的方式给执行任务传递参数, key 跟参数名要一致 |

```python
import multiprocessing
import time
import os
import math

def func(data):
    print(data, '进程PID', os.getpid())
    print(data, '进程父进程编号', os.getppid())
    for i in data:
        time.sleep(0.5)


def test_multiprocess():
    data_list = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    core = 4  # 4进程并行
    mod = math.ceil(len(data_list) / core)  # 向上取整
    group_data = [data_list[i:i + mod] for i in range(0, len(data_list), mod)]
    for data in group_data:
        # process = multiprocessing.Process(target=func, args=(5,))
        process = multiprocessing.Process(target=func, kwargs={'data': data})
        process.start()


if __name__ == '__main__':
    test_multiprocess()
```

## 2.2 多线程模式

启动一个进程，在一个进程内启动多个线程，这样，多个线程也可以一块执行多个任务。<font color=red>单个CPU核心可以同时做多个任务。</font>

[菜鸟教程——多线程](https://www.runoob.com/python/python-multithreading.html)

## 2.3 多进程+多线程模式

启动多个进程，每个进程再启动多个线程，这样同时执行的任务就更多了，当然这种模型更复杂，实际很少采用。

# 3 进程线程对比

## 3.1 关系对比

1. 线程是依附在进程里边的, 没有进程就没有线程
2. 一个进程默认提供一条线程, 进程可以创建多个线程

## 3.2 区别对比

1. 进程创建的开销要比线程的开销要大
2. 进程是操作系统资源分配的基本单位, 线程是cpu 调度的基本单位
3. 线程不能独立执行, 必须依存进程

## 3.3 优缺点对比

1. 进程优缺点:

   优点：可以用多核

   缺点：资源开销大

2. 线程优缺点

   优点: 资源开销小

   缺点：不能使用多核

# X 参考

* [在Python中优雅地用多进程](https://zhuanlan.zhihu.com/p/340657122)
* [python多进程多线程,多个程序同时运行](https://blog.csdn.net/qq_43475705/article/details/115518463)
