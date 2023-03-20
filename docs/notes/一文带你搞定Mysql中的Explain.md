---
title: 一文带你搞定Mysql中的Explain
permalink: /post/undefined.html
date: 2023-01-04 22:37:25
meta:
  - name: keywords
    content: ''
  - name: description
    content: >-
      一文带你搞定mysql中的explain一介绍使用explain关键字可以模拟优化器执行sql语句分析你的查询语句或者是结构的性能瓶颈。在select语句之前增加explain关键字mysql会在查询上设置一个标记执行查询会返回执行计划的信息而不是执行这条sql。二explain中的列id列id代表每个小的select查询语句的唯一标识id越大优先级越高id相同则从上往下执行id为null则最后执行。select_type列select_type表示对应行是简单还是复杂的查询simple_简单查询。查询不
tags: []
categories: []
author:
  name: XDUcoderT
  link: https://github.com/XDUcoderT
---

#  一文带你搞定Mysql中的Explain

## 一、介绍

使用Explain关键字可以模拟优化器执行sql语句，分析你的查询语句或者是结构的性能瓶颈。在select语句之前增加explain关键字，mysql会在查询上设置一个标记，执行查询会返回执行计划的信息，而不是执行这条sql。

## 二、Explain中的列

### 1.id列

id代表每个小的select查询语句的唯一标识，id越大优先级越高，id相同则从上往下执行，id为null则最后执行。

### 2.select_type列

select_type表示对应行是简单还是复杂的查询

> 1.simple：简单查询。查询不包含子查询和union  
> 2.primary：复杂查询最外层的select  
> 3.subquery：包含在select中的子查询(不在from语句中)  
> 4.derived: 包含在from子句中的子查询。mysql将结果存放在一个临时表中，也称为派生表  
> 5. union:在union中的第二个和随后的select

举例：

<pre><div class="hljs"><code class="lang-sql">mysql> set session optimizer_switch='derived_merge=off'; #关闭mysql5.7新特性对衍生表的合并优化 
mysql> explain select (select 1 from actor where id = 1) from (select * from film where id = 1) der;
</code></div></pre>

​![explain1.png](http://xducodert001.oss-cn-hangzhou.aliyuncs.com/articles/74fe2347f4f4d610acdb98ce62b66e23.png)​

<pre><div class="hljs"><code class="lang-sql">mysql> set session optimizer_switch='derived_merge=on'; #还原默认配置
</code></div></pre>

### 3.table列

这一列表示explain的一行正在访问哪个表。当from子句中有子查询时，table列是格式，表示当前查询依赖id=N的查询，于是先执行id=N的查询。当有union时，UNION RESULT的table列的值为<union1,2>,1和2表示参与union的select行id

### 4.type列

这一列表示**关联类型或访问类型**，即sql决定如何查找表中的行，查找数据记录的大概范围。

依次从最优到最差分别为：system ≈ const &gt; eq_ref &gt; ref &gt; range &gt; index &gt; all

一般来说，得保证查询达到range级别，最好达到ref

* NULL：mysql能够在优化阶段分解查询语句，在执行阶段用不着再访问表或者索引。  
  例如：在索引中选取最小值，可以单独查找索引来完成，不需要在执行时访问表。

  <pre><div class="hljs"><code class="lang-sql">mysql> explain select min(id) from film;
  </code></div></pre>

​![explain2.png](http://xducodert001.oss-cn-hangzhou.aliyuncs.com/articles/68f41b64dfd991a0c0003c882050bc91.png)​

* const，system：

  * const: 单表中最多只有一条匹配行，查询起来非常迅速，所以这个匹配行中的其他列中的值可以被优化器在当前查询中当做常量来处理。例如根据主键或者唯一索引进行的查询。
  * system: ​**system是const的特例**，如果表中只有一行数据，那么type就是system。

  <div>
  <pre><div class="hljs"><code class="lang-sql"> mysql> explain select * from film  where film_id = 1;
  </code></div></pre>
  </div>

​![explain3.png](http://xducodert001.oss-cn-hangzhou.aliyuncs.com/articles/9d1515d172ac7c4beab3fb21e6c5003a.png)​

* eq_ref: primary key 或 unique key索引的所有部分被连接使用 对于每个索引键值，只有唯一的一条匹配记录（在联表查询中使用primary key或者unique key作为关联条件）

  <div>
  <pre><div class="hljs"><code class="lang-sql">mysql> explain select * from film_actor left join film on film_actor.film_id = film.id;
  </code></div></pre>
  </div>

  ​![explain4.png](http://xducodert001.oss-cn-hangzhou.aliyuncs.com/articles/6534eaf7a5b95b3211d90a0f0dcb7fa2.png)​
* ref：相比eq_ref,不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行

  *  简单的select语句。name是普通索引(非唯一索引)

    <div>
    <pre><div class="hljs"><code class="lang-sql"> mysql> explain select * from film where name = 'film1';
    </code></div></pre>
    </div>

    ​![explain5.png](http://xducodert001.oss-cn-hangzhou.aliyuncs.com/articles/e6046e74816d7fbce2166a1c150ac01c.png)​
  * 非唯一性索引关联表查询，idx_film_actor_id是film_id和actor_id的联合索引，这里使用到了film_actor的左边前缀film_id部分。

    <pre><div class="hljs"><code class="lang-sql">mysql> explain select film_id from film left join film_actor on film.id = film_actor.fi
    lm_id;
    </code></div></pre>

    ​![explain6.png](http://xducodert001.oss-cn-hangzhou.aliyuncs.com/articles/1c58a1b3e025a5eaee17b726b7a3406f.png)​
* range：范围扫描通常出现在in(),between，&gt;,&lt;，&gt;=等操作的。使用一个索引来检索给定范围的行

  <pre><div class="hljs"><code class="lang-sql"> explain select * from actor where id > 1;
  </code></div></pre>

             ![explain7.png](http://xducodert001.oss-cn-hangzhou.aliyuncs.com/articles/ec2d6cbf4c353f18bcaeb9b092c15bc7.png)​

* index：扫描全表索引就能拿到结果（不需要进行回表操作），一般是扫描某个二级索引，这种扫描不会从索引树根节点开始快速查找，而是直接对二级索引的叶子节点遍历和扫描（叶子节点是双向链表的结构），速度还是比较慢的，这种查询一般为使用覆盖索引，二级索引一般比较小，所以这种通常比all快一些。

  <pre><div class="hljs"><code class="lang-sql">mysql> explain select * from film;
  </code></div></pre>

            ![explain8.png](http://xducodert001.oss-cn-hangzhou.aliyuncs.com/articles/68fc2406c3db1b5fffbcd5661d5ef0b4.png)​

* all：即全表扫描，扫描你的聚簇索引的所有叶子节点(需要优化)

  <pre><div class="hljs"><code class="lang-sql">mysql> explain select * from actor;
  </code></div></pre>

     ![explain9.png](http://xducodert001.oss-cn-hangzhou.aliyuncs.com/articles/0c7d38c5bf755e68d97bc93847c3ea5a.png)​

### 5.possible_keys列

标识innodb可能采取的索引列

**注意**

explain时可能出现possible_keys有列，而keys为NULL的情况，这种情况是因为表中数据不多,mysql认为索引对此帮助不大，选择了全表查询

如果该列是NULL，则没有相关的索引，可以检查where子句看是否可以创造一个适当的索引来提高查询性能，然后用explain查看结果

### 6. key列

这一列显示的mysql实际采用哪个索引来优化对该表的访问

如果没有使用索引，则该列是NULL。如果想强制mysql使用或忽视possible_keys中的列，可以使用force index、ignore index。

### 7. key_len列

这一列显示了mysql在做因里使用的字节数，通过这个值可以算出具体使用了索引中的哪些列。举例来说，film_actor的联合索引引idx_film_actor_id 由 film_id 和 actor_id 两个int列组成，并且每个int是4字节。通过结果中的key_len=4可推断出查询使用了第一个列：film_id列来执行索引查找。

```
 mysql> explain select * from film_actor where film_id = 2;
```

​![explain11.png](http://xducodert001.oss-cn-hangzhou.aliyuncs.com/articles/7f8be365de69648ee2da46986d194113.png)  
key_len计算规则如下：

* 字符串  
  char(n)和varchar(n) 5.0.3以后版本中，n均代表字符数，而不是字节数，如果是utf-8，一个数字或字母占1个字节，一个汉字占3个字节

  * char(n)：  
    如果存汉字长度就是 3n 字节
  * varchar(n)：  
    如果存汉字则长度是 3n + 2 字节，加的2字节用来存储字符串长度，因为 varchar是变长字符串
* 数值类型

  * tinyint：1字节
  * smallint：2字节
  * int：4字节
  * bigint：8字节
* 时间类型

  * date：3字节
  * timestamp：4字节
  * datetime：8字节
* 如果字段允许为 NULL，需要1字节记录是否为 NULL

注意：索引的最大长度是768字节，当字符串过长时，mysql会做一个类似左前缀的处理。将前半部分的字符提取出来左索引

### 8.ref列

这一列显示了在key列记录的索引中，表查找值所用到的列或者常量，常见的有：const(常量)，字段名(例：film.id)

### 9.rows列

这一列是mysql估计要读取并检测的行数，注意这个不是结果集里的行数

### 10.Extra列

这一列展示的是额外信息，但十分重要

1. ​**Using index**​：使用覆盖索引

> mysql执行计划explain结果里的key有使用索引，如果select后面查询的字段都可以从这个索引的树中获取，这种情况一般可以说是用到了覆盖索引，extra里一般都有using index；覆盖索引一般针对的是辅助索引，整个查询结果只通过辅助索引就可以拿到结果，不需要回表

2. ​**using where**​：使用where语句来处理结果，并且查询的列未被索引覆盖
3. ​**Using index condition**​：查询的列不完全被索引覆盖，where条件中是一个前导列的范围
4. ​**Using temporary**​: mysql需要创建一张临时表来处理查询，出现这种情况一般是要进行优化的，首先想到用索引进行优化
5. ​**Using filesort**​：将用外部排序而不是索引排序，数据较小时从内存排序，否则需要在磁盘完成排序。这种情况下一般也是要考虑索引优化的
6. Select tables optimized away： 使用某些聚集函数(比如 max min)来访问索引的某个字段  
    例如：

    ```
    mysql> explain select min(id) from film;
    ```
