---
title: '[doris] 支持删除回收站内容'
categories:
  - [DB,doris]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-27 00:09:09
  - pr
  - notion
---

# 1 issue 介绍

* **issue:** https://github.com/apache/doris/issues/31348
* **PR**: https://github.com/apache/doris/pull/31893

回收站（Catalog Recycle Bin）：

---

将数据库、表和分区删除后，会将他们放到回收站，然后经过 `catalog_trash_expire_second` 秒(在 `fe.conf` 中设置)后会被删除。目标是提供类似下面的命令来立即从回收站中删除他们：

```sql
// drop the database, table and partition with the specified db_id
drop catalog recycle bin where 'DbId' = '';

// drop the table and partition with the specified table_id
drop catalog recycle bin where 'TableId' = '';

// drop the partition with the specified partition_id
drop catalog recycle bin where 'PartitionId' = '';
```

# 2 实现

1. 这个任务涉及到添加语法，doris 是基于 antlr4 实现的，在 statementBase 位置加上新语法 `drop catalog recycle bin`，这样就可以生成 AST（抽象语法树）；
2. 然后就需要进行绑定，将 Java 中的类绑定到 AST 对应节点，antlr4 提供了 Visitor 去访问 ****AST 中的节点。在 doris 中调用 `LogicalPlanBuilder` 来基于 AST 节点去绑定数据库实体，从而构建逻辑计划树，这里只需要记录 `id` 和 `idType` 即可；
3. catalog recycle bin 有 hashmap 存放 id 和具体信息的映射，我们根据 idType 和 id 去到 hashmap 搜索，如果没有对应 id 则抛出异常，否则进行删除操作；对于删除数据库的操作，其下的所有表和分区需要删除。

# 3 测试

```bash
./run-regression-test.sh --run test_drop_catalog_recycle_bin
```

# X 参考

- 相关代码在 `CatalogRecycleBin.java`，参考 `erasePartition`, `eraseTable` 和 `erasePartition` 如何将回收站内容删除。
- 从回收站恢复数据库、表和分区的命令：https://doris.apache.org/zh-CN/docs/admin-manual/data-admin/delete-recover/