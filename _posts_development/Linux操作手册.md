---
title: Linux操作手册
tags:
categories:
  - 环境与工具
  - [操作系统, Linux]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-13 10:14:47
---

# 1 命令

## 1.1 基本

```sh
touch file #创建文件
mkdir dir #新建目录
cp -r src target #复制文件(夹)
rm -rf dir #强制删除文件(夹)
mv src target #移动或重命名文件(夹)
cd #切换目录
pwd #查看当前目录
man <command_name> #获得某个命令的说明和使用方式的详细介绍
who am i #查看用户
su <user> #可以切换到用户 user
su - <user> #切换用户，但是同时用户的环境变量和工作目录也会跟着改变成目标用户所对应的
sudo adduser <user> #新建用户
groups <user> #查看自己属于的用户组
sudo usermod -G sudo <user> #将用户添加到sudo用户组
sudo deluser <user> --remove-home #删除用户，参数是在删除用户时候会一并将该用户的工作目录一并删除
sudo groupdel <group> #删除用户组，倘若该群组中仍包括某些用户，则必须先删除这些用户后，才能删除群组
# 修改权限
chmod 777 file #读写执行rwx二进制位数分别是210，文件有三组固定的权限，拥有者，所属用户组，其他用户
chmod gou-rw file #g(group用户组)、o(others其他用户)、u(user用户)，+ - 表示增加和去掉相应的权限
file <file> #查看文件类型
#查看文件
cat -n file #参数是显示行号
nl -b a file #与上面一样
nl -b t file #只列出非空行，默认
head/tail -n 1 file #只看文件首部尾部一行
head/tail -f file #这个参数可以实现不停地读取某个文件的内容并显示，实现实时监视
```

## 1.2 压缩/解压

### 1.2.1 gzip

* 压缩后的格式为：*.gz

* 这种压缩方式不能保存原文件；且不能压缩目录

* 命令举例：

  ```shell
  #压缩
  gzip file
  #解压
  gunzip file.gz
  ```

### 1.2.2 tar

* 命令选项：

  ```shell
  -z(gzip)      用gzip来压缩/解压缩文件
  -j(bzip2)     用bzip2来压缩/解压缩文件
  -v(verbose)   详细报告tar处理的文件信息
  -c(create)    创建新的档案文件
  -x(extract)   解压缩文件或目录
  -f(file)      使用档案文件或设备，这个选项通常是必选的。
  ```

* 命令举例

  ```shell
  #压缩
  tar -zvcf file.tar.gz file
  tar -jvcf file.tar.bz2 file
  #解压
  tar -zvxf file.tar.gz 
  tar -jvxf file.tar.bz2
  ```

### 1.2.3 zip

* 与gzip相比：1）可以压缩目录； 2）可以保留原文件

* 命令选项：

  ```shell
  -r(recursive)    递归压缩目录内的所有文件和目录
  ```

* 命令举例：

  ```shell
  #压缩
  zip -r dir.zip dir
  #解压
  unzip dir.zip
  ```

# 2 快捷键

|      按键       |                     作用                     |
| :-------------: | :------------------------------------------: |
|    `Ctrl+c`     |               强行终止当前程序               |
|    `Ctrl+z`     | 将当前程序放到后台运行，恢复到前台为命令`fg` |
|    `Ctrl+a`     |      将光标移至输入行头，相当于`Home`键      |
|    `Ctrl+e`     |      将光标移至输入行末，相当于`End`键       |
|    `Ctrl+k`     |           删除从光标所在位置到行末           |
| `Alt+Backspace` |               向前删除一个单词               |
|  `Shift+PgUp`   |              将终端显示向上滚动              |
|  `Shift+PgDn`   |              将终端显示向下滚动              |

# 3 通配符

|          字符           |                    含义                     |
| :---------------------: | :-----------------------------------------: |
|           `*`           |              匹配 0 或多个字符              |
|           `?`           |              匹配任意一个字符               |
|        `[list]`         |         匹配 list 中的任意单一字符          |
|        `[^list]`        |   匹配 除 list 中的任意单一字符以外的字符   |
|        `[c1-c2]`        | 匹配 c1-c2 中的任意单一字符 如：[0-9\][a-z] |
| `{string1,string2,...}` | 匹配 string1 或 string2 (或更多)其一字符串  |
|       `{c1..c2}`        |       匹配 c1-c2 中全部字符 如{1..10}       |

# 4 目录结构

<img src="https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/4-1.png"/>
