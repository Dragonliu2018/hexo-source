---
title: '[doris][insert] 支持具有 CTE 的 Insert'
categories:
  - [DB,doris]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-27 00:17:55
tags:
---

# 1 介绍

* **issue:** https://github.com/apache/doris/issues/35784
* **PR**: https://github.com/apache/doris/pull/36150

让 doris(优化器 nereids) 支持具有 CTE 的 Insert，语法如下：

```sql
WITH cte1 as (
  ...
)
INSERT INTO tbl1
SELECT ...
```

# 2 实现

1. 这个任务涉及到修改语法，也就是词法分析和语法分析，doris 是基于 antlr4 实现的，在 insert 位置加上可选关键字 CTE（`cte?`），这样就可以生成 AST（抽象语法树）；
2. 然后就需要进行绑定，将 Java 中的类绑定到 AST 对应节点，antlr4 提供了 Visitor 去访问 AST 中的节点。在 doris 中调用 `LogicalPlanBuilder` 来基于 AST 节点去绑定数据库实体，从而构建逻辑计划树，这里就需要增加 CTE 的绑定，并且在构建 InsertCommand （逻辑计划节点）的时候将 CTE 变量传进去；
3. InsertCommand 里面调用 initPlan() 进行逻辑计划改写，主要是把 cte 作为孩子节点挂到原来的逻辑计划树。

# 3 测试

## 3.1 InsertIntoTableCommand

```sql
create database test;
use test;

create table t1 (val int) PROPERTIES('replication_num'='1');
create table t2 (id int, val int) unique key(id) DISTRIBUTED BY HASH (id) PROPERTIES('replication_num'='1');

insert into t1 values(1);

with cte1 as (select * from t1 where val <= 3) insert into table t2 select val+10, val+100 from cte1;

select * from t2;
```

## 3.2 InsertOverwriteTableCommand

```sql
create database test;
use test;

create table t1 (val int) PROPERTIES('replication_num'='1');
create table t2 (id int, val int) unique key(id) DISTRIBUTED BY HASH (id) PROPERTIES('replication_num'='1');

insert into t1 values(1);

with cte1 as (select * from t1 where val <= 3) insert overwrite table t2 select val+10, val+100 from cte1;
insert overwrite table t2 select val+10, val+100 from t1;

select * from t2;
```

```sql
insert overwrite table t1 partition(p2) select * from t2;
with cte1 as (select 4) insert overwrite table t1 partition(p2) select * from cte1;
```

```sql
insert overwrite table t1 partition(*) select * from t2;
with cte1 as (select 1) insert overwrite table t1 partition(*) select * from cte1;
```

## 3.3 BatchInsertIntoTableCommand

```sql
sql """begin"""
sql """insert into $t1 select * from $t2"""
sql """insert into $t1 select * from $t3"""
sql "commit"

// sql """insert into $t1 values(1)"""
// sql """insert into $t1 values(2)"""
// sql """insert into $t1 values(3)"""
// sql """with cte1 as (select 1) insert into $t1 select * from cte1"""
// sql """with cte2 as (select 2) insert into $t1 select * from cte2"""
// sql """with cte3 as (select 3) insert into $t1 select * from cte3"""
```

# X 参考

- insert 使用文档：[链接](https://doris.apache.org/docs/dev/sql-manual/sql-statements/Data-Manipulation-Statements/Manipulation/INSERT/)
- insert overwrite 使用文档: [链接](https://doris.apache.org/docs/dev/sql-manual/sql-statements/Data-Manipulation-Statements/Manipulation/INSERT-OVERWRITE/)
- Transaction Load 使用文档：[链接](https://doris.apache.org/zh-CN/docs/dev/data-operate/import/transaction-load-manual/)
- insert & delete support CTE:  https://github.com/apache/doris/pull/23384
- 划分 InsertIntoTableCommand：https://github.com/apache/doris/pull/27947
- [**语法解析器ANTLR4从入门到实践**](https://juejin.cn/post/7018521754125467661)
