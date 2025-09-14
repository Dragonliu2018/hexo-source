---
title: Python-解决循环引用问题
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-09 16:11:23
---

# 1 引入

当项目中的模块过多，或功能划分不够清晰时会出现循环引用的问题。最近在做毕设，涉及后台管理系统时出现循环引用的问题。

主要copy于：[python解决循环引用问题](https://www.jianshu.com/p/a1e91cc53b07)

# 2 示例

`moduleA.py`：

```python
from moduleB import b

def a():
    print('aaaaaaaa')
    b()

def c():
    print('cccc')

if __name__ == '__main__':
    a()
```

`moduleB.py`：

```python
from moduleA import c

def b():
    print('bbbbbbbbbb')
    c()
```

运行`moduleA.py`出现以下报错：

```sh
ImportError: cannot import name 'b' from partially initialized module 'moduleB' (most likely due to a circular import)
```

导入其实是要将 `被导入模块所有的代码都执行一遍，遇到函数和类的定义会作申明`。回到循环引用中，首先导入B，进入B中，发现B中又导入了A又回到A中，但是A又导入B这就形成了循环引用。

# 3 解决

## 3.1 直接导入模块名，通过模块调用其中的函数

`moduleA.py`：

```python
import moduleB

def a():
    print 'aaaaaaaa'
    moduleB.b()
def c():
    print 'cccccc'


if __name__ == '__main__':
    a()
```

`moduleB.py`：

```python
import moduleA

def b():
    print 'bbbbbbbbbb'

    moduleA.c()
```

## 3.2 使用延迟导入(lazy import)

在需要用的函数内部导入，或是在底部导入。

`moduleA.py`：

```python
# 不变
from moduleB import b

def a():
    print('aaaaaaaa')
    b()

def c():
    print('cccc')

if __name__ == '__main__':
    a()
```

`moduleB.py`：

```python
# 方法1
def b():
    print('bbbbbbbbbb')
    c()

from moduleA import c


# 方法2
def b():
    from moduleA import c
    print('bbbbbbbbbb')
    c()
```

## 3.3 重新设计代码结构，将代码和并或者分离

将公共功能的代码分离成单独模块，或者将一些具有关系的代码合并成一个模块。

本次毕设便采用了这个方法：将一些公共功能的代码分离，config.py(数据库配置)、entity.py(实体类)

`config.py`：

```python
#!/usr/bin/python3
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine, Column, Integer, String, TEXT
from sqlalchemy.orm import sessionmaker

HOST = '127.0.0.1'
PORT = 3306
USERNAME = 'xxx'  # mysql数据库用户名
PASSWORD = 'xxx'  # 密码
DB = 'xxx'  # 数据库名称
# f代表可以使用{}引用变量
DB_URI = f'mysql+pymysql://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DB}'

# 创建引擎
engine = create_engine(DB_URI)
# 创建SQLORM基类，注意这里要加参数，参数为上一步创建的引擎
Base = declarative_base(engine)
# 构建session对象
session = sessionmaker(engine)()
```

`entity.py`：

```python
from config import *


# 创建Report表
class Report(Base):
    __tablename__ = 'report'  # 表名
    id = Column(Integer, primary_key=True, autoincrement=True)
    report_name = Column(String(100))
    report_size = Column(String(100))
    source_info = Column(String(100))
    operation = Column(String(100))
    tag = Column(String(100))
    hazard_level = Column(String(100))
    malware_static_info = Column(TEXT())
    malware_dynamic_info = Column(TEXT())
    upload_time = Column(String(100))


# 创建MidFid表
class MidFid(Base):
    __tablename__ = 'malware_mid_fid'  # 表名
    id = Column(Integer, primary_key=True, autoincrement=True)
    mid = Column(Integer())  # 样本id
    fid = Column(Integer())  # 家族id
```

# X 参考

* 
