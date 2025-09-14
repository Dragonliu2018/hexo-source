---
title: Python-Mysql-sqlalchemy中的聚合函数
tags:
categories:
  - [Python, 数据库]
  - [数据库, MySQL]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-26 12:10:12
---

# 1 引入

**参考**：[SQL聚合函数](https://www.yiibai.com/sql/sql-aggregate-functions.html)

SQL聚合函数，包括：`AVG()`，`COUNT()`，`MIN()`，`MAX()`和`SUM()`。

SQL聚合函数计算一组值并返回单个值。 例如，平均函数(AVG)采用值列表并返回平均值。

# 2 实验

## 2.1 构建数据表

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import sessionmaker

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
    age = Column(Integer)


# 将模型映射到数据库中
# 即如果数据库中没有student表则创建映射表student
Base.metadata.create_all()


# 增：批量
def insert_batch(obj_list):
    session.add_all(obj_list)
    session.commit()


# 删除TestTable表
def del_table():
    Person.__table__.drop()


if __name__ == "__main__":
    obj_list = [Person(name="dragon", age=22),
                Person(name="pig", age=23),
                Person(name="SY", age=19),
                Person(name="cat", age=7),
                Person(name="toy", age=8),
                Person(name="sen", age=23),
                Person(name="tor", age=10),
                ]
    insert_batch(obj_list)
    # del_table()

```

数据表如下图所示：

<img src="https://s2.loli.net/2022/03/26/h8yAv2Ts1OfipVX.png" width = "800" height = "200" alt="图片名称" align=center id=120 />

**下面的聚合函数，需要导入`func`：**

```python
from sqlalchemy import func
```

## 2.2 AVG() - 返回集合中的平均值

**参考**：[SQLAlchemy（二)：SQLAlchemy对数据的增删改查操作、属性常用数据类型详解](https://blog.51cto.com/u_15127597/3800261)

```python
age_avg = session.query(func.avg(Person.age)).scalar()  # 16.0000
```

**注意**：这里的`age`虽然是int类型的，但是最后的平均值是“准确”的小数。

## 2.3 COUNT() - 返回集合中的项目数

**参考**：[SQLAlchemy —— query](https://www.jianshu.com/p/b03e20cde341)

返回查询结果的对象的数目，下面的例子是找到了`toy`和`tor`两个对象：

```python
u = session.query(Person).filter(Person.name.like('to%')).count()  # 2
```

需要单独计算某一类对象的数量，可以用 `func()` 和 `group_by()` 配合查询：

```python
u = session.query(Person.name, func.count(Person.name)).group_by(Person.name).all()
print(u)  # [('dragon', 1), ('pig', 1), ('SY', 1), ('cat', 1), ('toy', 1), ('sen', 1), ('tor', 1)]
```

返回查询得到的的数值，我们使用 `scalar()` 方法：

```python
u = session.query(func.count(Person.age)).scalar()  # 7
```

## 2.4 MIN() - 返回集合的最小值

**参考**：[SQLAlchemy（二)：SQLAlchemy对数据的增删改查操作、属性常用数据类型详解](https://blog.51cto.com/u_15127597/3800261)

```python
age_min = session.query(func.min(Person.age)).scalar()  # 7
```

## 2.5 MAX() - 返回集合的最大值

**参考**：[SQLAlchemy（二)：SQLAlchemy对数据的增删改查操作、属性常用数据类型详解](https://blog.51cto.com/u_15127597/3800261)

```python
age_max = session.query(func.max(Person.age)).scalar()  # 23
```

## 2.6 SUM() - 返回所有值的总和

**参考**：[sqlalchemy对数据进行求和](https://blog.csdn.net/weixin_43303732/article/details/86476069)

```python
age_sum = session.query(func.sum(Person.age)).scalar()  # 112
age_sum = session.query(func.sum(Person.age)).filter(Person.name.like("to%")).scalar()  # 18
```

# X 参考

* 
