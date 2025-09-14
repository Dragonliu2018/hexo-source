---
title: MySQL-错误1406：超长字符串
tags:
categories:
  - [Python, 数据库]
  - [数据库, MySQL]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-25 23:52:12
---

# 1 引入

在做毕设时，操作mysql时出现了这样的一个报错：

```sh
sqlalchemy.exc.DataError: (pymysql.err.DataError) (1406, "Data too long for column 'actions' at row 1")
```

这里显示字段的长度不够存放数据，也就是解决mysql超长字符串存储问题。

# 2 解决

## 2.1 问题复现

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import *
from sqlalchemy.orm import sessionmaker

HOST = '127.0.0.1'
PORT = 3306
USERNAME = 'xxx'
PASSWORD = 'xxx'
DB = 'xxx'
# f代表可以使用{}引用变量
DB_URI = f'mysql+pymysql://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DB}'

# 创建引擎
engine = create_engine(DB_URI)
# 创建SQLORM基类，注意这里要加参数，参数为上一步创建的引擎
Base = declarative_base(engine)
# 构建session对象
session = sessionmaker(engine)()


# 创建实体表Length
class Length(Base):
    __tablename__ = "string_length"
    id = Column(Integer, primary_key=True)
    actions = Column(String(10))  # 修改此处debug


# 将模型映射到数据库中
# 即如果数据库中没有student表则创建映射表student
Base.metadata.create_all()


# 增：一个
def insert(obj):
    session.add(obj)  # 添加到session
    session.commit()


# 删除Data表
def del_table():
    Length.__table__.drop()


if __name__ == "__main__":
    actions = ''
    for i in range(10):  # 修改长度进行测试
        actions += "SY"
    insert(Length(actions=actions))
    del_table()

```

出现报错：

```sh
sqlalchemy.exc.DataError: (pymysql.err.DataError) (1406, "Data too long for column 'actions' at row 1")
[SQL: INSERT INTO string_length (actions) VALUES (%(actions)s)]
[parameters: {'actions': 'SYSYSYSYSYSYSYSYSYSY'}]
(Background on this error at: https://sqlalche.me/e/14/9h9h)
```

## 2.2 尝试解决

1. 修改String的长度参数，可以解决：

   ```python
   actions = Column(String(21842))
   ```

   **注意**：VARCHAR(0-65535 bytes)，这里测试String的长度参数最大是`21842`，具体意思与大小还不清楚。

2. 修改长度后就发生了之前的报错：

   ```python
   actions = ''
   for i in range(21842):
       actions += "SY"
   insert(Length(actions=actions))
   ```

   修改字段类型为`TEXT`可以解决上述问题：

   ```python
   actions = Column(TEXT)  # 64K TEXT
   actions = Column(TEXT(65536))  # 16M MEDIUMTEXT
   actions = Column(TEXT(16777216))  # 32M LONGTEXT
   ```

   现在字符串最大长度可以到32M，可以cover大部分情况。

# X 参考

* [mysql报错：“DataError: (pymysql.err.DataError) (1406, "Data too long for column 'pwd' at row 1”](https://blog.csdn.net/Darkman_EX/article/details/85211597)
* [pymysql.err.DataError: (1406, "Data too long for column '字段名' at row 1")](https://blog.csdn.net/qq_41251963/article/details/86659549)
* [pymysql.err.DataError: (1406, "Data too long for column 'url' at row 1")](https://blog.csdn.net/qq392039757/article/details/83617260)

