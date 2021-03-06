#### 0. 数据库优化可以从下面几个方面着手

* SQL及索引
* 数据库表结构
* 系统配置
* 硬件

<!--more-->

#### 1. 数据库表结构优化

---------

> 选择合适的数据类型

```
数据类型的选择，重点在于 合适 ，如何确认选择的数据类型是否合适？
1. 使用可以存下要求数据的最小的数据类型
2. 使用简单的数据类型，INT 要比 VARCHAR 类型处理上简单
3. 尽可能使用 NOT NULL 定义字段，然后给定默认值
4. 尽量少用 TEXT 类型，一定要使用时最好考虑数据分表

#
# 使用 INT 来存储日期类型
# 利用 FROM_UNIXTIME(), UNIX_TIMESTAMP() 两个函数进行时间戳转换
   CREATE TABLE test(
     id INT AUTO_INCREMENT NOT NULL,
     timestr INT,
     PRIMARY KEY(id)
   );
   
   INSERT INTO test(timestr) VALUES(UNIX_TIMESTAMP('2017-12-01 12:00:00'));
   
   SELECT FROM_UNIXTIME(timestr) FROM test;
   
#
# 使用 BIGINT 来存储IP地址
# 利用 INET_ATON(), INET_NTOA() 两个函数进行IP地址转换
   CREATE TABLE sessions(
     id INT AUTO_INCREMENT NOT NULL,
     ipaddress BIGINT,
     PRIMARY KEY(id)
   );
   
   INSERT INTO sessions(ipaddress) VALUES(INET_ATON('192.168.0.1'));
   
   SELECT INET_NTOA(ipaddress) FROM sessions;
```

-----------

> 表的范式化和反范式化

```
# 范式化是指数据库设计的规范，目前说道的范式化一般是指第三设计范式
# 也就是要求数据表中不存在非关系字段对任意候选关键字段的传递函数依赖，则符合第三范式

# 不符合第三范式的数据库表结构:
1. 数据冗余，对于记录中分类描述等信息会重复出现
2. 数据的插入异常
3. 数据的更新异常
4. 数据的删除异常

#
# 可以将数据表中分类信息或着关联信息从表中分离出来，创建新的分类表或者关联表
#
# 但是，严格的第三范式会导致过多的表分离
# 在实际的数据查询中，可以适当的增加一些冗余字段，减少数据查询时的连表操作，优化查询效率
# 这是在适当的应用反范式化的方式，用表空间换取查询时间

```

----------

> 数据表的垂直拆分和水平拆分

```
#
# 垂直拆分
#
# 所谓的垂直拆分，就是把原来一个有很多列的表拆分成多个表，这解决了表的宽度问题
# 通常垂直拆分可以按照一下的原则进行：
1. 把不常用的字段单独放到一张表中
2. 把大字段独立放到一张表中(TEXT等)
3. 把经常一起使用的字段放到同一张表中

#
# 水平拆分
#
# 表的水平拆分，是为了解决单表中数据量过大的问题
# 水平拆分的表每一个表的结构都是完全一致的
# 水平拆分表要注意几个问题：
1. 拆分表后每条记录的储存位置设定
2. 跨分区表数据查询
3. 合并分区表完成数据统计
```



#### 2. 系统配置优化&服务器硬件优化 

----------

```
#
# 系统配置
# 优化系统配置方面，可以从增加网络连接数，减少断开连接时的资源回收
# 同时可以修改MySQL配置文件，配置缓冲池，
#
# 硬件优化
# 选择适当的CPU，并不是CPU核数越多越好
# 优化磁盘IO，选择合适的磁盘阵列，减少IO资源占用
```







