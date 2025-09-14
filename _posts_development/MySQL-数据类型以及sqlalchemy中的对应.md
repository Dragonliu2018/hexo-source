---
title: MySQL-数据类型以及sqlalchemy中的对应
tags:
categories:
  - [Python, 数据库]
  - [数据库, MySQL]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-25 21:17:46
---

# 1 MySQL数据类型

* **参考**：[MySQL 数据类型](https://www.runoob.com/mysql/mysql-data-types.html)

MySQL 支持多种类型，大致可以分为三类：数值、日期/时间和字符串(字符)类型。

## 1.1 数值类型

|     类型     |                   大小                   |                     范围（有符号）                      |         范围（无符号）          |      用途      |
| :----------: | :--------------------------------------: | :-----------------------------------------------------: | :-----------------------------: | :------------: |
|   TINYINT    |                 1 Bytes                  |                       (-128，127)                       |            (0，255)             |    小整数值    |
|   SMALLINT   |                 2 Bytes                  |                     (-32768，32767)                     |           (0，65 535)           |    大整数值    |
|  MEDIUMINT   |                 3 Bytes                  |                   (-8388608，8388607)                   |         (0，16 777 215)         |    大整数值    |
| INT或INTEGER |                 4 Bytes                  |             (-2 147 483 648，2 147 483 647)             |       (0，4 294 967 295)        |    大整数值    |
|    BIGINT    |                 8 Bytes                  | (-9,223,372,036,854,775,808，9 223 372 036 854 775 807) | (0，18 446 744 073 709 551 615) |   极大整数值   |
|    FLOAT     |                 4 Bytes                  |                            -                            |                -                | 单精度浮点数值 |
|    DOUBLE    |                 8 Bytes                  |                            -                            |                -                | 双精度浮点数值 |
| DECIMAL或DEC | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 |                     依赖于M和D的值                      |         依赖于M和D的值          |     小数值     |

## 1.2 日期和时间类型

每个时间类型有一个有效值范围和一个"零"值，当指定不合法的MySQL不能表示的值时使用"零"值。

|   类型    | 大小 ( bytes) |                             范围                             |        格式         |           用途           |
| :-------: | :-----------: | :----------------------------------------------------------: | :-----------------: | :----------------------: |
|   DATE    |       3       |                    1000-01-01/9999-12-31                     |     YYYY-MM-DD      |          日期值          |
|   TIME    |       3       |                   '-838:59:59'/'838:59:59'                   |      HH:MM:SS       |     时间值或持续时间     |
|   YEAR    |       1       |                          1901/2155                           |        YYYY         |          年份值          |
| DATETIME  |       8       |           1000-01-01 00:00:00/9999-12-31 23:59:59            | YYYY-MM-DD HH:MM:SS |     混合日期和时间值     |
| TIMESTAMP |       4       | 1970-01-01 00:00:00/2038结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07 |   YYYYMMDD HHMMSS   | 混合日期和时间值，时间戳 |

## 1.3 字符串类型

|    类型    |         大小          |              用途               |
| :--------: | :-------------------: | :-----------------------------: |
|    CHAR    |      0-255 bytes      |           定长字符串            |
|  VARCHAR   |     0-65535 bytes     |           变长字符串            |
|  TINYBLOB  |      0-255 bytes      | 不超过 255 个字符的二进制字符串 |
|  TINYTEXT  |      0-255 bytes      |          短文本字符串           |
|    BLOB    |    0-65 535 bytes     |     二进制形式的长文本数据      |
|    TEXT    |    0-65 535 bytes     |           长文本数据            |
| MEDIUMBLOB |  0-16 777 215 bytes   |  二进制形式的中等长度文本数据   |
| MEDIUMTEXT |  0-16 777 215 bytes   |        中等长度文本数据         |
|  LONGBLOB  | 0-4 294 967 295 bytes |    二进制形式的极大文本数据     |
|  LONGTEXT  | 0-4 294 967 295 bytes |          极大文本数据           |

* char(n) 和 varchar(n) 中括号中 n 代表字符的个数，并不代表字节个数，比如 CHAR(30) 就可以存储 30 个字符。
* CHAR 和 VARCHAR 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。
* BINARY 和 VARBINARY 类似于 CHAR 和 VARCHAR，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。
* BLOB 是一个二进制大对象，可以容纳可变数量的数据。有 4 种 BLOB 类型：TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB。它们区别在于可容纳存储范围不同。
* 有 4 种 TEXT 类型：TINYTEXT、TEXT、MEDIUMTEXT 和 LONGTEXT。对应的这 4 种 BLOB 类型，可存储的最大长度不同，可根据实际情况选择。

# 2 MySQL、sqlalchemy与Python数据类型对应

|    Mysql类型     |    sqlalchemy类型     |    python类型     |                       说明                       |
| :--------------: | :-------------------: | :---------------: | :----------------------------------------------: |
|   TINYINT(1B)    |        Boolean        |       bool        |                      布尔值                      |
|   SMALLINT(2B)   |     SmallInteger      |        int        |             取值范围较小，一般为16位             |
|  MEDIUMINT(3B)   |                       |                   |                                                  |
| INT或INTEGER(4B) |        Integer        |        int        |                普通整数，一般32位                |
|    BIGINT(8B)    |      BigInteger       |     int/long      |                  不限精度的整数                  |
|    FLOAT(4B)     |         Float         |       float       |                      浮点数                      |
|    DOUBLE(8B)    |                       |                   |                                                  |
|   DECIMAL或DEC   |        Numeric        |  decimal.Decimal  |                      定点数                      |
|     DATE(3B)     |         Date          |   datetime.date   |                       日期                       |
|     TIME(3B)     |         Time          |     date.time     |                       时间                       |
|     YEAR(1B)     |                       |                   |                                                  |
|   DATETIME(8B)   |       DateTime        | datetime.datetime |                    日期和时间                    |
|  TIMESTAMP(4B)   |                       |                   |                                                  |
|       CHAR       |                       |                   |                                                  |
|     VARCHAR      |        String         |        str        | 变长字符串，而且String后面需要长度参数，否则报错 |
|     TINYBLOB     |                       |                   |                                                  |
|     TINYTEXT     |                       |                   |                                                  |
|       BLOB       |      LargeBinary      |        str        |            二进制文件，64K=65535bytes            |
|       TEXT       |         Text          |        str        |       变长字符串，64K=65535bytes(255bytes)       |
|    MEDIUMBLOB    |  LargeBinary(65536)   |        str        |            二进制，16M=16777215bytes             |
|    MEDIUMTEXT    |      Text(65536)      |        str        |          变长字符串，16M=16777215bytes           |
|     LONGBLOB     | LargeBinary(16777216) |        str        |           二进制，32M=4294967295bytes            |
|     LONGTEXT     |    Text(16777216)     |        str        |         变长字符串，32M=4294967295bytes          |
|       ENUM       |         Enum          |        str        |                    一组字符串                    |

# 3 代码实战

## 3.1 建立MySQL表

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import *
from sqlalchemy.orm import sessionmaker

HOST = '127.0.0.1'
PORT = 3306
USERNAME = 'xxx'  # mysql的用户名
PASSWORD = 'xxx'  # mysql的密码
DB = 'xxx'  # 数据库名称
# f代表可以使用{}引用变量
DB_URI = f'mysql+pymysql://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DB}'

# 创建引擎
engine = create_engine(DB_URI)
# 创建SQLORM基类，注意这里要加参数，参数为上一步创建的引擎
Base = declarative_base(engine)
# 构建session对象
session = sessionmaker(engine)()


# 创建实体表Data
class Data(Base):
    __tablename__ = "data_type"
    id = Column(Integer, primary_key=True)
    booleanData = Column(Boolean(0))  # TINYINT(1B)
    smallInteger = Column(SmallInteger)  # SMALLINT(2B)
    integer = Column(Integer)  # INT或INTEGER(4B)
    bigInteger = Column(BigInteger)  # BIGINT(8B)
    floatData = Column(Float)  # FLOAT(4B)
    numericData = Column(Numeric)  # DECIMAL或DEC
    dateData = Column(Date)  # DATE(3B)
    timeData = Column(Time)  # TIME(3B)
    dateTime = Column(DateTime)  # DATETIME(8B)
    stringData = Column(String(250))  # VARCHAR
    textData = Column(Text)
    mediumText = Column(Text(65536))
    longText = Column(Text(16777216))
    largeBinary = Column(LargeBinary)
    mediumBlob = Column(LargeBinary(65536))
    longBlob = Column(LargeBinary(16777216))
    enumData = Column(Enum('father', 'mother'))

# 将模型映射到数据库中
# 即如果数据库中没有student表则创建映射表student
Base.metadata.create_all()


# 删除Data表
def del_table():
    Data.__table__.drop()


if __name__ == "__main__":
    print("test data type")
    # del_table()

```

## 3.2 MySQL表结构

```sh
mysql> DESC data_type;
+--------------+-------------------------+------+-----+---------+----------------+
| Field        | Type                    | Null | Key | Default | Extra          |
+--------------+-------------------------+------+-----+---------+----------------+
| id           | int                     | NO   | PRI | NULL    | auto_increment |
| booleanData  | tinyint(1)              | YES  |     | NULL    |                |
| smallInteger | smallint                | YES  |     | NULL    |                |
| integer      | int                     | YES  |     | NULL    |                |
| bigInteger   | bigint                  | YES  |     | NULL    |                |
| floatData    | float                   | YES  |     | NULL    |                |
| numericData  | decimal(10,0)           | YES  |     | NULL    |                |
| dateData     | date                    | YES  |     | NULL    |                |
| timeData     | time                    | YES  |     | NULL    |                |
| dateTime     | datetime                | YES  |     | NULL    |                |
| stringData   | varchar(250)            | YES  |     | NULL    |                |
| textData     | text                    | YES  |     | NULL    |                |
| mediumText   | mediumtext              | YES  |     | NULL    |                |
| longText     | longtext                | YES  |     | NULL    |                |
| largeBinary  | blob                    | YES  |     | NULL    |                |
| mediumBlob   | mediumblob              | YES  |     | NULL    |                |
| longBlob     | longblob                | YES  |     | NULL    |                |
| enumData     | enum('father','mother') | YES  |     | NULL    |                |
+--------------+-------------------------+------+-----+---------+----------------+
```

# X 参考

* [Flask SQLAlchemy 数据类型和数据库的对应关系 ](https://www.cnblogs.com/hzjdpawn/p/12893270.html)
* 
