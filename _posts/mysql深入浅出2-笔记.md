---
title: mysql深入浅出2 笔记
date: 2020-01-19 11:37:26
tags:
categories: 读书笔记
---

# 普通索引和唯一索引, 怎么选择

__查询__
select id from T where k=5
普通索引找到(5,100)后, 继续找直到遇到第一个不满足k=5; 唯一索引找到了就返回. 性能差异可忽略
因为InnoDB按照数据页为单位读写, 默认一个页大小16KB, 一般只要多一次计算就可。

__更新__
change buffer 如果数据不在内存中, 更新操作缓存, 等下次查询时再更新(merge), 适用于读少写多; 在内存中有拷贝, 会被写到磁盘上;
目的是为了减少磁盘读写次数, 通过innodb_change_buffer_max_size动态设置, 50表示最大占buffer pool 50%

只适用于普通索引, 因为唯一索引数据会读入内存要判断唯一性, 不会存在change buffer.

# 为什么会选错索引

```
set long_query_time=0;
select * from t where a between 10000 and 20000; /*Q1*/
select * from t force index(a) where a between 10000 and 20000;/*Q2*/
```

不断删除历史数据, 导致全表扫描, 优化器选错了索引

```
explain select * from t

show index from t

analyze table t

```

查看预计扫描行数, 查看索引基数, 重新统计索引信息

# 怎么给字符串加索引

```
mysql> select 
  count(distinct left(email,4)）as L4,
  count(distinct left(email,5)）as L5,
  count(distinct left(email,6)）as L6,
  count(distinct left(email,7)）as L7,
from SUser;

mysql> alter table SUser add index index1(email);
或指定前缀长度
mysql> alter table SUser add index index2(email(6));

```

没索引匹配时全表扫描, 统计后选择合适的前缀索引, 使用后就不会再有索引覆盖

1 直接创建索引, 会比较占空间
2 创建索引前缀, 节省空间, 但会增加扫描次数(区分度不高), 并且不能使用覆盖索引; 
3 倒序存储, 再创建前缀索引(省份证后6位), 饶过字符串本身区分度不够问题
4 hash索引, 使用crc32()函数定义校验码, 查询性能稳定, 有额外的存储和计算消耗, 和3一样不支持范围扫描


# 为什么mysql会抖一下

__影响性能场景__
redo log 写满了, 刷脏页, 这个时候更新是被阻塞的
内存不够用, 要先将脏也写入磁盘, 淘汰脏页

__InnoDB刷脏页控制策略__
需要测试磁盘读写能力, 再设置innodb_io_capacity参数

```
 fio -filename=$filename -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size=500M -numjobs=10 -runtime=10 -group_reporting -name=mytest 

```

刷脏页策略, 关注脏页比例, 尽量不要接口75%

```
mysql> select VARIABLE_VALUE into @a from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty';
select VARIABLE_VALUE into @b from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total';
select @a/@b;

```

innodb_flush_neighbors 参数控制是否只刷当前脏页, 0表示不带上邻居; SSD一般设置成0

# 为什么表数据删掉一半, 表文件大小没有变

数据表空间回收, 设计表定义和数据
innodb_file_per_table, on 表示InnoDB表数据存储在一个.idb为后缀的文件中, off 放在系统表共享空间

__数据删除__
delete 只是把数据页标记可复用, 磁盘文件大小不会变; 
增删操作量大的表, 会存在空洞, 重建表可以收缩表空间

__重建表__
alter table A engine=InnoDB

Online DDL
新建临时表=> 扫描表A数据=> 生成B+树,存储到临时文件=> 记录更新操作row log=> 更新数据, 替换表A的数据文件

optimize table t 等于 recreate+analyze


# count(*) 为什么这么慢

MyISAM 是直接返回, InnoDB 会遍历全表

用缓存来保存计数, 或者在mysql里先更新再插入

效率count(*) ~= count(1) > count(主键ID) >　count(字段)

# order by 是怎么工作的