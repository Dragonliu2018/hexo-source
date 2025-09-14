---
title: PA答疑之IDEX和IDEXW
tags:
  - PA
categories:
  - 计组
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-23 14:43:37
---

# 1 问题

`IDEX`其实就是相当于`IDEXW(,,0)`？可是最后一个参数不是操作数宽度吗？为什么会有0呢？

```c
#define IDEXW(id, ex, w)   {concat(decode_, id), concat(exec_, ex), w}
#define IDEX(id, ex)       IDEXW(id, ex, 0)
```

学妹的问题我不会🥦，于是请舍友帮忙解答，记录下。

# 2 指令的执行周期

## 2.1 取指(instruction fetch, IF)

取指令要做的事情自然就是将 `eip` 指向的指令从内存读入到CPU中。

## 2.2 译码(instruction decode, ID)

CPU拿到一条指令之后，可以通过查表的方式得知这条指令的操作数和操作码。这个过程叫译码。

计算机现在已经有存储器和寄存器了，它们都可以存放操作数，指令中也可以存放立即数，也可能还有二次译码的处理。

## 2.3 执行(execute, EX)

执行阶段就是真正完成指令的工作。现在 TRM 只有加法器这一个执行部件，必要的时候,，只需要往加法器输入两个源操作数, 就能得到执行的结果了。之后还要把结果写回到目的操作数中, 可能是寄存器, 也可能是内存。

## 2.4 更新 `eip`

执行完一条指令之后，CPU就要执行下一条指令。在这之前，CPU 需要更新 `eip` 的值，让 `eip` 加上刚才执行完的指令的长度, 即可指向下一条指令的位置。

# 3 一条指令在NEMU中的执行过程

## 3.1 IDEX与IDEXW

```c++
typedef struct {
  DHelper decode;
  EHelper execute;
  int width;
} opcode_entry;

#define IDEXW(id, ex, w)   {concat(decode_, id), concat(exec_, ex), w}
#define IDEX(id, ex)       IDEXW(id, ex, 0)

opcode_entry opcode_table [512] = {
  /* 0x00 */	IDEXW(G2E, add, 1), IDEX(G2E, add), IDEXW(E2G, add, 1), IDEX(E2G, add),
  /* 0x04 */	IDEXW(I2a, add, 1), IDEX(I2a, add), EMPTY, EMPTY,
    ...
}
```

`IDEXW`中的`w`也就是结构体`opcode_entry`中的`width`。

## 3.2 过程

### 3.2.1 main.c

nemu运行起来，主函数 `nemu/src/main.c`：

```c++
int main(int argc, char *argv[]) {
  /* Initialize the monitor. */
  int is_batch_mode = init_monitor(argc, argv);

  /* Receive commands from user. */
  ui_mainloop(is_batch_mode);

  return 0;
}
```

### 3.2.2 ui_mainloop() 

执行`ui_mainloop`函数

```c++
void ui_mainloop(int is_batch_mode) {
  if (is_batch_mode) { //批处理
    cmd_c(NULL);
    return;
  }

  while (1) {
	// 处理传来的字符串
  	...
    // 执行指令
    int i;
    for (i = 0; i < NR_CMD; i ++) {
      if (strcmp(cmd, cmd_table[i].name) == 0) {
        if (cmd_table[i].handler(args) < 0) { return; }
        break;
      }
    }

    if (i == NR_CMD) { printf("Unknown command '%s'\n", cmd); }
  }
}
```

### 3.2.3 cmd_c()

例如在nemu下执行`c`命令（继续运行被暂停的程序至结束）：

```c++
static int cmd_c(char *args) {
  cpu_exec(-1);
  return 0;
}
```

### 3.2.4 cpu_exec()

然后调用cpu_exec函数：

```c
/* Simulate how the CPU works. */
void cpu_exec(uint64_t n) {
  ...
  for (; n > 0; n --) {
    /* Execute one instruction, including instruction fetch,
     * instruction decode, and the actual execution. */
    exec_wrapper(print_flag);
  }
  ...
}
```

### 3.2.5 exec_wrapper()

> **`exec_wrapper()` 的执行过程**：
>
> 首先将当前的 `eip` 保存到全局译码信息 `decoding` 的成员 `seq_eip` 中,然后将其地址被作为参数送进`exec_real()` 函数中。`seq` 代表顺序的意思, 当代码从 `exec_real()` 返回时, `decoding.seq_eip` 将会指向下一条指令的地址.`exec_real()` 函数通过宏 `make_EHelper` 来定义:
>
> ```c
> #define make_EHelper(name) void concat(exec_, name) (vaddr_t *eip)
> ```
>
> 其含义是"定义一个执行阶段相关的helper函数", 这些函数都带有一个参数`eip`。NEMU通过不同的helper函数来模拟不同的步骤.

调用了exec_wrapper函数：

```c++
void exec_wrapper(bool print_flag) {
  ...
  decoding.seq_eip = cpu.eip;
  exec_real(&decoding.seq_eip);
  ...
}
```

获取当前cpu的eip，然后执行exec_real函数。

### 3.2.6 exec_real()

> 在 `exec_real()` 中:
>
> - 首先通过 `instr_fetch()` 函数(在`nemu/include/cpu/exec.h`中定义)进行**取指**, 得到指令的第一个字节, 将其解释成 `opcode` 并记录在全局译码信息 `decoding` 中；
> - 根据 `opcode` 查阅译码查找表, 得到操作数的宽度信息, 并通过调用 `set_width()` 函数将其记录在全局译码信息 `decoding` 中；
> - 调用 `idex()` 对指令进行进一步的译码和执行。

因为宏定义，exec_real也就是make_EHelper(real)函数：

```c++
#define make_EHelper(name) void concat(exec_, name) (vaddr_t *eip)
```

```c++
make_EHelper(real) {
  uint32_t opcode = instr_fetch(eip, 1);
  decoding.opcode = opcode;
  set_width(opcode_table[opcode].width);
  idex(eip, &opcode_table[opcode]);
}
```

decoding是保存当前单条命令译码结果的结构体。

#### 3.2.6.1 set_width()（解决问题）

**set_width函数使用到了width**：

```c++
static inline void set_width(int width) {
  if (width == 0) {
    width = decoding.is_operand_size_16 ? 2 : 4;
  }
  decoding.src.width = decoding.dest.width = decoding.src2.width = width;
}
```

**set_width函数中表明：如果width设置0，就可能是2字节或4字节；如果不设置就是width的值；就是设置操作数（src+dest）的宽度。**

#### 3.2.6.2 idex()

set_width函数之后执行idex函数，译码+执行：

```c++
/* Instruction Decode and EXecute */
static inline void idex(vaddr_t *eip, opcode_entry *e) {
  /* eip is pointing to the byte next to opcode */
  if (e->decode)
    e->decode(eip);
  e->execute(eip);
}
```

首先会调用译码查找表中的相应的译码函数进行操作数的**译码**. 译码函数统一通过宏 `make_DHelper` 来定义(在 `nemu/src/cpu/decode/decode.c` 中)：

```c++
#define make_DHelper(name) void concat(decode_, name) (vaddr_t *eip)
```

> 它们的名字主要采用 **i386 手册附录 A**中的操作数表示记号, 例如 `I2r` 表示将立即数移入寄存器, 其中 `I` 表示立即数, `2` 表示英文 `to`, `r` 表示通用寄存器, 更多的记号请参考 **i386 手册**.译码函数会把指令中的操作数信息分别记录在全局译码信息 `decoding` 中。
>
> 这些译码函数会进一步分解成各种不同操作数的译码的组合, 以实现操作数译码的解耦. 操作数译码函数统一通过宏 `make_DopHelper` 来定义 (在 `nemu/src/cpu/decode/decode.c` 中, `decode_op_rm()` 除外):
>
> ```c
> #define make_DopHelper(name) void concat(decode_op_, name) (vaddr_t *eip, Operand *op, bool load_val)
> ```
>
> 它们的名字主要采用 **i386 手册附录 A** 中的操作数表示记号. 操作数译码函数会把操作数的信息记录在结构体 `op` 中, 如果操作数在指令中, 就会通过 `instr_fetch()` 将它们从 `eip` 所指向的内存位置取出. 为了使操作数译码函数更易于复用, 函数中的 `load_val` 参数会控制是否需要将该操作数读出到全局译码信息 `decoding` 供后续使用. 例如如果一个内存操作数是源操作数, 就需要将这个操作数从内存中读出来供后续执行阶段来使用; 如果它仅仅是一个目的操作数, 就不需要从内存读出它的值了, 因为执行这条指令并不需要这个值, 而是将新数据写入相应的内存位置.

`idex()` 函数中的译码过程结束之后, 会调用译码查找表中的相应的执行函数来进行真正的**执行**操作. 执行函数统一通过宏 `make_EHelper` 来定义, 它们的名字是指令操作本身. 执行函数通过 RTL 来描述指令真正的执行功能(RTL 将在下文介绍). 其中 `operand_write()` 函数(在 `nemu/src/cpu/decode/decode.c` 中定义) 会根据第一个参数中记录的类型的不同进行相应的写操作, 包括写寄存器和写内存.

从 `idex()` 返回后, `exec_real()` 最后会通过 `update_eip()` 对 `eip` 进行更新。

### 3.2.7 指令执行结束后

执行完后，再次返回cpu_exec函数的for循环：

```c++
/* Simulate how the CPU works. */
void cpu_exec(uint64_t n) {
  ...
  for (; n > 0; n --) {
    /* Execute one instruction, including instruction fetch,
     * instruction decode, and the actual execution. */
    exec_wrapper(print_flag);
  }
  ...
}
```

循环上述流程，直到你指定的步数（c命令是-1，也就是最大值，也就是执行全部）。

# X 参考

* [讲义](https://zhong-kangwei.gitee.io/ics-pa-gitbook-2022/pa2/2.1.html)
* [NJU ICS Programming Assignment 代码分析 - NEMU](http://www.stardustdl.top/posts/learning/nju-icspa-analytics-nemu/)
* [NJU ics 2020 概要](https://ilern.github.io/2021/01/12/NJU-ics-2020-%E6%A6%82%E8%A6%81/#%E8%AF%B7%E6%95%B4%E7%90%86%E4%B8%80%E6%9D%A1%E6%8C%87%E4%BB%A4%E5%9C%A8NEMU%E4%B8%AD%E7%9A%84%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B)
