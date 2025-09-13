---
title: 中科大2021年Hackergame WP
tags:
categories:
  - [CTF, WP]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2021-10-30 21:27:28
---

# 1 签到

改变URL中的参数，使其在当前日期：`http://202.38.93.111:10000/?page=1635000000`

<img src="https://i.loli.net/2021/10/23/wxnQA8e1C6P2zK9.png" width = "600" height = "300" alt="图片名称" align=center id=168 />

# 2 进制十六——参上

使用文字识别工具识别十六进制串，转成字符串即可：https://www.bejson.com/convert/ox2str/

<img src="https://i.loli.net/2021/10/23/tqYaVTK3URO1wDl.png" width = "700" height = "300" alt="图片名称" align=center id=169 />



# 3 去吧！追寻自由的电波

此mp3文件明显是加速处理后的文件，拖到Pr减速可以听到是一个一个的英语单词，根据前四个单词的首字母(f l a g)可以判定是取所有的首字母，左右花括号可以直译即可。

# 4 猫咪问答 Pro Max

* 问题1：https://web.archive.org/web/20181004003308/http://sec.ustc.edu.cn/doku.php/codes
* 问题2：https://lug.ustc.edu.cn/wiki/intro/
* 问题3：https://lug.ustc.edu.cn/news/2016/06/new-activity-room-in-west-library/
* 问题4：http://sigbovik.org/2021/proceedings.pdf
* 问题5：https://datatracker.ietf.org/doc/html/rfc8962

# 5 卖瓜

使用`9斤的瓜`凑，大数使其溢出为负数，然后获得商并使用其凑`9斤的瓜`。重复几次可以凑出来。

# 6 透明的文件

...

# 7 旅行照片

百度搜索`海边KFC`关键字，打开百度地图，根据海的位置确定方位，根据阴影判断是下午(两个选择爆破下)，题目3需要爆破，题目4和5根据百度搜索内容就可得到。

<img src="https://i.loli.net/2021/10/23/ABcamulY4bJEHTt.png" width = "600" height = "300" alt="图片名称" align=center id=170 />

# 8 FLAG 助力大红包

...

# 9 Amnesia

## 轻度失忆

编译后 ELF 文件的 `.data` 和 `.rodata` 段会被清零。所以不能使用字符串赋值，putchar即可。

```c
#include <stdio.h>

int main() {
    putchar('H');
    putchar('e');
    putchar('l');
    putchar('l');
    putchar('o');
    putchar(',');
    putchar(' ');
    putchar('w');
    putchar('o');
    putchar('r');
    putchar('l');
    putchar('d');
    putchar('!');
    return 0;
}
```

## 记忆清除

```c++
#include <asm/unistd.h>
#include <unistd.h>
__attribute__((used, constructor, section(".init"))) void pre_array(); 
void _start() __attribute__((section(".init")));
ssize_t my_write(const void *buf) __attribute__((section(".init")));
ssize_t my_write(const void *buf)
{
    int fd = STDOUT_FILENO;
    ssize_t ret;
    size_t size = 1;
    asm volatile(
        "int $0x80"
        : "=a"(ret)
        : "0"(__NR_write), "b"(fd), "c"(buf), "d"(size)
        : "memory" // the kernel dereferences pointer args
    );
    return ret;
}
void pre_array()
{   
    char test1 = 'H';
    my_write((const void *)&test1);
    test1 = 'e';
    my_write((const void *)&test1);
    test1 = 'l';
    my_write((const void *)&test1);
    test1 = 'l';
    my_write((const void *)&test1);
    test1 = 'o';
    my_write((const void *)&test1);
    test1 = ',';
    my_write((const void *)&test1);
    test1 = ' ';
    my_write((const void *)&test1);
    test1 = 'w';
    my_write((const void *)&test1);
    test1 = 'o';
    my_write((const void *)&test1);
    test1 = 'r';
    my_write((const void *)&test1);
    test1 = 'l';
    my_write((const void *)&test1);
    test1 = 'd';
    my_write((const void *)&test1);
    test1 = '!';
    my_write((const void *)&test1);
    /* exit system call */
    asm("movl $1,%eax;"
        "xorl %ebx,%ebx;"
        "int  $0x80");
}
typedef void (*init_array_t)(void);
init_array_t __attribute__ ((section(".preinit_array"))) my_init_a1 = pre_array;
int main()
{
	return 0;
}
```



# 10 图之上的信息

# 11 Easy RSA

# 12 加密的 U 盘

# 13 赛博厨房

## Level 0

自然语言编程，按照每天给出的条件编程即可。

```
向右 2 步
拿起 1 个物品
向下 1 步
向左 2 步
放下 1 个物品
向右 1 步
向上 1 步
拿起 1 个物品
向下 1 步
向左 1 步
放下 1 个物品
```

## Level 1

`do...while`循环思路

```
向右 1 步
拿起 73 个物品
向下 1 步
向左 1 步
放下 1 个物品
如果手上的物品大于等于 0 向上跳转 1 行
```

