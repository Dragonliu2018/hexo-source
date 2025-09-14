---
title: CSAPP-Lab1-DataLab环境部署及题解
tags:
- Lab
categories:
  - 计组
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-09 01:26:20
---

# 1 介绍

该实验是《深入理解计算机系统》（英文缩写CSAPP）课程附带实验——Lab1：Data Lab。

大四下学期在做计组助教，需要部署Lab1，期间遇到了不少的问题，多亏了舍友的帮助，记录下。（🥦）

# 2 环境部署

部署时阅读`README`文件，包括子目录下的；项目使用Perl语言编写。

<img src="https://s2.loli.net/2022/04/12/e1jI2T7b3QhiqyU.png" width = "800" height = "300" alt="图片名称" align=center id=140 />

## 2.1 网站运行

在项目主目录运行`make start`：

```sh
[root@VM-4-8-centos lab1]# sudo make start
######################################
# start Data lab Online contest
######################################
(cd contest; make start)
make[1]: Entering directory '/root/Lab/lab1/contest'
/bin/sh: ./contest-timer.pl: Permission denied
/bin/sh: ./contest.pl: Permission denied
make[1]: *** [Makefile:13: start] Error 1
make[1]: Leaving directory '/root/Lab/lab1/contest'
make: *** [Makefile:15: start] Error 2
```

出现报错，是因为pl脚本无执行权限，需要加权限：

```sh
cd ./contest
chmod +x *.pl
```

之后报错：

```sh
Can't locate CGI.pm in @INC (you may need to install the CGI module)
```

原因是缺少CGI模块，[参考](https://blog.hostonnet.com/ah01215-cant-locate-cgi-pm-in-inc)

```sh
sudo yum install perl-CGI
```

此时浏览器访问`http://服务器IP:8080/`，（**注意服务器开启8080端口和8081端口**)

```html
Scoreboard for the Data Lab "Beat the Prof" Contest
Warning: The instructor (Tiger) must submit an entry before the results of the contest can be displayed.

To submit your instructor's entry: linux> ./src/driver.pl -u "The Prof"
```

这时候我们需要在后台执行提示的命令：

```sh
[root@VM-4-8-centos lab1]# ./src/driver.pl -u "The Prof"
-bash: ./src/driver.pl: Permission denied
```

报错显示需要加权限：

```sh
cd /root/Lab/lab1/src
chmod +x *.pl
```

之后再次报错：

```sh
[root@VM-4-8-centos lab1]# ./src/driver.pl -u "The Prof"
Can't locate Driverlib.pm in @INC (you may need to install the Driverlib module) (@INC contains: . /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5) at ./src/driver.pl line 19.
BEGIN failed--compilation aborted at ./src/driver.pl line 19.
```

这里的显示`Driverlib.pm `不存在，将往年Lab中的该文件复制到`/usr/lib64/perl5`（可变，只要在报错提示中的`@INC`中即可），然后再次执行：

```sh
[root@VM-4-8-centos lab1]# ./src/driver.pl -u "The Prof"
Can't locate Driverhdrs.pm in @INC (you may need to install the Driverhdrs module) (@INC contains: . /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5) at /usr/lib64/perl5/Driverlib.pm line 13.
BEGIN failed--compilation aborted at /usr/lib64/perl5/Driverlib.pm line 13.
Compilation failed in require at ./src/driver.pl line 20.
BEGIN failed--compilation aborted at ./src/driver.pl line 20.
```

`Driverhdrs.pm`文件与上面的`Driverlib.pm `文件同理，再次执行：

```sh
[root@VM-4-8-centos lab1]# ./src/driver.pl -u "The Prof"
./src/driver.pl: ERROR: No executable dlc binary.
```

进入`src`子目录下执行，出现报错：[参考](https://stackoverflow.com/questions/33059187/fatal-error-gnu-stubs-32-h-no-such-file-or-directory-but-stubs-64-h-is-prese)

```sh
[root@VM-4-8-centos src]# ./driver.pl -u "The Prof"
fatal error: gnu/stubs-32.h: No such file or directory" but stubs-64.h is present
```

```sh
sudo yum -y install glibc-devel.i686 glibc-devel
```

之后就可以成功执行了，但是前端页面仍然不变，需要提交如下指令：（非常坑，还是看文档不仔细，🥦

```sh
[root@VM-4-8-centos src]# ./driver.pl -u "Tiger"
```

在配置文件`./contest/Contest.pm`中可以查看：

```pm
$SERVER_NAME = "10.0.2.15";  # 需要修改成自己的服务器
$BASE_USERID = "Tiger";  # 初始用户名
$UPDATE_PERIOD = 10;  # 每10s刷新数据
$REQUESTD_PORT = 8080;  # Request server's port
$RESULTD_PORT = 8081;   # Result server's port
# ... 还有好多其他配置信息
```

到此前端可以正常显示了。

## 2.2 出题

具体操作参考`./src/dlcdir/README`

1. `./src/puzzles`目录下存放试题，新出的题目需要放在此处，由于本次实验出题主要更改原试题的名称，比较简单，魔改即可（包括文件名、函数名、注释等）

2. 将新出的题目名称添加到`./src/dlcdir/legallist.c`中的`legallist`中，魔改：

   ```c
   {{"isZero"}, 0, 3, {'~', '&', '!', '^', '|', '+', LS, RS, 0}},
   ```

3. 然后在`./src/dlcdir`目录下重新编译：

   ```
   make superclean
   make
   ```

4. 此时如果成功执行，`./src/dlcdir`目录下会生成`dlc`可执行文件，并将其复制到上级目录中；
5. 在`./src/selections.c`文件中存有你要选择出题的题目列表，默认13个，根据需求进行修改。

**下面是make过程中出现的报错记录：**

```sh
[root@VM-4-8-centos dlcdir]# make
bison -y -dv ANSI-C.y
make: bison: Command not found
make: *** [Makefile:40: y.tab.h] Error 127
```

原因是未安装bison，进行安装：[参考](https://blog.csdn.net/weborn/article/details/6794671)

```sh
yum install -y bison
```

然后继续报错：

```sh
[root@VM-4-8-centos dlcdir]# make
bison -y -dv ANSI-C.y
ANSI-C.y:653.15-16: error: $$ for the midrule at $4 of ‘declaring.list’ has no declared type
              $$ = AppendDecl($1, $3, Redecl);
               ^^
ANSI-C.y:708.15-16: error: $$ for the midrule at $4 of ‘default.declaring.list’ has no declared type
             { $$ = AppendDecl($1, $3, NoRedecl); }
               ^^
ANSI-C.y:1408.14-15: error: $$ for the midrule at $3 of ‘labeled.statement’ has no declared type
            { $$ = BuildLabel($1, NULL); }
              ^^
make: *** [Makefile:40: y.tab.h] Error 1
```

原因是bison版本太新，需要更换版本：[参考](https://lists.gnu.org/archive/html/help-bison/2009-03/msg00015.html)

查看版本：

```sh
[root@VM-4-8-centos dlcdir]# bison -V
bison (GNU Bison) 3.0.4
Written by Robert Corbett and Richard Stallman.

Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

删除bison：

```sh
yum remove bison.x86_64
```

下面更换bison版本：[参考](https://www.shuzhiduo.com/A/q4zVVOZWzK/)

1. 下载bison的压缩包，版本要**低于1.75**，[链接](http://ftp.gnu.org/gnu/bison/)

2. 安装：

   ```sh
   cd bison-x.x
   ./configure
   make
   sudo make install
   ```

***

bison版本正确后，下面继续做make，但是报了一大堆warning，然后报错：

```sh
heap.h:69:21: warning: inline function ‘HeapAllocate’ declared but never defined
 GLOBAL inline void *HeapAllocate(int number, int size);
...
/root/Lab/lab1/src/dlcdir/check.c:339: undefined reference to `MakeConstSlong'
/root/Lab/lab1/src/dlcdir/check.c:339: undefined reference to `MakeReturnCoord'
collect2: error: ld returned 1 exit status
make: *** [Makefile:36: dlc] Error 1
```

注意：这里搜索最后的报错部分无果，搜索warning部分可以解决。

这是内联函数`inline`引发的warning，添加编译选项`-fgnu89-inline`来修复，修改`./src/Makefile`，[参考](https://stackoverflow.com/questions/13120633/how-to-deal-with-warning-inline-function-stat64-declared-but-never-defined)

```makefile
_CFLAGS = -g  -O2 -fgnu89-inline
```

下面继续编译，之前需要清理下：

```sh
make clean
make 
```

现在就可以成功编译了，但是此时是`dlc`是64位可执行程序，在我们学生默认的32位debian虚拟机是无法运行的：

```sh
liuzhenlong@debian:~/Lab_new/lab1-handout$ ./dlc bits.c
-bash: ./dlc: cannot execute binary file: Exec format error
liuzhenlong@debian:~/Lab_new/lab1-handout$ file dlc
dlc: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=1dcd289bac34aee0dde34abd43a959588d79b382, with debug_info, not stripped
```

所以需要修改成32位编译：[参考](https://www.cnblogs.com/xuejianbest/p/10285173.html)

```makefile
_CFLAGS = -g  -O2 -fgnu89-inline -m32
```

下面继续编译，之前需要清理下：

```sh
make clean
make 
liuzhenlong@debian:~/Lab_new/lab1-handout$ file dlc
dlc: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=ec898c28449afe1d16254e2cbe67617c1874b026, with debug_info, not stripped
```

现在`dlc`可执行程序已经生成。

## 2.3 导出实验

在项目主目录下执行：

```sh
[root@VM-4-8-centos lab1]# make all
[root@VM-4-8-centos lab1]# ll
total 1232
drwxr-xr-x 2 root root    4096 Apr  9 22:20 contest
drwxr-xr-x 4 root root    4096 Apr  8 10:58 grade
drwxr-xr-x 2 root root    4096 Apr 13 00:13 lab1-handout
-rw-r--r-- 1 root root 1228800 Apr 13 00:13 lab1-handout.tar
-rw-r--r-- 1 root root    2436 Apr  7 23:06 Makefile
-rw-r--r-- 1 root root    8097 Apr  7 23:06 README
drwxr-xr-x 5 root root    4096 Apr 13 00:13 src
drwxr-xr-x 2 root root    4096 Apr  7 23:04 writeup
```

`lab1-handout.tar`是生成的实验，将其分发给同学们即可进行实验了。

# 3 题解

## 3.1 lsbZero.c

**注释**：

```c++
/*
 *   lsbZero - set 0 to the least significant bit of x
 *   Example: lsbZero(0x87654321) = 0x87654320
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 1
 */
```

**题目**：

给定一个`int`型整数`x`，将其最低一位有效位置0。

* 例如：`lsbZero(0x87654321) = 0x87654320`

**实现**：

```c++
// 法1（标准答案-3'）
/* */
int lsbZero(int x) {
  return (x | 1) + ~0;
}

// 法2（目前最佳答案-2'）
/* 右移一位再左移一位实现把最低一位有效位置0 */
int lsbZero(int x) {
  return x >> 1 << 1;
}

// 法3（来自学生-2'）
/* 和 0xfffffffe 相与即可 */
int lsbZero(int x) {
  return x & (~1);
}
```

## 3.2 byteNot.c

**注释**：

```c++
/*
 *   byteNot.c - bit-inversion to byte n from word x
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: getByteNot(0x12345678,1) = 0x1234A978
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */
```

**题目**：

给定一个`int`型整数`x`，将其第`n`个字节实现取反。

* 例如：`byteNot(0x12345678,1) = 0x1234A978`

**实现**：

```c++
// 法1（标准答案-3'）
/* 将0xFF移动到对应字节，然后取按位异或 */
int nuaa_question2(int x, int n) {
  /* Shift x n*8 positions right */
  int shift = n << 3;
  /* Mask byte */
  int mask = 0xFF << shift;
  return x ^ mask;
}
```

## 3.3 byteXor.c

**注释**：

```c++
/*
 *   byteXor - compare the nth byte of x and y, if it is same, return 0, if not, return 1
 *   example: byteXor(0x12345678, 0x87654321, 1) = 1
 *            byteXor(0x12345678, 0x87344321, 2) = 0
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 2
 */
```

**题目**：

给定三个`int`整数 `x` 、 `y `和 `n` ，比较 `x` 和 `y` 的第 `n` 字节，如果相同则返回 `0` ，不同则返回 `1`。

* 例如：`byteXor(0x12345678, 0x87654321, 1) = 1`
* 例如：`byteXor(0x12345678, 0x87344321, 2) = 0`

**实现**：

```c++
// 法1（标准答案-9'）
/* 移位取指定字节，异或，两次取逻辑非 */
int byteXor(int x, int y, int n) {
  int shift = n << 3;
  int xs = x >> shift;
  int ys = y >> shift;
  int cmp = (xs & 0xFF) ^ (ys & 0xFF);
  return !!cmp & 1 ;
}

// 法2（目前最佳答案-8'）
/* 法1最后的&1是多余的操作 */
int byteXor(int x, int y, int n) {
  int shift = n << 3;
  int xs = x >> shift;
  int ys = y >> shift;
  int cmp = (xs & 0xFF) ^ (ys & 0xFF);
  return !!cmp;
}

// 法3（来自学生-6'）
/* 把⽬标位移到最低位然后⽤异或⽐较⼀下是否相等即可 */
int lsbZero(int x) {
    _ = n << 3;
  __ = ((x ^ y) >> _) & 0xff;
  return !!(__);
}
```

## 3.4 logicalAnd

**注释**：

```c++
/*
 *   logicalAnd - x && y
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3
 */
```

**题目**：

实现逻辑与`&&`，当两个操作数都是ture的时候，才返回true，否则返回false。

* 例如：`logicalAnd(1, 0) = 0`
* 例如：`logicalAnd(2, 3) = 1`

**实现**：

```c++
// 法1（标准答案-8'）
/* */
int logicalAnd(int x, int y) {
  int z = ~!x & ~!y;
  return ~(z + 1) & 1;
}

// 法2（其他思路-5'）
/* 把x和y分别转换为逻辑的0和1，再相与 */
int logicalAnd(int x, int y) {
   return (!(!x))&(!(!y));
}

// 法3（目前最佳答案-4'）
/* 把x和y分别取NOT，二者相或后再取NOT，即可得到逻辑与 */
int logicalAnd(int x, int y) {
   return !((!x)|(!y));
}
```

## 3.5 logicalOr

**注释**：

```c++
/*
 *   logicalOr - x || y
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3
 */
```

**题目**：

实现逻辑或`||`，当两个操作数有一个为ture的时候，就返回true。

* 例如：`logicalOr(0, 0) = 0`
* 例如：`logicalOr(2, 0) = 1`

**实现**：

```c++
// 法1（标准答案）
/* */
int logicalOr(int x, int y) {
  return ~(!x & !y) & 1;
}

// 法2（目前最佳答案）
/* 把x和y分别转换为逻辑的0和1，再相或 */
int logicalOr(int x, int y) {
  return (!(!x))|(!(!y));
}
```

## 3.6 rotateLeft

**注释**：

```c++
/*
 *   rotateLeft - Rotate x to the left by n
 *   Can assume that 0 <= n <= 31
 *   Examples: rotateLeft(0x87654321,4) = 0x76543218
 *   Legal ops: ~ & ^ | + << >> !
 *   Max ops: 25
 *   Rating: 3
 */
```

**题目**：

给定一个`int`型整数`x`，实现循环左移，也就是把`x`的前`n`位移到末尾。

* 例如：`rotateLeft(0x87654321,4) = 0x76543218`

**实现**：

```c++
// 法1（标准答案-16'）
/* */
int rotateLeft(int x, int n) {
    /* Create mask for n = 0 */
    int zmask = (~!n)+1;
    int left = x << n;
    /* Arithmetic shift right by 32-n */
    int right = x >> (33 + ~n);
    /* Mask off upper 1's */
    int lmask = ~0 << n;
    right &= ~lmask;
    return (zmask&x) | (~zmask&(left|right));
}

// 法2(其他方法-10’)
/* 先构造y为高（32-n）位为0的y，再与x右移（32-n）的x相与，相当于储存了x的高n位数，最后再与x左移n位相加即可。 */
int rotateLeft(int x, int n) {
    int y;
    y=~((~0)<<n);
    x=(x<<n)+((x>>(32+(~n+1)))&y);
    return x;
}
```

## 3.7 float_abs.c

**注释**：

```c++
/*
 *   float_abs - Return bit-level equivalent of absolute value of f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representations of
 *   single-precision floating point values.
 *   When argument is NaN, return argument..
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 10
 *   Rating: 2
 */

```

**题目**：

给予⼀个无符号整数表示的浮点数 `uf` (你可以认为 `uf` 具有浮点数的比特级结构) ，函数返回它的绝对值，即 `|uf|` (返回的结果也是⼀个无符号整数表示的浮点数)，如果这个数是 `NaN` ，请返回它本身。

**实现**：

```c++
// 法1（标准答案）
/* */
unsigned float_abs(unsigned uf) {
  unsigned mask = 1 << 31;
  unsigned abs = uf & ~mask;
  if (abs > 0x7F800000)
    return uf;
  return abs;
}

// 法2（目前最佳答案）
/* https://blog.csdn.net/qq_43544682/article/details/102874816 */
unsigned float_abs(unsigned uf) {
    int x=uf&0x7fffffff;
    if(x>0x7f800000)
        return uf;//NaN
    else
        return x;
}
```

## 3.8 float_f2i

**注释**：

```c++
/*
 *   float_f2i - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */

```

**题目**：

给予⼀个无符号整数表示的浮点数 `uf` (你可以认为 `uf` 具有浮点数的比特级结构) ，函数返回它对应的强制类型转换后的整数（具有整数的比特级结构），即实现表达式 `(int)uf` ，如果这个数超过整数表示的范围（包括 `NaN` 和无穷），请返回 `0x80000000u`。

**实现**：

```c++
// 法1（标准答案）
/* */
int float_f2i(unsigned uf) {
  unsigned sign = uf >> 31;
  unsigned exp = (uf >> 23) & 0xFF;
  unsigned frac = uf & 0x7FFFFF;
  /* Create normalized value with leading one inserted,
     and rest of significand in bits 8--30.
  */
  unsigned val = 0x80000000u + (frac << 8);
  if (exp < 127) {
    /* Absolute value is < 1 */
    return 0;
  }
  if (exp > 158)
    /* Overflow */
    return 0x80000000u;
  /* Shift val right */
  val = val >> (158 - exp);
  /* Check if out of range */
  if (sign) {
    /* Negative */
    return val > 0x80000000u ? 0x80000000u : -val;
  } else {
    /* Positive */
    return val > 0x7FFFFFFF ? 0x80000000u : val;
  }
}

// 法2（目前最佳答案）
/*  */

```

# 4 其他

```sh
liuzhenlong@debian:~/Lab_new/lab1-handout$ ./driver.pl
1. Running './dlc -z' to identify coding rules violations.

2. Compiling and running './btest -g' to determine correctness score.
gcc -O -Wall -m32 -lm -o btest bits.c btest.c decl.c tests.c
btest.c: In function ‘test_function’:
btest.c:332:23: warning: ‘arg_test_range[1]’ may be used uninitialized in this function [-Wmaybe-uninitialized]
     if (arg_test_range[1] < 1)
         ~~~~~~~~~~~~~~^~~

3. Running './dlc -Z' to identify operator count violations.

4. Compiling and running './btest -g -r 2' to determine performance score.
gcc -O -Wall -m32 -lm -o btest bits.c btest.c decl.c tests.c
btest.c: In function ‘test_function’:
btest.c:332:23: warning: ‘arg_test_range[1]’ may be used uninitialized in this function [-Wmaybe-uninitialized]
     if (arg_test_range[1] < 1)
         ~~~~~~~~~~~~~~^~~


5. Running './dlc -e' to get operator count of each function.
```

运行`./driver.pl`会报warning，批改学生作业时，有一个同学提出了解答：👍

> 关于 ./driver.pl 中的预警问题，是因为 ./btest.c 在299行申请的变量没有进行初始化赋值，后面的程序检测到这个位置可能会被访问而提出的预警，给其付个初始值即可解决

# X 参考

* [计算机系统基础 实验一 csapp date lab1](https://blog.csdn.net/qq_43544682/article/details/102874816)
* [CSAPP:Lab1 -DataLab 超详解](https://zhuanlan.zhihu.com/p/339047608)
* [CSAPP: Lab1:Data Lab 实验 （虚拟机安装+Lab环境配置+README及实验引导翻译+如何编译+函数实现）](https://blog.csdn.net/qq_45677541/article/details/123955438)
