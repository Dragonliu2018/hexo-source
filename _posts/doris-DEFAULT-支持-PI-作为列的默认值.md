---
title: '[doris][DEFAULT] 支持 PI 作为列的默认值'
categories:
  - [DB,doris]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-27 00:22:11
tags: 
  - pr
  - notion
---

# 1 issue 介绍

* **issue:** https://github.com/apache/doris/issues/34228
* **PR**：https://github.com/apache/doris/pull/36280

doris 建表时，可以指定字段的默认值，比如日期类型 date，指定默认值为 current_timestamp，如果向表中插入数据，日期字段缺失的话，会设置默认值 current_timestamp。

这个任务是添加默认值 PI，可以支持以下语法：

```sql
CREATE TABLE IF NOT EXISTS t1
(
    k TINYINT,
    v double not null DEFAULT PI
)
UNIQUE KEY(k)
DISTRIBUTED BY HASH(k)
PROPERTIES("replication_num" = "1");
```

# 2 实现

1. 这个任务涉及到修改语法，也就是词法分析和语法分析，doris 是基于 antlr4 实现的，在 columnDef 的 DEFAULT 关键字后边加上可选关键字 `PI`；
2. 然后就需要进行绑定，将 Java 中的类绑定到 AST 对应节点，antlr4 提供了 Visitor 去访问 ****AST 中的节点。在 doris 中调用 `LogicalPlanBuilder` 来基于 AST 节点去绑定数据库实体，从而构建逻辑计划树，这里在 ColumDef 逻辑计划中就需要增加 PI 的绑定；
3. PI 精度选择：采用 <math.h> 中的 PI 值
    
    ```cpp
    # define M_PI           3.14159265358979323846  /* pi */
    ```
    

# 3 测试


```sql
create database test PROPERTIES ("replication_allocation"= "tag.location.default: 1" );
```

```sql
  CREATE TABLE t1
  (
      k TINYINT,
      v1 double not null DEFAULT PI
  )
  UNIQUE KEY(K)
  DISTRIBUTED BY HASH(k)
  PROPERTIES("replication_num" = "1");
```

```sql
set enable_unique_key_partial_update=true;
set enable_insert_strict=false;
```

```sql
insert into t1 (k) values (1);
```

https://doris.apache.org/zh-CN/docs/sql-manual/sql-statements/Data-Manipulation-Statements/Manipulation/INSERT?_highlight=enable_unique_key_partial_update#description

```sql
  CREATE TABLE t1
  (
      k TINYINT,
      v1 date not null DEFAULT now
  )
  UNIQUE KEY(K)
  DISTRIBUTED BY HASH(k)
  PROPERTIES("replication_num" = "1");
```

# X 参考

- **default value →  hll_empty**: https://github.com/apache/doris/pull/34447
- **default value →  uuid**：https://github.com/apache/doris/pull/34211