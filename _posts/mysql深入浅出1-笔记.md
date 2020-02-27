---
title: mysql深入浅出 笔记
date: 2019-06-04 09:49:28
tags:
categories: 读书笔记
---

# mysql基本架构
MySQL的逻辑架构：Server层和引擎层。
Server层：连接器+分析器(词法/语法分析)+优化器(索引选择)+执行器(操作引擎，返回结果) + 查询缓存(分析时命中则直接返回)
引擎层：

```
//链接数据库
mysql -h$ip -P$port -u$user -p  
//显示数据库连接情况
show processlist

```

一个链接长时间不用默认8小时断开，参数wait_timeout控制；通过链接对象管理内存，用久内存占用大。
 mysql_reset_connection 重新初始化链接。

查询缓存，key-value，key是sql，value是结果；在更新数据后清空，一般不建议用(参数query_cache_type)，在
MySQL8.0中没有缓存。

<!--more-->

# 日志系统
binlog（归档日志）是Server层的，所有引擎都可用; 追加写, 文件写到一定大小切换下一个; 逻辑日志记录原始语句;主要用于备份

redo log（重做日志）InnoDB引擎特有的日志; 循环写, 固定空间; 物理日志记录在某个数据页上的修改;主要用户crash恢复

redo log 为了支持crash-safe, WAL技术, Write-Ahead Logging, 关键点是先写日志再写磁盘, 还没写磁盘查询怎么办？

**redo log 二阶段提交, 包含prepare/commit**, 保证两个日志都写完整，可以恢复到任意时刻
crash恢复 和 扩容(备份+binlog)，保持逻辑上一致

innodb_flush_log_at_trx_commit = 1 , sync_binlog = 1 每次事务日志都持久化到磁盘

Binlog有两种模式, statement 格式的话是记sql语句， row格式会记录行的内容，记两条，更新前和更新后都有.

```
// 数据库
1 写redo log, 处于prepare状态 => 2 写binlog => 3 commit提交事务, redo log处于commit状态

不管在什么时候崩溃，binlog写成后事务算成功，就提交事务; 否则就回滚。
binlog是后悔，redo log是救命。

```

# 事务隔离
ACID（Atomicity, Consistency, Isolation, Durability）

**多个事务同时执行问题：**
脏读，读取了其他事务未提交的数据
不可重复读，针对其他提交，**读取数据本身的对比**；同一条记录多次读取值不一样，其他事务update
幻读，针对其他提交，**读取数据条数的对比**；多次查询结果集不一样，其他事务insert

**SQL标准的事务隔离级别：**
读未提交, 一个事务还没提交时, 变更就能不其他事务看到
读提交, 一个事务提交后, 变更才会被其他事务看到
可重复读, 一个事务在执行过程中看到的数据, 总是跟这个事务启动时看到的是一致的, 未提交事务对其他事务不可见; 一个事务在启动的时候, 看到所有已提交的事务结果, 之后在事务期间, 其他事务更新不可见。
串行化, 顺序执行

```
mysql> show variables like 'transaction_isolation';

+-----------------------+----------------+

| Variable_name | Value |

+-----------------------+----------------+

| transaction_isolation | READ-COMMITTED |

+-----------------------+----------------+
```

**事务隔离实现:**
通过__一致性视图__和回滚日志，同一条记录在系统中可以有多个版本(就是数据库多版本并发控制MVCC)

**事务的启动：**

```
1) begin或start transaction(在真正操作表时才启动), commit, rollback
2) set autocommint=0 (默认1)，不会主动提交，执行commit/rollback或断开连接
3) start transaction with consist snapshot(InnoDB)
```



# 索引

## 索引模型
__哈希表__, 适用于等值查询
__有序数组__, 有序数组索引只适用于静态存储引擎, 例如等值查询和范围查询
__搜索树__, 较少磁盘访问次数

主键和索引区别, 每个主键都会维护一个索引, 索引字段单独维护一个表

__索引类型__
1）主键索引(聚簇索引)，**值存的是整行内容**
2）非主键索引(二级索引)，**值存的是主键内容**

```
//drop主键索引会导致其他索引失效，但drop普通索引不会
//删除主键还是创建主键，都会将整个表重建
//重建索引 k 的做法是合理的，可以达到省空间的目的
alter table T drop index k;
alter table T drop primary key;

alter table T add index(k);
alter table T add primary key(id);
```

基于主键索引查询，只需查询一次；基于非主键索引查询，先得到主键值，再去查询主键索引。



## InnoDb 索引模型

索引的实现由存储引擎来决定, 每一个索引在 InnoDB里面对应一棵B+树(N叉树), 没有主键的表,  
innodb会给默认创建一个Rowid做主键

```
mysql> create table T(
id int primary key, 
k int not null, 
name varchar(16),
index (k))engine=InnoDB;

// NOT NULL PRIMARY KEY AUTO_INCREMENT 自增主键
```

## 索引维护
B+树的插入可能会引起数据页的分裂，删除可能会引起数据页的合并，二者都是比较重的IO消耗， 
所以比较好的方式是顺序插入数据，这也是我们一般使用自增主键的原因之一

在Key-Value的场景下，只有一个索引且是唯一索引，则适合直接使用业务字段作为主键索引

如何避免长事务对业务的影响？
客户端：分析general_log；select不需要begin/commit；SET MAX_EXECUTION_TIME命令控制最长执行时间；
数据库：监控infomation_schema.Innodb_trx，设置长事务阀值，超过就报警或kill
				pt-kill （KILL MySQL指定查询) 
							

```shell
mysql>set global general_log_file='/tmp/general.lg';    #设置路径
mysql>set global general_log=on;    # 开启general log模式
mysql>set global general_log=off;   # 关闭general log模式

mysql>set global log_output='table' # 在general_log表中查看

mysql>show global variables like '%general%';  # 查看


```



## 覆盖索引
非主键索引上已经存储了主键值, 查询主键值不需要回表
应用场景：身份证号/姓名 联合索引, 通过身份证号查询姓名频率很高

最左前缀原则, 联合索引顺序, 最左的N的字段或字符串的M个字符
索引下推, 在搜索过程中对索引中包含字段优先判断, 减少回表次数，满足条件的再比较其他字段

重建索引和主键，省空间？

```c++
alter table T engine=InnoDB
```



# 全局锁和表锁
__全局锁__
命令: 其他线程数据库操作被阻塞 

```
mysql>Flush tables with read lock
```

业务场景：用于全库逻辑备份; 
mysqldump , InnoDB使用-single-tranction参数导数据前 
拿到一个一致性试图, 数据可以正常更新; MyISAM引擎不支持事务, 只能使用全局锁

set global readonly=true 不推荐使用, 通常用做主备判断, 客户端异常断开数据库保持readonly

__表级别的锁__

__表锁__

```c++
// read 本线程和其他线程只能读; write 本线程可读写，其他线程读写阻塞
lock/unlock tables ... read/write

// 其他线程，t1可读，t1写和t2读写阻塞
// 本线程只能t1可读，t2可读写
lock tables t1 read, t2 write ; 
```



__元数据锁__
MDL锁(metadata lock)，防止DDL和DML并发的冲突，事务提交时释放。
增删改查加MDL读锁，表结构变更操作时加MDL写锁；读写锁之间互斥，读锁之间不互斥。
DML语句（数据操作语言）Insert、Update、 Delete、Merge
DDL语句（数据定义语言）Create、Alter、 Drop、Truncate

如何安全的给表加个字段？

```c++
// 使用NOWAIT/WAIT n 不影响其他线程查询和更新操作
alter table name NOWAIT add column...
alter talbe name WAIT N add column...
```

哪个索引是多余的？

```sql
CREATE TABLE `geek` (
  `a` int(11) NOT NULL,
  `b` int(11) NOT NULL,
  `c` int(11) NOT NULL,
  `d` int(11) NOT NULL,
  PRIMARY KEY (`a`,`b`),
  KEY `c` (`c`),
  KEY `ca` (`c`,`a`),
  KEY `cb` (`c`,`b`)
) ENGINE=InnoDB;

InnoDB会把主键字段放到索引定义字段后面, 同时也会去重
依次是 abc,cab,cab,cba , 所以第三个ca主键索引是多余的
```



# 行锁
在InnoDB事务中, 行锁是在需要的时候加入, 事务结束时释放

__交叉死锁办法__:
innodb_lock_wait_timeout 超时设置，默认50s; innodb_deadlock_dectect 设置on, 回滚其中一个交叉的事务

备库-single-tracation备份时，主库binlog传来一个DDL语句会怎么样？

```sql
Q1:SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
Q2:START TRANSACTION  WITH CONSISTENT SNAPSHOT；
/* other tables */
Q3:SAVEPOINT sp;
/* 时刻 1 */
Q4:show create table `t1`;
/* 时刻 2 */
Q5:SELECT * FROM `t1`;
/* 时刻 3 */
Q6:ROLLBACK TO SAVEPOINT sp;
/* 时刻 4 */
/* other tables */
```



# 事务
__视图__ : view 虚拟表, 一致性视图(MVCC多版本并发控制协议) 用于支持读提交/可重复读隔离级别实现

InnoDB里面有一个唯一事务ID, 按申请顺序严格递增, 每个版本都有row trx_id
InnoDB为每个事务创建了一个数组, 记录事务启动瞬间活跃的事务ID

__可重复读__ 查询只承认事务启动前就已经提交的数据

__读提交__ 查询只承认在语句启动前就已经提交的数据 



# Percona Toolkit

```shell
[root@localhost download]# rpm -ivh percona-toolkit-3.1.0-2.el7.x86_64.rpm
warning: percona-toolkit-3.1.0-2.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 8507efa5: NOKEY
error: Failed dependencies:
        perl(IO::Socket::SSL) is needed by percona-toolkit-3.1.0-2.el7.x86_64
        perl(Digest::MD5) is needed by percona-toolkit-3.1.0-2.el7.x86_64

[root@localhost download]# rpm -ivh percona-toolkit-3.1.0-2.el7.x86_64.rpm
warning: percona-toolkit-3.1.0-2.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 8507efa5: NOKEY
error: Failed dependencies:
        perl(Digest::MD5) is needed by percona-toolkit-3.1.0-2.el7.x86_64
		
yum insatll perl-DBI
yum install perl-DBD-MySQL
yum install perl-IO-Socket-SSL
yum install perl-Digest-MD5

[root@localhost download]# rpm -ivh percona-toolkit-3.1.0-2.el7.x86_64.rpm
warning: percona-toolkit-3.1.0-2.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 8507efa5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:percona-toolkit-3.1.0-2.el7      ################################# [100%]
   
[root@localhost download]# pt-duplicate-key-checker --help
```





























