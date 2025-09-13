---
title: CTF编码加密整理
date: 2020-07-13 09:04:26
tags:
categories:
    - [CTF, Crypto]
# keywords:
description:
top: #True #顶置
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
cover: https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/CTF-Crypto.png
toc:  #是否显示toc （除非特定文章设置，可以不写）
toc_number: #是否显示toc数字 （除非特定文章设置，可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不写）
---

# 01 常见编码

## 0x01 莫尔斯电码(Morse Code)

### 01 原理

　　摩尔斯电码也被称作摩斯密码，是一种时通时断的[信号](https://baike.baidu.com/item/信号/32683)代码，通过不同的排列顺序来表达不同的英文字母、数字和标点符号。

### 02 特征

1. 由`. -`组成的字符串；
2. 由`0 1`组成的字符串，且为空格间隔得一组一组的。

### 03 解题

* [通道1(英文字母小写)](http://ctf.ssleye.com/morse.html)
* [通道2(英文字母大写)](http://www.zhongguosou.com/zonghe/moErSiCodeConverter.aspx)
* [通道3(英文字母小写)](http://moersima.00cha.net/)
* 01破解脚本

```python
string = '...'
dic= {    '01': 'A',
        '1000': 'B',
        '1010': 'C',
         '100': 'D',
           '0': 'E',
        '0010': 'F',
         '110': 'G',
        '0000': 'H',
          '00': 'I',
        '0111': 'J',
         '101': 'K',
        '0100': 'L',
          '11': 'M',
          '10': 'N',
        '111': 'O',
        '0110': 'P',
        '1101': 'Q',
        '010': 'R',
        '000': 'S',
        '1': 'T',
        '001': 'U',
        '0001': 'V',
        '011': 'W',
        '1001': 'X',
        '1011': 'Y',
        '1100': 'Z',
        '01111': '1',
        '00111': '2',
        '00011': '3',
        '00001': '4',
        '00000': '5',
        '10000': '6',
        '11000': '7',
        '11100': '8',
        '11110': '9',
        '11111': '0',
        '001100': '?',
        '10010': '/',
        '101101': '()',
        '100001': '-',
        '010101': '.',
        '110011':',',
        '011010':'@',
        '111000':':',
        '101010':':',
        '10001':'=',
        '011110':"'",
        '101011':'!',
        '001101':'_',
        '010010':'"',
        '10110':'(',
        '1111011':'{',
        '1111101':'}'
        }; 

flag = ''
for item in key:
    flag += dic[item]
print( flag )
print( flag.lower() )
```

***

## 0x02 Base64/32/16编码

### 01 原理

　　base64、base32、base16可以分别编码转化8位字节为6位、5位、4位，16、32、64分别表示用多少个字符来编码。以base64为例：Base64编码要求把3个8位字节转化为4个6位的字节，之后在6位的前面补两个0，形成8位一个字节的形式，6位2进制能表示的最大数是2的6次方是64，这也是为什么是64个字符(A-Z,a-z，0-9，+，/这64个编码字符，=号不属于编码字符，而是填充字符)的原因。

### 02 特征

　　有大写、小写、数字，密文末尾一般为`==`。

### 03 解题

* [通道1](http://ctf.ssleye.com/base64.html)
* [通道2](https://tool.bugku.com/safe/base64.php)

***

## 0x03 Escape/Unescape编码

### 01 原理

　　Escape/Unescape加密解码/编码解码,又叫%u编码，采用UTF-16BE模式， Escape编码/加密,就是字符对应UTF-16 16进制表示方式前面加%u。Unescape解码/解密，就是去掉"%u"后，将16进制字符还原后，由utf-16转码到自己目标字符。如：字符“中”，UTF-16BE是：“6d93”，因此Escape是“%u6d93”。

### 02 特征

　　密文中含有`%u`。

### 03 解题

* [通道1](http://ctf.ssleye.com/escape.html)
* 工具Converter：Unescape

***

## 0x04 HTML编码

### 01 原理

完整编码手册：[我跳](http://www.w3school.com.cn/tags/html_ref_entities.html)

### 02 特征

由元素`&#x26;`构成，26可为别的数字。

### 03 解题

* [通道1](http://ctf.ssleye.com/html_en.html)
* [通道2](https://tool.chinaz.com/tools/htmlencode.aspx)
* 工具Converter：Decode HTML

***

## 0x05 Unicode编码

### 01 原理

Unicode编码有以下四种编码方式：

* 源文本： `The`
* &#x [Hex]： `&#x0054;&#x0068;&#x0065;`
* &# [Decimal]： `&#00084;&#00104;&#00101;`
* \U [Hex]： `\U0054\U0068\U0065`
* \U+ [Hex]： `\U+0054\U+0068\U+0065`　　

### 02 解题

* [通道1](https://tool.chinaz.com/tools/unicode.aspx)
* [通道2](http://www.mxcz.net/tools/Unicode.aspx)

***

## 0x06 URL编码

### 01 原理

　　url编码又叫百分号编码，是统一资源定位(URL)编码方式。URL地址（常说网址）规定了常用地数字，字母可以直接使用，另外一批作为特殊用户字符也可以直接用（/,:@等），剩下的其它所有字符必须通过%xx编码处理。 现在已经成为一种规范了，基本所有程序语言都有这种编码，如js：有encodeURI、encodeURIComponent，PHP有 urlencode、urldecode等。编码方法很简单，在该字节ascii码的的16进制字符前面加%. 如 空格字符，ascii码是32，对应16进制是'20'，那么urlencode编码结果是:%20。

### 02 解题

* [通道1](https://tool.chinaz.com/tools/urlencode.aspx)
* [通道2](http://web.chacuo.net/charseturlencode)

***

# 02 代换密码

## 0x01 凯撒密码(Caesar Cipher) 

### 01 原理

　　凯撒密码(Caesar Cipher或称恺撒加密、恺撒变换、变换加密、位移加密)是一种替换加密，明文中的所有字母都在字母表上向后（或向前）按照一个固定数目进行偏移后被替换成密文。例，当偏移量是3的时候，所有的字母A将被替换成D，B变成E，以此类推。

* **加密**：$c=E_3(m)=m+3(\bmod 26)　0\leq m\leq 25$
* **解密**：$m=D_3(c)=c-3(\bmod 26)　0\leq c\leq 25$

凯撒有两种编码脚本，一种是字母26内循环移位，一种是127次非字母内的循环移位。

### 02 样例

* c =  xyz
* m = abc

### 03 解题

* CTFcrackTools工具(所有结果罗列)
* [通道2(单个情况)](http://ctf.ssleye.com/caesar.html)
* 127次非字母内的循环移位脚本

```python
c = """...""" #密文

for p in range(127):
    m = '' #明文
    for i in c:
        temp = chr( (ord(i) + p) % 127 )
        if 32 < ord(temp) < 127:
            m += temp
            flag = 1
        else:
            flag = 0
            break
    if flag == 1:
        print('****%d****:'%p, m)
```

***

## 0x02 Brainfuck加密

### 01 原理

　　Brainfuck是一种极小化的计算机语言，它是由Urban Müller在1993年创建的。由于fuck在英语中是脏话，这种语言有时被称为brainf＊ck或brainf＊＊k，甚至被简称为BF。brainfuck语言用`> < + - . , [ ]`八种符号来替换C语言的各种语法和命令。

### 02 解题

* [通道1](https://tool.bugku.com/brainfuck/)
* [通道2](https://www.splitbrain.org/services/ook)

***

## 0x03 Ook加密

### 01 原理

　　Ook密码中只含`Ook. Ook? Ook!`三个不同的语法元素。

### 02 解题

* [通道1](https://tool.bugku.com/brainfuck/)
* [通道2](https://www.splitbrain.org/services/ook)

***

## 0x04 .!?加密

### 01 原理

　　.!?密码中只含`. ? !`三个不同的语法元素，使用Ook解密即可。

### 02 解题

* [通道1](https://tool.bugku.com/brainfuck/)
* [通道2](https://www.splitbrain.org/services/ook)

***

## 0x05 社会主义核心价值观

### 01 原理

　　富强、民主、文明、和谐，自由、平等、公正、法治，爱国、敬业、诚信、友善。

### 02 解题

* [通道1](http://ctf.ssleye.com/cvencode.html)
* [通道2](http://www.atoolbox.net/Tool.php?Id=850)

***

## 0x06 标准银河字母

### 01 原理

　　标准银河字母（Standard Galactic Alphabet）出自游戏《[指挥官基恩](https://baike.baidu.com/item/指挥官基恩/10345602)》系列。是系列中使用的书写系统。字母对应如下：

<img src="https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/eac4b74543a98226aab284e78a82b9014b90ebfb.png" width = "1000" height = "150" alt="图片名称" align=center id=185 />

***

## 0x07 仿射加密

### 01 原理

　　仿射密码是一种表单代换密码，字母表的每个字母相应的值使用一个简单的数学函数对应一个数值，再把对应数值转换成字母。

* **加密**：$c = E_{a,b}(m) \equiv am+b(\bmod 26)$
* **解密**：$m = D_{a,b} \equiv a^{-1}(c-d)(\bmod 26)$

　　其中，a、b是密钥，为满足$0\leq a,b\leq 25$和$gcd(a,26) = 1$的整数。其中$gcd(a,26) = 1$表示a和26的最大公因子，$a_{-1}$表示a的逆元，即$a_{-1}*a \equiv 1(\mod26)$。

### 02 解题

* **常规解法**

```python
import gmpy2

a = ... #密钥
b = ... #密钥
text = "..." #密文
flag = ""    #明文
 
text_list = []
 
for i in text:
    text_list.append( ord(i) - 97 )

aa = gmpy2.invert(a, 26) #求解模逆元
for i in text_list:
    tmp = (aa * (i - b)) % 26
    flag += chr(tmp + 97)

print(flag)
```

* **暴力破解**

```python
text = "..."
 
text_list = []
 
for i in text:
    text_list.append( ord(i) - 97 )
 
flag = ""
for i in text_list:
    for j in range(0, 26):
        c = (17 * j - 8) % 26
        if(c == i):
            flag += chr(j + 97)
print(flag)
```

***

## 0x08 

### 01 原理

　　

### 02 解题

* [通道1]()
* [通道2]()

***

# 03 换位加密

## 0x01 栅栏密码(Rail-fence Cipher)

### 01 原理

　　栅栏密码(Rail-fence Cipher)就是把要加密的明文分成N个一组，然后把每组的第1个字符组合，每组第2个字符组合...每组的第N(最后一个分组可能不足N个)个字符组合，最后把他们全部连接起来就是密文。

### 02 样例

* c = KYsd3js2E{a2jda}
  * KYsd3js2
  * E{a2jda}
* m = `KEY{sad23jjdsa2}`

### 03 解题

* CTFcrackTools工具(所有结果罗列)
* [通道2(不区分大小写)](http://ctf.ssleye.com/railfence.html)
* [通道3(区分大小写)](https://www.qqxiuzi.cn/bianma/zhalanmima.php)

***

## 0x02 转盘加密

### 01 原理

**题目**

```
1： <ZWAXJGDLUBVIQHKYPNTCRMOSFE <
2： <KPBELNACZDTRXMJQOYHGVSFUWI <
3： <BDMAIZVRNSJUWFHTEQGYXPLOCK <
4： <RPLNDVHGFCUKTEBSXQYIZMJWAO <
5： <IHFRLABEUOTSGJVDKCPMNZQWXY <
6： <AMKGHIWPNYCJBFZDRUSLOQXVET <
7： <GWTHSPYBXIZULVKMRAFDCEONJQ <
8： <NOZUTWDCVRJLXKISEFAPMYGHBQ <
9： <QWATDSRFHENYVUBMCOIKZGJXPL <
10：<WABMCXPLTDSRJQZGOIKFHENYVU <
11：<XPLTDAOIKFZGHENYSRUBMCQWVJ <
12：<TDSWAYXPLVUBOIKZGJRFHENMCQ <
13：<BMCSRFHLTDENQWAOXPYVUIKZGJ <
14：<XPHKZGJTDSENYVUBMLAOIRFCQW <

密钥： 2,5,1,3,6,4,9,7,8,14,10,13,11,12
密文：HCBTSXWCRQGLES
flag格式 flag{你解密的内容}
```

**解答**

这个转盘加密，比如第一个密钥匙：2、密文匙：H

把转盘第二行单独提出来  2： <KPBELNACZDTRXMJQOY**HGVSFUWI** < ；从H的地方一直剪切，把剪切的内容放在最前面，变成   2： <**HGVSFUWI**KPBELNACZDTRXMJQOY <

依次类推把14行都按这样的方式整一遍就得到这个：

```
2：  <HGVSFUWIKPBELNACZDTRXMJQOY <
5：  <CPMNZQWXYIHFRLABEUOTSGJVDK <
1：  <BVIQHKYPNTCRMOSFEZWAXJGDLU <
3：  <TEQGYXPLOCKBDMAIZVRNSJUWFH <
6：  <SLOQXVETAMKGHIWPNYCJBFZDRU <
4：  <XQYIZMJWAORPLNDVHGFCUKTEBS <
9：  <WATDSRFHENYVUBMCOIKZGJXPLQ <
7：  <CEONJQGWTHSPYBXIZULVKMRAFD <
8：  <RJLXKISEFAPMYGHBQNOZUTWDCV <
14：	<QWXPHKZGJTDSENYVUBMLAOIRFC <
10：	<GOIKFHENYVUWABMCXPLTDSRJQZ <
13：	<LTDENQWAOXPYVUIKZGJBMCSRFH <
11：	<ENYSRUBMCQWVJXPLTDAOIKFZGH <
12：	<SWAYXPLVUBOIKZGJRFHENMCQTD <
```

flag在倒数第六列。

flag{XSXSBUGKUADMIN}，可能要转化为小写。

***

# 04 文件加密

## 0x01 zip压缩包破解

[我跳]()

## 0x02 Linux shadow 文件破解密码

john爆破：`john filename`

# 05 其他加密

## 01 键盘布局加密

密文：`r5yG lp9I BjM tFhBT6uh y7iJ QsZ bhM `

明文：`TONGYUAN `

# 0x 参考

* [CTF中那些脑洞大开的编码和加密](http://blog.neargle.com/SecNewsBak/drops/CTF中那些脑洞大开的编码和加密 .html)
*  具体请看：https://ctf-wiki.github.io/ctf-wiki/crypto/classical/introduction-zh/