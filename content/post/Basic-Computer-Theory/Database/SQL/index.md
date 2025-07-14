+++
date = '2025-07-13T19:52:22+08:00'
draft = false
title = 'SQL'
categories = ['Sub Sections']
+++

# 什么是 SQL
SQL, 全称 Structured Query Language ，结构化查询语言。 SQL 是一种​​标准化的编程语言​​，专门用于​​管理关系型数据库​​ (Relational Database Management Systems - RDBMS) 和对其中的数据进行​​操作​​。

SQL 有以下特点：

1. **声明式**：​​用户主要​​描述需要什么​​（What），而不是详细指定​​如何获取​​（How）。数据库引擎内部的查询优化器负责找到最高效的执行路径。这与命令式语言（如 Python, Java）需要一步步指定操作流程不同。
1. **基于集合**：​​SQL 语句操作的对象通常是数据行（记录）的集合（表），而不是逐条处理记录（当然也能做到，但通常效率较低）。
1. **领域特定 (DSL)**：​​专门为数据库操作而设计，不是通用的编程语言。

# SQL 语句类别
| 中文名称 | 英文名称 | 英文缩写 | 说明 | SQL 语句 |
| :--: | :--: | :--: | :--: | :--: |
| 数据定义语句 | Data Definition Language | DDL |​ ​定义和修改数据库结构。 | `CREATE` `ALTER` `DROP` `TRUNCATE` |
| 数据操作语句 | Data Manipulation Language | DML |​ 操作数据库表中的数据行。 | `INSERT` `UPDATE` `DELETE` |
| 数据查询语句 | Data Manipulation Language | DQL |​ 查询数据库表中的数据行。 | `SELECT` |
| ​​数据控制语句 | Data Control Language | DCL |​ 控制对数据和数据库的访问权限。 | `GRANT` `REVOKE` |
| ​​事务控制语句 | Transaction Control Language | TCL |​ 管理数据库事务。 | `COMMIT` `ROLLBACK` `SAVEPOINT` |

有些人把 DQL 合并到 DML 中，但是由于 `SELECT` 语句太过复杂，而且是只读的操作，故也有人将其单独分为一类。

# SQL 标准
SQL 标准的目的是为了​​统一​​和​​标准化​​关系数据库管理语言。它定义了一套核心的语法、数据类型、操作符、关键字和功能，目的是确保不同厂商实现的数据库系统（如 Oracle, Microsoft SQL Server, MySQL, PostgreSQL, SQLite 等）能使用相同的基础语言进行交互，提高可移植性和互操作性。

SQL 标准主要由两个组织制定： ANSI (美国国家标准学会)和 ISO (国际标准化组织)。通常所说的 ​​ISO/IEC 9075​​ 标准就是官方的 SQL 标准文档。 SQL 标准的主要版本有： SQL-86, SQL-92, SQL:1999, SQL:2003, SQL:2011, SQL:2016, SQL:2019, SQL:2023 等。

尽管 SQL 标准存在，各大数据库厂商都会在核心标准之上实现各自的扩展（SQL 方言）。

# SQL 语句
有许多资料站，系统地记录了各种 SQL 语句，比如：

* [SQL 简介 | 菜鸟教程](https://www.runoob.com/sql/sql-intro.html)

# SQL 高级
## 触发器
触发器(Triggerss)首次出现在 SQL:1999(SQL3) 标准中。

功能：

* 当指定事件（ `INSERT` `UPDATE` `DELETE` ）发生时​​自动执行​​的代码块。
* 支持在操作​​前（ `BEFORE` ）​​ 或​​后（ `AFTER` ）​​ 触发。
* 可访问新旧数据（通过 `OLD` `NEW` 伪表）。

示例​​：当有新订单插入 orders 表时，自动在 order_audit 表中添加审计记录。

```SQL
CREATE TRIGGER audit_log                   -- 创建名为 audit_log 的触发器
AFTER INSERT ON orders                     -- 在 orders 表发生 INSERT 操作后触发
REFERENCING NEW ROW AS new_order           -- 使用别名 new_order 引用新插入的行
FOR EACH ROW                               -- 行级触发器（每条记录都触发）
INSERT INTO order_audit                    -- 向 order_audit 表插入记录
VALUES (new_order.id, CURRENT_TIMESTAMP);  -- 值：订单 ID + 当前时间戳
```

## 存储过程与函数
存储过程与函数(Stored Procedures & Functions)​首次出现在 SQL:1999(SQL3) 标准中。

功能：

* 存储过程：​​用 `CREATE PROCEDURE` 定义，支持参数输入/输出，无返回值。
​* ​函数：​​用 `CREATE FUNCTION` 定义，必须返回一个标量值或表。
* 支持变量、条件分支（ `IF` `CASE` ）、循环（ `LOOP` `WHILE` ）、异常处理。
* 存储过程和函数是预编译的，执行速度比 SQL 语句快。

示例：处理订单并同步更新库存。

```SQL
CREATE PROCEDURE add_order(                    -- 创建存储过程 add_order
    IN product_id INT,                         -- 输入参数：商品 ID （整数）
    IN quantity INT                            -- 输入参数：购买数量（整数）
)
BEGIN                                          -- 过程体开始
    INSERT INTO orders (product_id, quantity)  -- 向 orders 表插入新订单
    VALUES (product_id, quantity);             -- 值：传入的商品 ID 和数量

    UPDATE inventory                           -- 更新库存表
    SET stock = stock - quantity               -- 库存减少订购数量
    WHERE id = product_id;                     -- 只更新指定商品
END;                                           -- 过程体结束
```

## 查询的中间结果
有些时候，我们要进行复杂的查询，比如多表查询。我们可以使用中间结果来简化复杂查询的逻辑。

### 子查询
子查询(Subqueries)于 SQL-92​ 引入。

特点：
* 仅在​​单个查询​​内有效。

示例：子查询创建临时的"高额订单数据集"别名为 o ，然后与 products 表进行关联查询。

```SQL
SELECT o.id, p.name                 -- 选择订单 ID + 商品名称
FROM (                              -- 子查询结果作为临时表
    SELECT *                        -- 选择所有字段
    FROM orders                     -- 从订单表
    WHERE amount > 1000             -- 高额订单
) AS o                              -- 给临时结果命名别名为 o
JOIN products p                     -- 连接商品表（别名 p ）
ON o.product_id = p.id;             -- 按商品 ID 关联
```

### 公共表达式
公共表达式(Common Table Expression, CTE)于 SQL:1999​​ 引入，使用 `WITH` 子句定义临时数据集。

​​特点：​​

* 仅在​​单个查询​​内有效。
* 支持递归查询（ `WITH RECURSIVE` ）。

示例​：创建临时高额订单数据集，再筛选特定商品类型的订单

```SQL
WITH cte_orders AS (                  -- 创建临时数据集 cte_orders
    SELECT *                          -- 选择所有字段
    FROM orders                       -- 从订单表
    WHERE amount > 1000               -- 筛选金额大于 1000 的订单
)
SELECT *                              -- 从临时数据集查询
FROM cte_orders                       -- 指定数据源
WHERE product_id IN (                 -- 附加筛选条件
    SELECT id FROM products           -- 子查询获取商品 ID
);                                    -- 查询结束 CTE 自动销毁
```

### 临时表​
临时表(Temporary Tables)​于 ​​SQL-92​​ 引入。

​​特点：​​

* 在会话/事务结束时​​自动销毁​​。
* 语法与普通表相同（ `CREATE TEMPORARY TABLE` ）。

示例：

```SQL
CREATE TEMPORARY TABLE temp_orders  -- 创建临时表（会话结束时自动删除）
AS                                  -- 用查询结果填充表
SELECT *                            -- 选择所有字段
FROM orders                         -- 从订单表
WHERE amount > 1000;                -- 只取高额订单

SELECT * FROM temp_orders;          -- 直接查询临时表
```

### 视图
视图(Views)​于 ​​SQL-92​​ 引入。

​​特点：​​

* 将查询​​保存为数据库对象​​（虚拟表）。
​* ​长期存在​​，可跨会话复用。

示例：

```SQL
CREATE VIEW expensive_orders        -- 创建视图 expensive_orders
AS                                  -- 视图定义开始
SELECT *                            -- 选择所有字段
FROM orders                         -- 从订单表
WHERE amount > 1000;                -- 只包含金额>1000的记录

SELECT * FROM expensive_orders;     -- 像查表一样查询视图
```

