---
title: PA答疑之代码框架
tags:
  - PA
categories:
  - 计组
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-23 15:21:37

---

> **转载**：[NJU ICS Programming Assignment 代码分析 - NEMU](http://www.stardustdl.top/posts/learning/nju-icspa-analytics-nemu/)

> 部分内容框架代码并不包含（如扩展的 Debug 宏），均为我为编码而添加的内容。采用 `a_b` 方式命名的多为原内容，采用 `aB` 方式命名的多为补充内容。由于此项目是 NJU ICS PA 的一部分，其中会包含与相关项目的互操作内容。

NEMU (NJU EMUlator) 是在 Linux 上的一个 n86（x86 子集）模拟器，模拟了基本计算机系统的功能（内存，CPU等）。包含了：

- 内存
- CPU，寄存器
- 调试器（监视器）

# 1 框架代码结构

```sh
nanos-lite/
navy-apps/
nexus-am/
nemu/               # NEMU 项目
    build/          # 构建输出文件夹
        nemu        # NEMU 主程序（可执行文件）
    include/        # 头文件
    src/            # 源码文件
    tools/          # 工具文件
    runall.sh       # 测试 AM cputest 测试集 （nexus-am/tests/cputest）
    Makefile        # NEMU 构建命令
    Makefile.git    # NEMU Git 记录命令
```

nemu部分：

```sh
nemu
├── include                         #   存放全局使用的头文件 
|    ├── common.h                    #   公用的头文件 
|    ├── cpu
|    |    ├── decode.h                #   译码相关
|    |    ├── exec.h                  #   执行相关 
|    |    ├── reg.h                   #   寄存器结构体的定义 
|    |    └── rtl.h                   #   RTL指令 
|    ├── debug.h                     #   一些方便调试用的宏 
|    ├── device                      #   设备相关 
|    ├── macro.h                     #   一些方便的宏定义 
|    ├── memory                      #   访问内存相关 
|    ├── monitor
|    |    ├── expr.h
|    |    ├── monitor.h
|    |    └── watchpoint.h            #   监视点相关 
|    └── nemu.h
├── Makefile                        #   指示NEMU的编译和链接 
├── Makefile.git                    #   git版本控制相关 
├── runall.sh                       #   一键测试脚本 
└── src                             #   源文件
    ├── cpu
    |    ├── decode                  #   译码相关
    |    ├── exec                    #   执行相关 
    |    ├── intr.c                  #   中断处理相关
    |    └── reg.c                   #   寄存器相关    
    ├── device                      #   设备相关
    ├── main.c                        
    ├── memory
    |    └── memory.c
    ├── misc
    |    └── logo.c                  #   "i386"的logo
    └── monitor
        ├── cpu-exec.c              #   指令执行的主循环
        ├── diff-test
        ├── debug                   #   简易调试器相关
        |    ├── expr.c              #   表达式求值的实现
        |    ├── ui.c                #   用户界面相关    
        |    └── watchpoint.c        #   监视点的实现
        └── monitor.c
```

# 2 include/

## 2.1 nemu.h

基础头文件。包含了 `commom.h`，`memory/memory.h`，`cpu/reg.h`

## 2.2 macro.h

定义了一些字符串连接宏 `concat` 等

## 2.3 common.h

定义了一些类型别名。

|  类型别名  |   原类型   |     描述     |
| :--------: | :--------: | :----------: |
| `rtlreg_t` | `uint32_t` |  RTL寄存器   |
| `vaddr_t`  | `uint32_t` |   虚拟地址   |
| `paddr_t`  | `uint32_t` |   物理地址   |
| `ioaddr_t` | `uint16_t` | I/O 端口地址 |

- `relreg_t` 多用于寄存器访问
- `vaddr_t` `paddr_t` 多用于内存访问
- `ioaddr_t` 多用于设备 I/O 端口访问

定义了一些控制编译方式的宏。

|     宏      |       描述       |
| :---------: | :--------------: |
|   `DEBUG`   |     启用调试     |
| `DIFF_TEST` |  启用 diff-test  |
|  `HAS_IOE`  | 启用输入输出扩展 |

- `DIFF_TEST` 可启用一个差异测试工具，参见 `tools/qemu-diff` 部分。
- `HAS_IOE` 启用输入输出设备，参见设备部分。

## 2.4 debug.h

定义了便于调试的宏。

|               宏               |                     描述                      |
| :----------------------------: | :-------------------------------------------: |
|    `Log_write(format, ...)`    |                  仅记录日志                   |
|    `printflog(format, ...)`    |              显示文本并记录日志               |
|       `Log(format, ...)`       | 对 `printflog` 的扩展，包含当前文件，行，函数 |
|      `Info(format, ...)`       |        对 `Log` 的扩展，日志级别：提示        |
|     `Warning(format, ...)`     |        对 `Log` 的扩展，日志级别：警告        |
|      `Error(format, ...)`      |        对 `Log` 的扩展，日志级别：错误        |
|      `panic(format, ...)`      |         强制退出，显示文本并记录日志          |
| `Assert(cond [, format, ...])` | 设置断言，失败时强制退出，显示文本并记录日志  |
|            `TODO()`            |      标识待完成项，执行时会触发 `panic`       |

## 2.5 cpu/

### 2.5.1 reg.h

定义了寄存器结构，和辅助寄存器的一些宏和函数。

- 外部数组 `regsl, regsw, regsb` 不同寄存器名。实现在 `src/cpu/reg.c`

```c
extern const char* regsl[];
extern const char* regsw[];
extern const char* regsb[];
```

#### 结构体 CPU_state

寄存器结构，包含了所有寄存器，均为无符号整数。

- 对于 8 个通用寄存器，内部以 `gpr` 数组为基础结构，提供 `eax` 等别名方便访问。寄存器按照 i386 指令中寄存器标号顺序排列。可使用 `_16,_8[0],_8[1]` 访问寄存器低位部分。
- `eip` 当前执行指令位置寄存器
- `eflags`标志位寄存器（使用匿名结构体，可直接访问`CF,OF,ZF,SF`）
  - `eflags` 初始化为 `0x2`
- `cs,ss,ds,es,fs,gs`程序段寄存器（仅为支持 diff-test）
  - `cs` 初始化为 `8`
- `idtr`：48 位寄存器，存放 IDT (Interrupt Descriptor Table, 中断描述符表)的首地址和长度
  - `limit` 16位，长度，单位：字节
  - `base` 32位，IDT 基地址

|         函数/宏         |             描述             |
| :---------------------: | :--------------------------: |
|     `reg_l(index)`      |  获取指定下标处寄存器32位值  |
|     `reg_w(index)`      | 获取指定下标处寄存器低16位值 |
|     `reg_b(index)`      | 获取指定下标处寄存器低8位值  |
| `reg_name(index,width)` |  根据下标和位宽获得寄存器名  |

> 寄存器存储在变量 `cpu` 中。

#### 枚举

定义了形如 `R_NAME` 的寄存器枚举，其顺序与寄存器结构中的顺序一致。

```c
enum { R_EAX, R_ECX, R_EDX, R_EBX, R_ESP, R_EBP, R_ESI, R_EDI };
enum { R_AX, R_CX, R_DX, R_BX, R_SP, R_BP, R_SI, R_DI };
enum { R_AL, R_CL, R_DL, R_BL, R_AH, R_CH, R_DH, R_BH };
```

### 2.5.2 decode.h

定义了用于指令译码的结构和函数。

#### 结构体 Operand

操作数。

| 成员    | 描述                 |
| :------ | :------------------- |
| `type`  | 类型（见下方枚举）   |
| `width` | 位宽                 |
| `val`   | 实际值               |
| `str`   | 原串（用于调试输出） |
| `reg`   | 寄存器下标           |
| `addr`  | 内存地址             |
| `imm`   | 立即数               |
| `simm`  | 带符号立即数         |

#### 结构体 DecodeInfo

单条命令译码结果。

| 成员                 | 描述                                 | 对应x86指令部分     |
| :------------------- | :----------------------------------- | :------------------ |
| `opcode`             | 指令码                               | opcode              |
| `seq_eip`            | 序列 EIP 位置                        |                     |
| `is_operand_size_16` | 标识操作数是否为 16 位               | operand-size prefix |
| `ext_opcode`         | 额外指令码                           | ModR/M 中 opcode    |
| `is_jmp`             | 标识是否为跳转语句                   |                     |
| `jmp_eip`            | 跳转目标（绝对地址），仅对于跳转语句 |                     |
| `src`                | 源操作数                             |                     |
| `src2`               | 第二个源操作数                       |                     |
| `dest`               | 目标操作数                           |                     |
| `assembly`           |                                      |                     |
| `asm_buf`            |                                      |                     |
| `p`                  |                                      |                     |

- `seq_eip` 随译码过程改变，最终停留在需要译码的下一个位置，可根据这一值实现 eip 更新。
- `is_operand_size_16` 多用于实现单命令存在 16 位，32 位两个版本的情况
- `ext_opcode` 用于实现 `sub /5` 这种根据第二个指令码 `/5` 区分不同指令的情况，在译码中使用 `make_group` 实现。
- `is_jmp` 多在运行时指定（如 `rtl_j` 函数），如果标记，则不会再根据 `seq_eip` 更新 eip

| 函数/宏                                                    | 描述                                                         |
| :--------------------------------------------------------- | :----------------------------------------------------------- |
| `id_src`                                                   | `(&decoding.src)`                                            |
| `id_src2`                                                  | `(&decoding.src2)`                                           |
| `id_dest`                                                  | `(&decoding.dest)`                                           |
| `operand_write(Operand *, rtlreg_t *)`                     | 根据第一个参数中记录的类型的不同进行相应的写操作，包括写寄存器和写内存 |
| `load_addr(vaddr_t *, ModR_M *, Operand *)`                |                                                              |
| `read_ModR_M(vaddr_t *, Operand *, bool, Operand *, bool)` |                                                              |

> 译码内容存储在变量 `decoding` 中。

#### 结构体 ModR_M

指令中的 ModR/M。

#### 结构体 SIB

指令中的 SIB。

#### 枚举

定义了操作数的类型 `OP_TYPE_REG`，`OP_TYPE_MEM`，`OP_TYPE_IMM`，分别为寄存器，内存，立即数。

#### 宏 make_DHelper 与函数族 decode_name

由宏 `make_DHelper` 定义了一族函数（参数相同），用于指令译码，并定义了这些函数的指针类型 `DHelper`。

- 设计目的：由于大量指令的操作数模式相似，将这一点提取出来，实现解耦。

```c
#define make_DHelper(name) void concat(decode_, name) (vaddr_t *eip)
typedef void (*DHelper) (vaddr_t *);
```

函数族中部分函数命名规则（**不全**）：

| 名称    | 描述                                 |
| :------ | :----------------------------------- |
| `I`     | 立即数                               |
| `SI`    | 有符号立即数                         |
| `E`     | 内存或寄存器（对应指令描述中的 r/m） |
| `G`     | 通用寄存器                           |
| `r`     | 单一寄存器                           |
| `a`     | 指定寄存器为 `eax,ax,al`             |
| `I2G`   | 立即数到通用寄存器                   |
| `I_E2G` | 立即数与内存或寄存器到通用寄存器     |
| `O`     | 未知                                 |

- `r` 一般用于寄存器信息存储在 `opcode` 中的情况
- 还有一些专用于特定指令的译码函数

> 建议结合 i386 手册附录 C 理解。

函数族中特殊函数：

- `J` 跳转指令解码。单操作数，存储到 `jmp_eip` 中。

### 2.5.3 exec.h

定义了一些用于调试的指令打印宏：

| 宏                    | 描述                     |
| :-------------------- | :----------------------- |
| `print_asm`           | 打印指令                 |
| `suffix_char`         | 根据宽度获取指令宽度后缀 |
| `print_asm_template1` | 单操作数指令             |
| `print_asm_template2` | 双操作数指令             |
| `print_asm_template3` | 三操作数指令             |

#### 函数 instr_fetch

```c
uint32_t instr_fetch(vaddr_t *eip, int len)
```

从 `eip` 开始，读取 `len` 个字节，返回值，并自动增加 `eip`。

- 设计目的：与机器的大端小端解耦。

#### 宏 make_EHelper 与 函数族 exec_name

用于定义一族函数（参数相同），用于指令执行，并定义了这些函数的指针类型 `EHelper`。

```c
#define make_EHelper(name) void concat(exec_, name) (vaddr_t *eip)
typedef void (*EHelper) (vaddr_t *);
```

### 2.5.4 relop.h

定义了形如 `RELOP_NAME` 的枚举，标识不同类型的关系运算。对应了 `setcc,jcc` 命令的相应编码。

```c
enum {
  //            +-- unsign
  //            |   +-- sign
  //            |   |   +-- equal
  //            |   |   |   +-- invert
  //            |   |   |   |
  RELOP_FALSE = 0 | 0 | 0 | 0,
  RELOP_TRUE  = 0 | 0 | 0 | 1,
  RELOP_EQ    = 0 | 0 | 2 | 0,
  RELOP_NE    = 0 | 0 | 2 | 1,

  RELOP_LT    = 0 | 4 | 0 | 0,
  RELOP_LE    = 0 | 4 | 2 | 0,
  RELOP_GT    = 0 | 4 | 2 | 1,
  RELOP_GE    = 0 | 4 | 0 | 1,

  RELOP_LTU   = 8 | 0 | 0 | 0,
  RELOP_LEU   = 8 | 0 | 2 | 0,
  RELOP_GTU   = 8 | 0 | 2 | 1,
  RELOP_GEU   = 8 | 0 | 0 | 1,
};
```

### 2.5.5 cc.h

定义了函数 `get_cc_name` 根据编码获取指定关系运算字符串。

定义了 RTL 基本指令 `rtl_setcc` 用于根据当前关系运算和 eflags 寄存器标志位设置 dest。

```c
void rtl_setcc(rtlreg_t*, uint8_t);
```

### 2.5.6 rtl.h

定义和实现了一些 RTL 指令，用于提供对指令执行的底层建模。可使用这些操作将复杂指令分解成更简单的操作。

NEMU 中的 RTL 寄存器：

- x86的八个通用寄存器(在 `include/cpu/reg.h` 中定义)
- `id_src`, `id_src2` 和 `id_dest` 中的访存地址 `addr` 和操作数内容 `val` (在 `include/cpu/decode.h` 中定义). 从概念上看, 它们分别与MAR和 MDR有异曲同工之妙
- 临时寄存器 `t0~t3` 和 `at` (在 `src/cpu/decode/decode.c` 中定义)

```c
extern rtlreg_t t0, t1, t2, t3, at;
```

- 宏`make_rtl_arith_logic`根据算术运算符名创建对应 RTL 基本指令和 RTL 指令，使用了`include/util/c_op.h`中的运算。
  - 32位寄存器-寄存器类型的算术/逻辑运算
  - 32位寄存器-立即数类型的算术/逻辑运算
- 定义函数 `decoding_set_jmp(bool is_jmp)` ：将 当前指令标记为跳转（标记 `decoing.is_jmp`）
- 定义函数 `interpret_relop` ：实现两个值的关系运算，返回结果（实现在 `src/cpu/exec/relop.c`

#### RTL 基本指令

特点：不需要使用临时寄存器, 可以看做是最基本的x86指令中的最基本的操作。 实现时添加了 `interpret_` 前缀，但在 `include/cpu/rtl-wrapper.h` 作用下，其它代码中使用到这些RTL基本指令时会自动添加 `interpret_` 前缀。

- 立即数读入 `rtl_li`
- 寄存器传输 `rtl_mv`
- 32位寄存器-寄存器类型的算术/逻辑运算, 包括 `rtl_(add|sub|and|or|xor|shl|shr|sar|i?mul_[lo|hi]|i?div_[q|r])` , 这些运算的定义用到 `include/util/c_op.h` 中的C语言运算
- 被除数为64位的除法运算 `rtl_i?div64_[q|r]`
- guest内存访问 `rtl_lm` 和 `rtl_sm`
- host内存访问 `rtl_host_lm` 和 `rtl_host_sm`
- 关系运算 `rtl_setrelop`, 具体可参考 `src/cpu/exec/relop.c`
- 跳转, 包括直接跳转 `rtl_j` , 间接跳转 `rtl_jr` 和条件跳转 `rtl_jrelop`
- 终止程序 `rtl_exit`

具体声明：

- 未标明则函数修饰符均为 `static inline`。

```c
// 立即数读入
void interpret_rtl_li(rtlreg_t* dest, uint32_t imm);

// 寄存器传输
void interpret_rtl_mv(rtlreg_t* dest, const rtlreg_t *src1);

// 32位寄存器-寄存器类型的算术/逻辑运算
void interpret_rtl_add (rtlreg_t* dest, const rtlreg_t* src1, const rtlreg_t* src2);

// 被除数为64位的除法运算
void interpret_rtl_div64_q(rtlreg_t* dest,
    const rtlreg_t* src1_hi, const rtlreg_t* src1_lo, const rtlreg_t* src2);

// guest内存访问
void interpret_rtl_lm(rtlreg_t *dest, const rtlreg_t* addr, int len);
void interpret_rtl_sm(const rtlreg_t* addr, const rtlreg_t* src1, int len);

// host内存访问
void interpret_rtl_host_lm(rtlreg_t* dest, const void *addr, int len);
void interpret_rtl_host_sm(void *addr, const rtlreg_t *src1, int len);

// 关系运算
void interpret_rtl_setrelop(uint32_t relop, rtlreg_t *dest, const rtlreg_t *src1, const rtlreg_t *src2);

// 跳转
void interpret_rtl_j(vaddr_t target);
void interpret_rtl_jr(rtlreg_t *target);
void interpret_rtl_jrelop(uint32_t relop, const rtlreg_t *src1, const rtlreg_t *src2, vaddr_t target);

// 终止程序
void interpret_rtl_exit(int state);
```

#### RTL 伪指令

通过RTL基本指令或者已经实现的RTL伪指令来实现。

- 32位寄存器-立即数类型的算术/逻辑运算, 包括 `rtl_(add|sub|and|or|xor|shl|shr|sar|i?mul_[lo|hi]|i?div_[q|r])_i`
- 通用寄存器访问 `rtl_lr` 和 `rtl_sr`
- EFLAGS标志位的读写 `rtl_set_(CF|OF|ZF|SF)` 和 `rtl_get_(CF|OF|ZF|SF)`
- 其它常用功能, 如按位取反 `rtl_not` ，符号扩展 `rtl_sext` 等

具体声明：

- 未标明则函数修饰符均为 `static inline`。
- 宏`make_rtl_setget_eflags`声明了需要实现的 EFLAGS标志位的读写 指令
  - `rtl_set_name`
  - `rtl_get_name`

```c
// 32位寄存器-立即数类型的算术/逻辑运算
void rtl_addi (rtlreg_t* dest, const rtlreg_t* src1, int imm)

// 通用寄存器访问
void rtl_lr(rtlreg_t* dest, int r, int width);
void rtl_sr(int r, const rtlreg_t* src1, int width);

// EFLAGS标志位的读写
void rtl_set_CF (const rtlreg_t* src);
void rtl_get_CF (rtlreg_t* dest);
void rtl_set_OF (const rtlreg_t* src);
void rtl_get_OF (rtlreg_t* dest);
void rtl_set_ZF (const rtlreg_t* src);
void rtl_get_ZF (rtlreg_t* dest);
void rtl_set_SF (const rtlreg_t* src);
void rtl_get_SF (rtlreg_t* dest);

// 根据运算结构更新 ZF, SF 标志位
void rtl_update_ZF(const rtlreg_t* result, int width);
void rtl_update_SF(const rtlreg_t* result, int width);
void rtl_update_ZFSF(const rtlreg_t* result, int width);

// 按位取反
void rtl_not(rtlreg_t *dest, const rtlreg_t* src1);

// 符号扩展
void rtl_sext(rtlreg_t* dest, const rtlreg_t* src1, int width);

// 压栈
void rtl_push(const rtlreg_t* src1);

// 弹栈
void rtl_pop(rtlreg_t* dest);

// 32位寄存器-立即数类型 关系运算
void rtl_setrelopi(uint32_t relop, rtlreg_t *dest, const rtlreg_t *src1, int imm);

// 取符号位（最高位）
void rtl_msb(rtlreg_t* dest, const rtlreg_t* src1, int width);
```

我们定义RTL基本指令的时候, 约定了RTL基本指令不需要使用RTL临时寄存器. 但某些RTL伪指令需要使用临时寄存器存放中间结果, 才能实现其完整功能. 这样可能会带来寄存器覆盖的问题, 例如如下RTL指令序列:

```
(1) rtl_mv(&t0, &t1);
(2) rtl_sext(&t1, &t2, 1);  // use t0 temporarily
(3) rtl_add(&t2, &t0, &t1);
```

如果实现(2)的时候恰好使用到了t0作为临时寄存器, 在(3)中使用的t0就不再是(1)的结果了, 从而产生非预期的结果.

为了尽可能避免上述问题, 我们有两条约定:

- 实现RTL伪指令的时候, 尽可能不使用 `dest` 之外的寄存器存放中间结果. 由于 `dest` 最后会被写入新值, 其旧值肯定要被覆盖, 自然也可以安全地作为RTL伪指令的临时寄存器.
- 实在需要使用临时寄存器的时候, 使用 `at` . `at` 全称是assembly temporary, 是MIPS ABI中定义的一个特殊寄存器: 编译器并不会使用它, 它可以在编写汇编代码的时候安全地作为可使用的临时寄存器. 在这里， 我们借鉴它的功能来作如下约定: 不要在RTL伪指令的内部实现之外使用 `at` . 这样， `at` 就可以安全地作为RTL伪指令的临时寄存器了.

### 2.5.7 rtl-wrapper.h

为 rtl.h 中定义的 RTL 基本指令的调用省去 `interpret_` 前缀。

## 2.6 memory/

### 2.6.1 memory.h

定义了访问内存的函数。使用数组 `pmem` 模拟内存。

```c
extern uint8_t pmem[];
```

| 函数/宏                                    | 描述                                    |
| :----------------------------------------- | :-------------------------------------- |
| `uint32_t vaddr_read(vaddr_t, int)`        | 从虚拟内存指定位置读取指定数目个字节    |
| `void vaddr_write(vaddr_t, uint32_t, int)` | 向虚拟内存指定位置写入指定数目个字节    |
| `uint32_t paddr_read(paddr_t, int)`        | 从物理内存指定位置读取指定数目个字节    |
| `void paddr_write(paddr_t, uint32_t, int)` | 向物理内存指定位置写入指定数目个字节    |
| `guest_to_host(p)`                         | `((void *)(pmem + (unsigned)p))`        |
| `host_to_guest(p)`                         | `((paddr_t)((void *)p - (void *)pmem))` |

### 2.6.2 mmu.h

(TODO)

#### 结构体 GateDesc

指示中断操作的门描述符(Gate Descriptor)类型。门描述符是一个8字节的结构体, 里面包含着不少细节的信息, 在NEMU中简化了门描述符的结构, 只保留存在位P和偏移量OFFSET。

```
   31                23                15                7                0
  +-----------------+-----------------+---+-------------------------------+
  |           OFFSET 31..16           | P |          Don't care           |4
  +-----------------------------------+---+-------------------------------+
  |             Don't care            |           OFFSET 15..0            |0
  +-----------------+-----------------+-----------------+-----------------+
```

在 `raise_intr`（定义在 `intr.c` 中）中使用。

| 成员           | 描述            |
| :------------- | :-------------- |
| `offset_15_0`  | Offset 低位部分 |
| `offset_31_16` | Offset 高位部分 |
| `present`      | 标识是否有效    |

- 为方便从内存中读取，使用 union 结构以及 `val0 val1` 域简化读写。
- 此结构体与 AM 中定义的 `GateDesc` （在 `arch/x86-nemu/include/x86.h` 中）结构相同。

## 2.7 device/

### 2.7.1 mmio.h

对内存映射 I/O 编址方式的支持。注意，内存映射 I/O 的读写并不是面向 CPU 的。

> 端口映射I/O把端口号作为I/O指令的一部分, 这种方法很简单, 但同时也是它最大的缺点. 指令集为了兼容已经开发的程序, 是只能添加但不能修改的. 这意味着, 端口映射I/O所能访问的I/O地址空间的大小, 在设计I/O指令的那一刻就已经决定下来了. 所谓I/O地址空间, 其实就是所有能访问的设备的地址的集合. 随着设备越来越多, 功能也越来越复杂, I/O地址空间有限的端口映射I/O已经逐渐不能满足需求了. 有的设备需要让CPU访问一段较大的连续存储空间, 如VGA的显存, 24色加上Alpha通道的1024x768分辨率的显存就需要3MB的编址范围. 于是内存映射I/O(memory-mapped I/O)应运而生. 内存映射I/O这种编址方式非常巧妙, 它是通过不同的物理内存地址给设备编址的. 这种编址方式将一部分物理内存"重定向"到I/O地址空间中, CPU尝试访问这部分物理内存的时候, 实际上最终是访问了相应的I/O设备, CPU却浑然不知. 这样以后, CPU就可以通过普通的访存指令来访问设备. 这也是内存映射I/O得天独厚的好处: 物理内存的地址空间和CPU的位宽都会不断增长, 内存映射I/O从来不需要担心I/O地址空间耗尽的问题. 从原理上来说, 内存映射I/O唯一的缺点就是, CPU无法通过正常渠道直接访问那些被映射到I/O地址空间的物理内存了. 但随着计算机的发展, 内存映射I/O的唯一缺点已经越来越不明显了: 现代计算机都已经是64位计算机, 物理地址线都有48根, 这意味着物理地址空间有256TB这么大, 从里面划出3MB的地址空间给显存, 根本就是不痛不痒. 正因为如此, 内存映射I/O成为了现代计算机主流的I/O编址方式: RISC架构只提供内存映射I/O的编址方式, 而PCI-e, 网卡, x86的APIC等主流设备, 都支持通过内存映射I/O来访问.

> 在 NEMU 中， video memory是唯一使用内存映射 I/O 方式访问的 I/O 空间。

定义了类型 `mmio_callback_t` ，设备定义的回调函数，用以更新设备状态。

```c
typedef void(*mmio_callback_t)(paddr_t, int, bool);
```

| 函数                                                | 描述                                                         |
| :-------------------------------------------------- | :----------------------------------------------------------- |
| `void* add_mmio_map(paddr_t, int, mmio_callback_t)` | 注册一个内存映射 I/O 映射关系，返回该映射关系的 I/O 空间首地址 |
| `int is_mmio(paddr_t)`                              | 判断一个物理地址是否被映射到 I/O 空间，如果是，返回映射号, 否则返回 -1 |
| `uint32_t mmio_read(paddr_t, int, int)`             | 根据端口号和地址读取                                         |
| `void mmio_write(paddr_t, int, uint32_t, int)`      | 根据端口号和地址写入                                         |

### 2.7.2 port-io.h

对端口映射 I/O 编址方式的支持。端口映射I/O(port-mapped I/O)， CPU使用专门的I/O指令对设备进行访问， 并把设备的地址称作端口号。 有了端口号以后， 在I/O指令中给出端口号， 就知道要访问哪一个设备寄存器了。

定义了类型 `pio_callback_t` ，设备定义的回调函数，用以更新设备状态。

```c
typedef void(*pio_callback_t)(ioaddr_t, int, bool);
```

| 函数                                               | 描述                                                         |
| :------------------------------------------------- | :----------------------------------------------------------- |
| `void* add_pio_map(paddr_t, int, mmio_callback_t)` | 注册一个端口映射 I/O 映射关系，返回该映射关系的 I/O 空间首地址 |
| `uint32_t pio_read_[l,w,b](ioaddr_t)`              | 面向 CPU 的端口 I/O 读接口                                   |
| `void pio_write_[l,w,b](ioaddr_t, uint32_t)`       | 面向 CPU 的端口 I/O 写接口                                   |

## 2.8 monitor/

监视器部分（也包含 NEMU 执行主循环）。

### 2.8.1 expr.h

定义了计算表达式的值的函数 `expr`。

```c
uint32_t expr(char *, bool *);
```

### 2.8.2 monitor.h

定义了 NEMU 状态 变量 `nemu_state`，和枚举值 `NEMU_STOP, NEMU_RUNNING, NEMU_END, NEMU_ABORT`。 定义了 应用程序入口点 `ENTRY_START` ：

```c
#define ENTRY_START 0x100000
```

### 2.8.3 watchpoint.h

#### 结构体 WP

监视点结构。采用链表结构存储。

| 成员      | 描述               |
| :-------- | :----------------- |
| `NO`      | 序号               |
| `next`    | 下一监视点指针     |
| `expr`    | 监视的表达式       |
| `lastVal` | 表达式最近一次的值 |

## 2.9 util/

### 2.9.1 c_op.h

定义了一些形如 `c_opname_type` 的宏，用于表示基础 C 运算。在 RTL基本指令中的寄存器运算指令中使用。

# 3 src/

## main.c

NEMU 主程序。

调用 `init_monitor` （实现在 `/src/monitor/monitor.c`）初始化监视器，并获取当前是否为批处理模式。 调用 `ui_mainloop` （实现在 `/src/monitor/debug/ui.c`）进行指令执行模拟。

## cpu/

### reg.c

实现了 `include/cpu/reg.h` 中的 `regsl,regsw,regsb`，同时实现寄存器实际定义：变量 `cpu`。

- 函数 `reg_test`：测试寄存器结构定义（`CPU_state`）是否正确。

### intr.c

函数 `void raise_intr(uint8_t NO, vaddr_t ret_addr)` 为 `int` 指令（在 `system.c` 中实现）的内部实现。 实现了触发中断或异常后的硬件处理：

1. 依次将EFLAGS, CS(代码段寄存器), EIP寄存器（返回地址）的值压入堆栈
2. 根据中断码，从IDTR中读出IDT的首地址
3. 根据异常号在IDT中进行索引, 找到一个门描述符
4. 将门描述符中的offset域组合成目标地址
5. 跳转到目标地址

### decode/

指令译码相关。

#### decode.c

实现了 `include/cpu/decode.h` 中的译码函数族，函数 `operand_write` 以及译码信息变量 `decoding`。 实现了 `include/cpu/rtl.h` 中的临时寄存器 `t0,t1,t2,t3,at` 和函数 `decoding_set_jmp`。

##### 宏 make_DopHelper 与函数族 decode_op_name

```c
#define make_DopHelper(name) void concat(decode_op_, name) (vaddr_t *eip, Operand *op, bool load_val)
```

译码函数会进一步分解成各种不同操作数的译码的组合，以实现操作数译码的解耦. 操作数译码函数统一通过宏 `make_DopHelper` 来定义 （`decode_op_rm` 除外）。 操作数译码函数会把操作数的信息记录在结构体 `op` 中, 如果操作数在指令中， 就会通过 `instr_fetch()` 将它们从 `eip` 所指向的内存位置取出. 为了使操作数译码函数更易于复用， 函数中的 `load_val` 参数会控制 是否需要将该操作数读出到全局译码信息 `decoding` 供后续使用. 例如如果一个内存操作数是源操作数, 就需要将这个操作数从内存中读出来供后续执行阶段来使用； 如果它仅仅是一个目的操作数， 就不需要从内存读出它的值了，因为执行这条指令并不需要这个值， 而是将新数据写入相应的内存位置.

`decode_op_name` 函数族命名规则可参见 `decode_name` 函数族命名规则。

- `decode_op_a` 是一个特例，其用于将操作数标记为寄存器 `ax` 或 `eax`

#### modrm.c

实现了 `include/cpu/decode.h` 中的函数 `load_addr` 和 `read_ModR_M`。

### exec/

指令执行相关。

#### cc.c

实现了 `include/cpu/cc.h` 中的函数 `rtl_setcc`。根据指定关系运算以及条件标志位设置 dest。

#### relop.c

实现了 `include/cpu/relop.h` 中的函数 `interpret_relop`，使用 C语言关系运算符实现关系运算。

#### all-instr.h

定义了已经实现的指令执行函数（在 `exec.c` 中使用）。

#### arith.c

算术运算指令执行函数实现。

| 指令    | 描述          |
| :------ | :------------ |
| `add`   |               |
| `sub`   |               |
| `cmp`   |               |
| `inc`   |               |
| `dec`   |               |
| `neg`   |               |
| `adc`   |               |
| `sbb`   |               |
| `mul`   |               |
| `imul1` | imul 单操作数 |
| `imul2` | imul 双操作数 |
| `imul3` | imul 三操作数 |
| `div`   |               |
| `idiv`  |               |

#### control.c

控制指令执行函数实现。

| 指令      | 描述     |
| :-------- | :------- |
| `jmp`     | 直接跳转 |
| `jmp_rm`  | 间接跳转 |
| `jcc`     | 条件跳转 |
| `call`    | 直接调用 |
| `call_rm` | 间接调用 |
| `ret`     |          |

#### data-mov.c

数据移动指令执行函数实现。

| 指令    | 描述 |
| :------ | :--- |
| `mov`   |      |
| `movsx` |      |
| `movzx` |      |
| `lea`   |      |
| `push`  |      |
| `pop`   |      |
| `pusha` |      |
| `popa`  |      |
| `leave` |      |
| `cltd`  |      |
| `cwtl`  |      |

#### logic.c

逻辑运算指令执行函数实现。

| 指令    | 描述 |
| :------ | :--- |
| `test`  |      |
| `and`   |      |
| `xor`   |      |
| `or`    |      |
| `sar`   |      |
| `shl`   |      |
| `shr`   |      |
| `setcc` |      |
| `not`   |      |
| `rol`   |      |
| `ror`   |      |

#### special.c

特殊指令执行函数实现。

实现了 `include/cpu/rtl.h` 中的函数 `interpret_rtl_exit`。

| 指令        | 描述     |
| :---------- | :------- |
| `nop`       |          |
| `inv`       | 非法指令 |
| `nemu_trap` | 结束执行 |

#### prefix.c

定义了执行函数 `exec_real`。 定义并实现了执行函数 `exec_operand_size`。

- `exec_operand_size` 以 16 位操作数执行指令（标记 `decoding.is_operand_size_16`）

#### system.c

系统相关指令实现。

| 指令       | 描述                   |
| :--------- | :--------------------- |
| `lidt`     | 设置 IDTR 寄存器       |
| `mov_r2cr` |                        |
| `mov_cr2r` |                        |
| `int`      | 根据中断码进行中断跳转 |
| `iret`     | 从中断跳转返回         |
| `in`       | 读取端口映射 I/O       |
| `out`      | 写入端口映射 I/O       |

- x86 提供了 in 和 out 指令用于访问设备，其中 in 指令用于将设备寄存器中的数据传输到 CPU 寄存器中，out 指令用于将 CPU 寄存器中的数据传送到设备寄存器中

#### exec.c

指令执行过程核心实现。

##### 结构体 opcode_entry

译码查找表中元素。

| 成员              | 描述         |
| :---------------- | :----------- |
| `DHelper decode`  | 译码函数指针 |
| `EHelper execute` | 执行函数指针 |
| `width`           | 指令宽度     |

##### 数组 opcode_table

译码表。按指令第一个字节索引存放。分两段：单字节指令码和双字节指令码。

```c
opcode_entry opcode_table [512] = {
  /* 0x00 */	EMPTY, EMPTY, EMPTY, EMPTY,
  /* 0x04 */	EMPTY, EMPTY, EMPTY, EMPTY,
  /* 0x08 */	EMPTY, EMPTY, EMPTY, EMPTY,
  ...
};
```

| 宏                                                           | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `IDEXW(id, ex, w)`                                           | 根据译码函数名，执行函数名，宽度生成 `opcode_entry`          |
| `IDEX(id, ex)`                                               | 根据译码函数名，执行函数名，以宽度 0 生成 `opcode_entry`     |
| `EXW(ex, w)`                                                 | 根据执行函数名，宽度，生成无译码函数的 `opcode_entry`        |
| `EX(ex)`                                                     | 根据执行函数名，生成宽度为 0 且无译码函数的 `opcode_entry`   |
| `EMPTY`                                                      | 未实现的命令，使用 `exec_inv`（定义在 `special.c` 中） 构造 `opcode_entry` |
| `make_group(name, item0, item1, item2, item3, item4, item5, item6, item7)` | 用于实现 `sub /5` 这种根据第二个指令码 `/5` 区分不同指令的情况。会自动生成一个 `exec_name` 的统一执行函数，并根据 `decoding.ext_opcode` 分配到指定执行函数。 |

```c
#define IDEXW(id, ex, w)   {concat(decode_, id), concat(exec_, ex), w}
#define IDEX(id, ex)       IDEXW(id, ex, 0)
#define EXW(ex, w)         {NULL, concat(exec_, ex), w}
#define EX(ex)             EXW(ex, 0)
#define EMPTY              EX(inv)

#define make_group(name, item0, item1, item2, item3, item4, item5, item6, item7) \
  static opcode_entry concat(opcode_table_, name) [8] = { \
    /* 0x00 */	item0, item1, item2, item3, \
    /* 0x04 */	item4, item5, item6, item7  \
  }; \
static make_EHelper(name) { \
  idex(eip, &concat(opcode_table_, name)[decoding.ext_opcode]); \
}
```

使用 `make_group` 宏定义了一些组 `gp1` - `gp7`。对应于 80386 手册附录中组的划分。

##### 函数 exec_wrapper

```c
void exec_wrapper(bool print_flag);
```

执行下一条指令。

- 首先将当前的 `%eip` 保存到全局译码信息 `decoding` 的成员 `seq_eip` 中
- 然后将其地址被作为参数送进`exec_real()`函数中
  - `seq` 代表顺序的意思, 当代码从 `exec_real()` 返回时，`decoding.seq_eip` 将会指向下一条指令的地址.
- 调用 `update_eip` 更新 `%eip`
- 调试模式下
  - 记录日志（指令内容以及相关信息）
  - 若 `print_flag` 为真，则显示 `decoding.asm_buf`

##### 函数 exec_real

- 首先通过 `instr_fetch()` 函数(在 `include/cpu/exec.h` 中定义)进行取指， 得到指令的第一个字节, 将其解释成 `opcode` 并记录在全局译码信息 `decoding` 中.
- 根据 `opcode` 查阅译码查找表，得到操作数的宽度信息，并通过调用 `set_width()` 函数将其记录在全局译码信息 `decoding` 中
- 调用 `idex()` 对指令进行进一步的译码和执行

##### 函数 set_width

```c
static inline void set_width(int width);
```

根据指令定义宽度（`opcode_entry.width`）指定所有操作数宽度（`decoding.src.width`）。

- 如果定义宽度为 0，则采用译码结果（`decoding.is_operand_size_16`）

##### 函数 idex

```c
/* Instruction Decode and EXecute */
static inline void idex(vaddr_t *eip, opcode_entry *e);
```

调用译码查找表中的相应的译码函数（若存在）进行操作数的译码，译码过程结束之后, 会调用译码查找表中的相应的执行函数来进行真正的执行操作。

##### 函数 update_eip

根据当前指令是否为跳转指令，更新 `%eip`。

```c
static inline void update_eip(void) {
    if (decoding.is_jmp) { decoding.is_jmp = 0; }
    else { cpu.eip = decoding.seq_eip; }
}
```

## memory/

### memory.c

定义了宏 `PMEM_SIZE` 指定物理内存大小。 实现了 `include/memory/memory.h` 中的函数 `paddr_read`，`paddr_write`，`vaddr_read`，`vaddr_write`。

- `vaddr_read, vaddr_write` 的实现调用了 `paddr_read` 和 `paddr_write`。
- 为支持内存映射 I/O，`paddr_read, paddr_write` 的实现加入了对内存映射 I/O 的判断。

## device/

### io/

#### mmio.c

定义了宏 `MMIO_SPACE_MAX` 指定内存映射空间大小。 定义了结构体 `MMIO_t` 保存 MMIO 信息。

实现了 `include/device/mmio.h` 中的函数。

- 在 `mmio_read` 和 `mmio_write` 中，调用了回调函数。

#### port-io.c

定义了宏 `PORT_IO_SPACE_MAX` 指定内存映射空间大小。 定义了结构体 `PIO_t` 保存 MMIO 信息。

实现了 `include/device/port-io.h` 中的函数。

- 在 `pio_read_common` 和 `pio_write_common` 中，调用了回调函数。
- 基于 `pio_read_common` 和 `pio_write_common` 实现了不同的端口读写函数

### device.c

提供初始化和控制设备的一些函数。含有和SDL库相关的代码，NEMU使用SDL库来实现设备的模拟。

| 宏         | 描述         |
| :--------- | :----------- |
| `TIMER_HZ` | 时钟频率     |
| `VGA_HZ`   | VGA 刷新频率 |

#### 函数 init_device

用于初始化设备：串口， 时钟， 键盘， VGA四种设备。 其中在初始化 VGA 时还会进行一些和SDL相关的初始化工作， 包括创建窗口， 设置显示模式等. 最后还会注册一个100Hz的定时器， 每隔0.01秒就会调用一次 `device_update()` 函数。

#### 函数 device_update

主要进行一些设备的模拟操作, 包括以50Hz的频率刷新屏幕, 以及检测是否有按键按下/释放.

需要说明的是， 代码中注册的定时器是虚拟定时器， 它只会在 NEMU 处于用户态的时候进行计时： 如果 NEMU 在 `ui_mainloop()` 中等待用户输入， 定时器将不会计时; 如果 NEMU 进行大量的输出， 定时器的计时将会变得缓慢. 因此除非你在进行调试， 否则尽量避免大量输出的情况， 从而影响定时器的工作。

### serial.c

串口设备。 模拟了串口的功能。 其大部分功能也被简化，只保留了数据寄存器和状态寄存器。串口初始化时会分别注册 `0x3F8` 和 `0x3FC` 处长度为1个字节的端口，分别作为数据寄存器和状态寄存器。由于NEMU串行模拟计算机系统的工作，串口的状态寄存器可以一直处于空闲状态; 每当CPU往数据寄存器中写入数据时，串口会将数据传送到主机的标准输出。

| 函数/宏             | 描述          |
| :------------------ | :------------ |
| `init_serial()`     | 初始化设备    |
| `SERIAL_PORT=0x3F8` | 端口 I/O 地址 |

### timer.c

时钟设备。 模拟了i8253计时器的功能. 计时器的大部分功能都被简化, 只保留了"发起时钟中断"的功能. 同时添加了一个自定义的RTC(Real Time Clock), 初始化时将会注册0x48处的端口作为RTC寄存器, CPU可以通过I/O指令访问这一寄存器, 获得当前时间(单位是ms).

| 函数/宏         | 描述          |
| :-------------- | :------------ |
| `init_timer()`  | 初始化设备    |
| `RTC_PORT=0x48` | 端口 I/O 地址 |

### keyboard.c

键盘设备。 模拟了i8042通用设备接口芯片的功能. 其大部分功能也被简化, 只保留了键盘接口. i8042初始化时会注册 `0x60` 处的端口（长度为 4）作为数据寄存器. 每当用户敲下/释放按键时, 将会把相应的键盘码放入数据寄存器, CPU可以通过端口I/O访问数据寄存器, 获得键盘码; 当无按键可获取时, 将会返回 `_KEY_NONE` . 在AM中, 我们约定通码的值为 `断码 | KEYDOWN_MASK`.

| 函数/宏                | 描述          |
| :--------------------- | :------------ |
| `init_i8042()`         | 初始化设备    |
| `I8042_DATA_PORT=0x60` | 端口 I/O 地址 |
| `KEYDOWN_MASK=0x8000`  | 通码 MASK     |
| `KEY_QUEUE_LEN`        | 键队列长度    |

### vga.c

VGA 设备。 模拟了VGA的功能. VGA初始化时注册了从 `0x40000` 开始的一段用于映射到video memory的物理内存. 在NEMU中, video memory是唯一使用内存映射I/O方式访问的I/O空间. 代码只模拟了400x300x32的图形模式, 一个像素占32个bit的存储空间, R(red), G(green), B(blue), A(alpha)各占8 bit, 其中VGA不使用alpha的信息。VGA 设备同时注册了位于 `0x100` 的长度为 4 的端口存储屏幕大小信息。

| 函数/宏             | 描述              |
| :------------------ | :---------------- |
| `init_vga()`        | 初始化设备        |
| `SCREEN_PORT=0x100` | 端口 I/O 地址     |
| `VMEM=0x40000`      | 内存映射 I/O 地址 |
| `SCREEN_H`          | 屏幕高度          |
| `SCREEN_W`          | 屏幕宽度          |

## monitor/

监视器部分实现（也包含 NEMU 执行主循环）。

### monitor.c

#### 函数 init_monitor

初始化监视器并启动（用于 `main.c/main` 中）。

- 解析并处理命令行参数
- 初始化日志文件
- 寄存器测试（调用 `reg_test()`，实现在 `src/cpu/reg.c`）
- 加载程序镜像（根据命令行参数，如果为空，则调用 `load_default_img()` 加载默认镜像）
- 启动环境（调用 `restart()`，初始化应用程序入口点，寄存器值）
- 编译正则表达式（调用 `init_regex()`，实现在 `src/monitor/debug/expr.c`）
- 初始化监视点池（调用 `init_wp_pool()`，实现在 `src/monitor/debug/watchpoint.c`）
- 初始化设备（调用 `init_device()`，实现在 `src/device/device.c`）
- 初始化差异测试（调用 `init_difftest()`，实现在 `src/monitor/diff-test.c`）
- 显示欢迎界面
- 返回是否为批处理模式（根据命令行参数）

注：

- 命令行参数
  - `[img_file]` 指定应用程序镜像文件
  - `-b` 批处理模式
  - `-l log_file` 指定日志文件
  - `-d` 指定 Diff-Test 镜像文件

### cpu-exec.c

#### 函数 cpu_exec

```c
void cpu_exec(uint64_t n);
```

模拟 CPU 工作。

- 判断 NEMU 状态（查看 `nemu_state`，定义在 `include/monitor/monitor.h`）
- 若指令数 `n` 小于 `MAX_INSTR_TO_PRINT` （默认为 10），则打印每条指令。
- 开始执行指令
  - 调用 `exec_wrapper` 执行下一条指令（传入是否打印指令标记）
  - 检查监视点状态是否有更新
  - 更新设备信息
  - 判断 NEMU 状态（查看 `nemu_state`），决定是否退出
- 执行完 `n` 条指令后，将 NEMU 状态置为结束（`NEMU_END`）

注：

- 执行某条命令后
  - 若 NEMU 状态为结束（`NEMU_END`），则检查程序返回值（`cpu.eax`）是否为 0（是否正常退出）。并输出 `HIT GOOD TRAP`（正常退出） 或 `HIT BAD TRAP`（非正常退出）。

### debug/

#### watchpoint.c

定义了监视点内存池及其相关的函数。

- `wp_pool` 监视点池
- `head` 使用中的监视点链表头指针
- `free_` 监视点池中未使用的监视点链表头指针

| 函数              | 描述                                                       |
| :---------------- | :--------------------------------------------------------- |
| `init_wp_pool()`  | 初始化监视点内存池                                         |
| `clearWP(wp)`     | 清空某监视点的下一项指针                                   |
| `WP *getHeadWP()` | 获取 `head`                                                |
| `WP *createWP()`  | 申请使用一个新监视点（内部调用 `new_wp()` 并更新链表信息） |
| `removeWP(no)`    | 删除指定编号的监视点                                       |
| `WP *new_wp()`    | （私有）从内存池中获取下一个能使用的监视点，并作一定预处理 |
| `free_wp(wp)`     | （私有）释放一个监视点                                     |

#### expr.c

实现了 `expr.h/expr` 函数，实现表达式解析和求值。

##### 常量

- `PRI_NEG` 取负运算优先级
- `PRI_POINT` 解引用运算优先级
- 形如 `TK_TYPE` 的 Token 类型枚举

##### 数组 rules

规定了使用正则表达式解析 Token 的规则。

| 成员         | 描述                                                     |
| :----------- | :------------------------------------------------------- |
| `regex`      | 正则表达式字符串                                         |
| `token_type` | 对应 Token 类型，可用 `TK_TYPE` 枚举或字符（如 `+`）表示 |
| `opPri`      | 运算符 Token 的优先级                                    |

##### 数组 re

根据 `rules` 编译后的正则表达式。

##### 函数 `init_regex`

根据 `rules` 编译到 `re`

##### 结构体 Token

识别后的 Token.

| 成员       | 描述                                                |
| :--------- | :-------------------------------------------------- |
| `type`     | Token 类型，可用 `TK_TYPE` 枚举或字符（如 `+`）表示 |
| `isOp`     | 标记此 Token 是否是运算符                           |
| `isValue`  | 标记此 Token 是否是值                               |
| `str`      | Token 的原始字符串                                  |
| `data`     | 值类型的实际数据                                    |
| `priority` | 运算符的优先级                                      |

- `isOp` 和 `isValue` 多用于区分特殊单目运算符，如解引用和取负
- `data` 多存储经过预处理的数据，如转换后的整数

> 解析后的 Token 列表存储在数组 `tokens` 中。 `nr_token` 指示 `token` 有效长度。

##### 函数 make_token

```c
static bool make_token(char *e)
```

根据字符串解析 Token 列表，返回是否解析成功。

实现思路：使用 `re` 依次尝试每一种匹配，直到遇到第一个成功匹配，根据其规则的 `token_type` 生成 Token，存入 `token`。

- 函数 `toInteger` ：以指定进制完成字符串到数的转换，用于 十进制，二进制，八进制，十六进制 数的解析。

```c
static uint32_t toInteger(char *s, uint32_t base)
```

##### 函数 evalWithToken

```c
uint32_t evalWithToken(int l, int r, bool * success)
```

求解 `tokens[l..r]` 中的表达式的值。

实现思路：单 Token 特殊处理，然后处理外围括号情况，然后确定最后计算的运算符，分割成 `left` 和 `right` 两部分，然后递归解决，最后合并，

- 函数 `checkExtraP` 判断 `tokens[l..r]` 是否外围为括号且括号匹配正常。
- 函数 `getReg` 根据寄存器名获取寄存器值，使用了 `regMap`
- 数组 `regMap` 标识寄存器名与对应的偏移量（`cpu.gpr`）

##### 函数 expr

对 `expr.h/expr` 的实现，调用了 `make_token` 和 `evalWithToken`。

#### ui.c

监视器 CUI 部分。

##### 函数族 fc2color

控制台字体颜色控制。

| 函数        | 描述               |
| :---------- | :----------------- |
| `fc2red`    | 前景色设为红色     |
| `fc2green`  | 前景色设为绿色     |
| `fc2yellow` | 前景色设为黄色     |
| `fc2blue`   | 前景色设为蓝色     |
| `fc2purple` | 前景色设为紫色     |
| `csClear`   | 清除所有控制台设置 |

##### 函数族 cmd_item

不同命令的实现。

| 函数            | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| `cmd_help`      | 获取帮助                                                     |
| `cmd_q`         | 退出                                                         |
| `cmd_c`         | 继续执行（调用 `cpu_exec(-1)`，实现在 `src/monitor/cpu-exec.c`） |
| `cmd_si`        | 执行单步指令                                                 |
| `cmd_info name` | 查看信息                                                     |
| `cmd_x N expr`  | 显示地址从 `expr` 的值开始的 `N` 个字节值                    |
| `cmd_p expr`    | 计算表达式的值                                               |
| `cmd_w expr`    | 新建监视点，监视表达式为 `expr`                              |
| `cmd_d no`      | 删除指定编号的监视点                                         |

- ```
  cmd_info
  ```

  - `r` 打印所有寄存器信息
  - `w` 打印所有监视点信息

##### 数组 cmd_table

| 成员          | 描述                     |
| :------------ | :----------------------- |
| `name`        | 命令名（用于识别命令）   |
| `description` | 命令描述（用于帮助列表） |
| `handler`     | 命令实现函数指针         |

##### 函数 ui_mainloop

```c
void ui_mainloop(int is_batch_mode);
```

NEMU 以及其 CUI 主循环，不断读取命令，并执行。

- 如果是批处理模式（`is_batch_mode` 为真），则直接执行应用程序，不监听用户命令。

### diff-test/

差异测试实现。

> 如果有一种方法能够表达指令的正确行为, 我们就可以基于这种方法来进行类似assert()的检查了。那么, 究竟什么地方表达了指令的正确行为呢? 最直接的, 当然就是i386手册了, 但是我们恰恰就是根据i386手册中的指令行为来在NEMU中实现指令的, 同一套方法不能既用于实现也用于检查. 如果有一个i386手册的参考实现就好了. 嘿! 我们用的真机不就是根据i386手册实现出来的吗? 我们让在NEMU中执行的每条指令也在真机中执行一次, 然后对比NEMU和真机的状态, 如果NEMU和真机的状态不一致, 我们就捕捉到error了! 这实际上是一种非常奏效的测试方法, 在软件测试领域称为differential testing(后续简称DiffTest). 我们刚才提到了"状态", 那"状态"具体指的是什么呢? 我们在PA1中已经认识到, 计算机就是一个数字电路. 那么, “计算机的状态"就恰恰是那些时序逻辑部件的状态, 也就是寄存器和内存的值. 其实仔细思考一下, 计算机执行指令, 就是修改这些时序逻辑部件的状态的过程. 要检查指令的实现是否正确, 只要检查这些时序逻辑部件中的值是否一致就可以了! DiffTest可以非常及时地捕捉到error, 第一次发现NEMU的寄存器或内存的值与真机不一样的时候, 就是因为当时执行的指令实现有误导致的. 这时候其实离error非常接近, 防止了error进一步传播的同时, 要回溯找到fault也容易得多. 多么美妙的功能啊! 背后还蕴含着计算机本质的深刻原理! 但很遗憾, 不要忘记了, 真机上是运行了操作系统GNU/Linux的, 而NEMU中的测试程序是运行在x86-nemu上的, 我们无法在native中运行编译到x86-nemu的AM程序. 所以, 我们需要的不仅是一个i386手册的正确实现, 而且需要在上面能正确运行x86-nemu的AM程序. 事实上, QEMU就是一个不错的参考实现. 它是一个虚拟出来的完整的x86计算机系统, 而NEMU的目标只是虚拟出x86的一个子集, 能在NEMU上运行的程序, 自然也能在QEMU上运行. 因此, 为了通过DiffTest的方法测试NEMU实现的正确性, 我们让NEMU和QEMU逐条指令地执行同一个客户程序. 双方每执行完一条指令, 就检查各自的寄存器和内存的状态, 如果发现状态不一致, 就马上报告错误, 停止客户程序的执行.

#### diff-test.h

定义了宏 `DIFFTEST_REG_SIZE` 规定访问的寄存器大小。

```c
#define DIFFTEST_REG_SIZE (sizeof(uint32_t) * 9) // GRPs + EIP
```

#### ref.c

在 DUT(Design Under Test, 测试对象)和 REF(Reference, 参考实现) 之间定义了一组 API。

```c
// 从DUT host memory的 src 处拷贝 n 字节到REF guest memory的 dest 处
void difftest_memcpy_from_dut(paddr_t dest, void *src, size_t n);
// 获取REF的寄存器状态到 r 
void difftest_getregs(void *r);
// 设置REF的寄存器状态为 r 
void difftest_setregs(const void *r);
// 让REF执行 n 条指令
void difftest_exec(uint64_t n);
// 初始化REF的DiffTest功能
void difftest_init();
```

- 其中寄存器状态 `r` 要求寄存器的值按照某种顺序排列，若未按要求顺序排列， `difftest_getregs()` 和 `difftest_setregs()` 的行为是未定义的. REF 需要实现这些 API，DUT会使用这些 API 来进行 DiffTest 。

#### diff-test.c

定义了变量 `is_skip_ref`，`is_skip_dut` 用于标记忽视一些指令处的比对。（可结合 `difftest_step` 实现） 定义了函数 `difftest_skip_ref`，`difftest_skip_dut` 标记上述变量。

##### 函数 init_difftest

初始化 Diff-Test。

- 打开动态库文件 `ref_so_file`
- 从动态库中分别读取上述 API 的符号
- 对 REF 的 DIffTest功能进行初始化，此时会启动 REF，代码还会对 REF 的状态进行一些初始化工作，REF 运行在后台，因此将看不到 REF 的任何输出
- 将 DUT 的 guest memory 拷贝到 REF 中
- 将 DUT 的寄存器状态拷贝到 REF 中

##### 函数 difftest_step

用于逐条指令执行后的状态对比。它会在 `exec_wrapper()` 的最后被调用。在这里读取 REF 的寄存器并与 NEMU 寄存器状态比对。

## misc/

### logo.c

定义了字符数组 `logo` 存储 i386 Manual Logo。用于 `inv` 指令（位于 `special.c` 中）。

# 4 tools/

## gen-expr.c

生成 C 表达式，用于测试表达式求值功能。

## qemu-diff

QEMU 实现，用于 Diff-Test。编译成动态库 `qemu-so`，传入 nemu 的 `-d` 参数中。
