---
title: '[doris] PR TODO List'
tags:
---
下面可能存在问题:

- replace_empty 兼容性问题：https://github.com/apache/doris/pull/36283
- enable_unique_key_partial_update 部分更新问题：
    - **复现**
        
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
        
- nerids pi 函数实现

- [Featrue](default value) add e as default value 