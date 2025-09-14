---
title: Lab2实验总结
tags:
  - Lab
categories:
  - 计组
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-15 21:58:16
---

# 1 准备

## 1.1 安装gdb-peda

gdb-peda是gdb的插件，加强gdb调试能力。

安装方法如下：

```sh
gdb-peda$ set disassembly-flavor att$ git clone htgdb-peda$ set disassembly-flavor atttps://github.com/longld/peda.git ~/peda
$ echo "source ~/peda/peda.py" >> ~/.gdbinit 
```

安装插件后汇编指令格式改为了intel模式，不太习惯，改变为AT&T格式：

```sh
# 改为AT&T格式
gdb-peda$ set disassembly-flavor att

# 改为Intel格式
gdb-peda$ set disassembly-flavor intel
```

**参考**：[gdb-peda安装](https://blog.csdn.net/counsellor/article/details/81290335)

## 1.2 gdb&peda使用

```sh
# 载入可执行程序
gdb <file name>

# 设置断点
b <地址或函数名等>

# (重新)运行程序，若有断点，会卡在第一个断点
r

# 继续执行程序至下一个断点
c

# 执行一(n)条指令，进入call
s <n>
# 执行一(n)条指令，不进入call
n <n>

# 查看断点
info b
i b
# 删除断点 n为断点编号
delete <n>
d <n>

# 反汇编，func为某个函数
disassemble <func>

# 显示addr处的内存信息
x/nfu <addr>
#n表示输出单元的个数
#f是输出格式。比如x是以16进制形式输出，o是以8进制形式输出等等。
#u标明一个单元的长度。b是一个byte，h是两个byte（halfword），w是四个byte（word），g是八个byte（giant word）。
```

## 1.3 计组知识

 **汇编格式**：

* Intel格式：目的操作数在左，MASM采用
* AT&T格式：目的操作数在右，objdump和gcc默认格式

**栈**：

* 先进后出
* 从高地址向低地址增长
* ESP指向栈顶
* EBP指向栈底

**大小端模式**：

* 大端（Big-Endian）：高字节存低地址，低字节存低高地址
* 小端（Little-Endian）：低字节存低地址，高字节存高地址

**常见指令**：

* `push`：R[sp] ← R[sp] - 2 或者  R[esp] ← R[esp] - 4，然后将一个字或双字从指定寄存器送到SP或者ESP指示的单元；
* `pop`：然后将一个字或双字从SP或者ESP指示的单元送到指定寄存器，再执行R[sp] ← R[sp] + 2 或者  R[esp] ← R[esp] + 4
* `call <func_addr>`：先push EIP（当前指令的第一条指令地址），再执行jmp <func_addr>
* `ret`：pop EIP
* `leave`：先执行mov %ebp %esp，再pop %ebp

## 1.4 lab概览

通过IDA打开bomb文件，找到主函数main，发现需要需要进行六次输入，对应phase_1到phase_6函数，每次输入需要躲开explode_bomb()函数，否则拆炸弹失败。

# 2 phase_1

## 2.1 解析

IDA查看phase_1函数伪代码：

```c
int __cdecl phase_1(int a1)
{
  int result; // eax

  result = strings_not_equal(a1, "And they have no disregard for human life.");
  if ( result )
    explode_bomb();
  return result;
}
```

gdb查看phase_1函数汇编代码：

```sh
gdb-peda$ disassemble phase_1
Dump of assembler code for function phase_1:
=> 0x00401662 <+0>:     endbr32
   0x00401666 <+4>:     push   ebp
   0x00401667 <+5>:     mov    ebp,esp
   0x00401669 <+7>:     push   ebx
   0x0040166a <+8>:     sub    esp,0xc
   0x0040166d <+11>:    call   0x4013d0 <__x86.get_pc_thunk.bx>
   0x00401672 <+16>:    add    ebx,0x38f2
   0x00401678 <+22>:    lea    eax,[ebx-0x1e20]
   0x0040167e <+28>:    push   eax
   0x0040167f <+29>:    push   DWORD PTR [ebp+0x8]
   0x00401682 <+32>:    call   0x401bdf <strings_not_equal>
   0x00401687 <+37>:    add    esp,0x10
   0x0040168a <+40>:    test   eax,eax
   0x0040168c <+42>:    jne    0x401693 <phase_1+49>
   0x0040168e <+44>:    mov    ebx,DWORD PTR [ebp-0x4]
   0x00401691 <+47>:    leave
   0x00401692 <+48>:    ret
   0x00401693 <+49>:    call   0x401e64 <explode_bomb>
   0x00401698 <+54>:    jmp    0x40168e <phase_1+44>
End of assembler dump.
```

## 2.2 思路

看伪代码直接得出答案，输入需要与给定字符串一致。

***

如果不看伪代码，使用gdb动态调试则：

1. 载入bomb：

   ```sh
   $ gdb bomb
   GNU gdb (Debian 8.2.1-2+b3) 8.2.1
   Copyright (C) 2018 Free Software Foundation, Inc.
   License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
   This is free software: you are free to change and redistribute it.
   There is NO WARRANTY, to the extent permitted by law.
   Type "show copying" and "show warranty" for details.
   This GDB was configured as "i686-linux-gnu".
   Type "show configuration" for configuration details.
   For bug reporting instructions, please see:
   <http://www.gnu.org/software/gdb/bugs/>.
   Find the GDB manual and other documentation resources online at:
       <http://www.gnu.org/software/gdb/documentation/>.
   
   For help, type "help".
   Type "apropos word" to search for commands related to "word"...
   
   warning: build/bdist.linux-i686/wheel/peda/peda.py: No such file or directory
   Reading symbols from bomb...done.
   gdb-peda$
   ```

2. 在main函数处打断点，并执行到该断点：

   ```sh
   gdb-peda$ b main
   Breakpoint 1 at 0x14cd: file bomb.c, line 37.
   gdb-peda$ r
   ```

3. 在phase_1函数处打断点，并执行到该断点：

   ```sh
   gdb-peda$ b phase_1
   Breakpoint 2 at 0x401662
   gdb-peda$ c
   Continuing.
   Welcome to my fiendish little bomb. You have 6 phases with
   which to blow yourself up. Have a nice day!
   [-------------------------------------code-------------------------------------]
      0x401651 <main+388>: call   0x401300 <__printf_chk@plt>
      0x401656 <main+393>: mov    DWORD PTR [esp],0x8
      0x40165d <main+400>: call   0x4012a0 <exit@plt>
   => 0x401662 <phase_1>:  endbr32
      0x401666 <phase_1+4>:        push   ebp
      0x401667 <phase_1+5>:        mov    ebp,esp
      0x401669 <phase_1+7>:        push   ebx
      0x40166a <phase_1+8>:        sub    esp,0xc
   ```

4. 执行8步到`0x0040167e <+28>:    push   eax`：

   ```sh
   gdb-peda$ n 8
   [----------------------------------registers-----------------------------------]
   EAX: 0x403144 ("And they have no disregard for human life.")
   EBX: 0x404f64 --> 0x4e6c ('lN')
   ECX: 0x2b ('+')
   EDX: 0x1
   ESI: 0xbffff574 --> 0xbffff6bf ("/home/liuzhenlong/lab_debug/bomb_162020203/bomb95/bomb")
   EDI: 0xb7fbc000 --> 0x1d9d6c
   EBP: 0xbffff498 --> 0xbffff4c8 --> 0x0
   ESP: 0xbffff488 --> 0xb7e13cb9 (<__new_exitfn+9>:       add    ebx,0x1a8347)
   EIP: 0x40167e (<phase_1+28>:    push   eax)
   EFLAGS: 0x202 (carry parity adjust zero sign trap INTERRUPT direction overflow)
   [-------------------------------------code-------------------------------------]
      0x40166d <phase_1+11>:       call   0x4013d0 <__x86.get_pc_thunk.bx>
      0x401672 <phase_1+16>:       add    ebx,0x38f2
      0x401678 <phase_1+22>:       lea    eax,[ebx-0x1e20]
   => 0x40167e <phase_1+28>:       push   eax
      0x40167f <phase_1+29>:       push   DWORD PTR [ebp+0x8]
      0x401682 <phase_1+32>:       call   0x401bdf <strings_not_equal>
      0x401687 <phase_1+37>:       add    esp,0x10
      0x40168a <phase_1+40>:       test   eax,eax
   [------------------------------------stack-------------------------------------]
   0000| 0xbffff488 --> 0xb7e13cb9 (<__new_exitfn+9>:      add    ebx,0x1a8347)
   0004| 0xbffff48c --> 0x404f64 --> 0x4e6c ('lN')
   0008| 0xbffff490 --> 0xbffff574 --> 0xbffff6bf ("/home/liuzhenlong/lab_debug/bomb_162020203/bomb95/bomb")
   0012| 0xbffff494 --> 0x404f64 --> 0x4e6c ('lN')
   0016| 0xbffff498 --> 0xbffff4c8 --> 0x0
   0020| 0xbffff49c --> 0x40155a (<main+141>:      call   0x40203c <phase_defused>)
   0024| 0xbffff4a0 --> 0x405760 ("And they have no disregard for human life.")
   0028| 0xbffff4a4 --> 0x404f64 --> 0x4e6c ('lN')
   [------------------------------------------------------------------------------]
   Legend: code, data, rodata, value
   0x0040167e in phase_1 ()
   ```

   此时可以看到EAX中所指向的字符串，也就是答案。

**答案**：

```sh
And they have no disregard for human life.
```

# 3 phase_2

## 3.1 解析

IDA查看phase_2函数伪代码：

```c
int __cdecl read_six_numbers(int a1, int a2)
{
  int result; // eax

  result = __isoc99_sscanf(a1, &unk_33A1, a2, a2 + 4, a2 + 8);
  if ( result <= 5 )
    explode_bomb();
  return result;
}

unsigned int __cdecl phase_2(int a1)
{
  int *v1; // esi
  int v3[5]; // [esp+Ch] [ebp-34h] BYREF
  char v4; // [esp+20h] [ebp-20h] BYREF
  unsigned int v5; // [esp+24h] [ebp-1Ch]

  v5 = __readgsdword(0x14u);
  read_six_numbers(a1, v3);
  if ( v3[0] != 1 )
    explode_bomb();
  v1 = v3;
  do
  {
    if ( v1[1] != 2 * *v1 )
      explode_bomb();
    ++v1;
  }
  while ( v1 != (int *)&v4 );
  return __readgsdword(0x14u) ^ v5;
}
```

gdb查看phase_2函数汇编代码：

```sh
gdb-peda$ disassemble phase_2
Dump of assembler code for function phase_2:
   0x0040169a <+0>:     endbr32
   0x0040169e <+4>:     push   ebp
   0x0040169f <+5>:     mov    ebp,esp
   0x004016a1 <+7>:     push   edi
   0x004016a2 <+8>:     push   esi
   0x004016a3 <+9>:     push   ebx
   0x004016a4 <+10>:    sub    esp,0x34
   0x004016a7 <+13>:    call   0x4013d0 <__x86.get_pc_thunk.bx>
   0x004016ac <+18>:    add    ebx,0x38b8
   0x004016b2 <+24>:    mov    eax,gs:0x14
   0x004016b8 <+30>:    mov    DWORD PTR [ebp-0x1c],eax
   0x004016bb <+33>:    xor    eax,eax
   0x004016bd <+35>:    lea    eax,[ebp-0x34]
   0x004016c0 <+38>:    push   eax
   0x004016c1 <+39>:    push   DWORD PTR [ebp+0x8]
   0x004016c4 <+42>:    call   0x401eba <read_six_numbers>
   0x004016c9 <+47>:    add    esp,0x10
   0x004016cc <+50>:    cmp    DWORD PTR [ebp-0x34],0x1
   0x004016d0 <+54>:    jne    0x4016da <phase_2+64>
   0x004016d2 <+56>:    lea    esi,[ebp-0x34]
   0x004016d5 <+59>:    lea    edi,[ebp-0x20]
   0x004016d8 <+62>:    jmp    0x4016ed <phase_2+83>
   0x004016da <+64>:    call   0x401e64 <explode_bomb>
   0x004016df <+69>:    jmp    0x4016d2 <phase_2+56>
   0x004016e1 <+71>:    call   0x401e64 <explode_bomb>
   0x004016e6 <+76>:    add    esi,0x4
   0x004016e9 <+79>:    cmp    esi,edi
   0x004016eb <+81>:    je     0x4016f8 <phase_2+94>
   0x004016ed <+83>:    mov    eax,DWORD PTR [esi]
   0x004016ef <+85>:    add    eax,eax
   0x004016f1 <+87>:    cmp    DWORD PTR [esi+0x4],eax
   0x004016f4 <+90>:    je     0x4016e6 <phase_2+76>
   0x004016f6 <+92>:    jmp    0x4016e1 <phase_2+71>
   0x004016f8 <+94>:    mov    eax,DWORD PTR [ebp-0x1c]
   0x004016fb <+97>:    xor    eax,DWORD PTR gs:0x14
   0x00401702 <+104>:   jne    0x40170c <phase_2+114>
   0x00401704 <+106>:   lea    esp,[ebp-0xc]
   0x00401707 <+109>:   pop    ebx
   0x00401708 <+110>:   pop    esi
   0x00401709 <+111>:   pop    edi
   0x0040170a <+112>:   pop    ebp
   0x0040170b <+113>:   ret
   0x0040170c <+114>:   call   0x402e00 <__stack_chk_fail_local>
End of assembler dump.
```

## 3.2 思路

从伪代码看：需要输入6个数字，而且第一个数字为1，后面的数字是前面的两倍。

**答案**：

```sh
1 2 4 8 16 32
```

# 4 phase_3

## 4.1 解析

IDA查看phase_3函数伪代码：

```c
void __cdecl phase_3(int a1)
{
  unsigned int v1; // [esp+4h] [ebp-14h] BYREF
  int v2[3]; // [esp+8h] [ebp-10h] BYREF

  v2[1] = __readgsdword(0x14u);
  if ( __isoc99_sscanf(a1, "%d %d", &v1, v2) > 1 )
  {
    if ( v1 <= 7 )
      __asm { jmp     edx }
    explode_bomb();
  }
  explode_bomb();
}
```

gdb查看phase_3函数汇编代码：

```sh
gdb-peda$ disassemble phase_3
Dump of assembler code for function phase_3:
=> 0x00401711 <+0>:     endbr32
   0x00401715 <+4>:     push   ebp
   0x00401716 <+5>:     mov    ebp,esp
   0x00401718 <+7>:     push   ebx
   0x00401719 <+8>:     sub    esp,0x14
   0x0040171c <+11>:    call   0x4013d0 <__x86.get_pc_thunk.bx>
   0x00401721 <+16>:    add    ebx,0x3843
   0x00401727 <+22>:    mov    eax,gs:0x14
   0x0040172d <+28>:    mov    DWORD PTR [ebp-0xc],eax
   0x00401730 <+31>:    xor    eax,eax
   0x00401732 <+33>:    lea    eax,[ebp-0x10]
   0x00401735 <+36>:    push   eax
   0x00401736 <+37>:    lea    eax,[ebp-0x14]
   0x00401739 <+40>:    push   eax
   0x0040173a <+41>:    lea    eax,[ebx-0x1bb7]
   0x00401740 <+47>:    push   eax
   0x00401741 <+48>:    push   DWORD PTR [ebp+0x8]
   0x00401744 <+51>:    call   0x4012d0 <__isoc99_sscanf@plt>
   0x00401749 <+56>:    add    esp,0x10
   0x0040174c <+59>:    cmp    eax,0x1
   0x0040174f <+62>:    jle    0x40176a <phase_3+89>
   0x00401751 <+64>:    cmp    DWORD PTR [ebp-0x14],0x7
   0x00401755 <+68>:    ja     0x4017e1 <phase_3+208>
   0x0040175b <+74>:    mov    eax,DWORD PTR [ebp-0x14]
   0x0040175e <+77>:    mov    edx,ebx
   0x00401760 <+79>:    add    edx,DWORD PTR [ebx+eax*4-0x1dc4]
   0x00401767 <+86>:    notrack jmp edx
   0x0040176a <+89>:    call   0x401e64 <explode_bomb>
   0x0040176f <+94>:    jmp    0x401751 <phase_3+64>
   0x00401771 <+96>:    mov    eax,0x398
   0x00401776 <+101>:   sub    eax,0xd8
   0x0040177b <+106>:   add    eax,0x1ad
   0x00401780 <+111>:   sub    eax,0x3e
   0x00401783 <+114>:   add    eax,0x3e
   0x00401786 <+117>:   sub    eax,0x3e
   0x00401789 <+120>:   add    eax,0x3e
   0x0040178c <+123>:   sub    eax,0x3e
   0x0040178f <+126>:   cmp    DWORD PTR [ebp-0x14],0x5
   0x00401793 <+130>:   jg     0x40179a <phase_3+137>
   0x00401795 <+132>:   cmp    DWORD PTR [ebp-0x10],eax
   0x00401798 <+135>:   je     0x40179f <phase_3+142>
   0x0040179a <+137>:   call   0x401e64 <explode_bomb>
   0x0040179f <+142>:   mov    eax,DWORD PTR [ebp-0xc]
   0x004017a2 <+145>:   xor    eax,DWORD PTR gs:0x14
   0x004017a9 <+152>:   jne    0x4017ed <phase_3+220>
   0x004017ab <+154>:   mov    ebx,DWORD PTR [ebp-0x4]
   0x004017ae <+157>:   leave
   0x004017af <+158>:   ret
   0x004017b0 <+159>:   mov    eax,0x0
   0x004017b5 <+164>:   jmp    0x401776 <phase_3+101>
   0x004017b7 <+166>:   mov    eax,0x0
   0x004017bc <+171>:   jmp    0x40177b <phase_3+106>
   0x004017be <+173>:   mov    eax,0x0
   0x004017c3 <+178>:   jmp    0x401780 <phase_3+111>
   0x004017c5 <+180>:   mov    eax,0x0
   0x004017ca <+185>:   jmp    0x401783 <phase_3+114>
   0x004017cc <+187>:   mov    eax,0x0
   0x004017d1 <+192>:   jmp    0x401786 <phase_3+117>
   0x004017d3 <+194>:   mov    eax,0x0
   0x004017d8 <+199>:   jmp    0x401789 <phase_3+120>
   0x004017da <+201>:   mov    eax,0x0
   0x004017df <+206>:   jmp    0x40178c <phase_3+123>
   0x004017e1 <+208>:   call   0x401e64 <explode_bomb>
   0x004017e6 <+213>:   mov    eax,0x0
   0x004017eb <+218>:   jmp    0x40178f <phase_3+126>
   0x004017ed <+220>:   call   0x402e00 <__stack_chk_fail_local>
End of assembler dump.
```

## 4.2 思路

从伪代码看，phase_3需要输入两个整数，第一个整数需要小于等于7，然后跳转到EDX，其他信息无法获取，需要gdb调试。

假设第一个数为0，进行动态调试：

1. 运行到`<+79>`，此时EDX=0x401771 (`<phase_3+96>:    mov    eax,0x398`)

2. `<+86>`表示跳转到EDX对应的指令地址，也就是`<+96>`

3. `<+126>`和`<+130>`表示**第一个数还需要小于等于5**：

   ```sh
   0x0040178f <+126>:   cmp    DWORD PTR [ebp-0x14],0x5
   0x00401793 <+130>:   jg     0x40179a <phase_3+137>
   ```

4. `<+132>`表示第二个数需要等于EAX，此时的EAX为0x42f：

   ```sh
   gdb-peda$ p $eax
   $24 = 0x42f
   gdb-peda$ d $24
   No breakpoint number 1071.
   ```

5. 此时可以得到一组答案`0 1071`，答案不唯一。

**答案**：

```sh
0 1071
```

# 5 phase_4

## 5.1 解析

IDA查看phase_4函数伪代码：

```c
int __cdecl func4(int a1, int a2)
{
  int result; // eax
  int v3; // ebx

  result = 0;
  if ( a1 > 0 )
  {
    result = a2;
    if ( a1 != 1 )
    {
      v3 = func4(a1 - 1, a2) + a2;
      result = v3 + func4(a1 - 2, a2);
    }
  }
  return result;
}

unsigned int __cdecl phase_4(int a1)
{
  int v1; // eax
  int v3; // [esp+4h] [ebp-14h] BYREF
  int v4; // [esp+8h] [ebp-10h] BYREF
  unsigned int v5; // [esp+Ch] [ebp-Ch]

  v5 = __readgsdword(0x14u);
  if ( __isoc99_sscanf(a1, "%d %d", &v4, &v3) != 2 || (unsigned int)(v3 - 2) > 2 )
    explode_bomb();
  v1 = func4(5, v3);
  if ( v4 != v1 )
    explode_bomb();
  return __readgsdword(0x14u) ^ v5;
}
```

gdb查看phase_4函数汇编代码：

```sh
gdb-peda$ disassemble phase_4
Dump of assembler code for function phase_4:
   0x0040183f <+0>:     endbr32
   0x00401843 <+4>:     push   ebp
   0x00401844 <+5>:     mov    ebp,esp
   0x00401846 <+7>:     push   ebx
   0x00401847 <+8>:     sub    esp,0x14
   0x0040184a <+11>:    call   0x4013d0 <__x86.get_pc_thunk.bx>
   0x0040184f <+16>:    add    ebx,0x3715
   0x00401855 <+22>:    mov    eax,gs:0x14
   0x0040185b <+28>:    mov    DWORD PTR [ebp-0xc],eax
   0x0040185e <+31>:    xor    eax,eax
   0x00401860 <+33>:    lea    eax,[ebp-0x14]
   0x00401863 <+36>:    push   eax
   0x00401864 <+37>:    lea    eax,[ebp-0x10]
   0x00401867 <+40>:    push   eax
   0x00401868 <+41>:    lea    eax,[ebx-0x1bb7]
   0x0040186e <+47>:    push   eax
   0x0040186f <+48>:    push   DWORD PTR [ebp+0x8]
   0x00401872 <+51>:    call   0x4012d0 <__isoc99_sscanf@plt>
   0x00401877 <+56>:    add    esp,0x10
   0x0040187a <+59>:    cmp    eax,0x2
   0x0040187d <+62>:    jne    0x40188a <phase_4+75>
   0x0040187f <+64>:    mov    eax,DWORD PTR [ebp-0x14]
   0x00401882 <+67>:    sub    eax,0x2
   0x00401885 <+70>:    cmp    eax,0x2
   0x00401888 <+73>:    jbe    0x40188f <phase_4+80>
   0x0040188a <+75>:    call   0x401e64 <explode_bomb>
   0x0040188f <+80>:    sub    esp,0x8
   0x00401892 <+83>:    push   DWORD PTR [ebp-0x14]
   0x00401895 <+86>:    push   0x5
   0x00401897 <+88>:    call   0x4017f2 <func4>
   0x0040189c <+93>:    add    esp,0x10
   0x0040189f <+96>:    cmp    DWORD PTR [ebp-0x10],eax
   0x004018a2 <+99>:    jne    0x4018b5 <phase_4+118>
   0x004018a4 <+101>:   mov    eax,DWORD PTR [ebp-0xc]
   0x004018a7 <+104>:   xor    eax,DWORD PTR gs:0x14
   0x004018ae <+111>:   jne    0x4018bc <phase_4+125>
   0x004018b0 <+113>:   mov    ebx,DWORD PTR [ebp-0x4]
   0x004018b3 <+116>:   leave
   0x004018b4 <+117>:   ret
   0x004018b5 <+118>:   call   0x401e64 <explode_bomb>
   0x004018ba <+123>:   jmp    0x4018a4 <phase_4+101>
   0x004018bc <+125>:   call   0x402e00 <__stack_chk_fail_local>
End of assembler dump.
```

## 5.2 思路

从伪代码来看，phase_4需要输入两个整数，第二个可能是2、3、4，第一个需要满足一定的条件。

1. 假设第二个为2，进入gdb动态调试；

2. 执行到`<+96>`，第一个值需要等于此时的EAX，查看EAX的值：

   ```sh
   gdb-peda$ p $eax
   $28 = 0x18
   gdb-peda$ d $28
   No breakpoint number 24.
   ```

3. 所以其中一个答案是`24 2`

**答案**：

```sh
24 2
```

# 6 phase_5

## 6.1 解析

IDA查看phase_5函数伪代码：

```c
_BYTE *__cdecl phase_5(_BYTE *a1)
{
  _BYTE *result; // eax
  int v2; // ecx

  if ( string_length(a1) != 6 )
    explode_bomb();
  result = a1;
  v2 = 0;
  do
    v2 += array_3066[*result++ & 0xF];
  while ( result != a1 + 6 );
  if ( v2 != 44 )
    explode_bomb();
  return result;
}
```

gdb查看phase_5函数汇编代码：

```sh
gdb-peda$ disassemble phase_5
Dump of assembler code for function phase_5:
=> 0x004018c1 <+0>:     endbr32
   0x004018c5 <+4>:     push   ebp
   0x004018c6 <+5>:     mov    ebp,esp
   0x004018c8 <+7>:     push   edi
   0x004018c9 <+8>:     push   esi
   0x004018ca <+9>:     push   ebx
   0x004018cb <+10>:    sub    esp,0x18
   0x004018ce <+13>:    call   0x4013d0 <__x86.get_pc_thunk.bx>
   0x004018d3 <+18>:    add    ebx,0x3691
   0x004018d9 <+24>:    mov    esi,DWORD PTR [ebp+0x8]
   0x004018dc <+27>:    push   esi
   0x004018dd <+28>:    call   0x401bb9 <string_length>
   0x004018e2 <+33>:    add    esp,0x10
   0x004018e5 <+36>:    cmp    eax,0x6
   0x004018e8 <+39>:    jne    0x401917 <phase_5+86>
   0x004018ea <+41>:    mov    eax,esi
   0x004018ec <+43>:    add    esi,0x6
   0x004018ef <+46>:    mov    ecx,0x0
   0x004018f4 <+51>:    lea    edi,[ebx-0x1da4]
   0x004018fa <+57>:    movzx  edx,BYTE PTR [eax]
   0x004018fd <+60>:    and    edx,0xf
   0x00401900 <+63>:    add    ecx,DWORD PTR [edi+edx*4]
   0x00401903 <+66>:    add    eax,0x1
   0x00401906 <+69>:    cmp    eax,esi
   0x00401908 <+71>:    jne    0x4018fa <phase_5+57>
   0x0040190a <+73>:    cmp    ecx,0x2c
   0x0040190d <+76>:    jne    0x40191e <phase_5+93>
   0x0040190f <+78>:    lea    esp,[ebp-0xc]
   0x00401912 <+81>:    pop    ebx
   0x00401913 <+82>:    pop    esi
   0x00401914 <+83>:    pop    edi
   0x00401915 <+84>:    pop    ebp
   0x00401916 <+85>:    ret
   0x00401917 <+86>:    call   0x401e64 <explode_bomb>
   0x0040191c <+91>:    jmp    0x4018ea <phase_5+41>
   0x0040191e <+93>:    call   0x401e64 <explode_bomb>
   0x00401923 <+98>:    jmp    0x40190f <phase_5+78>
End of assembler dump.
```

## 6.2 思路

观察伪代码发现，输入是一段长度为6的字符串，而且需要满足一定的条件。

在IDA中双击`array_3066`查看该数组的值：

```sh
array_3066      dd 2, 0Ah, 6, 1, 0Ch, 10h, 9, 3, 4, 7, 0Eh, 5, 0Bh, 8
```

或者在gdb中查看，运行至`<+51>`：

```sh
gdb-peda$ x/20x $ebx-0x1da4
0x4031c0 <array.3066>:  0x00000002      0x0000000a      0x00000006      0x00000001
0x4031d0 <array.3066+16>:       0x0000000c      0x00000010      0x00000009      0x00000003
0x4031e0 <array.3066+32>:       0x00000004      0x00000007      0x0000000e      0x00000005
0x4031f0 <array.3066+48>:       0x0000000b      0x00000008      0x0000000f      0x0000000d
0x403200:       0x79206f53      0x7420756f      0x6b6e6968      0x756f7920
```

下标可以这样组合：1 1 1 1 0 0 （也就是10+10+10+10+2+2=44，满足条件）

对应到字符串是：`111100` 或 `AAAA@@`等。

**答案**：

```sh
111100
```

# 7 phase_6

## 7.1 解析

IDA查看phase_6函数伪代码：

```c
unsigned int __cdecl phase_6(int a1)
{
  int *v1; // esi
  int i; // esi
  int v3; // ecx
  int v4; // eax
  _DWORD *v5; // edx
  int v6; // esi
  int v7; // eax
  int v8; // edx
  int v9; // eax
  int v10; // edx
  int v11; // eax
  int v12; // edi
  char *v14; // [esp+Ch] [ebp-64h]
  int v15; // [esp+10h] [ebp-60h]
  int v16; // [esp+24h] [ebp-4Ch] BYREF
  char v17; // [esp+28h] [ebp-48h] BYREF
  int v18; // [esp+3Ch] [ebp-34h] BYREF
  int v19; // [esp+40h] [ebp-30h]
  int v20; // [esp+44h] [ebp-2Ch]
  int v21; // [esp+48h] [ebp-28h]
  int v22; // [esp+4Ch] [ebp-24h]
  int v23; // [esp+50h] [ebp-20h]
  unsigned int v24; // [esp+54h] [ebp-1Ch]

  v24 = __readgsdword(0x14u);
  read_six_numbers(a1, (int)&v16);
  v14 = &v17;
  v15 = 0;
  while ( 1 )
  {
    if ( (unsigned int)(*((_DWORD *)v14 - 1) - 1) > 5 )
      explode_bomb();
    if ( ++v15 > 5 )
      break;
    v1 = (int *)v14;
    do
    {
      if ( *((_DWORD *)v14 - 1) == *v1 )
        explode_bomb();
      ++v1;
    }
    while ( &v18 != v1 );
    v14 += 4;
  }
  for ( i = 0; i != 6; ++i )
  {
    v3 = *(&v16 + i);
    v4 = 1;
    v5 = &node1;
    if ( v3 > 1 )
    {
      do
      {
        v5 = (_DWORD *)v5[2];
        ++v4;
      }
      while ( v4 != v3 );
    }
    *(&v18 + i) = (int)v5;
  }
  v6 = v18;
  v7 = v19;
  *(_DWORD *)(v18 + 8) = v19;
  v8 = v20;
  *(_DWORD *)(v7 + 8) = v20;
  v9 = v21;
  *(_DWORD *)(v8 + 8) = v21;
  v10 = v22;
  *(_DWORD *)(v9 + 8) = v22;
  v11 = v23;
  *(_DWORD *)(v10 + 8) = v23;
  *(_DWORD *)(v11 + 8) = 0;
  v12 = 5;
  do
  {
    if ( *(_DWORD *)v6 > **(_DWORD **)(v6 + 8) )
      explode_bomb();
    v6 = *(_DWORD *)(v6 + 8);
    --v12;
  }
  while ( v12 );
  return __readgsdword(0x14u) ^ v24;
}
```

gdb查看phase_6函数汇编代码：

```sh
gdb-peda$ disassemble phase_6
Dump of assembler code for function phase_6:
=> 0x00401925 <+0>:     endbr32
   0x00401929 <+4>:     push   ebp
   0x0040192a <+5>:     mov    ebp,esp
   0x0040192c <+7>:     push   edi
   0x0040192d <+8>:     push   esi
   0x0040192e <+9>:     push   ebx
   0x0040192f <+10>:    sub    esp,0x64
   0x00401932 <+13>:    call   0x4013d0 <__x86.get_pc_thunk.bx>
   0x00401937 <+18>:    add    ebx,0x362d
   0x0040193d <+24>:    mov    eax,gs:0x14
   0x00401943 <+30>:    mov    DWORD PTR [ebp-0x1c],eax
   0x00401946 <+33>:    xor    eax,eax
   0x00401948 <+35>:    lea    eax,[ebp-0x4c]
   0x0040194b <+38>:    push   eax
   0x0040194c <+39>:    push   DWORD PTR [ebp+0x8]
   0x0040194f <+42>:    call   0x401eba <read_six_numbers>
   0x00401954 <+47>:    lea    eax,[ebp-0x48]
   0x00401957 <+50>:    mov    DWORD PTR [ebp-0x64],eax
   0x0040195a <+53>:    add    esp,0x10
   0x0040195d <+56>:    mov    DWORD PTR [ebp-0x60],0x0
   0x00401964 <+63>:    lea    eax,[ebp-0x34]
   0x00401967 <+66>:    mov    DWORD PTR [ebp-0x5c],eax
   0x0040196a <+69>:    jmp    0x40198d <phase_6+104>
   0x0040196c <+71>:    call   0x401e64 <explode_bomb>
   0x00401971 <+76>:    jmp    0x4019a0 <phase_6+123>
   0x00401973 <+78>:    add    esi,0x4
   0x00401976 <+81>:    cmp    DWORD PTR [ebp-0x5c],esi
   0x00401979 <+84>:    je     0x401989 <phase_6+100>
   0x0040197b <+86>:    mov    eax,DWORD PTR [esi]
   0x0040197d <+88>:    cmp    DWORD PTR [edi-0x4],eax
   0x00401980 <+91>:    jne    0x401973 <phase_6+78>
   0x00401982 <+93>:    call   0x401e64 <explode_bomb>
   0x00401987 <+98>:    jmp    0x401973 <phase_6+78>
   0x00401989 <+100>:   add    DWORD PTR [ebp-0x64],0x4
   0x0040198d <+104>:   mov    eax,DWORD PTR [ebp-0x64]
   0x00401990 <+107>:   mov    edi,eax
   0x00401992 <+109>:   mov    eax,DWORD PTR [eax-0x4]
   0x00401995 <+112>:   mov    DWORD PTR [ebp-0x68],eax
   0x00401998 <+115>:   sub    eax,0x1
   0x0040199b <+118>:   cmp    eax,0x5
   0x0040199e <+121>:   ja     0x40196c <phase_6+71>
   0x004019a0 <+123>:   add    DWORD PTR [ebp-0x60],0x1
   0x004019a4 <+127>:   mov    eax,DWORD PTR [ebp-0x60]
   0x004019a7 <+130>:   cmp    eax,0x5
   0x004019aa <+133>:   jg     0x4019b1 <phase_6+140>
   0x004019ac <+135>:   mov    esi,DWORD PTR [ebp-0x64]
   0x004019af <+138>:   jmp    0x40197b <phase_6+86>
   0x004019b1 <+140>:   mov    esi,0x0
   0x004019b6 <+145>:   mov    edi,esi
   0x004019b8 <+147>:   mov    ecx,DWORD PTR [ebp+esi*4-0x4c]
   0x004019bc <+151>:   mov    eax,0x1
   0x004019c1 <+156>:   lea    edx,[ebx+0x594]
   0x004019c7 <+162>:   cmp    ecx,0x1
   0x004019ca <+165>:   jle    0x4019d6 <phase_6+177>
   0x004019cc <+167>:   mov    edx,DWORD PTR [edx+0x8]
   0x004019cf <+170>:   add    eax,0x1
   0x004019d2 <+173>:   cmp    eax,ecx
   0x004019d4 <+175>:   jne    0x4019cc <phase_6+167>
   0x004019d6 <+177>:   mov    DWORD PTR [ebp+edi*4-0x34],edx
   0x004019da <+181>:   add    esi,0x1
   0x004019dd <+184>:   cmp    esi,0x6
   0x004019e0 <+187>:   jne    0x4019b6 <phase_6+145>
   0x004019e2 <+189>:   mov    esi,DWORD PTR [ebp-0x34]
   0x004019e5 <+192>:   mov    eax,DWORD PTR [ebp-0x30]
   0x004019e8 <+195>:   mov    DWORD PTR [esi+0x8],eax
   0x004019eb <+198>:   mov    edx,DWORD PTR [ebp-0x2c]
   0x004019ee <+201>:   mov    DWORD PTR [eax+0x8],edx
   0x004019f1 <+204>:   mov    eax,DWORD PTR [ebp-0x28]
   0x004019f4 <+207>:   mov    DWORD PTR [edx+0x8],eax
   0x004019f7 <+210>:   mov    edx,DWORD PTR [ebp-0x24]
   0x004019fa <+213>:   mov    DWORD PTR [eax+0x8],edx
   0x004019fd <+216>:   mov    eax,DWORD PTR [ebp-0x20]
   0x00401a00 <+219>:   mov    DWORD PTR [edx+0x8],eax
   0x00401a03 <+222>:   mov    DWORD PTR [eax+0x8],0x0
   0x00401a0a <+229>:   mov    edi,0x5
   0x00401a0f <+234>:   jmp    0x401a19 <phase_6+244>
   0x00401a11 <+236>:   mov    esi,DWORD PTR [esi+0x8]
   0x00401a14 <+239>:   sub    edi,0x1
   0x00401a17 <+242>:   je     0x401a29 <phase_6+260>
   0x00401a19 <+244>:   mov    eax,DWORD PTR [esi+0x8]
   0x00401a1c <+247>:   mov    eax,DWORD PTR [eax]
   0x00401a1e <+249>:   cmp    DWORD PTR [esi],eax
   0x00401a20 <+251>:   jle    0x401a11 <phase_6+236>
   0x00401a22 <+253>:   call   0x401e64 <explode_bomb>
   0x00401a27 <+258>:   jmp    0x401a11 <phase_6+236>
   0x00401a29 <+260>:   mov    eax,DWORD PTR [ebp-0x1c]
   0x00401a2c <+263>:   xor    eax,DWORD PTR gs:0x14
   0x00401a33 <+270>:   jne    0x401a3d <phase_6+280>
   0x00401a35 <+272>:   lea    esp,[ebp-0xc]
   0x00401a38 <+275>:   pop    ebx
   0x00401a39 <+276>:   pop    esi
   0x00401a3a <+277>:   pop    edi
   0x00401a3b <+278>:   pop    ebp
   0x00401a3c <+279>:   ret
   0x00401a3d <+280>:   call   0x402e00 <__stack_chk_fail_local>
End of assembler dump.
```

## 7.2 思路

1. 查看伪代码需要输入六个数字；

2. 观察这部分代码块（对应汇编代码`<+47>`到`<+98>`）可得，这六个数字必须从`1 2 3 4 5 6`中选，而且互不相同；

   ```c
   while ( 1 )
   {
       if ( (unsigned int)(*((_DWORD *)v14 - 1) - 1) > 5 )
           explode_bomb();
       if ( ++v15 > 5 )
           break;
       v1 = (int *)v14;
       do
       {
           if ( *((_DWORD *)v14 - 1) == *v1 )
               explode_bomb();
           ++v1;
       }
       while ( &v18 != v1 );
       v14 += 4;
   }
   ```

3. 查看这部分代码块可得，是根据输入的六个数字对node链表进行排序，node链表最后要升序排列（可以有相等的）；

   ```c
   for ( i = 0; i != 6; ++i )
   {
       v3 = *(&v16 + i);
       v4 = 1;
       v5 = &node1;
       if ( v3 > 1 )
       {
           do
           {
               v5 = (_DWORD *)v5[2];
               ++v4;
           }
           while ( v4 != v3 );
       }
       *(&v18 + i) = (int)v5;
   }
   
   do
   {
       if ( *(_DWORD *)v6 > **(_DWORD **)(v6 + 8) )
           explode_bomb();
       v6 = *(_DWORD *)(v6 + 8);
       --v12;
   }
   while ( v12 );
   ```

4. 运行至`<+156> lea    edx,[ebx+0x594]`，查看各个node结点的数值：

   ```sh
   [----------------------------------registers-----------------------------------]
   EAX: 0x1
   EBX: 0x404f64 --> 0x4e6c ('lN')
   ECX: 0x4
   EDX: 0x4054f8 --> 0x2a0
   ESI: 0x1
   EDI: 0x1
   EBP: 0xbffff498 --> 0xbffff4c8 --> 0x0
   ESP: 0xbffff430 --> 0x3
   EIP: 0x4019c7 (<phase_6+162>:   cmp    ecx,0x1)
   EFLAGS: 0x293 (CARRY parity ADJUST zero SIGN trap INTERRUPT direction overflow)
   [-------------------------------------code-------------------------------------]
      0x4019b8 <phase_6+147>:      mov    ecx,DWORD PTR [ebp+esi*4-0x4c]
      0x4019bc <phase_6+151>:      mov    eax,0x1
      0x4019c1 <phase_6+156>:      lea    edx,[ebx+0x594]
   => 0x4019c7 <phase_6+162>:      cmp    ecx,0x1
      0x4019ca <phase_6+165>:      jle    0x4019d6 <phase_6+177>
      0x4019cc <phase_6+167>:      mov    edx,DWORD PTR [edx+0x8]
      0x4019cf <phase_6+170>:      add    eax,0x1
      0x4019d2 <phase_6+173>:      cmp    eax,ecx
   [------------------------------------stack-------------------------------------]
   0000| 0xbffff430 --> 0x3
   0004| 0xbffff434 --> 0xbffff464 --> 0x405528 --> 0x133
   0008| 0xbffff438 --> 0x6
   0012| 0xbffff43c --> 0xbffff464 --> 0x405528 --> 0x133
   0016| 0xbffff440 --> 0x4058f0 ("5 4 2 1 6 3")
   0020| 0xbffff444 --> 0x50 ('P')
   0024| 0xbffff448 --> 0xb7fbc5c0 --> 0xfbad2288
   0028| 0xbffff44c --> 0x5
   [------------------------------------------------------------------------------]
   Legend: code, data, rodata, value
   0x004019c7 in phase_6 ()
   gdb-peda$ x/21x $ebx+0x594
   0x4054f8 <node1>:       0x000002a0      0x00000001      0x00405504      0x0000026d
   0x405508 <node2+4>:     0x00000002      0x00405510      0x000001ea      0x00000003
   0x405518 <node3+8>:     0x0040551c      0x00000038      0x00000004      0x00405528
   0x405528 <node5>:       0x00000133      0x00000005      0x00405080      0x00000000
   0x405538:       0x00000000      0x00000000      0x00403417      0x00000000
   0x405548 <host_table+8>:        0x00000000
   gdb-peda$ x 0x00405080
   0x405080 <node6>:       0x000000ca
   gdb-peda$ x/3x 0x00405080
   0x405080 <node6>:       0x000000ca      0x00000006      0x00000000
   ```

   **注意**：在IDA中查看注意截断

   ```sh
   .data:000054F8                 public node1
   .data:000054F8 node1           db 0A0h                 ; DATA XREF: phase_6+9C↑o
   .data:000054F9                 db    2
   .data:000054FA                 db    0
   .data:000054FB                 db    0
   .data:000054FC                 db    1
   .data:000054FD                 db    0
   .data:000054FE                 db    0
   .data:000054FF                 db    0
   .data:00005500                 dd offset node2
   .data:00005504                 public node2
   .data:00005504 node2           db  6Dh ; m             ; DATA XREF: .data:00005500↑o
   .data:00005505                 db    2
   .data:00005506                 db    0
   .data:00005507                 db    0
   .data:00005508                 db    2
   .data:00005509                 db    0
   .data:0000550A                 db    0
   .data:0000550B                 db    0
   .data:0000550C                 dd offset node3
   ```

   可以得出：

   * node结点的结构：

     ```c
     struct node {
         int val;
         int index;
         struct node * next;
     };
     ```

   * node排列：

     ```sh
     |node1=0x2a0| -> |node2=0x260| -> |node3=0x1ea| -> |node4=0x038| -> |node5=0x133| -> |node6=0x0ca| 
     ```

   所以应输入从小到大排列序号为：4 6 5 3 2 1

**答案**：

```sh
4 6 5 3 2 1
```

# 8 bomblab隐藏关卡

## 8.1 解析

在bomb的伪代码内找到调用 secret_phase 的函数-- phase_defused ，研究phase_defused 汇编代码：首先调用 sscanf ，要求输入 %d %d %s ，而且调用strings_not_equal 函数，只有第三个输入是 `DrEvil` 时触发 secret_phase() 函数。

```c
unsigned int phase_defused()
{
  char v1[4]; // [esp+10h] [ebp-64h] BYREF
  char v2[4]; // [esp+14h] [ebp-60h] BYREF
  char v3[80]; // [esp+18h] [ebp-5Ch] BYREF
  unsigned int v4; // [esp+68h] [ebp-Ch]

  v4 = __readgsdword(0x14u);
  send_msg(1);
  if ( num_input_strings == 6 )
  {
    if ( __isoc99_sscanf(&input_strings[240], "%d %d %s", v1, v2) == 3 && !strings_not_equal(v3, "DrEvil") )
    {
      puts("Curses, you've found the secret phase!");
      puts("But finding it and solving it are quite different...");
      secret_phase();
    }
    puts("Congratulations! You've defused the bomb!");
    puts("Your instructor has been notified and will verify your solution.");
  }
  return __readgsdword(0x14u) ^ v4;
}
```

将已经解决的问题答案放到txt文件中，便于操作；找到一个输入是两个整数的阶段，这里选择了phase_4，成功触发隐藏关卡；（第三阶段无法触发，不知为何？  

```txt
And they have no disregard for human life.
1 2 4 8 16 32
0 1071 
24 2 DrEvil
111100
4 6 5 3 2 1
```

```sh
$ ./bomb flag.txt
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Phase 1 defused. How about the next one?
That's number 2.  Keep going!
Halfway there!
So you got that one.  Try this one.
Good work!  On to the next...
Curses, you've found the secret phase!
But finding it and solving it are quite different...
```

secret_phase伪代码 ：  

```c
unsigned int secret_phase()
{
  const char *v0; // eax
  int v1; // esi

  v0 = read_line();
  v1 = strtol(v0, 0, 10);
  if ( (unsigned int)(v1 - 1) > 0x3E8 )
    explode_bomb();
  if ( fun7(n1, v1) != 3 )
    explode_bomb();
  puts("Wow! You've defused the secret stage!");
  return phase_defused();
}
```

fun7伪代码：

```c
int __cdecl fun7(_DWORD *a1, int a2)
{
  int result; // eax

  if ( !a1 )
    return -1;
  if ( *a1 > a2 )
    return 2 * fun7(a1[1], a2);
  result = 0;
  if ( *a1 != a2 )
    result = 2 * fun7(a1[2], a2) + 1;
  return result;
}
```

## 8.2 思路

1. 进入 secret_phase：

   ```sh
   gdb-peda$ disassemble secret_phase
   Dump of assembler code for function secret_phase:
      0x00401a98 <+0>:     endbr32
      0x00401a9c <+4>:     push   ebp
      0x00401a9d <+5>:     mov    ebp,esp
      0x00401a9f <+7>:     push   esi
      0x00401aa0 <+8>:     push   ebx
      0x00401aa1 <+9>:     call   0x4013d0 <__x86.get_pc_thunk.bx>
      0x00401aa6 <+14>:    add    ebx,0x34be
      0x00401aac <+20>:    call   0x401f09 <read_line>
      0x00401ab1 <+25>:    sub    esp,0x4
      0x00401ab4 <+28>:    push   0xa
      0x00401ab6 <+30>:    push   0x0
      0x00401ab8 <+32>:    push   eax
      0x00401ab9 <+33>:    call   0x401340 <strtol@plt>
      0x00401abe <+38>:    mov    esi,eax
      0x00401ac0 <+40>:    lea    eax,[eax-0x1]
      0x00401ac3 <+43>:    add    esp,0x10
      0x00401ac6 <+46>:    cmp    eax,0x3e8
      0x00401acb <+51>:    ja     0x401b03 <secret_phase+107>
      0x00401acd <+53>:    sub    esp,0x8
      0x00401ad0 <+56>:    push   esi
      0x00401ad1 <+57>:    lea    eax,[ebx+0x540]
      0x00401ad7 <+63>:    push   eax
      0x00401ad8 <+64>:    call   0x401a42 <fun7>
      0x00401add <+69>:    add    esp,0x10
      0x00401ae0 <+72>:    cmp    eax,0x3
      0x00401ae3 <+75>:    jne    0x401b0a <secret_phase+114>
      0x00401ae5 <+77>:    sub    esp,0xc
      0x00401ae8 <+80>:    lea    eax,[ebx-0x1df4]
      0x00401aee <+86>:    push   eax
      0x00401aef <+87>:    call   0x401280 <puts@plt>
      0x00401af4 <+92>:    call   0x40203c <phase_defused>
      0x00401af9 <+97>:    add    esp,0x10
      0x00401afc <+100>:   lea    esp,[ebp-0x8]
      0x00401aff <+103>:   pop    ebx
      0x00401b00 <+104>:   pop    esi
      0x00401b01 <+105>:   pop    ebp
      0x00401b02 <+106>:   ret
      0x00401b03 <+107>:   call   0x401e64 <explode_bomb>
      0x00401b08 <+112>:   jmp    0x401acd <secret_phase+53>
      0x00401b0a <+114>:   call   0x401e64 <explode_bomb>
      0x00401b0f <+119>:   jmp    0x401ae5 <secret_phase+77>
   End of assembler dump.
   ```

   可以看到`<+20>`调用了read_line函数，接着把read_line的返回值赋给了eax，并调用了strtol函数，这个标准库函数的作用是把一个字符串转换成对应的长整型数值。返回值还是存放在eax中，`<+38>`将eax赋值给了esi，`<+40>`将eax减1赋给eax，`<+46>`与1000(0x3e8)进行比较，如果这个值小于等于0x3e8就跳过引爆代码。看到这里可以知道我们需要再加入一行数据，它应该是一个小于等于1001的数值。

2. `<+53>`将 esi 赋给了 M[esp + 4] ，也就是一开始输入的 eax 值。`<+57>`将ebx+0x540赋给了 M[esp] ，`<+64>`调用了 fun7 函数。函数返回后令返回值 eax 与 0x3 做了一个比较，如果相等则跳过引爆代码。所以fun7函数需要返回3；

3. 查看 fun7 函数：

   ```sh
   gdb-peda$ disassemble fun7
   Dump of assembler code for function fun7:
      0x00401a42 <+0>:     endbr32
      0x00401a46 <+4>:     push   ebp
      0x00401a47 <+5>:     mov    ebp,esp
      0x00401a49 <+7>:     push   ebx
      0x00401a4a <+8>:     sub    esp,0x4
      0x00401a4d <+11>:    mov    edx,DWORD PTR [ebp+0x8]
      0x00401a50 <+14>:    mov    ecx,DWORD PTR [ebp+0xc]
      0x00401a53 <+17>:    test   edx,edx
      0x00401a55 <+19>:    je     0x401a91 <fun7+79>
      0x00401a57 <+21>:    mov    ebx,DWORD PTR [edx]
      0x00401a59 <+23>:    cmp    ebx,ecx
      0x00401a5b <+25>:    jg     0x401a69 <fun7+39>
      0x00401a5d <+27>:    mov    eax,0x0
      0x00401a62 <+32>:    jne    0x401a7c <fun7+58>
      0x00401a64 <+34>:    mov    ebx,DWORD PTR [ebp-0x4]
      0x00401a67 <+37>:    leave
      0x00401a68 <+38>:    ret
      0x00401a69 <+39>:    sub    esp,0x8
      0x00401a6c <+42>:    push   ecx
      0x00401a6d <+43>:    push   DWORD PTR [edx+0x4]
      0x00401a70 <+46>:    call   0x401a42 <fun7>
      0x00401a75 <+51>:    add    esp,0x10
      0x00401a78 <+54>:    add    eax,eax
      0x00401a7a <+56>:    jmp    0x401a64 <fun7+34>
      0x00401a7c <+58>:    sub    esp,0x8
      0x00401a7f <+61>:    push   ecx
      0x00401a80 <+62>:    push   DWORD PTR [edx+0x8]
      0x00401a83 <+65>:    call   0x401a42 <fun7>
      0x00401a88 <+70>:    add    esp,0x10
      0x00401a8b <+73>:    lea    eax,[eax+eax*1+0x1]
      0x00401a8f <+77>:    jmp    0x401a64 <fun7+34>
      0x00401a91 <+79>:    mov    eax,0xffffffff
      0x00401a96 <+84>:    jmp    0x401a64 <fun7+34>
   End of assembler dump.
   ```

   查看一下 M[esp] 这个地址里存放的什么数据结构：  

   ```sh
   gdb-peda$ b *0x00401ad8  #打断点至<+64>
   gdb-peda$ x/24xw $eax
   0x4054a4 <n1>:  0x00000024      0x004054b0      0x004054bc      0x00000008
   0x4054b4 <n21+4>:       0x004054e0      0x004054c8      0x00000032      0x004054d4
   0x4054c4 <n22+8>:       0x004054ec      0x00000016      0x0040505c      0x00405044
   0x4054d4 <n33>: 0x0000002d      0x00405020      0x00405068      0x00000006
   0x4054e4 <n31+4>:       0x0040502c      0x00405050      0x0000006b      0x00405038
   0x4054f4 <n34+8>:       0x00405074      0x000002a0      0x00000001      0x00000000
   ```

   仔细观察可以发现这是一个二叉树的结构，每个节点第1个4字节存放数据，第2个4字节存放左子树地址，第3个4字节存放右子树位置。并且命名也有规律，`nab`，`a` 代表层数，`b` 代表从左至右第b个节点。整理可得： 

   ```sh
   0x4054a4 <n1>:  0x00000024      0x004054b0      0x004054bc
   0x4054b0 <n21>: 0x00000008      0x004054e0      0x004054c8
   0x4054bc <n22>: 0x00000032      0x004054d4      0x004054ec
   0x4054e0 <n31>: 0x00000006      0x0040502c      0x00405050
   0x4054c8 <n32>: 0x00000016      0x0040505c      0x00405044
   0x4054d4 <n33>: 0x0000002d      0x00405020      0x00405068
   0x4054ec <n34>: 0x0000006b      0x00405038      0x00405074
   0x40502c <n41>: 0x00000001      0x00000000      0x00000000
   0x405050 <n42>: 0x00000007      0x00000000      0x00000000
   0x40505c <n43>: 0x00000014      0x00000000      0x00000000
   0x405044 <n44>: 0x00000023      0x00000000      0x00000000
   0x405020 <n45>: 0x00000028      0x00000000      0x00000000
   0x405068 <n46>: 0x0000002f      0x00000000      0x00000000
   0x405038 <n47>: 0x00000063      0x00000000      0x00000000
   0x405074 <n48>: 0x000003e9      0x00000000      0x00000000
   ```

   

   ```mermaid
   graph TB
     0x24 --> 0x08
     0x24 --> 0x32
     0x08 --> 0x06
     0x08 --> 0x16
     0x32 --> 0x2d
     0x32 --> 0x6b
     0x06 --> 0x01
     0x06 --> 0x07
     0x2d --> 0x28
     0x2d --> 0x2f
     0x6b --> 0x63
     0x6b --> 0x3e9
     0x16 --> 0x14
     0x16 --> 0x23
   ```

   <img src="https://s2.loli.net/2022/05/26/3jUNCrcuDWAf5Ms.png" width = "600" height = "300" alt="图片名称" align=center id=201 />

4. 总结上面的过程： M[esp] 指向树的一个节点，令节点的值与读入的值进行比较。

   * 如果前者大于后者： M[esp] 移至左子树，返回 2 * eax ；
   * 如果前者不等于后者： M[esp] 移至右子树，返回 2 * eax + 1 ；
   * 如果前者等于后者：返回0  

5. 那么我们需要返回3，根据递归可得当输入是 0x63（99）时返回3。

**答案**：

```sh
99
```

# 9 最终答案

`flag.txt`：

```sh
And they have no disregard for human life.
1 2 4 8 16 32
0 1071 
24 2 DrEvil
111100
4 6 5 3 2 1
99
```

运行：

```sh
$ ./bomb flag.txt
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Phase 1 defused. How about the next one?
That's number 2.  Keep going!
Halfway there!
So you got that one.  Try this one.
Good work!  On to the next...
Curses, you've found the secret phase!
But finding it and solving it are quite different...
Wow! You've defused the secret stage!
Congratulations! You've defused the bomb!
Your instructor has been notified and will verify your solution.
```

