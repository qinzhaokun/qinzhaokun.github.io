---
title: SQL语法大全
date: 2017-09-09 18:54:25
tags: 数据库
categories: 数据库
---

MYSQL是非常流行的关系型数据库，而SQL语句是数据库交互的主要工具。

## 数据库创建、删除、备份和还原

### 创建

```
语法：CREATE DATABASE _NAME

e.x. CREATE DATABASE user
```

### 删除

```
语法：DROP DATABASE _NAME

e.x. DROP DATABASE user
```

### 备份
```
语法：mysqldump -u username -p dbname table1 table2... > backipname.sql
```
+ dbname参数表示数据库的名称；
+ table1和table2参数表示需要备份的表的名称，为空则整个数据库备份；
+ BackupName.sql参数表设计备份文件的名称，文件名前面可以加上一个绝对路径。通常将数据库被分成一个后缀名为sql的文件

### 还原
```
语法：mysql -u root -p < D:\back.sql
```

## 数据库表操作

首先进入某个数据库
```
USE DATABASE;
```
### 创建新表
```
CREATE TABLE _TABLE_NAME (COL1 TYPE1, COL2 TYPE2, COL3, TYPE3)
```

如上看到很多列，那么如果定义各列的属性呢？如字段**自增**或者设定**主键**，**非空**属性

+ 自增是两种方法：`AUTO_INVREMENT`和`IDENTITY(1,1)`

+ 主键：`PRIMARY KEY`, 或者在最后写`PRIMARY KEY('id')`

+ 非空：关键字`NOT NULL`

+ 检查范围：`CHECK` 如：`age int check (age >= 0 and age < 200)

+ 默认值：`DEFAULT` 如：sex int default ('male')

+ 外键：`CONSTRAINT _KEY_NAME FOREIGN KEY REFERENCES _TABLE_NAME(COL1)` dept_id int constraint fk_dept_id_heihei foreign key references dept(dept_id), 

```
e.x. CREATE TABLE user (id int PRIMARY KEY AUTO_INVREMENT NOT NULL, age int check (age >= 0 and age < 200), name varchar(55) DEFAULT ('zhangshan'))
```

### 从已有表创建新表

```
语法：CREATE _NEW_TABLE LIKE _OLD_TABLE;

CREATE _NEW_TABLE AS SELECT COL1, COL2 FROM _OLD_TABLE
```

### 删除表

关于删除表，有：`DROP, DELETE, TRUNCATE`这三个关键字

#### DROP

删除表的所有数据，以及表结构，同时删除表的结构被依赖的约束（constrain),触发器（trigger)索引（index)
```
语法： DROP TABLE _TABLE_NAME
```

#### TRUNCATE

清空数据，内容，表结构，索引，触发器等保留。
```
语法： TRUNCATE TABLE _TABLE_NAME
```

#### DELETE

清空数据，内容，表结构，索引，触发器等保留，可以和`WHERE`一起使用，删除指定的行。同时将该行的删除操作作为事务记录在日志中保存,以便进行进行回滚操作
```
语法：DELETE TABLE _TABLE_NAME
```

#### 区别

1. 速度上：drop> truncate > delete

2. truncate、drop 是数据库定义语言(ddl)，操作立即生效，原数据不放到 rollback segment 中，不能回滚，操作不触发 trigger; delete语句是数据库操作语言(dml)，这个操作会放到 rollback segement 中，事务提交之后才生效；如果有相应的 trigger，执行的时候将被触发

### 给表添加'东西'

如果表已经存在，那么想给某个表添加一个属性，设置新属性等,使用ALERT关键字

#### 添加列
```
ALERT TABLE _TABLE_NAME ADD COLUMN COL5 TYPE1
```

`COLUMN COL5 TYPE1`关于列的属性可以参照上面

#### 添加主键

新增/删除一个主键：
```
ALERT TABLE _TABLE_NAME ADD/DROP PRIMARY KEY(COL2)
```

## 数据库表的索引

索引是提升数据查询的关键，也是相比于表的创建，删除和修改等操作更重要

### 普通索引

可以使用两种方式来创建索引，分别是`ALTER TABLE`和`CREATE INDEX`

```
ALERT TABLE _TABLE_NAME ADD INDEX _INDEX_NAME (COL3)

CREATE INDEX _INDEX_NAME ON _TALBE_NAME (COL3)
```

### 唯一索引

和普通索引不同，唯一索引要求索引列的值唯一，如果是多列索引，则多列的组合唯一
```
ALERT TABLE _TABLE_NAME ADD UNIQUE _INDEX_NAME(COL3)

CREATE UNIQUE INDEX _INDEX_NAME ON _TABLE_NAME (COL3)
```

### 全文索引

旧版的MySQL的全文索引只能用在MyISAM表格的char、varchar和text的字段上。 

不过新版的MySQL5.6.24上**InnoDB引擎也加入了全文索引**
```
ALERT TABLE _TABLE_NAME ADD FULLTEXT INDEX _INDEX_NAME (COL3)

CREATE FULLTEXT INDEX _INDEX_NAME ON _TALBE_NAME (COL3)
```
使用全文索引时，和其他的索引有所不同，

```
MATCH (columnName) AGAINST ('string')
```

### 删除索引

```
DROP INDEX _INDEX_NAME ON _TABLE_NAME
ALTER TABLE _TABLE_NAME DROP INDEX _INDEX_NAME
```

## 数据库视图

视图是由查询结果形成的一张虚拟表，如果某个查询的结果非常频繁，要经常基于这个查询来做子查询，我们可以创建视图
```
CREATE VIEW _VIEW_NAME AS SELECT 语句;

e.x. ： CREATE VIEW AVGPRICE as SELECT cat_id, avg(shop_price) FROM goods GROUPBY cat_id

SELCET * FROM AVEPRICE;
```


## 数据库触发器

触发器是一种与表操作有关的数据库对象，当触发器所在表上出现指定事件时，将调用该对象，即表的操作事件触发表上的触发器的执行。

```
CREATE TRIGGER trigger_name trigger_time trigger_event ON tbl_name FOR EACH ROW trigger_stmt
```

+ trigger_name：标识触发器名称，用户自行指定；

+ trigger_time：标识触发时机，取值为 BEFORE 或 AFTER；

+ trigger_event：标识触发事件，取值为 INSERT、UPDATE 或 DELETE；

+ tbl_name：标识建立触发器的表名，即在哪张表上建立触发器；

+ trigger_stmt：触发器程序体，可以是一句SQL语句，或者用 BEGIN 和 END 包含的多条语句。

```
DELIMITER $
create trigger tri_stuInsert after insert
on student for each row
begin
declare c int;
set c = (select stuCount from class where classID=new.classID);
update class set stuCount = c + 1 where classID = new.classID;
end$
DELIMITER ;
```

## 数据插入

插入一行数据：
```
INSERT INTO _TABLE_NAME(COL1,COL2) VALUES(_VALUE1,_VALUE2);
或者
INSERT INTO _TABLE_NAME VALUES(COL1，COL2，...,COLN);
```

## 数据更新
```
UPDATE _TABLE_NAME SET COL1 = VALUE1, COL1=VALUE2 WHERE COL3=VALUE3;
```

## 数据删除
```
DELETE * FROM _TABLE_NAME WHERE COL3=VALUE3;
```

## 数据查询

查询语句是SQL中最复杂也是用得最多的，必须要牢固的掌握：
```
SELECT COL1,COL2 FROM _TABLE_NAME WHERE CONDITION;
```

关于`CONDITION`，有很多：

+ `ORDER BY COL1 DESC`

+ `COL1 > VALUE1，COL1 < VALUE1, COL1 = VALUE1`

+ `COL1 BETWEEN VALUE1 AND VALUE2`和`COL1 NOT BETWEEN VALUE1 AND VALUE2`, 可用<>代替

+ `IN/NOT IN (VALUE1,VALUE2,VALUE3)` 可用于连接两个查询：`SELECT * FORM _TABLE_NAME WHERE ID IN (SELECT user_id FROM WHERE _TABLE_NAME2 WHERE age > 20)`

+ `EXISTS (SELECT ...)`exists做为where 条件时，是先对where 前的主查询询进行查询，然后用主查询的结果一个一个的代入exists的查询进行判断，如果为真则输出当前这一条主查询的结果

### 四种连接

当两个表联合查询时，会使用连接进行查询。如有以下两个表：
```
CREATE TABLE goods (id int, name varchar(55));

CREATE TABLE orders (id int, good_id int, num int);
```

#### 内连接

像=，>,<这样的比较运算符，**内连接使用比较运算符根据每个表共有的列的值匹配两个表中的行**
```
SELECT goods.*,orders.* FROM goods inner join orders on goods.id = orders.goods_id;
```

这样在两个表中都出现的才有。

#### 左连接

结果集包含左表中的所有行，至于右表中的行，只有在左表中有对应匹配的行，才会出现在结果集。对于左表中的一行，如果右表没有匹配则以空值代替。
```
SELECT goods.*, orders.* FROM goods LEFT JOIN orders ON goods.id = orders.goods_id;
```

#### 右连接

和左连接相反，以右表为基础，左表去匹配
```
SELECT goods.*, orders.* FROM goods RIGHT JOIN orders ON goods.id = orders.goods_id;
```

#### 全连接
相当于左连接和右连接的并集，左表和右表的所有行都会出现，无法匹配的填空值
```
SELECT goods.*, orders.* FROM goods FULL JOIN orders ON goods.id = orders.goods_id;
```

#### SQL查询的基本原理：两种情况介绍。
+ 单表查询：根据WHERE条件过滤表中的记录，形成中间表（这个中间表对用户是不可见的）；然后根据SELECT的选择列选择相应的列进行返回最终结果。

+ 两表连接查询：对两表求积（笛卡尔积）并用ON条件和连接连接类型进行过滤形成中间表；然后根据WHERE条件过滤中间表的记录，并根据SELECT指定的列返回查询结果。

+ 多表连接查询：先对第一个和第二个表按照两表连接做查询，然后用查询结果和第三个表做连接查询，以此类推，直到所有的表都连接上为止，最终形成一个中间的结果表，然后根据WHERE条件过滤中间表的记录，并根据SELECT指定的列返回查询结果。
理解SQL查询的过程是进行SQL优化的理论依据。

### GROUP BY

根据给定数据列的每个成员对查询结果进行分组统计，最终得到一个分组汇总表。SELECT子句中的列名必须为分组列或列函数。列函数对于GROUP BY子句定义的每个组各返回一个结果。

假设有个ORDERS表,其中包括订单的自增id，商品的id，商品的销售额， 时间
```
CREATE TABLE orders (id int, goods_ids int, amount int, time timestamp);
```

现在我想在所有订单中统计每种商品的总销售额：
```
SELECT goods_id, SUM(amount) as total_amount FROM orders GROUP BY goods_ids
```

也可以和WHERE合用，过滤数据
```
SELECT goods_id, SUM(amount) as total_amount FROM orders WHERE time > '2017-09-09' GROUP BY goods_ids
```

和`HAVING`连用，求销售额大于10000的商品
```
SELECT goods_id, SUM(amount) as total_amount FROM orders WHERE time > '2017-09-09' GROUP BY goods_ids HAVING SUM(amount) > 10000
```

### HAVING和WHERE的区别
`Select city FROM weather WHERE temp_lo = (SELECT max(temp_lo) FROM weather);` 作用的对象不同。WHERE 子句作用于表和视图，HAVING 子句作用于组。 WHERE 在分组和聚集计算之前选取输入行（因此，它控制哪些行进入聚集计算）， 而 HAVING 在分组和聚集之后选取分组的行。因此，WHERE 子句不能包含聚集函数； 因为试图用聚集函数判断那些行输入给聚集运算是没有意义的。 相反，HAVING 子句总是包含聚集函数








