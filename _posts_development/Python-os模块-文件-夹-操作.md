---
title: Python-os模块-文件(夹)操作
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-23 16:47:23
---

# 1 判断文件(夹)是否存在

**参考**：[Python判断文件是否存在的三种方法](https://www.cnblogs.com/jhao/p/7243043.html)

`os`模块中的`os.path.exists()`方法用于检验文件是否存在。

判断文件(夹)是否存在：

```python
import os
os.path.exists(path)
#True

os.path.exists(path)
#False
```

**问题**：如果文件夹和文件同路径+同名会出现bug。

只检查文件：

```python
import os
os.path.isfile("test-data")
```

# 2 创建目录

**参考**：[python创建目录（文件夹）](https://blog.csdn.net/MuWinter/article/details/77215768)

```python
import os

os.path.exists(path) # 判断一个目录是否存在

os.makedirs(path) # 创建多级目录

os.mkdir(path) # 创建单级目录
```

# 3 复制文件

**参考**：[用Python复制文件的9个方法](https://zhuanlan.zhihu.com/p/35725217)

这是运行任何系统命令的最常用方式。使用 system() 方法，你可以调用 subshell 中的任何命令。在内部，该方法将调用 C 语言的标准库函数。该方法返回该命令的退出状态。

对于 Windows 系统：

```python
import os

os.system('copy 1.txt.py 2.txt.py')
os.system(f'copy {source} {target}')
```

对于 Liunx 系统：

```python
import os

os.system('cp 1.txt.py 2.txt.py')
```

速度较慢，大量文件复制效果不佳。

# 4 路径拼接

os.path.join()函数用于路径拼接文件路径：

```python
import os

dir = "xxx"
file_name = "xxx"

path = os.path.join(dir, file_name)
```

# 5 删除文件(夹)

**参考**：[Python中删除文件的几种方法](https://developer.51cto.com/article/648822.html)

**删除文件**：

```python
import os

os.remove(path)
```

**注意**：如果文件在之前被打开，会出现报错：

```python
    os.remove(file_path)
PermissionError: [WinError 32] 另一个程序正在使用此文件，进程无法访问。: 'data3\\dataset\\test\\Benign\\4447.txt'
```

所以删除前要关闭文件：

```python
file_path = os.path.join(type_dir, name)
f = open(file_path, "r")
f.close()
os.remove(file_path)
```

***

**删除目录**：Python中的`os.remove()`方法用于删除文件路径。此方法无法删除目录。如果指定的路径是目录，则该方法将引发OSError。可以使用下面代码删除目录：

```python
import os

os.rmdir(dir_path)
```



