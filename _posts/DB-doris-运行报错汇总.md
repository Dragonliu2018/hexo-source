---
title: '[DB][doris] 运行报错汇总'
categories:
  - [DB,doris]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-26 09:08:53
tags:
---

# Cluster default_cluster has no available capacity
【**问题**】

建表报错：

```sql
MySQL [demo]> create table mytable (
	k1 TINYINT,
  k2 DECIMAL(10, 2) DEFAULT "10.05",
  k3 CHAR(10) COMMENT "string column",
  k4 INT NOT NULL DEFAULT "1" COMMENT "int column" )
  COMMENT "my first table"
  DISTRIBUTED BY HASH(k1)
  BUCKETS 1
  PROPERTIES ("replication_num" = "1");
ERROR 1105 (HY000): errCode = 2, detailMessage = Cluster default_cluster has no available capacity
```

***

【**原因**】


没有将 be 添加到集群。

```sql
MySQL [test]> show backends;
Empty set (0.03 sec)
```

***


【**解决**】

添加 be 到集群：

```sql
# ALTER SYSTEM ADD BACKEND "be_host_ip:heartbeat_service_port";
ALTER SYSTEM ADD BACKEND "127.0.0.1:9050";
```

ref: https://doris.apache.org/zh-CN/docs/2.1/gettingStarted/quick-start

# Failed to find 1 backends for policy

【**问题**】

建表报错：

```sql
MySQL [demo]> create table mytable (
	k1 TINYINT,
  k2 DECIMAL(10, 2) DEFAULT "10.05",
  k3 CHAR(10) COMMENT "string column",
  k4 INT NOT NULL DEFAULT "1" COMMENT "int column" )
  COMMENT "my first table"
  DISTRIBUTED BY HASH(k1)
  BUCKETS 1
  PROPERTIES ("replication_num" = "1");
ERROR 1105 (HY000): errCode = 2, detailMessage = Failed to find 1 backends for policy: cluster|query|load|schedule|tags|medium: default_cluster|false|false|true|[{"location" : "default"}]|HDD
```

***

【**原因**】

BE 的 IP 地址错误:

```sql
MySQL [demo]> show backends;
+-----------+-----------------+-----------+---------------+--------+----------+----------+---------------+---------------+-------+----------------------+-----------------------+-----------+------------------+---------------+---------------+---------+----------------+--------------------------+--------------------------------------+---------+---------------------------------------------------------------------------------------------------------------+----------+
| BackendId | Cluster         | IP        | HeartbeatPort | BePort | HttpPort | BrpcPort | LastStartTime | LastHeartbeat | Alive | SystemDecommissioned | ClusterDecommissioned | TabletNum | DataUsedCapacity | AvailCapacity | TotalCapacity | UsedPct | MaxDiskUsedPct | Tag                      | ErrMsg                               | Version | Status                                                                                                        | NodeRole |
+-----------+-----------------+-----------+---------------+--------+----------+----------+---------------+---------------+-------+----------------------+-----------------------+-----------+------------------+---------------+---------------+---------+----------------+--------------------------+--------------------------------------+---------+---------------------------------------------------------------------------------------------------------------+----------+
| 11002     | default_cluster | 127.0.0.1 | 9050          | -1     | -1       | -1       | NULL          | NULL          | false | false                | false                 | 0         | 0.000            | 1.000 B       | 0.000         | 0.00 %  | 0.00 %         | {"location" : "default"} | actual backend local ip: 172.17.0.26 |         | {"lastSuccessReportTabletsTime":"N/A","lastStreamLoadTime":-1,"isQueryDisabled":false,"isLoadDisabled":false} |          |
+-----------+-----------------+-----------+---------------+--------+----------+----------+---------------+---------------+-------+----------------------+-----------------------+-----------+------------------+---------------+---------------+---------+----------------+--------------------------+--------------------------------------+---------+---------------------------------------------------------------------------------------------------------------+----------+
```

***

【**解决**】

根据上面的 ErrMsg 提示，改正 BE 的 IP 地址:

```sql
MySQL [(none)]>  ALTER SYSTEM ADD BACKEND "172.17.0.26:9050";
```
