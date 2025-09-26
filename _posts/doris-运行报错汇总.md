---
title: '[doris] 运行报错汇总'
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

# mysql 可以登录，不可以建立分区表 → 多个 be

【**问题**】

无法建立分区表。

***

【**原因 & 解决**】

原因是有多个 be 实例（对应多个网卡），删除多余 be即可。

```sql
SHOW PROC '/backends';
```

```sql
ALTER SYSTEM DROPP BACKEND "59982";
```

![](https://raw.githubusercontent.com/Dragonliu2018/hexo-images-bed/main/2025/Snipaste_2025-09-26_23-46-06.png)

# mysql 可以登录，不可以建立分区表 → 没有 be

【**问题**】

无法建立分区表，报错如下：

```sql
ERROR 1105 (HY000): errCode = 2, detailMessage = System has no available disk capacity or no available BE nodes
```

***

【**原因 & 解决**】

原因是没有 be 实例，添加一个 be 实例并且重启 be 即可：

````sql
-- ALTER SYSTEM ADD BACKEND "host:port";
ALTER SYSTEM ADD BACKEND "127.0.0.1:9050";
````

# mysql 可以登录，不可以建立分区表 → be 数量不够

【**问题**】

不可以建立分区表，报错如下：

```sql
ERROR 1105 (HY000): errCode = 2, detailMessage = replication num should be less than the number of available backends. replication num is 3, available backend num is 1
```

***

【**原因**】

be 数量不够。

***

【**解决**】

在创建 database 的时候进行指定：

```sql
create database test PROPERTIES ("replication_allocation"= "tag.location.default: 1" );
```

# mysql 可以登录，不可以建立分区表 → be 数量为0

【**问题**】

- **client 报错**

  ```bash
  ERROR 1105 (HY000): errCode = 2, detailMessage = errCode = 2, detailMessage = errCode = 2, detailMessage = replication num should be less than the number of available backends. replication num is 1, available backend num is 0
  ```

- **日志报错**

  查看日志发现 be 说找不到 fe，fe 找不到 be。

  ```bash
  I20240613 21:15:45.467008 1111236 mem_info.cpp:459] Refresh cgroup memory win, refresh again after 10s, cgroup mem limit: 9223372036854771712, cgroup mem usage: 13997985792, cgroup mem info cached: 0
  W20240613 21:15:49.361387 1110431 olap_server.cpp:714] Have not get FE Master heartbeat yet
  W20240613 21:15:50.491954 1109571 fragment_mgr.cpp:886] Could not find any running frontends, maybe we are upgrading? We will not cancel any running queries in this situation.
  I20240613 21:15:51.379978 1110598 task_worker_pool.cpp:686] waiting to receive first heartbeat from frontend before doing report
  I20240613 21:15:51.836712 1110309 wal_manager.cpp:480] Scheduled(every 10s) WAL info: [/home/zhenlong/code/doris/output/be/storage/wal: limit 795363328 Bytes, used 0 Bytes, estimated wal bytes 0 Bytes, available 795363328 Bytes.];
  W20240613 21:15:52.586161 1116046 status.h:423] meet error status: [INTERNAL_ERROR]invalid cluster id. ignore. Record cluster id =789047984, record frontend info . Invalid cluster_id=1306576199, invalid frontend info TFrontendInfo(coordinator_address=TNetworkAddress(hostname=172.17.1.228, port=9020), process_uuid=1718284452423) 
  
  	0#  doris::Status doris::Status::Error<6, true, int&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, int const&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > >(std::basic_string_view<char, std::char_traits<char> >, int&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >&&, int const&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >&&) at /home/zhenlong/code/doris/be/src/common/status.h:422
  	1#  doris::Status doris::Status::InternalError<true, int&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, int const&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > >(std::basic_string_view<char, std::char_traits<char> >, int&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >&&, int const&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >&&) at /home/zhenlong/code/doris/be/src/common/status.h:468
  	2#  doris::HeartbeatServer::_heartbeat(doris::TMasterInfo const&) at /home/zhenlong/code/doris/be/src/agent/heartbeat_server.cpp:116
  	3#  doris::HeartbeatServer::heartbeat(doris::THeartbeatResult&, doris::TMasterInfo const&) at /home/zhenlong/code/doris/be/src/agent/heartbeat_server.cpp:74
  	4#  doris::HeartbeatServiceProcessor::process_heartbeat(int, apache::thrift::protocol::TProtocol*, apache::thrift::protocol::TProtocol*, void*) at /home/zhenlong/code/doris/gensrc/build/gen_cpp/HeartbeatService.cpp:298
  	5#  doris::HeartbeatServiceProcessor::dispatchCall(apache::thrift::protocol::TProtocol*, apache::thrift::
  ```

***

【**解决**】删除 output 目录中的子目录 fe 和 be，重新编译安装成功了。

【**更新**】删除 `output/be/storage` 内容，重新启动 be。

# 执行回归测试时出现 jar 缺失报错

回归测试：

```shell
./run-regression-test.sh --run test_remove
```

报错：

```shell
Could not resolve dependencies for project org.apache.doris:regression-test:jar:1.0-SNAPSHOT: Could not find artifact jdk.tools:jdk.tools:jar:1.7 at specified path /usr/lib/jvm/java-11-openjdk-amd64/../lib/tools.jar
```

原因是 java11 下没有对应 jar 包，但是 java8 下有，所以 JAVA_HOME 改成 java8 即可：

```shell
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

# fe无法启动，log显示：image does not exist: /home/zhenlong/code/doris/output/fe/doris-meta/image/image.0

观察 output/fe/log/fe.warn.log，发现是因为磁盘空间不足造成的。

# be无法启动 -> Address already in use

【**问题**】

```shell
start BE in local mode
Doris BE server did not start correctly, exiting
*** Query id: 0-0 ***
*** is nereids: 0 ***
*** tablet id: 0 ***
*** Aborted at 1711251949 (unix time) try "date -d @1711251949" if you are using GNU date ***
*** Current BE git commitID: 2c377e69a9 ***
*** SIGSEGV address not mapped to object (@0x80) received by PID 2331840 (TID 2331840 OR 0x7f88bd98db80) from PID 128; stack trace: ***

W20240324 11:45:45.365067 2333118 doris_main.cpp:125] thrift internal message: TServerSocket::listen() Could not bind to port 9060
E20240324 11:45:45.365289 2333118 thrift_server.cpp:165] ThriftServer 'backend' (on port: 9060) exited due to TException: Could not bind: Address already in use
```

***

【**解决**】

若仍没有解决，则先kill掉be进程: `lsof -i:9060`

# client 无法登陆，9030 端口没开启

【**问题**】

```bash
WARN (UNKNOWN fe_7e29d323_27e2_4414_88e0_65bca5217033(-1)|1) [Env.notifyNewFETypeTransfer():2612] notify new FE type transfer: UNKNOWN
```

***

【**解决**】

清空fe的元数据目录doris-meta下的所有数据。重新启动即可：

https://blog.csdn.net/u011385544/article/details/118603874

# be 因为 swap 无法启动

/home/dragon/code/doris/output/be/bin/start_be.sh

去掉下面的：

```shell
if [[ "$(swapon -s | wc -l)" -gt 1 ]]; then
    echo "Disable swap memory before starting be"
    exit 1
fi
```

# （TO FIX）fe 无法启动：SLF4J: Class path contains multiple SLF4J bindings.

【**问题**】

- **报错**

  ```sql
  2024-06-12 17:58:35,795 INFO (main|1) [DorisFE.start():156] Doris FE starting...
  2024-06-12 17:58:35,798 INFO (main|1) [FrontendOptions.initAddrUseIp():101] local address: /192.168.1.103.
  2024-06-12 17:58:35,939 INFO (main|1) [ConsistencyChecker.initWorkTime():105] consistency checker will work from 23:00 to 23:00
  2024-06-12 17:58:36,027 ERROR (main|1) [Util.report():128] SLF4J: Class path contains multiple SLF4J bindings.
  2024-06-12 17:58:36,027 ERROR (main|1) [Util.report():128] SLF4J: Found binding in [jar:file:/home/dragon/code/doris/output/fe/lib/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  2024-06-12 17:58:36,027 ERROR (main|1) [Util.report():128] SLF4J: Found binding in [jar:file:/home/dragon/code/doris/output/fe/lib/log4j-slf4j-impl-2.18.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  2024-06-12 17:58:36,028 ERROR (main|1) [Util.report():128] SLF4J: See <http://www.slf4j.org/codes.html#multiple_bindings> for an explanation.
  2024-06-12 17:58:36,036 ERROR (main|1) [Util.report():128] SLF4J: Actual binding is of type [org.slf4j.impl.Reload4jLoggerFactory]
  2024-06-12 17:58:36,320 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] java.lang.ExceptionInInitializerError
  2024-06-12 17:58:36,320 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.catalog.AliasFunction.getExpr(AliasFunction.java:105)
  2024-06-12 17:58:36,321 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.catalog.AliasFunction.initBuiltins(AliasFunction.java:91)
  2024-06-12 17:58:36,321 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.catalog.FunctionSet.init(FunctionSet.java:97)
  2024-06-12 17:58:36,321 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.catalog.Env.<init>(Env.java:711)
  2024-06-12 17:58:36,321 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.catalog.EnvFactory.createEnv(EnvFactory.java:71)
  2024-06-12 17:58:36,321 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.catalog.Env$SingletonHolder.<clinit>(Env.java:656)
  2024-06-12 17:58:36,322 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.catalog.Env.getCurrentEnv(Env.java:815)
  2024-06-12 17:58:36,323 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.DorisFE.start(DorisFE.java:178)
  2024-06-12 17:58:36,323 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.DorisFE.main(DorisFE.java:95)
  2024-06-12 17:58:36,323 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] Caused by: java.lang.RuntimeException: Cannot find external parser table action_table.dat
  2024-06-12 17:58:36,323 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.analysis.SqlParser.loadTableFromFile(SqlParser.java:2964)
  2024-06-12 17:58:36,324 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	at org.apache.doris.analysis.SqlParser.<clinit>(SqlParser.java:604)
  2024-06-12 17:58:36,324 ERROR (main|1) [Throwable$WrappedPrintStream.println():763] 	... 9 more
  2024-06-12 17:58:36,324 WARN (main|1) [DorisFE.start():225] 
  java.lang.ExceptionInInitializerError: null
  	at org.apache.doris.catalog.AliasFunction.getExpr(AliasFunction.java:105) ~[doris-fe.jar:1.2-SNAPSHOT]
  	at org.apache.doris.catalog.AliasFunction.initBuiltins(AliasFunction.java:91) ~[doris-fe.jar:1.2-SNAPSHOT]
  	at org.apache.doris.catalog.FunctionSet.init(FunctionSet.java:97) ~[doris-fe.jar:1.2-SNAPSHOT]
  	at org.apache.doris.catalog.Env.<init>(Env.java:711) ~[doris-fe.jar:1.2-SNAPSHOT]
  	at org.apache.doris.catalog.EnvFactory.createEnv(EnvFactory.java:71) ~[doris-fe.jar:1.2-SNAPSHOT]
  	at org.apache.doris.catalog.Env$SingletonHolder.<clinit>(Env.java:656) ~[doris-fe.jar:1.2-SNAPSHOT]
  	at org.apache.doris.catalog.Env.getCurrentEnv(Env.java:815) ~[doris-fe.jar:1.2-SNAPSHOT]
  	at org.apache.doris.DorisFE.start(DorisFE.java:178) ~[doris-fe.jar:1.2-SNAPSHOT]
  	at org.apache.doris.DorisFE.main(DorisFE.java:95) ~[doris-fe.jar:1.2-SNAPSHOT]
  Caused by: java.lang.RuntimeException: Cannot find external parser table action_table.dat
  	at org.apache.doris.analysis.SqlParser.loadTableFromFile(SqlParser.java:2964) ~[doris-fe.jar:1.2-SNAPSHOT]
  	at org.apache.doris.analysis.SqlParser.<clinit>(SqlParser.java:604) ~[doris-fe.jar:1.2-SNAPSHOT]
  	... 9 more
  ```

***

【**解决**】

解决：删除 `output/fe`，重新编译 fe 即可。

**每次修改少量 fe 代码，重新编译 fe，都会遇到这种问题，待解决。**

**开发机上删除 `output/fe/doris_meta`，重新编译 fe 即可。**

参考：

- https://github.com/apache/doris/issues/29122
- https://github.com/apache/doris/discussions/6610

