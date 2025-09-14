---
title: Python-文件-csv文件读取
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-03 10:09:58
---

# 1 前言

最近在做毕设，数据集使用的天池竞赛的，csv数据格式的，下面整理下csv读取的几种方法。

**主要参考**：[Python读取csv文件的几种方法](https://blog.csdn.net/domoNaruto/article/details/81193281)

下面是测试文件`test.csv` 的内容（与下面的python代码同目录）：

```csv
file_id,label,api,tid,index
1,5,LdrLoadDll,2488,0
1,5,LdrGetProcedureAddress,2488,1
1,5,LdrGetProcedureAddress,2488,2
1,5,LdrGetProcedureAddress,2488,3
1,5,LdrGetProcedureAddress,2488,4
```

# 2 读取操作

## 2.1 利用Python I/O读取文件

### 2.1.1 csv.reader()

```python
import csv


def read_csv(csv_path):
    data = []
    with open(csv_path) as f:
        csv_reader = csv.reader(f)  # 使用csv.reader读取f中的内容
        header = next(csv_reader)  # 读取第一行每一列的标题
        print(header)  # ['file_id', 'label', 'api', 'tid', 'index']
        for row in csv_reader:  # 将csv 文件中的数据保存到data中
            data.append(row[1])  # 选择某一列加入到data数组中
        print(data)  # ['5', '5', '5', '5', '5']
    f.close()


if __name__ == "__main__":
    csv_path = ".\\test.csv"
    read_csv(csv_path=csv_path)
```

### 2.1.2 csv.DictReader()

可以获取字段名，参考 [csv.DictReader 读取字段名（headers）](https://zhuanlan.zhihu.com/p/405847004)

```python
import csv


def read_csv(csv_path):
    with open(csv_path, 'r') as f:
        reader = csv.DictReader(f)
        headers = reader.fieldnames  # 字段名
        print(headers)  # ['file_id', 'label', 'api', 'tid', 'index']
        column = [row['index'] for row in reader]  # weight 同列的数据
        print(column)  # ['0', '1', '2', '3', '4']
    f.close()


if __name__ == "__main__":
    csv_path = ".\\test.csv"
    read_csv(csv_path=csv_path)
```

### 2.1.3 open()

把csv文件当作文本文件，使用`open()`和`split`进行处理，但是存在明显缺点——**如果字段中含有分隔符`,`，分割将会比较麻烦**。

```python
def read_csv(csv_path):
    with open(csv_path) as f:
        column = []
        column_id = 0
        for line in f.readlines():
            column.append(line.rstrip("\n").split(',')[column_id])
        print(column)  # ['file_id', '1', '1', '1', '1', '1']
    f.close()


if __name__ == "__main__":
    csv_path = ".\\test.csv"
    read_csv(csv_path=csv_path)
```

## 2.2 利用numpy读取

缺点：**csv里面的值只能是数值型的，字符串类型会报错，读取的值是按照float存储的**

解释：`delimiter`是分隔符，`skiprows`是跳过前`n`行，`usecols`是使用的列数，例子中读取的是`3,4`列。

```python
import numpy as np


def read_csv(csv_path):
    n = 1
    data = np.loadtxt(open(csv_path, "rb"), delimiter=",", skiprows=n, usecols=[2, 3])
    print(data)


if __name__ == "__main__":
    csv_path = ".\\test.csv"
    read_csv(csv_path=csv_path)
```

## 2.3 利用pandas读取

```python
import pandas as pd


def read_csv(csv_path):
    data = pd.read_csv(csv_path, sep=',', header='infer', usecols=[2])
    print(data)
    array = data.values[0::, 0::]  # 读取全部行，全部列
    print(array)  # array是数组形式存储，顺序与data读取的数据顺序格式相同
    print(array[0][0])  # LdrLoadDll


if __name__ == "__main__":
    csv_path = ".\\test.csv"
    read_csv(csv_path=csv_path)
```

# X 参考

