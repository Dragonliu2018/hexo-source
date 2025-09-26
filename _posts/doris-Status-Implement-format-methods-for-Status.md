---
title: '[doris][Status] Implement format methods for Status'
categories:
  - [DB,doris]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-14 17:26:44
tags: pr
---

# 1 issue 介绍

* **issue**: https://github.com/apache/doris/issues/25974
* **PR**: https://github.com/apache/doris/pull/26133

当前 `Status` 类型进行 format 时，需要调用 `to_string` 函数：

```cpp
fmt::format("{}", status.to_string());
```

想要的效果是只需要传入 Status 类型即可：

```cpp
fmt::format("{}", status);
```

# 2 实现

## 2.1 解析

`fmt` 库提供了一个类模板 `fmt::formatter`，通过模板特化为 `Status` 类型提供适当的格式化方式。 

然后实现 `parse` 和 `format` 函数即可：

- **`parse`** 函数负责解析格式字符串。在通常情况下，用户可能不需要提供自己的 **`parse`** 函数，而可以使用 fmt 默认提供的版本，该版本简单地返回解析上下文的开始迭代器。
- **`format`** 函数是用户必须提供的关键成员函数。它定义了如何将自定义类型格式化为字符串。

## 2.2 功能代码

>👉 **参考：**https://wgml.pl/blog/formatting-user-defined-types-fmt.html

```cpp
// specify formatter for Status
template <>
struct fmt::formatter<doris::Status> {
    template <typename ParseContext>
    constexpr auto parse(ParseContext& ctx) {
        return ctx.begin();
    }

    template <typename FormatContext>
    auto format(doris::Status const& status, FormatContext& ctx) {
        return fmt::format_to(ctx.out(), "{}", status.to_string());
    }
};
```


## 2.3 测试代码

实现完成后，编写测试样例，这里需要注意覆盖率。

reviewer 提出的建议是：

```cpp
please add 3 tests：
1. status == ok
2. status == error && has stacktrace
3. status == error && do not have stacktrace
```

# 3 问题

## 3.1 ld: symbol(s) not found for architecture x86_64

在本地 macos 上尝试写了个测试程序，但是发生下面的报错：

```bash
➜  test-reset git:(main) ✗ g++ test.cpp -std=c++20 -o test
Undefined symbols for architecture x86_64:
  "void fmt::v10::detail::vformat_to<char>(fmt::v10::detail::buffer<char>&, fmt::v10::basic_string_view<char>, fmt::v10::detail::vformat_args<char>::type, fmt::v10::detail::locale_ref)", referenced from:
      fmt::v10::appender fmt::v10::vformat_to<fmt::v10::appender, 0>(fmt::v10::appender, fmt::v10::basic_string_view<char>, fmt::v10::basic_format_args<fmt::v10::basic_format_context<fmt::v10::appender, char> >) in test-878b3f.o
  "fmt::v10::detail::assert_fail(char const*, int, char const*)", referenced from:
      std::__1::make_unsigned<long>::type fmt::v10::detail::to_unsigned<long>(long) in test-878b3f.o
  "fmt::v10::vprint(fmt::v10::basic_string_view<char>, fmt::v10::basic_format_args<fmt::v10::basic_format_context<fmt::v10::appender, char> >)", referenced from:
      _main in test-878b3f.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

**解决**：原因是未链接 fmt 库文件

```bash
g++ test.cpp -std=c++20 -o test -lfmt
```

**参考**：https://stackoverflow.com/questions/56608684/how-to-use-the-fmt-library-without-getting-undefined-symbols-for-architecture-x

## 3.2 `fmt::formatter` 模板特化代码不能在 doris 的 namespace

开始将特化代码放在了 doris namespace 中，发生了下面的报错：

```bash
Class template specialization of 'formatter' not in a namespace enclosing 'v8' is a Microsoft extension clang(-Wmicrosoft-template)

[core.h(707, 8): ] Explicitly specialized declaration is here
```

**解决**：

```cpp
namespace doris {

// doris code

} // end doris_namespace

// define formatter here
```

**参考：**https://github.com/fmtlib/fmt/issues/2767