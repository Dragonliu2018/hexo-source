---
title: RSA题型整理
date: 2020-07-07 10:09:17
tags: 
categories:
    - [CTF, Crypto]
# keywords:
description:
top: #True #顶置
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
cover: https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/maxresdefault.jpg
toc:  #是否显示toc （除非特定文章设置，可以不写）
toc_number: #是否显示toc数字 （除非特定文章设置，可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不写）
mathjax: true
---

# 0x00 前期准备

## 01 基础知识

　　**RSA加密算法**是一种[非对称加密算法](https://zh.wikipedia.org/wiki/非对称加密演算法)，在[公开密钥加密](https://zh.wikipedia.org/wiki/公开密钥加密)和[电子商业](https://zh.wikipedia.org/wiki/电子商业)中被广泛使用。RSA是由[罗纳德·李维斯特](https://zh.wikipedia.org/wiki/罗纳德·李维斯特)（Ron Rivest）、[阿迪·萨莫尔](https://zh.wikipedia.org/wiki/阿迪·萨莫尔)（Adi Shamir）和[伦纳德·阿德曼](https://zh.wikipedia.org/wiki/伦纳德·阿德曼)（Leonard Adleman）在1977年一起提出的。当时他们三人都在[麻省理工学院](https://zh.wikipedia.org/wiki/麻省理工学院)工作。RSA 就是他们三人姓氏开头字母拼在一起组成的。

### 1.1 安全保证

安全性依赖于大整数分解的难题：寻找两个不同的大素数是容易的，但将两个大素数的乘积分解成原来的两个素数是极其困难的。  

### 1.2 具体内容

假设Bob想给Alice送一个消息m：

1. 选择两个大素数 p 和 q ，计算$n=p q$；
2. 随机选取整数e和 d ，满足$e d \equiv 1(\bmod \varphi(n))$  ，其中$\varphi(n)$ 为 $n$的欧拉函数；（$r = \varphi(n)=\varphi(p) \varphi(q)=(p-1)(q-1)$，选择一个小于 r 的整数 e，使 r 与 e 互质，并求得 e 关于 r 的[模逆元](https://zh.wikipedia.org/wiki/模反元素)，命名为 d 求 $ e d \equiv 1(\bmod r)$。模逆元存在，当且仅当 e 与 r 互质。）
3. 发布e和 n 为公钥， d 为私钥；  
4. 设明文为m，加密函数为$c \equiv E(m) \equiv m^{e}(\bmod n)$，其中$1<m, c<n$；
5. 解密函数为 $m \equiv D(c) \equiv c^{d}(\bmod n)$。

$(n, e)$ 是公钥, $(n, d)$ 是私钥。Alice将她的公钥$(n, e)$ 传给Bob，而将她的私钥 $(n, d)$ 藏起来。

### 1.3 证明

下面使用欧拉定理证明解密函数的正确性，即已知1-4，证明 5 成立。  

**证明**：由于 $c \equiv m^{e}(\bmod n),$ 所以 $c^{d} \equiv m^{e d}(\bmod n),$ 即证 $m^{e d} \equiv m(\bmod n)$

1. 当 $(m, n)=1$ 时，由欧拉定理知 $, m^{\varphi(n)} \equiv 1(\bmod n),$ 而由条件2知 $e d \equiv 1(\bmod \varphi(n)),$ 即存
   在整数 $k ，$ 使得 $e d=k \varphi(n)+1,$ 因此 $, \quad m^{e d} \equiv m^{k \varphi(n)+1} \equiv m(\bmod n)$；
2. 当 $(m, n) \neq 1$ 时，由于 $n=p q,$ 因此 $(m, n)=p$ 或 $(m, n)=q,$ 即 $p|m$ 或 $q | m$ ，
   若 $p| m,$ 则显然 $m^{e d} \equiv m^{k \varphi(n)+1} \equiv m \equiv 0(\bmod p)$；
   若 $p\nmid m,$ 则由欧拉定理知 $m^{p-1} \equiv 1(\bmod p),$ 于是$m^{k \varphi(n)+1} \equiv m^{k(p-1)(q-1)+1} \equiv m(\bmod p)$；
   因此，对任意$m$，$m^{e d} \equiv m^{k \varphi(n)+1} \equiv m(\bmod p)$ 成立，同理可证，对任意 $m, \quad m^{e d} \equiv m^{k \varphi(n)+1} \equiv m(\bmod q)$ 成立，因此 $m^{e d} \equiv m(\bmod n)$ 成立。(同余性质10-最小公倍数)

### 1.4 安全性

假设偷听者Eve获得了Alice的公钥 $n$ 和 e 以及Bob的加密消息 $c,$ 但她无法直接获得Alice的私钥 $d 。$ 要获得 $d,$ 最简单的方法是将 $n$ 分解为 $p$ 和 $q,$ 这样她可以得到同余方程$d e=1(\bmod (p-1)(q-1))$ 并解出 $d,$ 然后代入解密公式$m\equiv c^{d} (\bmod n)$导出$m$(破密) 。
但至今为止还没有人找到一个多项式时间的算法来分解一个大的整数的因子，同时也还没有人能够证明这种算法不存在 (见因数分解) 
至今为止也没有人能够证明对$n$进行因数分解是唯一的从$c$导出$m$的方法，直到今天也还没有找到比它更简单的方法。（至少没有公开的方法）
因此今天一般认为只要$n$足够大，那么黑客就没有办法了。
假如$n$的长度小于或等于256位，那么用一台个人电脑在几个小时内就可以分解它的因子了。1999年，数百台电脑合作分解了一个512位长的$n$。一个由Shamir 和Tromer在2003年从理论上构建的硬件TWIRL，使人们开始质疑1024位长的$n$的安全性，目前推荐$n$的长度至少为2048位。
1994年[彼得·秀尔](https://zh.wikipedia.org/wiki/彼得·秀爾)（Peter Shor）证明一台[量子计算机](https://zh.wikipedia.org/wiki/量子计算机)可以在多项式时间内进行因数分解。假如量子计算机有朝一日可以成为一种可行的技术的话，那么秀尔的算法可以淘汰RSA和相关的派生算法。（即依赖于分解大整数困难性的加密算法）
假如有人能够找到一种有效的分解大整数的算法的话，或者假如量子计算机可行的话，那么在解密和制造更长的钥匙之间就会展开一场竞争。但从原理上来说RSA在这种情况下是不可靠的。

### 1.5 参考

* 《信息安全数学基础》(主编：常相茂 周玉倩)
* [维基百科](https://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95#%E5%AE%89%E5%85%A8)

## 02 Ubuntu 安装gmpy2模块

PARI/GP是一个比较强大的数论库，“针对数论中的快速计算（大数分解，代数数论，椭圆曲线…）而设计”。

**需要的依赖库 gmp mpfr mpc**

### gmp 库安装

```bash
sudo apt-get install libgmp-dev
```

### mpfr 库安装

```bash
sudo apt-get install libmpfr-dev
```

### mpc 库安装

```bash
sudo apt-get install libmpc-dev
```

### gmpy2 安装

```bash
#python3
sudo pip3 install gmpy2
#python2
sudo pip install gmpy2
```

***

# 0x01 已知n、e、c，求m

## 01 思路

1. 利用 http://factordb.com/ 分解n获得p和q；
2. 计算d；
3. 解密得明文m。

## 02 代码

* **代码1**

```python
from Crypto.Util.number import long_to_bytes,bytes_to_long,getPrime,isPrime
import primefac

def modinv(a, n):
    return primefac.modinv(a, n) % n
n = ...
e = ...
c = ...
p = ...
q = ...

phi_n = (p-1)*(q-1)
d = modinv(e, phi_n) % phi_n #求模逆元
m = pow(c, d, n) #求明文
print long_to_bytes(m)
```

* **代码2**

```python
#!/usr/bin/env python
# coding=utf-8
import gmpy2

#分解n得p,q
p = gmpy2.mpz(...) #填写p
q = gmpy2.mpz(...) #填写q
e = gmpy2.mpz(...) #填写e
c = gmpy2.mpz(...) #填写c

#计算d
phi_n = (p - 1) * (q - 1)
d = gmpy2.invert(e, phi_n) #求解模逆元
print "private key:"
print d

#求明文
print "plaintext:"
M  =  pow(c, d, p*q)
print '[10进制]: ' + str(M)
flag = str( hex(M) )[2:] #[2:-1]
print '[16进制]: ' + flag
print '[ASCII码]: ' + flag.decode('hex')
```

***

# 0x02 低加密指数分解攻击(e = 1)

## 01 思路

1. 加密过程：$c \equiv E(m) \equiv m^{e}(\bmod n) \equiv m(\bmod n)$，所以明文与密文模n同余；
2. $m = c + n*k (k=0,1,2,3...)$，暴力破解即可。

## 02 代码

```python
#!/usr/bin/env python
# coding=utf-8
import libnum

n = ... #填写n
c = ... #填写c

max_num = 6 #遍历上限

for k in range( max_num ):
    m = c + n*k
    print libnum.n2s(m)
```

***

# 0x03 Rabin加密(e=2)

## 01 思路

理论知识：[我跳](http://dragonliu.tk/2020/07/08/Rabin加密/)

## 02 代码

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import libnum
import gmpy2

#从文件中读取密文
f = open('flag.enc','r')
c = f.read()
c = libnum.s2n(c)#字符串转数字

#分解n
p = ... #填写p
q = ... #填写p
n = p * q

#求解mp与mq
mp = pow(c, (p+1)/4, p)
mq = pow(c, (q+1)/4, q)

#求解yp与yq
yp = gmpy2.invert(p, q)
yq = gmpy2.invert(q, p)

#获得四个解
r1 = (yp*p*mq + yq*q*mp) % n
r2 = (-r1) % n
r3 = (yp*p*mq - yq*q*mp) % n
r4 = (-r3) % n

print libnum.n2s( r1 )#数字转字符串
print libnum.n2s( r2 )
print libnum.n2s( r3 )
print libnum.n2s( r4 )
```

当$p = q$时，使用python命令行将16进制转十进制，然后直接求解$c$模 n 时的平方根：

```mathematica
n = ...;
c = ...;
PowerMod[c, 1/2, n] 
```

Wolfram 语言在线编辑：[我跳](https://www.wolframcloud.com/)

***

# 0x04 flag.enc + pubkey.pem

## 01 思路

1. 解压得到两个文件【flag.enc】和【pubkey.pem】，其中【flag.enc】从文件名含有flag可以判断是加密后的密文，【pubkey.pem】是公钥文件，通过公钥文件可以得到e和n；
2. 通过openssl对公钥文件【pubkey.pem】进行分解，使用命令【openssl rsa -pubin -text -modulus -in warmup -in pubkey.pem】，得到 e【Exponent】和 n【Modulus】；
3. 其他根据类型判断。

## 02 代码

```python
#!/usr/bin/env python
# coding=utf-8
import gmpy2
import rsa

#分解n得p,q
n = ... #填写n
p = ... #填写p
q = ... #填写q
e = ... #填写e

#计算私钥
phi_n = (p - 1) * (q - 1)
d = int( gmpy2.invert(e, phi_n) ) #求解模逆元
privatekey = rsa.PrivateKey(n , e , d , p , q) #根据已知参数，计算私钥

with open("./flag.enc" , "rb") as f: #填写文件
    print rsa.decrypt(f.read(), privatekey).decode()  #使用私钥对密文进行解密，并打印
```

***

# 0x05 共模攻击

## 01 思路

用相同的N，不同的e进行加密的，可以使用共模攻击。

## 02 代码

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from libnum import n2s, s2n
from gmpy2 import invert, gcdext

n  = ... #填写n
e1 = ... #填写e1
e2 = ... #填写e2

s = gcdext(e1, e2)  
s1 = s[1]
s2 = -s[2]

#读取密文
file1 = open('flag.enc1', 'r')
c1 = file1.read()
c1 = s2n(c1)#字符串转数字

file2 = open('flag.enc2', 'r')
c2 = file2.read()
c2 = s2n(c2)#字符串转数字

c2 = invert(c2, n)   
m = (pow(c1,s1,n) * pow(c2 , s2 , n)) % n
print n2s(m)
```

***

# 0x06 低加密指数分解攻击(e = 3)

## 01 思路

1. 公钥中，e=3，N非常大。
2. 加密过程：$c \equiv E(m) \equiv m^{e}(\bmod n) \equiv m^3(\bmod n)$，所以明文与密文的3次方模n同余；
3. $m = c + n*k (k=0,1,2,3...)$，然后开三次方，暴力破解即可。

## 02 代码

```python
#!/usr/bin/env python
# coding=utf-8
from libnum import s2n, n2s
from gmpy2 import iroot

n = ... #填写n
e = 3

#读取密文
file = open('flag.enc', 'r')
c = file.read()
c = s2n(c)#字符串转数字
file.close()

i = 0 
while True:
    res = iroot( (c + i*n), 3 )
    if( res[1] ==  True):
        print res
        break
    print "i = " + str(i)
    i += 1

m = ... #暴力获得的m
print n2s(m)
```

***

# 0x07 私钥修复+最优非对称加密填充(God Like RSA)

## 01 思路

压缩包里有一个密文，一个部分缺失的私钥，一个公钥，读公钥可知 N 是 4096 位的，分解无望，肯定要从私钥着手。

1. 【vscode】打开【private.corrupted】，将对应变量填入下列脚本；
2. 执行脚本后得到私钥，新建文件【private.pem】并将私钥复制进去；
3. 然后执行【最优非对称加密填充】脚本。

## 02 代码

**私钥修复脚本**

```python
#!/usr/bin/python
#-*- coding:utf-8 -*-

import re
import pickle
from itertools import product
from libnum import invmod, gcd


def solve_linear(a, b, mod):
    if a & 1 == 0 or b & 1 == 0:
        return None
    return (b * invmod(a, mod)) & (mod - 1)  # hack for mod = power of 2


def to_n(s):
    s = re.sub(r"[^0-9a-f]", "", s)
    return int(s, 16)


def msk(s):
    cleaned = "".join(map(lambda x: x[-2:], s.split(":")))
    return msk_ranges(cleaned), msk_mask(cleaned), msk_val(cleaned)


def msk_ranges(s):
    return [range(16) if c == " " else [int(c, 16)] for c in s]


def msk_mask(s):
    return int("".join("0" if c == " " else "f" for c in s), 16)


def msk_val(s):
    return int("".join("0" if c == " " else c for c in s), 16)

#根据文件pubilc.pem得到
E = ...
#文件pubilc.pem中第一个Modulus
N = to_n("""...""")

#private.corrupted中的prime1
p_ranges, pmask_msk, pmask_val = msk(""" ... """)

#prime2
q_ranges, qmask_msk, qmask_val = msk(""" ... """)

#privateExponent
_, dmask_msk, dmask_val = msk(""" ... """)

#exponent1
_, dpmask_msk, dpmask_val = msk(""" ... """)

#exponent2
_, dqmask_msk, dqmask_val = msk(""" ... """)


def search(K, Kp, Kq, check_level, break_step):
    max_step = 0
    cands = [0]
    for step in range(1, break_step + 1):
        #print " ", step, "( max =", max_step, ")"
        max_step = max(step, max_step)

        mod = 1 << (4 * step)
        mask = mod - 1

        cands_next = []
        for p, new_digit in product(cands, p_ranges[-step]):
            pval = (new_digit << ((step - 1) * 4)) | p

            if check_level >= 1:
                qval = solve_linear(pval, N & mask, mod)
                if qval is None or not check_val(qval, mask, qmask_msk, qmask_val):
                    continue

            if check_level >= 2:
                val = solve_linear(E, 1 + K * (N - pval - qval + 1), mod)
                if val is None or not check_val(val, mask, dmask_msk, dmask_val):
                    continue

            if check_level >= 3:
                val = solve_linear(E, 1 + Kp * (pval - 1), mod)
                if val is None or not check_val(val, mask, dpmask_msk, dpmask_val):
                    continue

            if check_level >= 4:
                val = solve_linear(E, 1 + Kq * (qval - 1), mod)
                if val is None or not check_val(val, mask, dqmask_msk, dqmask_val):
                    continue

                if pval * qval == N:
                    print "Kq =", Kq
                    print "pwned"
                    print "p =", pval
                    print "q =", qval
                    p = pval
                    q = qval
                    d = invmod(E, (p - 1) * (q - 1))
                    coef = invmod(p, q)

                    from Crypto.PublicKey import RSA
                    print RSA.construct(map(long, (N, E, d, p, q, coef))).exportKey()
                    quit()

            cands_next.append(pval)

        if not cands_next:
            return False
        cands = cands_next
    return True

def check_val(val, mask, mask_msk, mask_val):
    test_mask = mask_msk & mask
    test_val = mask_val & mask
    return val & test_mask == test_val


for K in range(1, E):
    if K % 100 == 0:
        print "checking", K
    if search(K, 0, 0, check_level=2, break_step=20):
        print "K =", K
        break

for Kp in range(1, E):
    if Kp % 1000 == 0:
        print "checking", Kp
    if search(K, Kp, 0, check_level=3, break_step=30):
        print "Kp =", Kp
        break

for Kq in range(1, E):
    if Kq % 100 == 0:
        print "checking", Kq
    if search(K, Kp, Kq, check_level=4, break_step=9999):
        print "Kq =", Kq
        break
```

**解题脚本(最优非对称加密填充)**

```python
#!/usr/bin/python
# coding=utf-8
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_OAEP

with open('pubkey.pem', 'r') as f:
    key = RSA.importKey(f)
    N = key.n
    e = key.e

print N
print e

with open('private.pem', 'r') as f:
    private = RSA.importKey(f)
    oaep = PKCS1_OAEP.new(private)

with open('flag.enc', 'r') as f:
    print oaep.decrypt(f.read())
```

## 03 参考

[RSA 私钥恢复和最优非对称加密填充](https://www.40huo.cn/blog/rsa-private-key-recovery-and-oaep.html)

***

# 0x08 wiener attack(e特别大)

## 01 思路

1. 给出的n分解无望，而且e特别大，利用`wiener attack`脚本分解；
2. 然后利用一般方法求解即可。

## 02 代码

```python
import gmpy2
import libnum

def continued_fractions_expansion(numerator,denominator):#(e,N)
    result=[]
 
    divident=numerator%denominator
    quotient=numerator/denominator
    result.append(quotient)
 
    while divident!=0:
        numerator=numerator-quotient*denominator
 
        tmp=denominator
        denominator=numerator
        numerator=tmp
 
        divident=numerator%denominator
        quotient=numerator/denominator
        result.append(quotient)
 
    return result
 
def convergents(expansion):
    convergents=[(expansion[0],1)]
    for i in range(1,len(expansion)):
        numerator=1
        denominator=expansion[i]
        for j in range(i-1,-1,-1):
            numerator+=expansion[j]*denominator
            if j==0:
                break
            tmp=denominator
            denominator=numerator
            numerator=tmp
        convergents.append((numerator,denominator))#(k,d)
    return convergents
 
def newtonSqrt(n):
    approx = n/2
    better = (approx + n/approx)/2
    while better != approx:
        approx = better
        better = (approx + n/approx)/2
    return approx
 
def wiener_attack(cons,e,N):
    for cs in cons:
        k,d=cs
        if k==0:
            continue
        phi_N=(e*d-1)/k
        #x**2-((N-phi_N)+1)*x+N=0
        a=1
        b=-((N-phi_N)+1)
        c=N
        delta = b*b - 4*a*c
        if delta<=0:
            continue
        x1= (newtonSqrt(delta)-b)/(2*a)
        x2=-(newtonSqrt(delta)+b)/(2*a)
        if x1*x2==N:
            return [x1,x2,k,d]
 
 
N = ...
e = ...
 
expansion = continued_fractions_expansion(e,N)
cons = convergents(expansion)
 
p, q, k, d = wiener_attack(cons, e, N)
print p
print q

c = ... #密文

d = gmpy2.invert(e, (p - 1) * (q - 1))
m = pow(c, d, N)
print libnum.n2s(m)
```

***

# 0x0 reference

* [Ubuntu 安装gmpy2模块](https://blog.csdn.net/qq_28573835/article/details/86164877)
* [一大波RSA就要来了~~](https://bestwing.me/Common-types-of-RSA.html)





