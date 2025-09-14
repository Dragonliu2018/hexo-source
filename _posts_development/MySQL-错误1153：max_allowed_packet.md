---
title: MySQL-错误1153：max_allowed_packet
tags:
categories:
  - [Python, 数据库]
  - [数据库, MySQL]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-20 14:53:01
---

# 1 问题

编码过程中，有个字段使用`TEXT`无法存储，改用`TEXT(65536)` 后成功存储，但是导入mysql时报错：

```sh
sqlalchemy.exc.OperationalError: (pymysql.err.OperationalError) (1153, "Got a packet bigger than 'max_allowed_packet' bytes")
```

原因：`TEXT(65536)`为16M，超过了上限`max_allowed_packet`。

# 2 解决

修改`max_allowed_packet`值即可，在Mysql命令行运行：

```sh
mysql> set global net_buffer_length=1000000; 
Query OK, 0 rows affected (0.00 sec)

mysql> set global max_allowed_packet=1000000000;
Query OK, 0 rows affected (0.00 sec)
```

# 3 参考

* [MySQL Error 1153 - Got a packet bigger than 'max_allowed_packet' bytes](https://stackoverflow.com/questions/93128/mysql-error-1153-got-a-packet-bigger-than-max-allowed-packet-bytes)
