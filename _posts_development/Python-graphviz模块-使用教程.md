---
title: Python-graphviz模块-使用教程
tags:
  - Graphviz
categories:
  - [环境与工具]
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-03 11:11:14
---

# 1 情景引入

前几个月师姐给了个任务——恶意样本行为可视化，输入为xml文件，一个文件代表一个恶意样本。请教学姐学长同学等，加上观看效果图，得出实现思路：python解析xml文件，使用python模块graphviz进行可视化；之后使用图数据库neo4j进行对比。

之后发现neo4j创建多个图数据库比较麻烦，但是按照行为分类等都比较直观，所以做辅助作用应该不错。

<img src="https://s2.loli.net/2022/02/10/jnNf9Xrms8CYgxz.png" width = "600" height = "400" alt="图片名称" align=center id=65 />

下面对graphviz进行整理。

# 2 Graphviz

Graphviz 是一个自动排版的作图软件，可以生成 png pdf 等格式。

* [Graphviz官网](https://graphviz.org/documentation/)
* [Graphviz 画图的一些总结](https://www.cnblogs.com/shuqin/p/11897207.html)

# 3 python graphviz

python graphviz则是graphviz的python实现。我们可以通过python graphviz实现轻松完成各种流程图的绘制。

* [python graphviz官方文档](https://graphviz.readthedocs.io/en/stable/manual.html)
* [[python] python模块graphviz使用入门](https://blog.csdn.net/LuohenYJ/article/details/106172201)

## 3.1 安装

1. 安装python graphviz木块（本机使用的是python3.9）:

   ```sh
   pip install graphviz
   ```

2. 除了python安装外，本机还需要安装Graphviz[(下载页面)](https://www.graphviz.org/download/)，并确保包含dot可执行文件的目录在系统路径上。安装过程中需要添加PATH：

   <img src="https://s2.loli.net/2022/02/10/mJkzIuyaMLCQbAS.png" width = "400" height = "300" alt="图片名称" align=center id=35 />

   否则出现下面的报错：(重启Pycharm生效)

   ```sh
   graphviz.backend.execute.ExecutableNotFound: failed to execute WindowsPath('dot'), make sure the Graphviz executables are on your systems' PATH
   ```

   [graphviz.backend.ExecutableNotFound: failed to execute ['dot', '-Tpng', '-O', 't est.gv']问题解决方法](https://blog.csdn.net/lizzy05/article/details/88529030)

   解决报错：添加环境变量

   <img src="https://s2.loli.net/2022/02/10/Nbm9RlSDYFyxtZX.png" width = "800" height = "400" alt="图片名称" align=center id=66 />

## 3.2 使用

该graphviz模块提供了两个类：Graph和 Digraph。它们分别以DOT语言为无向图和有向图创建图描述。它们具有相同的 API。示例如下：

```python
from graphviz import Digraph

dot = Digraph(comment='恶意样本行为可视化', format='jpg')

# 初始化端点
dot.node(name='n1', label='端点A', color="#008000", fontcolor="#000000", style="filled", shape="rectangle", fontname="Microsoft YaHei")
dot.node(name='n2', label='端点B', color="#FFD700", fontcolor="#000000", style="filled", shape="circle", fontname="Microsoft YaHei")
dot.node(name='n3', label='端点C', color="#0000FF", fontcolor="#FFFFFF", style="filled", shape="oval", fontname="Microsoft YaHei")

# 布局 L-左 R-右 T-上 B-下
dot.attr(rankdir='LR')  # 自左向右
# dot.attr(rankdir='RL')
# dot.attr(rankdir='TB')  # 默认
# dot.attr(rankdir='BT')
# 初始化边
dot.edge('n1', 'n2', label="边1", color="#000000", fontcolor="#000000", style="", fontname="Microsoft YaHei")
dot.edge('n1', 'n3', label="边2", color="#FF0000", fontcolor="#FF0000", style="dashed", fontname="Microsoft YaHei")

# 保存文件，view=False时将不自动打开文件
# dot.render("test_graphviz.gv", format='jpg', view=True)
# 不保存文件，仅仅自动打开
u = dot.unflatten(stagger=1)
u.view()
```

显示如下：

<img src="https://s2.loli.net/2022/02/10/hID7tuFsmfijoQv.png" width = "400" height = "300" alt="图片名称" align=center id=67 />

## 3.3 中文乱码

如果出现中文乱码，多半是因为没有设置fontname为支持中文显示的字体，只需要在node或者edge中添加`fontname="Microsoft YaHei"`，即可正常显示。

* [参考](https://blog.csdn.net/MAILLIBIN/article/details/100581857)

## 3.4 端点、边、字体颜色

在node或者edge中添加`color="#000000", fontcolor="#000000"`即可，具体颜色可以参考：

* [官网颜色](https://graphviz.gitlab.io/doc/info/colors.html)
* [维基百科——X11名称编码](https://zh.wikipedia.org/wiki/%E7%BD%91%E9%A1%B5%E9%A2%9C%E8%89%B2)

## 3.5 端点的形状

在node中添加`shape="xxx"`即可，具体需要什么形状可以在[官网](https://graphviz.gitlab.io/doc/info/shapes.html)中查找。

## 3.6 Rank

[GraphViz DOT有向图 (四)node节点布局控制之rank,group,subgraph](https://blog.csdn.net/youwen21/article/details/98397954)

## 3.7 排版

如上面的示例，使用`dot.attr(rankdir='xx')`来确定排版，作用范围为使用该行代码之后，下一个`dot.attr(rankdir='xx')`代码之前。

## 3.8 name参数中不能有英文冒号

```python
dot.node(name='n1', label='n1')
dot.node(name='n2:3', label='n2:3')
dot.edge('n1', 'n2:3')
```

`dot.node(name="xxx")` 如果name参数含有英文逗号，则会报错，导致画图错误：

```sh
Warning: node n2, port 3 unrecognized
```

常见的思路是使用python将name参数改成中文冒号，label参数是原来的英文冒号，达到正确显示的目的。

```python
string = "http://gsstudio.info/panel"
label = repr(string)
name = repr(string.replace(":", "："))
```

`repr`函数是防止字符串转义的，详细见[博客——python中防止字符串转义](dragonliu.tk/2022/02/09/python中防止字符串转义/)。

## 3.9 报错：graphviz.backend.execute.CalledProcessError

**报错信息：**

```sh
subprocess.CalledProcessError: Command '[WindowsPath('dot'), '-Kdot', '-Tjpg', '-O', '57630-0.xml.gv']' returned non-zero exit status 3221225477.
...
graphviz.backend.execute.CalledProcessError: Command '[WindowsPath('dot'), '-Kdot', '-Tjpg', '-O', '57630-0.xml.gv']' returned non-zero exit status 3221225477. [stderr: b'dot: graph is too large for cairo-renderer bitmaps. Scaling by 0.876568 to fit\r\n']
```

**现象：**

`.gv`文件可以正常生成，图片文件`.jpg`无法生成。

定位报错语句：

```python
dot = Digraph(comment='恶意样本行为可视化', format='jpg')
...
dot.render(output_dir + "\\" + file_name + ".gv", format='jpg')
```

**原因：**

生成的graph太大，dot无法导出，换成`pdf`或`svg`就可以了，`png`和`jpg`都不行。

最后选择pdf格式，因为导出的文件小。

```python
dot = Digraph(comment='恶意样本行为可视化', format='pdf')
...
dot.render(output_dir + "\\" + file_name + ".gv", format='pdf')
```

<img src="https://s2.loli.net/2022/02/12/59CK8EwoYSsyNiJ.png" width = "900" height = "200" alt="图片名称" align=center id=76 />

# X 参考

* 



