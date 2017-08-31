---
title: SQL 体系结构
date: 2017-08-31
tags: 数据库
categories: 数据库
---

了解MySql必须牢牢记住其体系结构图，Mysql是由SQL接口，解析器，优化器，缓存，存储引擎组成的。

1. Connectors指的是不同语言中与SQL的交互
 
2. Management Serveices & Utilities： 系统管理和控制工具
 
3. Connection Pool: 连接池。管理缓冲用户连接，线程处理等需要缓存的需求

4. SQL Interface: SQL接口。接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface

5. Parser: 解析器。SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的，是一个很长的脚本。

主要功能: 1) 将SQL语句分解成数据结构，并将这个结构传递到后续步骤，以后SQL语句的传递和处理就是基于这个结构; 2) 如果在分解构成中遇到错误，那么就说明这个sql语句是不合理的
