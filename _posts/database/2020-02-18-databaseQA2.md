---
title:  关于数据库的QA2
categories:
- SQL
---

### mysql的划分以及sql的执行过程

#### 分层

**server层**

1. 连接器
2. 查询缓存
3. 分析器
4. 优化器
5. 执行器

**存储引擎层**

![](../../../sources/img/database/mysql划分.png)

#### sql更新流程分析

```
update tb_student A set A.age='19' where A.name=' 张三 ';
```

1. 先查询到张三这一条数据，如果有缓存，也是会用到缓存。
2. 然后拿到查询的语句，把 age 改为 19，然后调用引擎 API 接口，写入这一行数据，InnoDB 引擎把数据保存在内存中，同时记录 redo log，此时 redo log 进入 prepare 状态，然后告诉执行器，执行完成了，随时可以提交。
3. 执行器收到通知后记录 binlog，然后调用引擎接口，提交 redo log 为提交状态。
4. 更新完成。

##### redolog和binlog的顺序原因

**先写 redo log 直接提交，然后写 binlog**

假设写完 redo log 后，机器挂了，binlog 日志没有被写入，那么机器重启后，这台机器会通过 redo log 恢复数据，但是这个时候 bingog 并没有记录该数据，后续进行机器备份的时候，就会丢失这一条数据，同时主从同步也会丢失这一条数据。**先写 binlog，然后写 redo log**

假设写完了 binlog，机器异常重启了，由于没有 redo log，本机是无法恢复这一条记录的，但是 binlog 又有记录，那么和上面同样的道理，就会产生数据不一致的情况。

如果采用 redo log 两阶段提交的方式，写完 binglog 后，然后再提交 redo log 就会防止出现上述的问题，从而保证了数据的一致性。

**假设 redo log 处于预提交状态，binglog 也已经写完**了，这个时候发生了异常重启会怎么样？

Mysql会自己去处理：

1.判断 redo log 是否完整，如果判断是完整的，就立即提交。

2.如果 redo log 只是预提交但不是 commit 状态，这个时候就会去判断 binlog 是否完整，如果完整就提交 redo log, 不完整就回滚事务。