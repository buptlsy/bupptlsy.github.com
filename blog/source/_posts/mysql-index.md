---
title: mysql索引那些事儿
categories: mysql
tags: msql

---
在创建数据库表时，需要考虑到很多的因素。不仅仅是每个字段的构成，还有主键。其实主键就是唯一索引的一种。因为mysql有多种存储引擎，每种引擎用的索引类型也不一样。
<!--more-->
## 索引的基本操作

    #查看表所用的存储引擎
    show table status from database where name='table' \G;
    #更改数据库表所用的引擎
    alter table 表名 engine=myisam;
    #创建多列索引
    alter table 表名 add index 索引名(col1, col2);
    #创建唯一索引
    alter table 表名 add unique 索引名(col1, col2);
    #创建主键索引
    alter table 表名 add primary key 索引名(col1);
    #查看一条查询语句所用到的索引情况
    explain 查询语句

## 索引的分类

### b树索引（innodb引擎）

myisam引擎和innodb引擎是用b+树实现的索引。b+树的叶子节点存储具体的行信息。叶子节点同时构成了一个链表。因为b+       树的一些特性，所以此索引也具有很多优点：查询效率高（O(h),h为树高))，范围查询效率高。

#### myisam引擎和innodb引擎的区别

myisam引擎和innodb引擎在一些地方存在差异（分别从主键索引和二级索引方面考虑）：
主键索引：myisam引擎的数据文件和索引文件是分开的。而innodb引擎的数据和索引是放在一起的。
myisam引擎索引：
![myisam引擎索引](https://raw.githubusercontent.com/buptlsy/images/master/mysql_index_my_primarykey.jpg)
innodb引擎索引：
![innodb引擎索引](https://raw.githubusercontent.com/buptlsy/images/master/mysql_index_innodb_primarykey.jpg)
二级索引：myisam引擎的二级索引的data部分存放的是行指针，而innodb引擎存放的是主键。
myisam引擎索引：
![myisam引擎索引](https://raw.githubusercontent.com/buptlsy/images/master/mysql_index_myisam_secondarykey.jpg)
innodb引擎索引：
![innodb引擎索引](https://raw.githubusercontent.com/buptlsy/images/master/mysql_index_innodb_secondarykey.jpg)

#### b+树索引生效的地方

举个例子（以myisam引擎为例）：假设表student主键为id, 多列索引(A, B)。分析以下几个例子，是否用到了索引(最左前缀原则和隔离列)：
1. select * from student where A=xxx and B like '%xxx%';
2. select * from student where A=xxx;
3. select * from student where A=xxx and id=xx;
4. select * from student where B=xxx;
5. select * from student where A=xxx order by id asc;
6. select * from student where A=xxx and B=xxx order by id asc;
7. select A, B from student where id=xxx;
8. select * from student where A like '%xx%' and B=xxx;
9. select * from student where A like 'xxx%' and B=xxx;
10. select * from student where A+1=10;

#### b+树索引的优势

innodb引擎是用b+树实现的索引。b+树的叶子节点存储具体的行信息。叶子节点同时构成了一个链表。因为b+树的一些特性，所以此索引也具有很多优点：查询效率高（O(h),h为树高))，范围查询效率高。

### 哈希索引 （memory）



## 全文索引（暂时没有研究）

### 唯一索引

唯一索引保证任意两行的索引都不相同。

### 聚簇索引

行的物理位置和索引的顺序一致。

###  

## 索引的作用

## 引用[reference]
http://database.51cto.com/art/201504/473322_all.htm
