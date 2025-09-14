---
title: Windows下安装与使用neo4j
tags:
categories:
  - [环境与工具]
  - [数据库, Neo4j]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-10 11:17:06
---

# 1 引言

最近在实现恶意样本行为可视化，使用 [python模块graphviz使用教程](dragonliu.tk/2022/02/03/python模块graphviz使用教程/) 可以达到既定要求；但是图数据库neo4j理论上也是个不错的选择，所以整理下。

思路：安装Neo4j数据库，python语言使用py2neo库进行使用。

# 2 安装Neo4j

主要copy于 [图数据库neo4j的安装与基本使用(一)](https://cloud.tencent.com/developer/article/1387732)

## 2.1 安装JDK

Neo4j是基于Java的图形[数据库](https://cloud.tencent.com/solution/database?from=10680)，运行Neo4j需要启动JVM进程，因此必须安装JAVA SE的JDK。从Oracle官方网站下载 [Java SE JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)，当前的版本是JDK8。<font color=red>java版本的高低决定了Neo4j的版本</font>

## 2.2 安装Neo4j

官网下载最新版本[Neo4j](https://neo4j.com/download/other-releases/)

Neo4j应用程序有如下主要的目录结构：

```sh
bin目录：用于存储Neo4j的可执行程序；
conf目录：用于控制Neo4j启动的配置文件；
data目录：用于存储核心数据库文件；
plugins目录：用于存储Neo4j的插件；
```

## 2.3 配置环境变量

创建主目录环境变量NEO4J_HOME，并把主目录设置为变量值。

<img src="https://s2.loli.net/2022/02/11/lkopux9h8Zn7PeE.png" width = "600" height = "200" alt="图片名称" align=center id=72 />

## 2.4 启动neo4j

### 2.4.1 通过控制台启动Neo4j程序

打开cmd，切换到Neo4j主目录下的`bin`目录，运行下面命令即可：

```sh
neo4j.bat console
```

<img src="https://s2.loli.net/2022/02/11/3d7LJMoKpr2WxNC.png" width = "800" height = "200" alt="图片名称" align=center id=73 />

<img src="https://s2.loli.net/2022/02/11/nFHPDW9Tfxz6LXY.png" width = "600" height = "300" alt="图片名称" align=center id=74 />

用户名和密码默认为`neo4j`，首次登陆需要修改密码(Neo4j)

<font color=red>java与neo4j版本不对应的报错：</font>

```python
警告: ERROR! Neo4j cannot be started using java version 1.8.0_301
警告: * Please use Oracle(R) Java(TM) 11, OpenJDK(TM) 11 to run Neo4j Server.
* Please see https://neo4j.com/docs/ for Neo4j installation instructions.
Invoke-Neo4j : This instance of Java is not supported
所在位置 D:\developer\neo4j\neo4j-community-4.4.3\bin\neo4j.ps1:21 字符: 7
+ Exit (Invoke-Neo4j -Verbose:$Arguments.Verbose -CommandArgs $Argument ...
+       ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
    + FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException,Invoke-Neo4j
```

当时我测试的版本(JDK8+Neo4j4.4.4)出现上面的报错，该换JDK8+Neo4j3.5.30后成功运行。

### 2.4.2 把Neo4j安装为服务（Windows Services）

请查阅  [图数据库neo4j的安装与基本使用(一)](https://cloud.tencent.com/developer/article/1387732)

# 3 python使用

python通过调用`py2neo` 库实现对数据库的操作。

* [py2neo官网](https://py2neo.org/v4/data.html#node-and-relationship-objects)

* [py2neo——Neo4j&python的配合使用](https://www.jianshu.com/p/a2497a33390f)
* [用Python中的py2neo库调用neo4j，搭建简单关联图谱](https://cloud.tencent.com/developer/article/1693445)

## 3.1 连接数据库

安装py2neo库：`pip install py2neo`

```python
from py2neo import Graph

test_graph = Graph(
    "http://localhost:7474",
    username="neo4j",
    password="Neo4j"  #修改后的密码
)
```

测试时出现报错：

```sh
ValueError: The following settings are not supported: {'username': 'neo4j'}
```

因为py2neo版本问题，需要做出以下修改：

```python
from py2neo import Graph

test_graph = Graph("http://localhost:7474", auth=("neo4j", "Neo4j"))
```

详见 [关于使用Py2neo连接Neo4j图数据库出现“ValueError: The following settings are not supported”报错的解决方案](https://blog.csdn.net/u010785550/article/details/116856031)

## 3.2 基本操作

参考：[py2neo在已有节点上批量创建关系](https://blog.csdn.net/michaelguangg/article/details/100704303)

```python
from py2neo import Graph, Node, Relationship, Subgraph

test_graph = Graph("http://localhost:7474", auth=("neo4j", "Neo4j"))

# 删除数据库中以往的图，确保在一个空白的环境中进行操作
test_graph.delete_all()

# 创建顶点
node_list = []
node_list.append(Node("Teacher", name="Alice"))  # label(type) name(显示值)
node_list.append(Node("Student", name="Bob"))
node_list.append(Node("Student", name="Dragon"))
node_list.append(Node("Student", name="Pig"))
##subgraph加快导入速度
nodes = Subgraph(node_list)
test_graph.create(nodes)

# 创建边
edge_list = []
##找到对应节点
node1 = test_graph.nodes.match(name="Alice").first()
node2 = test_graph.nodes.match(name="Bob").first()
edge_list.append(Relationship(node1, "teach", node2))
node1 = test_graph.nodes.match(name="Alice").first()
node2 = test_graph.nodes.match(name="Dragon").first()
edge_list.append(Relationship(node1, "teach", node2))
edges = Subgraph(relationships=edge_list)
test_graph.create(edges)
```

效果图如下：

<img src="https://s2.loli.net/2022/02/11/4Li9RKwmuN7t6kV.png" width = "700" height = "400" alt="图片名称" align=center id=75 />

# 4 拓展——SPADE

当时参加A-ST竞赛时使用过，本次测试没进行。

[Github-Spade](https://github.com/ashish-gehani/SPADE)

# X 参考

* 
