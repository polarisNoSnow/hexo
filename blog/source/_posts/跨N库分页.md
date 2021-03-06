---
title: 跨N库分页
date: 2017-07-07 20:01:00
tags:
- 数据库
categories:
- 技术
---

# 跨N库分页
样例：如果分为ABC三库，每页显示20条数据，数据量足够多。
## 实现方案
### 方法一：全局视野法
如果取第一页，则三个库都取前20条，共60条。然后排序取前20条。（因为第一页就在这60条数据里）
如果取第二页，则三个库都取前40条，共120条。然后排序取第21~40条。（因为第二页就在这120条数据里）
依次类推。。。。。
缺点：越往后需要的数据越多，当你取N页的时候，其实已经把N页以前的数据也获取了过来。

### 方法二：业务折衷法-禁止跳页查询
不允许跳往指定页，只有首页上一页下一页末页（目前公司大数据做法而且因为统计总记录数很慢，导致也没有总页数）
第一次进入是首页，做法和全局视野类似，这时候会记录第20条数的流水号
然后点击下一页，ABC三库获取记录的那条流水之后的20条数据，共60条，然后排序取前20条。
上一页，三库都获取流水之前的20条数据（不足20条，全获取），然后排序取前20条。
依次类推。。。。。。。
缺点：无法跳转指定的页数
设想：前几页可以任意跳转，后几页可以任意跳转（逆排序），当第五页之后采用上下页的方式。
首页 2 3 4 5.... 末页，
首页 ....47 48 49 末页，
首页....11 12 13.......末页
前后几页用全局视野法，中间用禁止跳页查询方法。

### 方法三：业务折衷法-允许模糊数据
假设ABC库数据均匀，获取第N+1页的数据的时候，三库偏移20N/3条，各自获取20/3数据。（可以改为都获取20条共60条，然后排序取前20条）
缺点：数据偏差，不精确。


### 方法四：二次查询法
如跳转第N+1页时
第一次查询：三库偏移20N/3,各自获取20条，找出所有的最小值min,和每个库的最大值max_i(1,2,3)
第二次查询: 三库查询between (min, max_i)之间的数据，计算min在每个库的偏移量，然后相加等于min在全库的偏移量，将第二次查询的数据整合排序，已经知道第一条数据min的全局偏移量，那么偏移量20N所在的位置及之后的20条数据。

## 方案落实

### 方法一：全局视野法

（1）将order by time offset X limit Y，改写成order by time offset 0 limit X+Y

（2）服务层对得到的N*(X+Y)条数据进行内存排序，内存排序后再取偏移量X后的Y条记录

这种方法随着翻页的进行，性能越来越低。


### 方法二：业务折衷法-禁止跳页查询

（1）用正常的方法取得第一页数据，并得到第一页记录的time_max

（2）每次翻页，将order by time offset X limit Y，改写成order by time where time>$time_max limit Y

以保证每次只返回一页数据，性能为常量。


### 方法三：业务折衷法-允许模糊数据

（1）将order by time offset X limit Y，改写成order by time offset X/N limit Y/N


### 方法四：二次查询法

（1）将order by time offset X limit Y，改写成order by time offset X/N limit Y

（2）找到最小值time_min

（3）between二次查询，order by time between $time_min and $time_i_max

（4）设置虚拟time_min，找到time_min在各个分库的offset，从而得到time_min在全局的offset

（5）得到了time_min在全局的offset，自然得到了全局的offset X limit Y
