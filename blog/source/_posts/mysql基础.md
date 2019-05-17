---
title: mysql基础
date: 2017-07-10 10:52:00
tags:
- mysql
- 数据库
categories:
- 技术
---

# 前言
EQUI JOIN: join、outer join 
SEMI JOIN: from a,b

# 索引
- 存储过程至批量数据导入

```mysql
create PROCEDURE bigData_test(IN num int,IN begin_num int)
BEGIN
	 DECLARE i INT DEFAULT  0;
	 DECLARE y INT DEFAULT  begin_num;
	 WHILE i < num DO
			INSERT INTO bs_user VALUES (y,concat(y,'.jpg'),concat('tangyb',y),concat('tangyb',y),concat('tangyb',y),NULL,NULL,'00');
			SET i = i + 1;
			SET y = y + 1;
	 END WHILE;
END
```



- 执行存储函数
```mysql
START TRANSACTION;
call bigData_test(100000,1); //调用存储过程（手动开启事务，否则每次insert都会commit，会导致执行速度慢到令人发指）
commit;
```

- 新建并查询索引（自己测试的时候先新增数据在建立索引，否则在添加数据的时候等候时间太长）
```mysql
ALTER TABLE bs_user ADD INDEX index_uname (u_name); //给bs_user添加一个名为index_uname的索引
show index from bs_user; /*查询bs_user表中的索引
```

**小知识**

```mysql
show global variables like '%query_cache%'; //查询query_cache 是否开启（走索引第一次会很慢，第二次会很快）
```
```mysql
show variables like '%storage_engine%'; //表引擎使用innodb.第一次查询也会走数据文件，第二次直接走buffer_pool,也比直接查询数据文件要快
```
原理分析：http://blog.jobbole.com/24006/

# 分表、分库、分区
在大数据的基础上需要考虑这些数据主要是读还是更新（根据不同的操作也可以选择不同特征的数据库，冷热数据分离）。
## 分表
- 垂直-按照字段（如：文章的内容常常用于查询，访问量常常更新）
- 水平-保持表的结构相同，只是把数据放到不同的表中（user表：user1,user2），根据uid段来区分1~1000000放到user1，1000001~2000000放到user2等等
每张表都需要带上，MYD数据文件，.MYI索引文件，.frm表结构文件

## 分区
将一张表的数据分为N个区块，可以放置在相同或是不同的磁盘上，散列在不同的位置。操作的同一张表名，由数据库自己选择分区。

## 分库
当一台服务器的磁盘IO遇到瓶颈或是磁盘剩余空间过小等等，可以采用分库到不同服务器数据库。

简单介绍：http://www.cnblogs.com/langtianya/p/4997768.html	
存在问题：http://wentao365.iteye.com/blog/1740874

# 引擎
mysql中 myisam 引擎不支持事务的概念，多用于数据仓库这样查询多而事务少的情况，速度较快。

mysql中 innoDB 引擎支持事务的概念，多用于web网站后台等实时的中小型事务处理后台。

一种是表锁定（myisam存储引擎），一个是行锁定（innodb存储引擎）。

# 小结
## mysql大约执行流程
1. 接收到sql; 
2. 把sql放到排队队列中;
3. 执行sql; 
4. 返回执行结果。

##  注意点
1. 做mysql集群，用调度算法选择数据库（但是每张表的数据还是那么多，只是改变了连接队列方面的效率，而且耗硬件）。

2. 预计会出现大数据量并访问频繁，按照user1,user2的方式分表（缩短每张表的数据，但是前期如果没有规划好，后期就需要修改大量的sql）。

3. merge存储引擎来实现分表（用一个总表allUser,然后做user1和User2）,具体http://www.cnblogs.com/miketwais/articles/mysql_partition.html。

4. 存储引擎的使用不同，冷数据使用MyIsam 可以有更好的查询数据。活跃数据，可以使用Innodb ,可以有更好的更新速度。

5. 对冷数据进行更多的从库配置，因为更多的操作是查询，这样来加快查询速度。对热数据，可以相对有更多的主库的横向分表处理。

6. 对于一些特殊的活跃数据，也可以考虑使用memcache ,redis之类的缓存，等累计到一定量再去更新数据库.

## 提升检索速度
1. like '%tangyb' 这种前面模糊匹配会严重影响检索速度
2. limit 的offset 是取offset+N行 并不是从offset开始,所以offset特别大的时候 影响效率
3. count(*)会统计所有行 ，count(1)必须确保第一列不为null，否则不会统计，所以mysql推荐使用count（*）
4. 禁止使用外键（不适合分布式、高并发）和存储过程（不利于调试、扩展、移植）
5. 最好避免in的使用，最好在1000以内
6. mybatis多用resultMap,禁止返回resultClass，减少耦合，方便维护
7. 表的设计 包括 id、gmt_creat、gmt_modified
