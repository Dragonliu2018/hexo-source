---
title: MySQL-错误1366：字段编码报错
tags:
categories:
  - [Python, 数据库]
  - [数据库, MySQL]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-26 11:32:44
---

# 1 引入

在做毕设时，操作mysql时出现了这样的一个报错：

```sh
sqlalchemy.exc.DataError: (pymysql.err.DataError) (1366, "Incorrect string value: '\\xF2\\x88\\xBF\\xB6E ...' for column 'property' at row 1")
```

**原因**：UTF-8编码有可能是两个、三个、四个字节。Emoji表情或者某些特殊字符是4个字节，而Mysql的utf8编码最多3个字节，所以数据插不进去。

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


# 创建实体表Encode
class Encode(Base):
    __tablename__ = "string_encode"
    id = Column(Integer, primary_key=True)
    property = Column(TEXT)


# 将模型映射到数据库中
# 即如果数据库中没有student表则创建映射表student
Base.metadata.create_all()


# 增：一个
def insert(obj):
    session.add(obj)  # 添加到session
    session.commit()


# 删除Encode表
def del_table():
    Encode.__table__.drop()


if __name__ == "__main__":
    property = "📦 🚀 森罗万象"
    insert(Encode(property=property))

    # del_table()

```

出现报错：

```sh
sqlalchemy.exc.DataError: (pymysql.err.DataError) (1366, "Incorrect string value: '\\xF0\\x9F\\x93\\xA6 \\xF0...' for column 'property' at row 1")
[SQL: INSERT INTO string_encode (property) VALUES (%(property)s)]
[parameters: {'property': '📦 🚀 森罗万象'}]
(Background on this error at: https://sqlalche.me/e/14/9h9h)
```

## 2.2 尝试解决

### 2.2.1 Sqlalchemy 设置表编码及引擎

**参考**：[Sqlalchemy 设置表编码及引擎](https://blog.csdn.net/weixin_34015336/article/details/93760632)

1. 设置引擎编码方式为utf8：

   ```python
   DB_URI = f'mysql+pymysql://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DB}?charset=utf8'
   
   # 默认是utf8，所以上述代码无需修改
   DB_URI = f'mysql+pymysql://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DB}'
   ```

2. 设置数据库表编码方式为utf8mb4：

   ```python
   class Encode(Base):
       __tablename__ = "string_encode"
       id = Column(Integer, primary_key=True)
       property = Column(TEXT)
       __table_args__ = {
           "mysql_charset": "utf8mb4"
       }
   ```

3. 删除原来的表，再次生成表结构就可以正常插入了：

   <img src="https://s2.loli.net/2022/03/26/TlzLxG3yScio7EY.png" width = "800" height = "200" alt="图片名称" align=center id=119 />

**缺点**：需要重新构建表，所以旧表存在大量数据就会比较麻烦；在这种情况下可以使用下面的方法。

### 2.2.2 修改表的字符集为utf8mb4

**参考**：[mysql之（1366，"Incorrect string value:'\\xF0\\x9F\\x98\\x82...' for column 'content' at row 1")20](https://blog.csdn.net/qlzy_5418/article/details/91973353)

1. navicat右击需要修改的数据表，点击设计表；

2. 选择要修改的字段`property`，将字符集修改为`utf8mb4`，然后保存：

   <img src="https://s2.loli.net/2022/03/26/Ne1f5wn8GOLZqxt.png" width = "800" height = "300" alt="图片名称" align=center id=120 />

3. 现在就可以正常插入了：

   <img src="https://s2.loli.net/2022/03/26/8pfV65gsinLQFMS.png" width = "800" height = "300" alt="图片名称" align=center id=121 />

**缺点**：这种方法仅仅改变了当前表的字符集，如果以后项目迁移，重新生成表的情况，相同的问题将会再次出现。

# X 参考

* [Mysql错误1366的解决办法：Incorrect string value: '\xF0\x9F...' for column 'XXX' at row 1](https://blog.csdn.net/zz975896590/article/details/119991096)
* [pymysql.err.DataError: (1366, “Incorrect string value: ‘\\xF0\\x9F\\x92\\xA1 \\xE4...‘ for column ‘d](https://blog.csdn.net/wenxingchen/article/details/118021258)
