---
title: Python-文件-表格读写
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-09 18:45:09
---

# 1 问题引入

今天实现恶意样本标签提取，用到了表格处理，下面进行整理。

python操作excel主要用到`xlrd`和`xlwt`这两个库，即`xlrd`是读excel，`xlwt`是写excel的库。这两个库使用pip进行安装。

# 2 Python写excel——xlwt

```python
import xlwt


# 导出表格
def export_excel(output_file):
    fields = ['姓名', '年龄']  # 设置自己需要的Excel表头
    book = xlwt.Workbook(encoding='utf-8')  # 获取excel对象
    sheet = book.add_sheet('人员信息表')  # 设置excel的sheet名称
    for col, field in enumerate(fields):  # 写入excel表头
        sheet.write(0, col, field)
    info_list = [['Dragon', 18], ['Pig', 21], ['Cat', 22]]
    row = 1
    for name, age in info_list:  # 根据数据写入excel，col-单元格行标，field-单元格列标
        sheet.write(row, 0, name)
        sheet.write(row, 1, age)
        row += 1
    book.save(output_file)


if __name__ == '__main__':
    output_file = "output.xls"
    export_excel(output_file)
```

表格信息如下：

<img src="https://s2.loli.net/2022/02/09/B8mGXdCqurtDVJ6.png" width = "600" height = "300" alt="图片名称" align=center id=64 />

# 3 Python读excel——xlrd

整体思路为，打开文件，选定表格，读取行列内容，读取表格内数据

详细代码如下：

```python
import xlrd


# 导入表格
def read_excel(input_file):
    # 打开文件
    wb = xlrd.open_workbook(filename=input_file)
    # 获取所有表格名字
    print(wb.sheet_names())  # ['人员信息表']
    # 通过索引获取表格
    sheet1 = wb.sheet_by_index(0)
    # 通过名字获取表格
    sheet2 = wb.sheet_by_name('人员信息表')
    print(sheet1, sheet2)  # Sheet  0:<人员信息表> Sheet  0:<人员信息表>

    print(sheet1.name, sheet1.nrows, sheet1.ncols)  # 人员信息表 4 2
    # 获取第1行内容（编号从0开始）
    rows = sheet1.row_values(1)
    # 获取第1列内容（编号从0开始）
    cols = sheet1.col_values(1)
    print(rows)  # ['Dragon', 18.0]
    print(cols)  # ['年龄', 18.0, 21.0, 22.0]
    # 获取表格里的内容，三种方式
    print(sheet1.cell(1, 0).value)  # Dragon
    print(sheet1.cell_value(1, 0))  # Dragon
    print(sheet1.row(1)[0].value)  # Dragon


if __name__ == '__main__':
    input_file = "output.xls"
    read_excel(input_file)
```

# 4 xls和xlsx的异同

* 文件核心结构不同：
  * xls核心结构是复合文档类型的；
  * xlsx 的核心结构是 XML 类型的结构，并且基于XML进行压缩（占用空间更小），所以也可以看做zip文件，将一个“.xlsx”文件的后缀改为ZIP后，用解压软件解压，可以看到里面有一个xml文件和文件的主要内容。

* 版本不同：
  * xls是excel2003及以前版本所生成的文件格式
  * xlsx是excel2007及以后版本所生成的文件格式
  * （excel 2007之后版本可以打开上述两种格式，但是excel2013只能打开xls格式）

进一步的详细解释参见：[excel后缀.xls和.xlsx有什么区别](https://jingyan.baidu.com/article/1974b289c9aaefb5b0f7746b.html)

# X 参考

* [python如何处理表格？](https://www.php.cn/python-tutorials-451228.html)
* [python实现——处理Excel表格（超详细）](https://blog.csdn.net/weixin_44288604/article/details/120731317)
