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
| ​​事务控制语句 | Transaction Control Language | TCL |​ 管理数据库事务。 | `CREATE` `ALTER` `DROP` `TRUNCATE` |

有些人把 DQL 合并到 DML 中，但是由于 `SELECT` 语句太过复杂，而且是只读的操作，故也有人将其单独分为一类。

# SQL 标准
SQL 标准的目的是为了​​统一​​和​​标准化​​关系数据库管理语言。它定义了一套核心的语法、数据类型、操作符、关键字和功能，目的是确保不同厂商实现的数据库系统（如 Oracle, Microsoft SQL Server, MySQL, PostgreSQL, SQLite 等）能使用相同的基础语言进行交互，提高可移植性和互操作性。

SQL 标准主要由两个组织制定： ANSI (美国国家标准学会)和 ISO (国际标准化组织)。通常所说的 ​​ISO/IEC 9075​​ 标准就是官方的 SQL 标准文档。 SQL 标准的主要版本有： SQL-86, SQL-92, SQL:1999, SQL:2003, SQL:2011, SQL:2016, SQL:2019, SQL:2023 等。

尽管 SQL 标准存在，各大数据库厂商都会在核心标准之上实现各自的扩展（SQL 方言）。

