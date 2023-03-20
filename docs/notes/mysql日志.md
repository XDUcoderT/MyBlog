---
title: mysql日志
permalink: /post/mysql-log-z2diocn.html
date: 2023-01-09 13:30:34
meta:
  - name: keywords
    content: ''
  - name: description
    content: >-
      mysql日志本文转载了竹子爱熊猫的文章https_juejincnpost一undolog回滚日志当一条写入类型的sql执行时候都会记录undolog日志我们在执行执行一条“增删改”语句的时候虽然没有输入begin开启事务和commit提交事务但是mysql会隐式开启事务来执行“增删改”语句undo就是撤销的意思所以我们也叫他回滚日志用来撤销sql操作的。当一条写入类型的sql执行时都会记录undolog日志会生成相反的sql放入到undolog中例如_如果目前是insert插入操作则生成一个对应的de
tags: []
categories: []
author:
  name: XDUcoderT
  link: https://github.com/XDUcoderT
---

# mysql日志

> 本文转载了竹子爱熊猫的文章https://juejin.cn/post/7157956679932313608

# 一、Undo-log回滚日志

当一条写入类型的SQL执行时候，都会记录Undo-log日志

我们在执行执行一条“增删改”语句的时候，虽然没有输入 begin 开启事务和 commit 提交事务，但是 MySQL 会**隐式开启事务**来执行“增删改”语句

Undo就是撤销的意思，所以我们也叫他回滚日志，**用来撤销sql操作**的。

当一条写入类型的sql执行时，都会记录Undo-log日志，会生成相反的sql放入到undo-log中，例如：

* 如果目前是insert插入操作，则生成一个对应的delete操作。
* 如果目前是delete删除操作，InnoDB会修改隐藏字段deleted_bit=1，则生成改为0的语句
* 如果目前的update修改操作，比如将姓名从aa改成bb，那就生成一个从bb改为aa的操作

当事务中某条`SQL`​执行失败时，`MySQL`​就需要回滚事务中其他执行成功的`SQL`​，此时就会找到这个事务在`Undo-log`​中生成的反`SQL`​，然后将库中的数据改回事务发生前的样子。

but!!

**实际上不会生成反sql**，以上叙述是方便理解，**实际会将Undo-log存储在xx.ibdata共享表数据文件**当中，默认采用**段的形式存储**。

也就是当一个事务尝试**写某行数据时**，**首先会将数据拷贝到xx.ibdata文件中**，将表中行数据的隐藏字段:**roll_ptr回滚指针会指向xx.ibdata文件中的旧数据**，然后再写表上的数据。

在共享表数据文件中，有一块区域名为**`Rollback Segment`**​**回滚段**，每个回滚段中有`1024`​​个`Undo-log Segment`​​，每个`Undo`​​段可存储一条旧数据，而执行写`SQL`​​时，`Undo-log`​​就是写入到这些段中  

## 真正的事务回滚原理

先拷贝，后修改，回滚时覆盖

实际上当一个事务需要回滚时，本质上并不会以执行反sql的模式还原数据，而是直接将roll_ptr回滚指针指向的Undo记录，**从xx.ibdata共享表数据文件拷贝到xx.ibd表数据文件，覆盖掉原本修改过的数据。**

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130151334-1lcbw7d.png)​

一条写sql执行的流程如上图所示，需要回滚事务时，直接用Undo旧纪录覆盖表中修改过的新纪录即可

> 如果是`insert`​​操作，由于插入之前这条数据都不存在，**那么就不会产生**​**`Undo`**​**记录，此时回滚时如何删除这条记录呢？因为插入操作不会产生**​**`Undo`**​**旧记录，因此隐藏字段中的**​**`roll_ptr=null`**​**，因此直接用**​**`null`**​**覆盖插入的新记录即可**，这样也就实现了删除数据的效果

## 基于Undo版本链实现MVCC

这里顺便复习以下MVCC知识

Undo-log中的记录的旧数据并不仅仅只有一条，一条相同的行数据可能存在多条不同版本的Undo记录，内部会通过roll_ptr回滚指针，组成一个单向链表，而这个链表则被称为Undo版本链。

```c
-- 事务T1：trx_id=1（两次修改同一条数据）
UPDATE `zz_users` SET user_name = "竹子" WHERE user_id = 1;
UPDATE `zz_users` SET user_sex = "男" WHERE user_id = 1;
```

​`Undo-log`​中的旧数据版本链示意图大致如下

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130152210-g3vggqy.png)

一条记录的每一次更新操作产生的 undo log 格式都有一个 roll_pointer 指针和一个 trx_id 事务id：

* 通过 trx_id 可以知道该记录是被哪个事务修改的；
* 通过 roll_pointer 指针可以将这些 undo log 串成一个链表，这个链表就被称为版本链；

## Undo-log的内存缓冲区

InnoDB在mysql启动的时候，会在内存中创建一个bufferPool，而这个缓冲池主要存放两类东西，一类是**数据相关的缓冲**，如索引、表、表数据等，另一类则是**各种日志的缓冲**，如Undo、Bin、Redo...等日志。

而当一条写SQL执行时，**不会直接去往磁盘中的xx.ibdata文件写数据**，**而是会写在undo_log_buffer缓冲区中，因为工作线程直接去写磁盘太影响效率了，写进缓冲区后会由后台线程去刷写磁盘。**

如果当**一个事务提交时，**​**`Undo`**​**的旧记录不会立马被删除，**​**`InnoDB`**​**中会有专门的**​**`purger`**​**线程负责**，`purger`​​​线程内部会维护一个`ReadView`​​​，它会以此作为判断依据，来决定何时移除`Undo`​​​记录。为什么不是事务提交后立马删除`Undo`​​​记录呢？因为可能会有其他事务在通过快照，读`Undo`​​​版本链中的旧数据，直接移除可能会导致其他事务读不到数据，因此删除的工作就交给了`purger`​​​线程。

## Undo-log的作用

* **实现事务回滚，保障事务的原子性**
* **实现MVCC(多版本并发控制)关键因素之一**

# 二、Redo-log重做日志

## 概述

Redo-log主要用来**实现数据的恢复**。

Mysql绝大部分引擎都是基于磁盘存储数据的，但若每次读写数据都走磁盘，其效率必然十分低下。因此InnoDB引擎在设计时，**当mysql启动后就会在内存中创建一个BufferPool**，运行过程中会将大量操作集中在内存中进行，比如**写入数据时，先写到内存中，然后由后台线程再刷写到磁盘。**

​`MySQL`​ 中数据是以页为单位，你查询一条记录，会从硬盘把一页的数据加载出来，加载出来的数据叫数据页，会放入到 `Buffer Pool`​ 中。

后续的查询都是先从 `Buffer Pool`​ 中找，没有命中再去硬盘加载，减少硬盘 `IO`​ 开销，提升性能。

更新表数据的时候，也是如此，发现 `Buffer Pool`​​ 里存在要更新的数据，就**直接在 ​**​**`Buffer Pool`**​**​ 里更新。**

然后会把“**在某个数据页上做了什么修改”记录到重做日志缓存**（`redo log buffer`​​​）里，接着刷盘到 `redo log`​​​ 文件里。

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230121191644-ugcil68.png)​

虽然使用了BufferPool提升了mysql整体的读写性能，**但是它是基于内存的，也就意味着随着服务器的宕机、重启会有数据丢失的风险，Redo-log就用来解决这个问题。**

​**`Redo-log`**​**是一种预写式日志**，即在**向内存写入数据前，会先写日志**，当后续数据未被刷写到磁盘、`MySQL`​崩溃时，就可以通过日志来恢复数据，确保所有提交的事务都会被持久化。这就是 **WAL （Write-Ahead Logging）技术**。**WAL 技术指的是， MySQL 的写操作并不是立刻写到磁盘上，而是先写日志，然后在合适的时间再写到磁盘上**。

## 执行过程

1. 执行sql
2. 将数据页加载到buffer pool
3. 在bufferpool中更新数据页
4. 将redolog写入磁盘

但是要注意：工作线程执行`SQL`​​​前，写的`Redo-log`​​​日志，**也是写在了内存中的**​**`redo_log_buffer`**​**缓冲区**。既然`Redo-log`​​​日志也是先写内存，那`Redo-log`​​​有没有丢失的风险呢？这跟`Redo-log`​​​的刷盘策略有关。

## Redo log 和 Undo log区别在哪

这两种日志是属于InnoDB存储引擎的日志

* redo log记录了此次事务完成后的数据状态，记录的是更新之后的值
* undo log记录了此次事务开始前的数据状态，记录的是更新之前的值

事务提交之前发生了崩溃，重启后会通过undo log回滚事务，事务提交之后发生了崩溃，重启后会通过redo log恢复事务

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130155238-5k7kq0q.png)​

所以有了 redo log，再通过 WAL 技术，InnoDB 就可以保证即使数据库发生异常重启，之前已提交的记录都不会丢失，这个能力称为 **crash-safe**（崩溃恢复）。可以看出来， **redo log 保证了事务四大特性中的持久性**。

再次复习一下：

undo-log保证了事务的原子性(只要事务还没提交发生崩溃，我就给你回滚)

redo-log保证了事务的持久性(只要事务提交，我就保证能落盘)

## Redo log落盘策略

### redo log缓存

执行一个事务的过程中，产生的 redo log 也不是直接写入磁盘的，因为这样会产生大量的 I/O 操作，而且磁盘的运行速度远慢于内存。

所以，redo log 也有自己的缓存—— ​**redo log buffer**​，每当产生一条 redo log 时，会先写入到 redo log buffer，后续在持久化到磁盘如下图

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130160543-0orf2gc.png)​

### redo log刷盘时机

主要有下面几个时机

1. Mysql正常关闭时
2. 当redo log buffer中写入量大于redo log buffer内存空间的一半时，会触发落盘
3. InnoDB的后台线程每隔1秒，将redo log buffer持久化到磁盘
4. 每次事务提交时都将缓存在redo log buffer里的redo log直接持久化到磁盘（这个策略可由 innodb_flush_log_at_trx_commit 参数控制，下面会说）。

### 刷盘参数

单独执行一个更新语句的时候，InnoDB 引擎会自己启动一个事务，在执行更新语句的过程中，生成的 redo log 先写入到 redo log buffer 中，然后等事务提交的时候，再将缓存在 redo log buffer 中的 redo log 按组的方式「顺序写」到磁盘。

上面这种 redo log 刷盘时机是在事务提交的时候，这个默认的行为。

除此之外，InnoDB 还提供了另外两种策略，由参数 `innodb_flush_log_at_trx_commit`​ 参数控制，可取的值有：0、1、2，默认值为 1，这三个值分别代表的策略如下：

* 当设置该​**参数为 0 时**​，表示每次事务提交时 ，还是**将 redo log 留在 redo log buffer 中** ，该模式下在事务提交时不会主动触发写入磁盘的操作。
* 当设置该​**参数为 1 时**​，表示每次事务提交时，都​**将缓存在 redo log buffer 里的 redo log 直接持久化到磁盘**​，这样可以保证 MySQL 异常重启之后数据不会丢失。
* 当设置该**参数为 2 时**，表示每次事务提交时，都只是缓存在 redo log buffer 里的 redo log **写到 redo log 文件，注意写入到「 redo log 文件」并不意味着写入到了磁盘**，Page Cache 是专门用来缓存文件数据的，所以写入「 redo log文件」意味着写入到了操作系统的文件缓存。

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130161043-876jfot.png)​

InnoDB 的后台线程每隔 1 秒：

* 针对参数 0 ：会把缓存在 redo log buffer 中的 redo log ，通过调用 `write()`​ 写到操作系统的 Page Cache，然后调用 `fsync()`​ 持久化到磁盘。​**所以参数为 0 的策略，MySQL 进程的崩溃会导致上一秒钟所有事务数据的丢失**​;
* 针对参数 2 ：调用 fsync，将缓存在操作系统中 Page Cache 里的 redo log 持久化到磁盘。​**所以参数为 2 的策略，较取值为 0 情况下更安全，因为 MySQL 进程的崩溃并不会丢失数据，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失**​。

加入了后台线程后，innodb_flush_log_at_trx_commit 的刷盘时机如下图

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130161210-caaig7w.png)

​

## Redo log 写满了怎么办

默认情况下， InnoDB 存储引擎有 1 个重做日志文件组( redo log Group），「重做日志文件组」由有 2 个 redo log 文件组成，这两个 redo 日志的文件名叫 ：`ib_logfile0`​ 和 `ib_logfile1`​ 。

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130162209-vp61eo6.png)​

在重做日志组中，每个redo log File的大小是固定且一致的，假设每个redo log File设置的上限是1GB 那么总共就可以记录2GB的操作

重做日志文件组是以循环写的方式工作的，从头开始写，写到末尾就又回到开头，相当于一个环形

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130162355-bietm1u.png)​

我们知道 redo log 是为了防止 Buffer Pool 中的脏页丢失而设计的，那么如果随着系统运行，Buffer Pool 的脏页刷新到了磁盘中，那么 redo log 对应的记录也就没用了，这时候我们擦除这些旧记录，以腾出空间记录新的更新操作。

redo log 是循环写的方式，相当于一个环形，InnoDB 用 write pos 表示 redo log 当前记录写到的位置，用 checkpoint 表示当前要擦除的位置，如下图：

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130162422-1c3y64c.png)

​

图中的：

* write pos 和 checkpoint 的移动都是顺时针方向；
* write pos ～ checkpoint 之间的部分（图中的红色部分），用来记录新的更新操作；
* check point ～ write pos 之间的部分（图中蓝色部分）：待落盘的脏数据页记录；

如果 write pos 追上了 checkpoint，就意味着 ​**redo log 文件满了，这时 MySQL 不能再执行新的更新操作，也就是说 MySQL 会被阻塞**​（​*因此所以针对并发量大的系统，适当设置 redo log 的文件大小非常重要*​），此时​**会停下来将 Buffer Pool 中的脏页刷新到磁盘中，然后标记 redo log 哪些记录可以被擦除，接着对旧的 redo log 记录进行擦除，等擦除完旧记录腾出了空间，checkpoint 就会往后移动（图中顺时针）**​，然后 MySQL 恢复正常运行，继续执行新的更新操作。

所以，一次 checkpoint 的过程就是脏页刷新到磁盘中变成干净页，然后标记 redo log 哪些记录可以被覆盖的过程。

# 三、binlog

​`redo log`​它是物理日志，记录内容是“在某个数据页上做了什么修改”，属于`InnoDB`​物理引擎

而`binlog`​是逻辑日志，记录内容是语句的原始逻辑，类似于“给 ID=2 这一行的 c 字段加 1”属于`MySQL Server`​ 层。

不管用什么存储引擎，只要发生了表数据更新，都会产生 ​`binlog`​ 日志。

可以说`MySQL`​数据库的**数据备份、主备、主主、主从**都离不开`binlog`​，需要依靠`binlog`​来同步数据，保证数据一致性。

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230121193933-w2fz1f6.png)​

​`binlog`​​会记录所有涉及更新数据的逻辑操作，并且是顺序写。

## bin-log的缓冲区

和之前的两种日志相同，bin-log也由内存日志缓冲区 + 本地磁盘文件两部分组成，这也就意味着：写bin-log日志时，也会先写缓冲区，然后由后台线程去刷盘。

​`bin-log`​的缓冲区跟`redo-log、undo-log`​的缓冲区并不同，前面分析的两种日志缓冲区，都位于`InnoDB`​创建的共享`BufferPool`​中，而`bin_log_buffer`​是位于每条线程中的，关系图如下

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130163509-nl4o9el.png)​

也就是说，`MySQL-Server`​会给每一条工作线程，都分配一个`bin_log_buffer`​，而并不是放在共享缓冲区中，这是为啥呢？因为`MySQL`​设计时要兼容所有引擎，直接将`bin-log`​的缓冲区，设计在线程的工作内存中，这样就能够让所有引擎通用，并且不同线程/事务之间，由于写的都是自己工作内存中的`bin-log`​缓冲，因此并发执行时也不会冲突！

## 记录格式

binlog日志有三种格式，可以通过`binlog_format`​参数指定。

* statement
* row
* mixed

### statement

如果设置为statement，则记录的是sql语句全文，比如执行一条`update T set update_time=now() where id=1`​，记录的内容如下

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230121214710-x18hybe.png)​

但是会有一个问题：

在同步数据时，会执行记录的sql语句，`update_time=now()`这里会获取当前系统时间，直接执行会导致与原库的数据不一致。

为了解决这种问题，需要指定为`row`​

### row

**row记录的内容不是简单的sql语句，还包含操作的具体数据**，记录内容如下

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230121215316-gjwlew2.png)​

row格式记录的内容看不到详细信息，要通过mysqlbinlog工具解析出来

​`update_time=now()`​变成了具体的时间`update_time=1627112756247`​，条件后面的@1、@2、@3 都是该行数据第 1 个~3 个字段的原始值（**假设这张表只有 3 个字段**）

这样就能**保证同步数据的一致性**，通常情况下都是指定为`row`​，这样可以为数据库的同步和恢复带来更好的可靠性

但是这种格式，需要更大的容量来记录，恢复与同步时会更消耗`IO`资源，影响执行速度。

所以就有了一种折中的方案，指定为`mixed`​，记录的内容是前两者的混合。

### mixed

​**`MySQL`**​**会判断这条**​**`SQL`**​**语句是否可能引起数据不一致，如果是，就用**​**`row`**​**格式，否则就用**​**`statement`**​**格式。、**

> 其实看到这里，如果比较熟悉`Redis4.x`​版本的小伙伴应该会有种熟悉感，`Redis`​的`RDB、AOF`​持久化模式，正好对应`MySQL`​的`Statment、Row`​模式，而`Redis4.0`​引入了混合持久化机制，`MySQL5.1`​版本也引入了混合日志模式~

## 写入机制

​`binlog`​的写入时机也非常简单，事务执行过程中，**先把日志写到**​**`binlog cache`**​**，事务提交的时候，再把**​**`binlog cache`**​**写到**​**`binlog`**​**文件中**

**因为一个事务的**​**`binlog`**​**不能被拆开，无论这个事务多大，也要确保一次性写入，所以系统会给每个线程分配一块内存作为**​**`binlog cache`**​****

我们可以通过`binlog_cache_size`​参数控制单个线程 binlog cache 大小，如果存储内容超过了这个参数，就要暂存到磁盘（`Swap`​）

binlog日志刷盘流程如下

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230121222535-xo6z5gh.png)​

* **上图的write，是指把日志写入到文件系统的page cache，并没有把数据持久化到磁盘，所以速度比较快**
* **上图的fsync，才是将数据持久化到磁盘的操作**

​`write`​和`fsync`​的时机，可以由参数`sync_binlog`​控制，默认是`0`​。

为`0`​的时候，表示每次提交事务都只`write`​，由系统自行判断什么时候执行`fsync`​。

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230121224549-uo882pr.png)​

虽然性能得到提升，但是机器宕机，`page cache`​里面的`binlog`​会丢失

为了安全起见，可以设置为`1`​，表示每次提交事务都会执行`fsync`​，就如同 **redo log 日志刷盘流程** 一样。

最后还有一种这种方式，可以设置为N(N>1)，表示每次提交事务都`write`​,但累积`N`​个事务后才`fsync`​

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230121224822-5m1x91l.png)​

## 一些题目

### redo log 和 bin log有什么区别

1. 适用对象不同：

    * binlog是Mysql的Server层实现的日志，所有存储引擎都可以适用
    * redolog是Innodb存储引擎实现的日志
2. 文件格式不同：

    * binlog有3种格式类型，分别是STATEMENT(默认格式)、ROW、MIXED
    * redo log 是物理日志，记录的是在某个数据页做了什么修改，比如对 XXX 表空间中的 YYY 数据页 ZZZ 偏移量的地方做了AAA 更新
3. 写入方式不同：

    * binlog 是追加写，写满一个文件，就创建一个新的文件继续写，不会覆盖以前的日志，保存的是全量的日志。
    * redo log 是循环写，日志空间大小是固定，全部写满就从头开始，保存未被刷入磁盘的脏页日志。
4. 用途不同：

    * binlog 用于备份恢复(也就是数据库级别的恢复)、主从复制；
    * redo log 用于掉电等故障恢复。

### 如果不小心整个数据库的数据被删除了，能使用 redo log 文件恢复数据吗？

不可以使用 redo log 文件恢复，只能使用 binlog 文件恢复。

因为 redo log 文件是循环写，是会边写边擦除日志的，只记录未被刷入磁盘的数据的物理日志，已经刷入磁盘的数据都会从 redo log 文件里擦除。

binlog 文件保存的是全量的日志，也就是保存了所有数据变更的情况，理论上只要记录在 binlog 上的数据，都可以恢复，所以如果不小心整个数据库的数据被删除了，得用 binlog 文件恢复数据

## 两阶段提交

事务两阶段提交就是指redo-log分两次写入

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130165916-3532ms4.png)​

主要解决的是主从数据不一致的问题：

1. 先写`bin-log`​，再写`redo-log`​：当事务提交后，先写`bin-log`​成功，结果在写`redo-log`​时断电宕机了，再重启后由于`redo-log`​中没有该事务的日志记录，因此不会恢复该事务提交的数据。但要注意，主从架构中同步数据是使用`bin-log`​来实现的，而宕机前`bin-log`​写入成功了，就代表这个事务提交的数据会被同步到从机，也就意味着从机会比主机多出一条数据。
2. 先写`redo-log`​，再写`bin-log`​：当事务提交后，先写`redo-log`​成功，但在写`bin-log`​时宕机了，主节点重启后，会根据`redo-log`​恢复数据，但从机依旧是依赖`bin-log`​来同步数据的，因此从机无法将这个事务提交的数据同步过去，毕竟`bin-log`​中没有撒，最终从机会比主机少一条数据。

经过上述分析后可得知：如果`redo-log`​只写一次，那不管谁先写，都有可能造成主从同步数据时的不一致问题出现，为了解决该问题，`redo-log`​就被设计成了两阶段提交模式，设置成两阶段提交后，整个执行过程有三处崩溃点：

1. `redo-log(prepare)`​：在写入准备状态的`redo`​记录时宕机，事务还未提交，不会影响一致性。
2. ​`bin-log`​：`MySQL`​根据`redo log`​日志恢复数据时，发现`redo log`​还处于`prepare`​阶段，并且没有对应`binlog`​日志，就会回滚该事务，所以不会出现主库比从库多数据的情况
3. ​`redo-log(commit)`​：在`bin-log`​写入成功后，写`redo(commit)`​记录时崩溃，因为`bin-log`​中已经写入成功了，所以从机也可以同步数据，因此重启时直接再次提交事务，写入一条`redo(commit)`​记录即可。所以不会出现主库比从库少数据的情况

# 总结

Mysql InnoDB引擎使用redo log(重做日志)保证事务的**持久性，**使用**undo log(回滚日志)**来保证事务的事务的原子性

​`MySQL`​​数据库的**数据备份、主备、主主、主从**都离不开`binlog`​​，需要依靠`binlog`​​来同步数据，保证数据一致性。

mysql写操作全流程总结：

​![](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/image-20230130163159-s99qdxw.png)​

‍
