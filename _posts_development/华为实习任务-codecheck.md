---
title: 华为实习任务-codecheck
tags:
  - 重构
categories:
  - 软件开发
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-18 13:40:01
---

# 1 引入

在华为实习的三个月(2021.9-2021.11)中有两个月(9&11月)都在做codecheck的工作，重复性操作，"毫无"技术含量可言。

令我惊讶的是：2022年4月份在实习答辩时，我说做了codecheck的工作。一位老师说（具体咋说的忘了，大致意思是：你是实习生，才三个月，华为不会让你改代码的（指核心代码？），你不要吹过头了），当时我愣住了，哈哈哈哈🥦。

答辩结束后仔细想了下，应该这样说：“实习期间，部门正好有codecheck的任务，我在其中做了一部分工作（包括xxx），改完代码后有检视人员保障代码质量。”

值得肯定的是：面对老师的奇怪语气+奇怪问法，没有进行反驳，毕竟到时间了（答辩每人10min），而且老师也没有听的意思。

总而言之，答辩时表达要得体（本次实习答辩自己多多少少存在这方面的问题，可能真是吹的口吻，只是自己没察觉）；认真听取老师的意见，一些情况下没必要进行反驳。还有就是要努力🥦🥦🥦

> [华为c++语言编程规范](https://www.bookstack.cn/read/openharmony-1.0-zh-cn/contribute-OpenHarmony-cpp-coding-style-guide.md)

# 2 冗余代码

**规则4.4.3 不用的代码段直接删除，不要注释掉**

被注释掉的代码，无法被正常维护；当企图恢复使用这段代码时，极有可能引入易被忽略的缺陷。 正确的做法是，不需要的代码直接删除掉。若再需要时，考虑移植或重写这段代码。

这里说的注释掉代码，包括用 `/* */ `和 `//`，还包括 `#if 0， #ifdef NEVER_DEFINED` 等等。

# 3 圈复杂度（Cyclomatic complexity）

详见 博客：`圈复杂度`

定义：软件源码某部分的圈复杂度就是这部分代码中[线性无关](https://baike.baidu.com/item/线性无关/4705660)路径的数量。[百度百科](https://baike.baidu.com/item/%E5%9C%88%E5%A4%8D%E6%9D%82%E5%BA%A6/828737?fr=aladdin)

含huge_cyclomatic_complexity和huge_cca_cyclomatic_complexity两个指标；cyclomatic_complexity和cca_cyclomatic_complexity的计算差异：

* cca_cyclomatic_complexity：不管switch里面有多少个case，圈复杂度只加1
* cyclomatic_complexity：switch里面有多少个case，圈复杂度就加多少

精髓：覆盖所有的可能情况最少使用的测试用例个数

1. ?: 三元运算符 +1
2. case +1
3. default +1
4. else +1

华为规则：圈复杂度改到15以下

# 4 超大目录

华为规则：超大目录门槛50

拆分标准：根据数据库表，PDMReader

# 5 降低文件重复率

**问题**：采用抽离函数的方式降DT重复率，但是cpp文件中重复代码过多且存在不同，所以会导致函数参数过多；抽离的函数中使用的if-else也会过多，导致圈复杂度上升；

**解决**：将抽离函数按照功能块再拆分

# 6 llvm(clang-tidy)

llvm原生的，华为版本的

重新下库后需要执行：

```sh
mkdir cmake_build
cd  cmake_build
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 -DCMAKE_TOOLCHAIN_FILE=../cmake/scripts/gcc_all_toolchain.cmake ..
cp ./compile_commands.json ..
cd ..
```

在01_RB下执行命令，工具路径替换成自己的：

```sh
/usr1/dragon/llvm/tools/clang/tools/extra/clang-tidy/tool/run-clang-tidy.py -check=-*,modernize-use-nullptr,modernize-use-using,modernize-use-override -clang-tidy-binary='/usr1/dragon/llvm/build/bin/clang-tidy' -clang-apply-replacements-binary='/usr1/dragon/llvm/build/bin/clang-apply-replacements' -header-filter=./rating_charging/cppcode/tools/* -fix ./rating_charging/cppcode/tools
```

自动化修复：

* modernize-use-nullptr
* modernize-use-using：typedef->using
* modernize-use-override

# 7 c++类型风格转换

* 先去了解清楚C++11类型转换的语法
* 几种转换使用的场景

# 8 超大函数（huge method）

代码行大于阈值（阈值由产品CMC根据自身情况决策、定义）的函数

# 9 G.EXP.35-CPP 使用`nullptr`作为空指针常量

**【级别】** 要求

**【描述】**

C++11开始引入了nullptr关键字代表空指针，在此之前使用NULL宏来表示空指针常量，导致出现如下问题：

```cpp
#define NULL ((void*)0)

char* str = NULL; // 编译错误: void* 不能自动转换为 char*

void(C::*pmf)() = &C::Func;
if (pmf == NULL) { // 编译错误: void* 不能自动转换为指向成员函数的指针
    ...
}
```

如果把`NULL`被定义为`0`或`0L`。可以解决上面的问题。或者在需要空指针的地方直接使用`0`。

但这引入另一个问题，代码不清晰，特别是使用`auto`自动推导时更为突出：

```cpp
auto result =Find(id);
if (result == 0) {// Find() 返回的是 指针 还是 整数?
	// do something
}
```

`0`字面上是`int`类型(`0L`是`long`)，所以`NULL`和`0`都不是指针类型。

 当重载指针和整数类型的函数时，传递`NULL`或`0`都调用到整数类型重载的函数:

```cpp
void F(int);
void F(int*);

F(0);	// 调用 F(int)，而非 F(int*)
F(NULL);// 调用 F(int)，而非 F(int*)
```

另外，`sizeof(NULL) == sizeof(void*)`并不一定总是成立的，直接使用`0`或`0L`，代码不清晰，且无法做到类型安全；使用`NULL`无法做到类型安全。这些都是潜在的风险。

`nullptr`的优势不仅仅是在字面上代表了空指针，使代码清晰，而且它不再是一个整数类型。

`nullptr`是`std::nullptr_t`类型，而`std::nullptr_t`可以隐式的转换为所有的原始指针类型，这使得`nullptr`可以表现成指向任意类型的空指针。

```cpp
void F(int);
void F(int*);
F(nullptr);	// 调用 F(int*)

auto result =Find(id);
if (result ==nullptr) { // Find() 返回的是 指针
	// do something
}
```

# 10 G.EXP.03-CPP 使用using定义类型别名

**【级别】**建议

**【描述】**

类型的别名实际是对类型的封装。而通过封装，可以让代码更清晰，同时在很大程度上避免类型变化带来的散弹式修改。 

在`C++11`之前，可以通过`typedef`定义类型的别名：

```cpp
typedef std::map<uint32_t, std::vector<int>> SomeType;
```

在`C++11`之后，提供`using`，实现`声明别名(alias declarations)`:

```cpp
using SomeType = std::map<uint32_t, std::vector<int>>;
```

对比两者的格式，使用using更容易理解：

```cpp
typedef Type Alias; // Type 在前，还是 Alias 在前，不宜历届
using Alias = Type; // 符合'赋值'的用法，容易理解，不易出错
```

如果觉得这点还不足以切换到`using`，我们接着看看`模板别名(alias template)`更简洁:

```cpp
// 定义模板的别名，一行代码
template<class T> // template
using MyAllocatorVector = std::vector<T,MyAllocator<T>>;

MyAllocatorVector<int> data;// 使用 using 定义的别名

template<class T> // template
class MyClass {
    ...

    private:
        MyAllocatorVector<int> data_;// 模板类中使用 using 定义的别名
};
```

而`typedef`不支持带模板参数的别名，只能”曲线救国”:

```cpp
// 通过模板包装 typedef，需要实现一个模板类
template<class T>
struct MyAllocatorVector {
    typedef std::vector<T,MyAllocator<T>> type;
};

MyAllocatorVector<int>::type data; // 使用 typedef 定义的别名，多写 ::type

template<class T>
class MyClass {
    ...
        
    private:
    typename MyAllocatorVector<int>::type data_; // 模板类中使用，除了 ::type，还需要加上 typename
};
```

实习中遇到的实例：

```cpp
// 数组
typedef TChar NumArray[24];
using NumArray = TChar[24];

// 结构体c++中，定义变量无需加关键字struct
typedef struct {
    TUInt4 uiAVPcode;
    TUChar ucAVPflag_r : 5;
    ...
} AVPHEAD;
struct AVPHEAD {
    ...
} ;

// 函数指针
// typedef 返回类型(*新类型)(参数表)
typedef RETCODE (*FnFmtFunc)(const TChar* cdrBuff, TUInt4 cdrLen) const;
using FnFmtFunc = RETCODE (*)(const TChar* cdrBuff, TUInt4 cdrLen) const;
```

# 10  G.CLS.12-CPP 在重写虚函数时应明确指定`override`或`final`

**【级别】**要求

**【描述】**

在重写虚函数时明确指定override或final，能保证该函数是重写了基类的虚函数，并且无需再添加virtual关键字。如果派生类函数与基类函数原型不一致，则产生编译错误。`final`还保证虚函数不会再被派生类重写。

* virtual 表示一个新的虚函数
* override 表示重写了基类中的虚函数，可以再被派生类重写
* final 表示重写了基类中的虚函数，不能再被派生类重写

使用`override`或`final`关键字后，如果修改了基类虚函数原型，但忘记修改子类重写的虚函数，在编译期就可以发现。也可以避免有多个子类时，重写虚函数的修改遗漏。

**【正例】**

```cpp
class Base {
public:
    Base();
    virtual ~Base();
    virtual void Foo();
    virtual void Foo(int var);
    void Bar();
};

class Derived(): public Base {
public:
    Derived();
    ~Derived() override; // 符合：析构函数添加override
    void Foo(int var) final; // 符合：重写基类函数，且Derived的派生类不能再重写此函数
    void Foo() override;// 符合：重写基类函数
    void Foo() const override;// 编译失败: Derived::Foo 和 Base::Foo 原型不一致，
    void Bar() override;// 编译失败: Base::Bar 不是虚函数
};
```

**总结**

1. 基类首次定义虚函数，使用`virtual`关键字
2. 子类重写基类虚函数（包括析构函数），使用`override`或`final`关键字（但不要两者一起使用），并且不使用`virtual`关键字
3. 非虚函数，`virtual`、`override`和`final`都不使用

【实习实例】

```cpp
DECLARE_SERVICE_PLUGIN(CBypassMsgController);

#define DECLARE_SERVICE_PLUGIN(CLASS_NAME) \
DECLARE_SERVICE_COMPONET(CLASS_NAME); \
protected: \
	virtual const TChar* MyName() const \
    {
        return #CLASS_NAME; \
    } \
        
#define DECLARE_SERVICE_PLUGIN(CLASS_NAME) \
DECLARE_SERVICE_COMPONET(CLASS_NAME); \
protected: \
	virtual const TChar* MyName() const override \
    {
        return #CLASS_NAME; \
    } \
```

# 11 G.EXP.29-CPP 比较两个表达式，左侧倾向于变化，右侧倾向于不变

**【级别】**建议

**【描述】**

当变量与常量比较时，如果常量放左边，如 if (MAX == v) 不符合阅读习惯，而 if (MAX > v) 更是难于理解。 应当按人的正常阅读、表达习惯，将常量放右边。写成如下方式：

**【正例】**

```cpp
if (value == MAX) {
	...
}

if (value < MAX) {
	...
}
```

也有特殊情况，如：`if (MIN < value && value < MAX)` 用来描述区间时，前半段是常量在左的。

不用担心将 `==` 误写成 `=`，因为`if (value = MAX)` 会有编译告警，其他静态检查工具也会报错。让工具去解决笔误问题，代码要符合可读性第一。

**【脚本误报】**

```cpp
// 逻辑或
if ('N' == cSIPChgSubsNoType || '\0' == cSIPChgSubsNoType) 
if (cSIPChgSubsNoType == 'N' == cSIPChgSubsNoType || '\0')

// 逻辑并
TBool bContinue = 0 >= lSecUnpaid && bSingle;
TBool bContinue = lSecUnpaid && bSingle <= 0;
```

# 12 其他

* G.FMT.02-CPP 使用空格进行缩进，每次缩进4个空格
* G.CMT.03-CPP 文件头注释包含版权说明
* G.CMT.02-CPP 代码注释置于对应代码的上方或右边
* G.FMT.06-CPP 换行时将操作符留在行末，新行缩进一层或进行同类对齐
* G.FMT.09-CPP 构造函数初始化列表放在同一行或按4空格缩进并排多行
* G.FMT.13-CPP case/default语句相对于switch缩进一层
