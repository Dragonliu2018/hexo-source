---
title: Python-Mysql-通过SQLAlchemy操作
tags:
categories:
  - [Python, 数据库]
  - [数据库, MySQL]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-01 15:01:57
---

# 1 简介

Windows下安装Mysql参考：[博客——Windows下安装MySQL](http://dragonliu.tk/2022/02/15/Windows%E4%B8%8B%E5%AE%89%E8%A3%85MySQL/)

SQLAlchemy是用Python编程语言开发的一个开源项目。它提供了SQL工具包和ORM（对象关系映射）工具，使用MIT许可证发行。

# 2 安装

```sh
pip3 install sqlalchemy
pip3 install pymysql
```

本文使用MySQL作为数据库，使用pymysql作为驱动，因此需要安装pymysql。

# 3 简单使用

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine, Column, Integer, String, TEXT
from sqlalchemy.orm import sessionmaker
from sqlalchemy import or_

HOST = '127.0.0.1'
PORT = 3306
USERNAME = 'xxx'  # mysql用户名
PASSWORD = 'xxx'  # mysql密码
DB = 'xxx'  # 数据库名称
# f代表可以使用{}引用变量
DB_URI = f'mysql+pymysql://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DB}'

# 创建引擎
engine = create_engine(DB_URI)
# 创建SQLORM基类，注意这里要加参数，参数为上一步创建的引擎
Base = declarative_base(engine)
# 构建session对象
session = sessionmaker(engine)()


# 创建实体表Person
class Person(Base):
    __tablename__ = 'person'  # 表名
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(100))
    value = Column(TEXT())


# 将模型映射到数据库中
# 即如果数据库中没有student表则创建映射表student
Base.metadata.create_all()


# 增：一个
def insert(obj):
    session.add(obj)  # 添加到session
    session.commit()


# 增：批量
def insert_batch(obj_list):
    session.add_all(obj_list)
    session.commit()


# 删
def delete():
    session.query(Person).filter(Person.name == 'Pig').delete()
    session.commit()


# 改
def update():
    # 将查找到的对象全部修改
    session.query(Person).filter(Person.name == 'dragon').update({'value': 'waiting'})
    session.commit()


# 查：所有数据
def select_all():
    item_list = session.query(Person).all()
    for item in item_list:
        print(item.name, item.value)
    # 若没有此句，后面执行删除表时会卡住
    session.commit()


# 查：指定列查询
def select_in_cols():
    item_list = session.query(Person.name).all()
    print(item_list)
    # 若没有此句，后面执行删除表时会卡住
    session.commit()


# 查：获取返回数据的第一行
def select_first_row():
    item = session.query(Person.name).first()
    print(item)
    session.commit()


# 查：使用filter()方法过滤
def select_filter():
    item_list = session.query(Person.name).filter(Person.value != "love").all()
    print(item_list)
    session.commit()


# 查：使用order_by()进行排序
def select_order():
    # 默认正序
    item_list = session.query(Person.name, Person.value).order_by(Person.name).all()
    print(item_list)
    # desc()倒序
    item_list = session.query(Person.name, Person.value).order_by(Person.name.desc()).all()
    print(item_list)
    session.commit()


# 查：多条件查询
def select_multi_condition():
    # 多条件查询条件以,分隔默认为and
    item_list = session.query(Person.name).filter(Person.name != "dragon", Person.value != "love").all()
    print(item_list)
    # 使用or_连接多个条件
    item_list = session.query(Person.name).filter(or_(Person.name != "dragon", Person.value != "love")).all()
    print(item_list)
    session.commit()


# 查：模糊查询
def select_like():
    item_list = session.query(Person.name).filter(Person.name.like('%ag%')).all()
    print(item_list)
    session.commit()


# 查：是否包含传递参数列表类元素
def select_in():
    item_list = session.query(Person.name).filter(Person.name.in_(["SY", "dragon"])).all()
    print(item_list)
    session.commit()


# 查：计算个数
def select_count():
    num = session.query(Person).count()
    print(num)
    session.commit()


# 删除TestTable表
def del_table():
    Person.__table__.drop()


if __name__ == "__main__":
    insert(Person(name="SY", value="afascinatinggril"))
    obj_list = [Person(name="dragon", value="aunreasoningboy"),
                Person(name="pig", value="aquerulousboy"),
                ]
    insert_batch(obj_list)
    delete()
    update()
    select_all()
    select_in_cols()
    select_first_row()
    select_filter()
    select_order()
    select_multi_condition()
    select_like()
    select_in()
    select_count()
    del_table()
```

**注意**：

1. 增删改查除了查询不设计修改操作，其他增删改都需要使用commit()方法提交事务
2. 执行查询操作后，不执行`session.commit()`，直接执行删除表操作会卡住

# X 参考

* [sqlalchemy的基本用法](https://www.cnblogs.com/minseo/p/15305003.html)

* [如何在SQLAlchemy中删除表？](https://www.cnpython.com/qa/33649)
