---
title: '[Databend][pr] pivot 支持 subquery'
categories:
  - [DB,Databend,pr]
  - [DB,Databend,执行器,表达式]
  - [DB,执行器,表达式]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-14 17:45:42
tags: pr
---

# 1 任务背景

* **Issue**：https://github.com/databendlabs/databend/issues/16556
* **PR：**https://github.com/databendlabs/databend/pull/16631

## 1.1 任务介绍

**目前 databend 中的 pivot 和 unpivot 不支持 `FROM/IN + subquery`**。→ [文档链接](https://docs.databend.com/sql/sql-commands/query-syntax/query-pivot)

**要求实现类似于 snowflake 的语法，也就是让 pivot 支持 `FROM/IN + subquery`，unpivot 支持 `FROM + subquery`**。→ [文档链接](https://docs.snowflake.com/en/sql-reference/constructs/pivot)

- **databend PIVOT 语法**
  
    ```sql
    SELECT ...
    FROM ...
       PIVOT ( <aggregate_function> ( <pivot_column> )
                FOR <value_column> IN ( <pivot_value_1> [ , <pivot_value_2> ... ] ) )
    
    [ ... ]
    ```
    
    - `<aggregate_function>`：用于组合来自`pivot_column`的分组值的聚合函数。
    - `<pivot_column>`：将使用指定的`<aggregate_function>`进行聚合的列。
    - `<value_column>`：其唯一值将在旋转结果集中成为新列的列。
    - `<pivot_value_N>`：来自`<value_column>`的唯一值，将在旋转结果集中成为新列。
- **databend UNPIVOT 语法**
  
    ```sql
    SELECT ...
    FROM ...
        UNPIVOT ( <value_column>
        FOR <name_column> IN ( <column_list> ) )
    
    [ ... ]
    ```
    
    - `<value_column>`：将存储从 `<column_list>` 中列出的列中提取的值的列。
    - `<name_column>`：将存储从中提取值的列的名称的列。
    - `<column_list>`：要取消透视的列的列表，用逗号分隔。
- **snowflake PIVOT 语法**
  
    ```sql
    SELECT ...
    FROM ...
       PIVOT ( <aggregate_function> ( <pivot_column> )
                FOR <value_column> IN (
                  <pivot_value_1> [ , <pivot_value_2> ... ]
                  | ANY [ ORDER BY ... ]
                  | <subquery>
                )
                [ DEFAULT ON NULL (<value>) ]
             )
    
    [ ... ]
    ```
    
- **snowflake UNPIVOT 语法**
  
    ```sql
    SELECT ...
    FROM ...
        UNPIVOT [ { INCLUDE | EXCLUDE } NULLS ]
          ( <value_column>
            FOR <name_column> IN ( <column_list> ) )
    
    [ ... ]
    ```
    

## 1.2 介绍 pivot & unpivot

### 1.2.1 **pivot（透视，一列中的属性值转成多个对应属性）**

> https://docs.databend.com/sql/sql-commands/query-syntax/query-pivot
> 

---

**`PIVOT`操作通过旋转表格并基于指定列聚合结果来转换表格**，这是一个对于总结和分析大量数据以更可读格式显示非常有用的操作。

---

- **databend PIVOT 语法**
  
    ```sql
    SELECT ...
    FROM ...
       PIVOT ( <aggregate_function> ( <pivot_column> )
                FOR <value_column> IN ( <pivot_value_1> [ , <pivot_value_2> ... ] ) )
    
    [ ... ]
    ```
    
    - `<aggregate_function>`：用于组合来自`pivot_column`的分组值的聚合函数。
    - `<pivot_column>`：将使用指定的`<aggregate_function>`进行聚合的列。
    - `<value_column>`：其唯一值将在旋转结果集中成为新列的列。
    - `<pivot_value_N>`：来自`<value_column>`的唯一值，将在旋转结果集中成为新列。

---

假设我们有一个名为 monthly_sales 的表，其中**包含不同员工在不同月份的销售数据**。我们**可以使用`PIVOT`操作来总结数据并计算每个员工在每个月的总销售额**。

- **创建和插入数据**
  
    ```sql
    -- 创建monthly_sales表
    CREATE TABLE monthly_sales(
      empid INT,
      amount INT,
      month VARCHAR
    );
    
    -- 插入销售数据
    INSERT INTO monthly_sales VALUES
      (1, 10000, 'JAN'),
      (1, 400, 'JAN'),
      (2, 4500, 'JAN'),
      (2, 35000, 'JAN'),
      (1, 5000, 'FEB'),
      (1, 3000, 'FEB'),
      (2, 200, 'FEB'),
      (2, 90500, 'FEB'),
      (1, 6000, 'MAR'),
      (1, 5000, 'MAR'),
      (2, 2500, 'MAR'),
      (2, 9500, 'MAR'),
      (1, 8000, 'APR'),
      (1, 10000, 'APR'),
      (2, 800, 'APR'),
      (2, 4500, 'APR');
    ```
    
- **使用 PIVOT**
  
    现在，我们可以使用`PIVOT`操作来计算每个员工在每个月的总销售额。我们将使用`SUM`聚合函数来计算总销售额，**MONTH 列将被旋转以为每个月创建新列。**
    
    ```sql
    SELECT *
    FROM monthly_sales
    PIVOT(SUM(amount) FOR MONTH IN ('JAN', 'FEB', 'MAR', 'APR'))
    ORDER BY EMPID;
    ```
    
    输出：
    
    ```sql
    +-------+-------+-------+-------+-------+
    | empid | jan   | feb   | mar   | apr   |
    +-------+-------+-------+-------+-------+
    |     1 | 10400 |  8000 | 11000 | 18000 |
    |     2 | 39500 | 90700 | 12000 |  5300 |
    +-------+-------+-------+-------+-------+
    ```
    

### 1.2.2 **unpivot（取消透视，多个属性转成一列中对应的属性值）**

> https://docs.databend.com/sql/sql-commands/query-syntax/query-unpivot
> 

---

`UNPIVOT` 操作通过将列转换为行来旋转表。它是一个关系操作符，接受两列（来自表或子查询），以及列的列表，并为列表中指定的每列生成一行。在查询中，它在 FROM 子句中指定，位于表名或子查询之后。

---

- **databend UNPIVOT 语法**
  
    ```sql
    SELECT ...
    FROM ...
        UNPIVOT ( <value_column>
        FOR <name_column> IN ( <column_list> ) )
    
    [ ... ]
    ```
    
    - `<value_column>`：将存储从 `<column_list>` 中列出的列中提取的值的列。
    - `<name_column>`：将存储从中提取值的列的名称的列。
    - `<column_list>`：要取消透视的列的列表，用逗号分隔。

---

让我们取消透视个别月份列，以返回每位员工每月的单一销售值：

- **创建和插入数据**
  
    ```sql
    -- 创建 unpivoted_monthly_sales 表
    CREATE TABLE unpivoted_monthly_sales(
      empid INT,
      jan INT,
      feb INT,
      mar INT,
      apr INT
    );
    
    -- 插入销售数据
    INSERT INTO unpivoted_monthly_sales VALUES
      (1, 10400,  8000, 11000, 18000),
      (2, 39500, 90700, 12000,  5300);
    ```
    
- **使用 UNPIVOT**
  
    ```sql
    SELECT *
    FROM unpivoted_monthly_sales
        UNPIVOT (amount
        FOR month IN (jan, feb, mar, apr));
    ```
    
    输出：
    
    ```sql
    +-------+-------+--------+
    | empid | month | amount |
    +-------+-------+--------+
    |     1 | jan   |  10400 |
    |     1 | feb   |   8000 |
    |     1 | mar   |  11000 |
    |     1 | apr   |  18000 |
    |     2 | jan   |  39500 |
    |     2 | feb   |  90700 |
    |     2 | mar   |  12000 |
    |     2 | apr   |   5300 |
    +-------+-------+--------+
    ```
    

# 2 设计参考（竞品分析）

# 3 思路与实现

## 3.1 思路

实现 `from + subquery` 较为简单，在 TableReference::Subquery 加上 pivot 和 unpivot 字段，修改 parser 及其他相关地方。

```rust
let subquery = map(
    rule! {
        LATERAL? ~ "(" ~ #query ~ ")" ~ #table_alias? ~ #pivot? ~ #unpivot?
    },
    |(lateral, _, subquery, _, alias, pivot, unpivot)| TableReferenceElement::Subquery {
        lateral: lateral.is_some(),
        subquery: Box::new(subquery),
        alias,
        pivot: pivot.map(Box::new),
        unpivot: unpivot.map(Box::new),
    },
);
```

---

实现 `in + subquery` 相对复杂，因为在 binder 的时候必须知道 column 的 name 才能生成逻辑计划，所以只能先执行一下这个 subquery ，然后把返回值当作 values 处理；但是执行 query/plan 是在 service 模块下，binder 是在 sql 模块，存在循环调用的问题。→ [解决循环依赖问题](https://www.notion.so/125e3d1c52c6806ca52dedaa716ef5ae?pvs=21)

解决思路有三种：

1. 【**思路-1**】在 pivot_rewrite 阶段（binder）执行 Subquery 得到答案，参考 QuerySampleExecutor 写个回调函数在 binder 里面调一下 service 的函数执行。
    1. Pivot 结构体新添 Subquery 字段（Query），用于存储 `in + subquery` 中的 subquery；
        - **Pivot**
          
            ```rust
            // 原版
            pub struct Pivot {
                pub aggregate: Expr,
                pub value_column: Identifier,
                pub values: Vec<Expr>,
            }
            
            // 修改后
            pub enum PivotValues {
                ColumnValues(Vec<Expr>),
                Subquery(Box<Query>),
            }
            
            pub struct Pivot {
                pub aggregate: Expr,
                pub value_column: Identifier,
                pub values: PivotValues,
            }
            ```
        
    2. 在 rewrite_pivot 函数中对 Subquery 进行 parser + rewrite + opt + exec 等操作，并将结果存在原来的 `values` 字段，后续操作一致。
        1. 执行 query 的回调函数目前有基于 PhysicalPlan 的，一种方案是基于该函数进行操作；另一种方案是新添基于 Query String 执行的回调函数。
2. 【**思路-2**】在发现 subquery 的时候返回，在 service 里面执行之后重写 sql，然后再次执行。
3. 【**思路-3**】现在的 plan_sql 函数里面会执行 parser 并生成 plan，可以把这个函数拆成两个：
    1. 第一个函数执行 parser，生成 stmt，同时进行一些 rewrite，如果这时发现有 subquery 的话，就先执行一下，然后 rewrite。
    2. 第二个函数传入 rewrite 之后的 stmt，生成 plan。

本着尽可能少的改动当前代码结构的原则，选择**【思路-1】**进行实现。

## 3.2 review 相关修改

- 【**review-1**】避免不必要的 clone
  
    ```rust
    // 原版
    let value = columns[0].to_column(block.num_rows());
     
    // 修改
    for row in 0..block.num_rows() {
        match columns[0].value.index(row).unwrap() {
            ScalarRef::String(s) => {
                let literal = Expr::Literal {
                    span,
                    value: Literal::String(s.to_string()),
                };
                values.push(literal);
            }
            _ => {
                return Err(ErrorCode::SemanticError(
                    "The subquery of `pivot in` must return a string type",
                )
                .set_span(span));
            }
        }
    }
    ```
    
- 【**review-2**】针对于 subquery 返回的每个 String 不需要设置 span，使用 subquery 的 span 即可。
- 【**review-3**】使用 `block.num_columns()` 计算 column 的数量。
  
    ```rust
    // 原版
    let columns = block.columns();
    if columns.len() != 1 {
    	// ...
    }
    
    // 修改后
    if block.num_columns() != 1 {
    	// ...
    }
    ```
    
- 【**review-4**】将 sample_executor 和 subquery_executor 合并成一个 query_executor，实现不同的函数用来做区分。
- 【**review-5**】对返回的 ErrorCode 设置 span。→ [span](https://www.notion.so/span-123e3d1c52c68003850bf86ae43a4c3d?pvs=21)
  
    不使用 span：
    
    ```rust
    return Err(ErrorCode::SemanticError(
        "The subquery of `pivot in` must return one column",
    ));
    ```
    
    ```sql
    MySQL [dev]> SELECT *  FROM (select * from monthly_sales) PIVOT(SUM(amount) FOR MONTH IN (select distinct month, month from monthly_sales)) ORDER BY EMPID;
    ERROR 1105 (HY000): SemanticError. Code: 1065, Text = The subquery of `pivot in` must return one column.
    ```
    
    ---
    
    使用 span：
    
    ```rust
    return Err(ErrorCode::SemanticError(
        "The subquery of `pivot in` must return one column",
    )
    .set_span(span));
    ```
    
    ```sql
    MySQL [dev]> SELECT *  FROM (select * from monthly_sales) PIVOT(SUM(amount) FOR MONTH IN (select distinct empid from monthly_sales)) ORDER BY EMPID;
    ERROR 1105 (HY000): SemanticError. Code: 1065, Text = error: 
      --> SQL:1:78
      |
    1 | SELECT *  FROM (select * from monthly_sales) PIVOT(SUM(amount) FOR MONTH IN (select distinct empid from monthly_sales)) ORDER BY EMPID
      |                                                                              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ The subquery of `pivot in` must return a string type
    
    .
    ```
    

## 3.3 bugs

- 【**bug-1**】因为需要两次用到 pivot 变量，所以会报 error: use of moved value 的错误。→ [error: use of moved value](https://www.notion.so/error-use-of-moved-value-11fe3d1c52c68078839ae6d3f66d8f62?pvs=21)
  
    把 pivot 和 unpivot 挪出去，改成两个单独的函数。
    
- 【**bug-2**】rewrite_pivot 是非 async 函数，但是 subquery_executor.execute_query_with_sql_string 是 async 函数，rewrite_pivot 中需要调用 subquery_executor.execute_query_with_sql_string。所以会报错：`await` is only allowed inside `async` functions and blocks。→ [在非 async 函数里调用 async 函数](https://www.notion.so/async-async-125e3d1c52c680008cb5e6a86c6a6554?pvs=21)
  
    `databend_common_base::runtime::block_on`
    

# 4 测试

## 4.1 功能测试

与 snowflake 语法保持一致：

- pivot 支持三种 subquery，`from + subquery`、`in + subquery`、`from + subquery + in + subquery`。
- unpivot 只支持 `from + subquery` 一种。

### 4.1.1 手动测试

- **测试 Pivot**
    - **Create the monthly_sales table**
      
        ```sql
        CREATE TABLE monthly_sales(
          empid INT, 
          amount INT, 
          month VARCHAR
        );
        ```
        
    - **Insert sales data**
      
        ```sql
        INSERT INTO monthly_sales VALUES
          (1, 10000, 'JAN'),
          (1, 400, 'JAN'),
          (2, 4500, 'JAN'),
          (2, 35000, 'JAN'),
          (1, 5000, 'FEB'),
          (1, 3000, 'FEB'),
          (2, 200, 'FEB'),
          (2, 90500, 'FEB'),
          (1, 6000, 'MAR'),
          (1, 5000, 'MAR'),
          (2, 2500, 'MAR'),
          (2, 9500, 'MAR'),
          (1, 8000, 'APR'),
          (1, 10000, 'APR'),
          (2, 800, 'APR'),
          (2, 4500, 'APR');
        ```
        
    - **Pivot（原版）**
      
        ```sql
        SELECT * 
        FROM monthly_sales
        PIVOT(SUM(amount) FOR MONTH IN ('JAN', 'FEB', 'MAR', 'APR'))
        ORDER BY EMPID;
        ```
        
    - **from + subquery**
      
        ```sql
        SELECT *
        FROM (select * from monthly_sales)
        PIVOT(SUM(amount) FOR MONTH IN ('JAN', 'FEB', 'MAR', 'APR'))
        ORDER BY EMPID;
        ```
        
    - **in + subquery**
      
        ```sql
        SELECT * 
        FROM monthly_sales
        PIVOT(SUM(amount) FOR MONTH IN (select distinct month from monthly_sales))
        ORDER BY EMPID;
        ```
        
    - **from + in + subquery**
      
        ```sql
        SELECT * 
        FROM (select * from monthly_sales)
        PIVOT(SUM(amount) FOR MONTH IN (select distinct month from monthly_sales))
        ORDER BY EMPID;
        ```
        
    - **Answer**
      
        ```sql
        +-------+-------+-------+-------+-------+
        | empid | jan   | feb   | mar   | apr   |
        +-------+-------+-------+-------+-------+
        |     1 | 10400 |  8000 | 11000 | 18000 |
        |     2 | 39500 | 90700 | 12000 |  5300 |
        +-------+-------+-------+-------+-------+
        ```
    
- **测试 Unpivot**
    - **Create the unpivoted_monthly_sales table**
      
        ```sql
        CREATE TABLE unpivoted_monthly_sales(
          empid INT,
          jan INT,
          feb INT,
          mar INT,
          apr INT
        );
        ```
        
    - **Insert sales data**
      
        ```sql
        INSERT INTO unpivoted_monthly_sales VALUES
          (1, 10400,  8000, 11000, 18000),
          (2, 39500, 90700, 12000,  5300);
        ```
        
    - **unpivot（原版）**
      
        ```sql
        SELECT *
        FROM unpivoted_monthly_sales
            UNPIVOT (amount
            FOR month IN (jan, feb, mar, apr));
        ```
        
    - **from + subquery**
      
        ```sql
        SELECT * FROM (SELECT * FROM monthly_sales_1)
          UNPIVOT(sales FOR month IN (jan, feb, mar, april))
          ORDER BY empid
        ```
        
    - **Answer**
      
        ```sql
        +-------+-------+--------+
        | empid | month | amount |
        +-------+-------+--------+
        |     1 | jan   |  10400 |
        |     1 | feb   |   8000 |
        |     1 | mar   |  11000 |
        |     1 | apr   |  18000 |
        |     2 | jan   |  39500 |
        |     2 | feb   |  90700 |
        |     2 | mar   |  12000 |
        |     2 | apr   |   5300 |
        +-------+-------+--------+
        ```
        

### 4.1.2 单元测试（ast 相关）

```bash
make unit-test
```

parser 改了会导致之前的测试失败，修复下面两个文件：

1. `src/query/ast/tests/it/testdata/stmt.txt`
2. `src/query/ast/tests/it/testdata/query.txt`

---

另外在 test_query 函数加上一个 pivot & unpivot 带 subquery 的测试用例，相关文件：`src/query/ast/tests/it/parser.rs`。

### 4.1.3 逻辑测试（sql 执行结果）

```bash
make sqllogic-test
```

为了加快测试速度，可以**指定目录**进行执行，甚至可以**修改文件名**使其在当前目录先执行：

```bash
./target/debug/databend-sqllogictests --handlers mysql --run_dir query
```

---

添加 pivot & unpivot 相关 sql 查询，相关文件：

- `tests/sqllogictests/suites/query/pivot.test`
- `tests/sqllogictests/suites/query/unpivot.test`

## 4.2 性能测试

# 5 难点

# 6 TODO

# X 参考

- https://docs.snowflake.com/en/sql-reference/constructs/pivot
- https://docs.snowflake.com/en/sql-reference/constructs/unpivot
- https://docs.databend.com/sql/sql-commands/query-syntax/query-pivot
- https://docs.databend.com/sql/sql-commands/query-syntax/query-unpivot