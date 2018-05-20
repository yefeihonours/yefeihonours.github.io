#### 0. 数据库优化可以从下面几个方面着手

* SQL及索引
* 数据库表结构
* 系统配置
* 硬件

<!--more-->


#### 1. 导入示例数据库

----------

> 下载MySQL官网提供的 `sakila` 数据库：[https://dev.mysql.com/doc/index-other.html](https://dev.mysql.com/doc/index-other.html)，然后在MySQL命令模式下导入数据库：[https://dev.mysql.com/doc/sakila/en/sakila-installation.html](https://dev.mysql.com/doc/sakila/en/sakila-installation.html)。

```
# 注： docker 进入 mysql 数据库
shell> docker start mysql_instance 
shell> docker exec -it mysql_instance bash
# 拷贝数据库文件到 docker mysql_instance 后执行下面的命令

shell> mysql -u root -p

mysql> SOURCE C:/temp/sakila-db/sakila-schema.sql;
mysql> SOURCE C:/temp/sakila-db/sakila-data.sql;

mysql> select @@version;
+-----------+
| @@version |
+-----------+
| 5.7.19    |
+-----------+
1 row in set (0.00 sec)

mysql> use sakila;
Database changed
mysql> show tables;
+----------------------------+
| Tables_in_sakila           |
+----------------------------+
| actor                      |
| actor_info                 |
| address                    |
| category                   |
| city                       |
| country                    |
| customer                   |
| customer_list              |
| film                       |
| film_actor                 |
| film_category              |
| film_list                  |
| film_text                  |
| inventory                  |
| language                   |
| nicer_but_slower_film_list |
| payment                    |
| rental                     |
| sales_by_film_category     |
| sales_by_store             |
| staff                      |
| staff_list                 |
| store                      |
+----------------------------+
23 rows in set (0.00 sec)
```

--------------------

#### 2. SQL语句优化

>  如何发现有问题的SQL？
>
>  使用MySQL慢查询日志对有效率问题的SQL进行监控

```
# 查看是否开启慢查询
show variables like 'slow_query_log';

# 指定慢查询日志文件位置
set global slow_query_log_file='/home/mysql/mysql_slow.log';

# 指定是否把没有使用索引的SQL记录到日志中
set global log_queries_not_using_indexes=on;

# 指定查询时间超过 1s 的SQL记录到慢查询日志
# mysql 5.7版本可以省略 global
set global long_query_time=1;

# 开启慢查询日志记录
set global slow_query_log=on;
```

--------------------

> 查看慢查询日志是否正常记录

```
mysql> show tables;
...		# 省略查询输出

mysql> select * from store limit 10;
...

# 查看慢查询日志
shell> tail -n 10 /home/mysql/mysql_slow.log
# Time: 2017-12-13T09:47:45.452804Z
# User@Host: root[root] @ localhost []  Id:     4
# Query_time: 0.000499  Lock_time: 0.000103 Rows_sent: 23  Rows_examined: 23
SET timestamp=1513158465;
show tables;
# Time: 2017-12-13T09:47:57.221642Z
# User@Host: root[root] @ localhost []  Id:     4
# Query_time: 0.000265  Lock_time: 0.000081 Rows_sent: 2  Rows_examined: 2
SET timestamp=1513158477;
select * from store limit 10;
```

--------------------

> 慢查询日志所包含的信息

```
# 日志记录时间
# Time: 2017-12-13T09:47:57.221642Z

# 执行SQL的主机信息
# User@Host: root[root] @ localhost []  Id:     4

# SQL的执行状态信息
# Query_time: 0.000265  Lock_time: 0.000081 Rows_sent: 2  Rows_examined: 2

# SQL执行时间
SET timestamp=1513158477;

# SQL执行语句
select * from store limit 10;
```

--------------------

> 慢查询日志分析工具：`mysqldumpslow` ，MySQL安装后自带有的慢查询日志查看工具

```
shell>  mysqldumpslow --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

Parse and summarize the MySQL slow query log. Options are

  --verbose    verbose
  --debug      debug
  --help       write this text to standard output

  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
                al: average lock time
                ar: average rows sent
                at: average query time
                 c: count
                 l: lock time
                 r: rows sent
                 t: query time
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time

# 查看并分析慢查询日志
shell> mysqldumpslow -t 3 /home/mysql/mysql_slow.log
  
Reading mysql slow query log from /var/lib/mysql/d60e3989c9d8-slow.log

Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=23.0 (23), root[root]@localhost
  show tables

Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=2.0 (2), root[root]@localhost
  select * from store limit N

Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=0.0 (0), 0users@0hosts
  mysqld, Version: N.N.N (MySQL Community Server (GPL)). started with:
  # Time: N-N-13T09:N:N.731872Z
  # User@Host: root[root] @ localhost []  Id:     N
  # Query_time: N.N  Lock_time: N.N Rows_sent: N  Rows_examined: N
  use sakila;
  SET timestamp=N;
  set global slow_query_log=on
  
# Count			SQL执行次数
# Time			SQL执行时间
# Lock			使用锁时间
# Rows			SQL发送的内容
```

--------------------

> 慢查询日志分析工具：`pt-query-digest` 。附工具下载地址。

```
# 安装 pt-* percona工具
shell> wget percona.com/get/percona-toolkit.tar.gz
shell> tar -zxvf percona.com/get/percona-toolkit.tar.gz
# percona 包内的工具都在 bin 目录下，将 bin 目录加入 PATH 环境变量即可使用pt-query-digest

shell> pt-query-digest /home/mysql/mysql_slow.log

# 160ms user time, 0 system time, 25.19M rss, 74.70M vsz
# Current date: Wed Dec 13 10:32:09 2017
# Hostname: d60e3989c9d8
# Files: /var/lib/mysql/d60e3989c9d8-slow.log
# Overall: 3 total, 3 unique, 0.00 QPS, 0.00x concurrency ________________
# Time range: 2017-12-13T09:47:45 to 2017-12-13T10:32:03
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time           25ms   242us    23ms     5ms    23ms     9ms   316us
# Lock time          463us    81us   106us    92us   103us     9us    89us
# Rows sent         15.72k       2  15.67k   3.14k  15.20k   6.07k    9.83
# Rows examine      15.72k       2  15.67k   3.14k  15.20k   6.07k    9.83
# Query size           414      11     297      69  284.79   97.86   28.07

# Profile
# Rank Query ID           Response time Calls R/Call V/M   Item
# ==== ================== ============= ===== ====== ===== ==============
#    1 0x0F48AC368B665207  0.0232 94.6%     1 0.0232  0.00 SELECT payment
#    2 0x132628303F99240D  0.0005  2.0%     1 0.0005  0.00 SHOW TABLES
# MISC 0xMISC              0.0008  3.4%     3 0.0003   0.0 <3 ITEMS>

# Query 1: 0 QPS, 0x concurrency, ID 0x0F48AC368B665207 at byte 1260 _____
# This item is included in the report because it matches --limit.
# Scores: V/M = 0.00
# Time range: all events occurred at 2017-12-13T10:32:03
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count         20       1
# Exec time     94    23ms    23ms    23ms    23ms    23ms       0    23ms
# Lock time     17    83us    83us    83us    83us    83us       0    83us
# Rows sent     99  15.67k  15.67k  15.67k  15.67k  15.67k       0  15.67k
# Rows examine  99  15.67k  15.67k  15.67k  15.67k  15.67k       0  15.67k
# Query size     5      21      21      21      21      21       0      21
# String:
# Hosts        localhost
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms  ################################################################
# 100ms
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS LIKE 'payment'\G
#    SHOW CREATE TABLE `payment`\G
# EXPLAIN /*!50100 PARTITIONS*/
select * from payment\G

# Query 1: ...
```

--------------------

> 如何通过慢查询日志发现有问题的SQL？

```
1. 查询次数多且每次查询占用时间长的SQL
   通常为 pt-query-digest 分析的前几个查询
2. IO大的SQL
   注意 pt-query-digest 分析中的 Rows examine 项，扫面行数据量
3. 未命中索引的SQL
   注意 pt-query-digest 分析中 Rows examine 和 Rows Send 的对比 [扫描行/命中行]
```

--------------------

> 如何分析SQL查询，使用 `explain` 

```
mysql> explain select customer_id, first_name from customer \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: customer
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 599
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)

# explain 返回各列的含义
table:          显示这一行的数据是关于哪张表的
type:           显示查询使用的类型。优->劣: const,eq_reg,ref,range,indec,ALL
possible_keys:  可能应用的索引，NULL，表示没使用索引
key:            实际使用的索引
key_len:        实际使用的索引长度，在不损失精确性的情况下，长度越短越好
ref:            显示索引被使用的列，如果可能的话，是一个常数
rows:           MySQL认为必须检查的用来返回请求数据的行数
Extra:          Using filesort: 需要优化，MySQL需要进行额外的步骤来发现对返回的行排序。
                它根据连接类型及存储排序键值和匹配条件的全部的行指针来排序
                Using temporary: 需要优化，MySQL需要创建一个临时表存储结果，
                通常发生在对不同的列集进行 ORDER BY 上，而不是 GROUP BY 上。
```

--------------------

> Count() 和 Max()函数优化

```
#
# MAX
#
mysql> explain select MAX(payment_date) from payment \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: payment
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 16086
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)

# 优化，建立索引
mysql> create index idx_paydate on payment(payment_date);
Query OK, 0 rows affected (0.05 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> explain select max(payment_date) from payment \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: NULL
   partitions: NULL
         type: NULL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: Select tables optimized away
1 row in set, 1 warning (0.00 sec)

# 不需要扫描表数据，通过索引就可以查询到需要的结果
# 对于简单的MAX查询操作，可以使用索引查询的方式

# 在添加了索引之后，那么会增加一个索引表，这个索引表记录了索引值与对应字段的关系
# 然后，以该字段进行的查询操作，将不再需要扫描原来的数据表的每一行，而是扫描这个建立的索引表
# 显然，这个索引表的IO的操作就比原来的数据表要小很多了，所以可以提升查询的速度
# 并且如果表的字段比较多的情况，那么建立索引的总用越明显
# 同时，因为要维护这个索引表，所以当进行增，删，改的时候，性能会相对下降；

# 索引的应用:
# 覆盖索引，就是说 通过索引的值，在索引表中就可以找到需要的值；

#
# COUNT
#
mysql> select COUNT(release_year='2006' OR NULL) AS '2006YEARS',
       COUNT(release_year='2007' OR NULL) AS '2007YEARS' from film;
+-----------+-----------+
| 2006YEARS | 2007YEARS |
+-----------+-----------+
|      1000 |         0 |
+-----------+-----------+
1 row in set (0.01 sec)

# NULL 在 COUNT查询时，会被略过查询，忽略扫描不需要的数据行
```

--------------------

> 子查询的优化

```
子查询一般使用 JOIN 优化成表连接的形式，但是要注意，使用 JOIN 时，如果出现数据重复，需要使用 distinct 来去除重复的数据行
```



> GROUP BY 优化方式

```
#
# 示例SQL
#
mysql> explain select actor.first_name,
         Count(*) from sakila.film_actor INNER JOIN sakila.actor USING(actor_id)
         GROUP BY film_actor.actor_id \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: actor
   partitions: NULL
         type: ALL
possible_keys: PRIMARY
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 200
     filtered: 100.00
        Extra: Using temporary; Using filesort
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: film_actor
   partitions: NULL
         type: ref
possible_keys: PRIMARY,idx_fk_film_id
          key: PRIMARY
      key_len: 2
          ref: sakila.actor.actor_id
         rows: 27
     filtered: 100.00
        Extra: Using index
2 rows in set, 1 warning (0.00 sec)

# 在上面的执行计划中显示，表连接使用 GROUP BY 查询时
# 使用了 Using temporary; Using filesort ，这是需要做优化的地方
# Using index 表示使用索引查询

#
# 改写SQL
#
mysql> explain select actor.first_name, c.cnt
         from sakila.actor INNER JOIN
           ( select  actor_id,COUNT(*) AS cnt 
             from sakila.film_actor GROUP BY actor_id
           ) AS c USING(actor_id) \G

*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: actor
   partitions: NULL
         type: ALL
possible_keys: PRIMARY
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 200
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: PRIMARY
        table: <derived2>
   partitions: NULL
         type: ref
possible_keys: <auto_key0>
          key: <auto_key0>
      key_len: 2
          ref: sakila.actor.actor_id
         rows: 27
     filtered: 100.00
        Extra: NULL
*************************** 3. row ***************************
           id: 2
  select_type: DERIVED
        table: film_actor
   partitions: NULL
         type: index
possible_keys: PRIMARY,idx_fk_film_id
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 5462
     filtered: 100.00
        Extra: Using index
3 rows in set, 1 warning (0.00 sec)

# 改写SQL后，将分组 GROUP BY 放在子查询中
# 查询时不再使用 Using temporary; Using filesort，而是只有扫描表和使用索引的形式
# 这样可以节省大量的 IO 资源
```

--------------------

> Limit 条件优化
>
> limit 常用于数据的分页处理，经常伴随 ORDER BY 从句使用，因此会使用文件排序(Using filesort)，这样会造成大量的IO操作，占用系统IO资源

```
# 示例
mysql> explain select film_id, description
         from sakila.film ORDER BY title LIMIT 50, 5 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: film
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1000
     filtered: 100.00
        Extra: Using filesort
1 row in set, 1 warning (0.00 sec)

# 上面的SQL语句使用的表扫描的方式扫描了1000行数据，还使用到了文件排序(Using filesort)方式
# 当表中的数据量非常大的时候，会产生IO资源问题

# 优化步骤 1
# 使用有索引的列或主键进行 ORDER BY 操作
mysql> explain select film_id, description
       from sakila.film ORDER BY film_id LIMIT 50, 5 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: film
   partitions: NULL
         type: index
possible_keys: NULL
          key: PRIMARY
      key_len: 2
          ref: NULL
         rows: 55
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)

# 排序换成主键后，仅仅扫描了55行，并且没有使用文件排序，而是使用索引查询
# 但是随着翻页增加，扫描的行数也会增加，数据量很大的时候也会占用大量IO资源

# 优化步骤 2
# 如果使用排序查询的主键是`整数且连续递增`的
# 在分页查询的时候，使用 WHERE 条件进行过滤，会减少表扫描的行数
# 但是使用这中方式，是有限制的，需要使用查询的主键必须是 `整数且连续递增` 的

```

--------------------

#### 3. 索引优化

> 如何选择合适的列建立索引

```
1. 在 WHERE 从句，GROUP BY 从句，ORDER BY从句， OR 从句中出现的列
2. 索引字段越小越好
3. 离散度大的列放到联合索引的前面
   SELECT * FROM payment WHERE staff_id = 2 AND customer_id = 584;
   在建立索引时，因为customer_id的离散度更大，索引应该使用联合索引 index(customer_id, staff_id)
```

----------

> 索引优化SQL的方法

```
1. 建立索引可以提高查询的效率，但是，过多的索引会影响数据库写入的效率
   同时，也可能会影响查询时索引的选择，从而减低了索引查询本身的高效性
2. 删除重复和冗余的索引:
   重复索引是指相同的列以相同的顺序建立的同类型的索引
   如下表中 primary key 和在 id 列上建立的索引就是重复索引
   create table test(
     id int not null promary key,		# 指定主键，建立索引
     name varchar(10) not null,
     unique(id)							# 指定唯一键，建立索引
   )engine=innodb;
   
   冗余索引是指多个索引的前缀列相同，或是在联合索引中包含了主键的索引
   下面例子中 key(name, id) 就是一个冗余索引
   create table test(
     id int not null promary key,		# 指定主键，建立索引
     name varchar(10) not null,
     key(name, id)						# 指定联合索引  
   )engine=innodb;
   
3. 查看数据库中的重复索引
mysql> use information_schema;
mysql> SELECT
         TS1.TABLE_SCHEMA AS 'TABLE_NAME',
         TS1.TABLE_NAME AS 'TABLE_NAME',
         TS1.INDEX_NAME AS 'INDEX_1',
         TS2.INDEX_NAME AS 'INDEX_2',
         TS1.COLUMN_NAME AS 'DUPLICATE_COLUMN'
       FROM STATISTICS TS1 JOIN STATISTICS TS2
         ON TS1.TABLE_SCHEMA=TS2.TABLE_SCHEMA
         AND TS1.TABLE_NAME=TS2.TABLE_NAME
         AND TS1.SEQ_IN_INDEX=TS2.SEQ_IN_INDEX
         AND TS1.COLUMN_NAME=TS2.COLUMN_NAME
       WHERE
         TS1.SEQ_IN_INDEX=1
         AND TS1.INDEX_NAME<>TS2.INDEX_NAME;
         
+------------+------------+----------+----------+------------------+
| TABLE_NAME | TABLE_NAME | INDEX_1  | INDEX_2  | DUPLICATE_COLUMN |
+------------+------------+----------+----------+------------------+
| sakila     | country    | idx_cnty | PRIMARY  | country_id       |
| sakila     | country    | PRIMARY  | idx_cnty | country_id       |
+------------+------------+----------+----------+------------------+
2 rows in set (0.01 sec)
```

---------

> 使用工具查询重复索引；`py-duplicate-key-checker` 

```
# 使用工具查询重复索引的示例命令
shell> pt-duplicate-key-checker -u root -p 'password' -h 127.0.0.1:3306
# ########################################################################
# sakila.country
# ########################################################################

# idx_cnty is a duplicate of PRIMARY
# Key definitions:
#   KEY `idx_cnty` (`country_id`)
#   PRIMARY KEY (`country_id`),
# Column types:
#	  `country_id` smallint(5) unsigned not null auto_increment
# To remove this duplicate index, execute:
ALTER TABLE `sakila`.`country` DROP INDEX `idx_cnty`;

# ########################################################################
# Summary of indexes
# ########################################################################

# Size Duplicate Indexes   218
# Total Duplicate Indexes  1
# Total Indexes            97

# 上面的命令输出，会显示表中的重复索引，并显示索引对应的列，以及给定的移除索引建议
```



#### 附 percona 工具下载地址

安装percona [https://www.percona.com/doc/percona-toolkit/2.1/installation.html](https://www.percona.com/doc/percona-toolkit/2.1/installation.html)

