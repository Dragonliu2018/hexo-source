---
title: 《鸟哥的Linux私房菜基础学习篇（第四版）》第4章笔记与习题
tags:
categories:
  - 阅读
  - [操作系统, Linux]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-07-25 23:53:32
---

> 第4章：首次登录与在线求助

# 1 笔记

## 1.1 X Window 与命令行模式的切换

Linux 预设的情况下会提供六个Terminal 来让使用者登入， 切换的方式为使用：

* `[Ctrl] + [Alt] + [F2] ~ [F6]`：命令行模式登入 tty2 ~ tty6 终端；
* `[Ctrl] + [Alt] + [F1]` ：图形用户界面模式 tty1。

纯文本界面下 (不能有 X 存在) 启动窗口界面的作法：`startx`

在 X 的环境下想要『强制』重新启动 X 的组合按键为：『[alt]+[ctrl]+[backspace]』；

## 1.2 基本命令操作

* 显示日期与时间的命令date：

  ```sh
  [dragon@192 ~]$ date
  2022年 07月 27日 星期三 22:12:06 CST
  [dragon@192 ~]$ date +%Y/%m/%d
  2022/07/27
  [dragon@192 ~]$ date +%H:%M:%S
  22:12:57
  ```

* 显示日历的命令cal：

  ```sh
  [dragon@192 ~]$ cal
        七月 2022     
  日 一 二 三 四 五 六
                  1  2
   3  4  5  6  7  8  9
  10 11 12 13 14 15 16
  17 18 19 20 21 22 23
  24 25 26 27 28 29 30
  31
  
  [dragon@192 ~]$ cal [month] [year]
  ```

* 计算器bc：

  ```sh
  bc
  ```

## 1.3 热键

* `[Tab]`
  * 命令补全
  * 文件补全
  * 选项/参数补全
* `Ctrl + C`：中断目前程序
* `Ctrl + Q`：
  * 键盘输入结束
  * 相当于输入 exit
* `[shift]+{[PageUP]|[Page Down]}`：命令行界面翻页

## 1.4 在线求助

### 1.4.1 --help

eg: date的基本用法与选项参数：

```sh
date --help
```

### 1.4.2 man page

date的 manual(操作说明)：

```sh
[dragon@192 ~]$ man date
DATE(1)
# 括号内的数字
# 1表示用户在 shell 环境中可以操作的指令或可执行文件
# 5表示配置文件或者是某些文件的格式
# 8表示系统管理员可用的管理指令
```

常用的按键：

|    按键     |                           进行工作                           |
| :---------: | :----------------------------------------------------------: |
|   空格键    |                          向下翻一页                          |
| [Page Down] |                          向下翻一页                          |
|  [Page Up]  |                          向上翻一页                          |
|   [Home]    |                          去到第一页                          |
|    [End]    |                         去到最后一页                         |
|   /string   | 向『下』搜寻 string 这个字符串，如果要搜寻 vbird 的话，就输入 /vbird |
|   ?string   |                向『上』搜寻 string 这个字符串                |
|    n, N     | 利用 / 或 ? 来搜寻字符串时，可以用 n 来继续下一个搜寻 (不论是 / 或 ?) ，可以利用 N 来进行『反向』搜寻。举例来说，我以 /vbird 搜寻 vbird 字符串， 那么可以 n 继续往下查询，用 N 往上查询。若以 ?vbird 向上查询 vbird 字符串， 那我可以用 n 继续『向上』查询，用 N 反向查询。 |
|      q      |                     结束这次的 man page                      |

### 1.4.3 info page

```sh
info date
```

## 1.5 关机

### 1.5.1 sync

数据同步写入磁盘

### 1.5.2 shutdown

shutdown 指令会通知系统内的各个程序 (processes)，并且将通知系统中的一些服务来关闭。

```sh
[root@study ~]# shutdown -h now
立刻关机，其中 now 相当于时间为 0 的状态
[root@study ~]# shutdown -h 20:25
系统在今天的 20:25 分会关机，若在 21:25 才下达此指令，则隔天才关机
[root@study ~]# shutdown -h +10
系统再过十分钟后自动关机
[root@study ~]# shutdown -r now
系统立刻重新启动
[root@study ~]# shutdown -r +30 'The system will reboot' 再过三十分钟系统会重新启动，并显示后面的讯息给所有在在线的使用者
[root@study ~]# shutdown -k now 'This system will reboot' 
仅发出警告信件的参数！系统并不会关机啦！吓唬人！
```

### 1.5.3 reboot & halt & poweroff

```sh
[root@study ~]# sync; sync; sync; reboot  # 先数据同步写入磁盘 后重启
[root@study ~]# halt # 系统停止～屏幕可能会保留系统已经停止的讯息！
[root@study ~]# poweroff # 系统关机，所以没有提供额外的电力，屏幕空白！
```

# 2 习题

## 2.1 简单的查询一下，Physical console / Virtual console / Terminal 的说明为何？

console 有『控制台』的意思在里面，因此你可以这样看的：

* 实体控制面板：实体的屏幕、键盘、鼠标等界面，让妳可以使用该配备来操作系统的环境，就称为实体控制面板(Physical console)
* 虚拟控制台：由系统衍生出的虚拟控制面板，你可以透过该虚拟控制面板搭配你自己系统的实体配备，来操作远程系统的环境。每个虚拟控制台都是独立运作的。
* 终端机：你可以用该界面来取得一个可以控制系统的 shell 环境。

由这些定义来看，一般来说，我们取得可以与系统互动的环境，大致上都称为 terminal 就是了。

## 2.2 man page 显示的内容的文件是放置在哪些目录中？

放置在 /usr/share/man/ 与 /usr/local/man 等默认目录中。

## 2.3 当我输入 man date 时，在我的终端机却出现一些乱码，请问可能的原因为何？如何修正？

如果没有其他错误的发生，那么发生乱码可能是因为语系的问题所致。 可以利用 `export LANG=en_US.utf8` 或者是 `export LC_ALL=en_US.utf8` 等设定来修订这个问题。
