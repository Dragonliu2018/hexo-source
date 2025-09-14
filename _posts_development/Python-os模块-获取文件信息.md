---
title: Python-os模块-获取文件信息
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-28 21:44:44
---

# 1 获取文件大小，创建时间，访问时间，修改时间

```python
import time
import os

'''把时间戳转化为时间: 1479264792 to 2016-11-16 10:53:12'''
def TimeStampToTime(timestamp):
    timeStruct = time.localtime(timestamp)
    return time.strftime('%Y-%m-%d %H:%M:%S',timeStruct)

'''获取文件的大小,结果保留两位小数，单位为MB'''
def get_FileSize(filePath):
    filePath = filePath.encode('utf-8')
    fsize = os.path.getsize(filePath)
    print(fsize, "B")
    print(fsize/float(1024), "KB")
    print(fsize/float(1024*1024), "MB")
    fsize = fsize/float(1024*1024)
    return round(fsize, 2)

'''获取文件的访问时间'''
def get_FileAccessTime(filePath):
    filePath = filePath.encode('utf-8')
    t = os.path.getatime(filePath)
    return TimeStampToTime(t)

'''获取文件的创建时间'''
def get_FileCreateTime(filePath):
    filePath = filePath.encode('utf-8')
    t = os.path.getctime(filePath)
    return TimeStampToTime(t)

'''获取文件的修改时间'''
def get_FileModifyTime(filePath):
    filePath = filePath.encode('utf-8')
    t = os.path.getmtime(filePath)
    return TimeStampToTime(t)


if __name__ == "__main__":
    file_path = "test1.py"
    print(get_FileSize(file_path))
    print(get_FileAccessTime(file_path))
    print(get_FileCreateTime(file_path))
    print(get_FileModifyTime(file_path))
```

# X 参考

* [python 获取文件大小，创建时间和访问时间](https://www.cnblogs.com/shaosks/p/5614630.html)
