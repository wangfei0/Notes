# MySQL

# 执行流程

![img](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/sql执行流程)

# 引擎

## InnoDB

MySQL 5.5版本后默认的存储引擎

- 事务性数据库引擎

## MyISAM

- MyISAM不支持事务和行级锁
- 崩溃后无法安全恢复

## 二者区别

1. **是否支持行级锁** : MyISAM 只有表级锁(table-level locking)，而InnoDB 支持行级锁(row-level locking)和表级锁,默认为行级锁。
2. **是否支持事务和崩溃后的安全恢复：** 
   - MyISAM强调的是性能，每次查询具有原子性,其执行速度比InnoDB类型更快，但是不提供事务支持。
   - **InnoDB** 提供事务支持，外部键等高级数据库功能。 具有事务(commit)、回滚(rollback)和崩溃修复能力(crash recovery capabilities)的事务安全(transaction-safe (ACID compliant))型表。
3. **是否支持外键：** MyISAM不支持，而InnoDB支持。
4. **是否支持MVCC** ：仅 InnoDB 支持。应对高并发事务, MVCC比单纯的加锁更高效;MVCC只在 `READ COMMITTED` 和 `REPEATABLE READ` 两个隔离级别下工作;MVCC可以使用 乐观(optimistic)锁 和 悲观(pessimistic)锁来实现;各数据库中MVCC实现并不统一。推荐阅读：[MySQL-InnoDB-MVCC多版本并发控制](https://segmentfault.com/a/1190000012650596)

[9个不同点](https://blog.csdn.net/qq_35642036/article/details/82820178)

**1.  InnoDB支持事务，MyISAM不支持**

**2. InnoDB支持外键，而MyISAM不支持**

**3. InnoDB是聚集索引，使用B+Tree作为索引结构，数据文件是和（主键）索引绑在一起的（表数据文件本身就是按B+Tree组织的一个索引结构）。 MyISAM是非聚集索引，也是使用B+Tree作为索引结构，索引和数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。**

**4. InnoDB不保存表的具体行数，执行select count(\*) from table时需要全表扫描。而MyISAM用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快（注意不能加有任何WHERE条件）**

**5. Innodb不支持全文索引，而MyISAM支持全文索引**

**6. MyISAM表格可以被压缩后进行查询操作**

.**7. InnoDB支持表、行(默认)级锁，而MyISAM支持表级锁**

InnoDB的行锁是实现在索引上的，而不是锁在物理行记录上。潜台词是，如果访问没有命中索引，也无法使用行锁，将要退化为表锁。

**8、InnoDB表必须有主键（用户没有指定的话会自己找或生产一个主键），而Myisam可以没有**

**9、Innodb存储文件有frm、ibd，而Myisam是frm、MYD、MYI**

​        **Innodb：frm是表定义文件，ibd是数据文件**

​        **Myisam：frm是表定义文件，myd是数据文件，myi是索引文件**

# 索引

索引在MySQL中也叫做 **键key**。是存储引擎快速找到记录的一种数据结构。

### 哈希索引（hash index）

基于哈希表实现，只有精准匹配索引所有列的查询才有效。

- **查找速度非常快**
- **无法用于排序**
- **哈希索引只支持等值比较查询，不支持任何范围查询**
- **有一个变量对全部数据计数了，当查询数据总数时，也就是执行select count(*)时，直接返回总数，时间为O(1)的。**
- **底层的数据结构就是哈希表，因此在绝大多数需求为单条记录查询的时候，可以选择哈希索引，查询性能最快；**

### B+Tree索引

#### InnoDB引擎下

- 其数据文件本身就是索引文件
- 树的叶节点data域保存了完整的数据记录
- 辅助索引的data域存储相应记录主键的值而不是地址
- **在根据主索引搜索时，直接找到key所在的节点即可取出数据；在根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引。** **因此，在设计表的时候，不建议使用过长的字段作为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂。**

#### MyISAM引擎下

- B+Tree叶节点的data域存放的是数据记录的地址。

## 索引的底层结构

![img](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/5373082.jpg)

要找到id为8的记录简要步骤：

![img](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/89338047.jpg)



### 高性能索引策略

也就是什么情况下会用到索引，什么情况下不会用到。

1. 独立的列

   始终将索引列单独放在比较符的一侧。

   如：

   ...where id = 1;会用到索引

   ...where id+1 = 1;不会用到索引

2. 前缀索引

   对于TEXT和VARCHAR格式的数据，必须使用前缀索引，不允许索引列的完整长度。

   使用方法

   alter table table_name add key (cloum(7))

   建立了一个已cloum的前7个前缀为索引的前缀索引。

### 聚簇索引

是一种数据存储方式

一个表只能有一个聚簇索引，数据行存放在索引列的叶子页中。

在innodb中必须有主键，主键索引就是聚簇索引。如果没定义主键，innodb会选择一个唯一的非空索引代替。如果没有这样的索引，innodb会隐式定义一个主键来作为聚簇索引。

### 覆盖索引

如果一个索引包含所有的需要查询字段的值，就称之为覆盖索引。

## 冗余和重复索引

重复索引指的是在同一列上，按照相同顺序建立了相同类型的索引。

冗余索引：如有索引(A,B)，再有索引(A)。就是冗余索引，但是(B,A)和(B)，不是冗余索引。

# 事务特性

## ACID

1. **A(atomicity) 原子性：**

   一个事务必须被视为一个不可分割的最小工作单元。整个事务中的所有操作要么全部提交成功，要么全部回滚，对于一个事务来说，不可能只执行其中一部分操作，这就是事务的原子性。

2. **C(consistency) 一致性：**

   数据库总是从一个一致性的状态转换到另一个一致性的状态。

3. **I(isolation) 隔离性：**

   一个事务所做的修改在最终提交之前，对于其他事务是不可见的。

4. **D(durability) 持久性：**

   一旦事务提交，则其所作的修改就会永久保存到数据库中。

# InnoDB怎么解决的幻读问题

## 间隙锁

**REPEATABLE-READ（可重读）隔离级别下**

InnoDB 存储引擎在 **REPEATABLE-READ（可重读）** 事务隔离级别下使用的是Next-Key Lock 锁算法，因此可以避免幻读的产生

**Next-Key Lock 锁算法**

是两种锁的结合：

- 行级锁Record lock：单个行记录上的锁
- 间隙锁Gap lock：间隙锁，锁定一个范围，不包括记录本身
- Next-key lock：record+gap 锁定一个范围，包含记录本身

**重要：**

1. innodb对于行的查询使用next-key lock（解决幻读问题）
2. Next-locking keying为了解决Phantom Problem幻读问题
3. 当查询的索引含有唯一属性时，将next-key lock降级为record key
4. Gap锁设计的目的是为了阻止多个事务将记录插入到同一范围内，而这会导致幻读问题的产生
5. 有两种方式显式关闭gap锁：（除了外键约束和唯一性检查外，其余情况仅使用record lock） A. 将事务隔离级别设置为RC B. 将参数innodb_locks_unsafe_for_binlog设置为1

InnoDB 存储引擎在 **分布式事务** 的情况下一般会用到 **SERIALIZABLE(可串行化)** 隔离级别。

## MVCC解决幻读

//todo

# 最左匹配原则⭐

# 索引失效的情况⭐

[原文在这里](https://juejin.im/post/6844903954392825869)

[文章1](https://blog.csdn.net/wuseyukui/article/details/72312574)

![](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\索引失效的情况总结.png)

#### 第一类：普通索引

在key1上建立索引

1. **不等式** `<>` 或 `!=` 会导致索引失效

2. **数据不一致：**字段 `key1` 为字符串，传入的值为数字类型，会导致索引失效

   ~~~sql
   SELECT * FROM `t_index` WHERE key1 = 1;
   ~~~

3. **函数计算导致索引失效：** 

   如：

   ~~~sql
   SELECT * FROM `t_index` WHERE key1 + 1 = 1;
   SELECT * FROM `t_index` WHERE CHAR_LENGTH(key1) = 1;
   ~~~

   函数计算 `x+1` 、 `x-1` 、`CHAR_LENGTH(x)` 等会导致索引失效

4. **模糊查询：**

   ~~~sql
   SELECT * FROM `t_index` WHERE key1 LIKE  '3';
   SELECT * FROM `t_index` WHERE key1 LIKE  '%3';
   SELECT * FROM `t_index` WHERE key1 LIKE  '3%';
   ~~~

   模糊查询查询条件前缀模糊不会走索引（%开头）

#### 第二类：组合索引

在字段 `key1, key2, key3` 上创建复合索引



正常走索引在and连接符下

~~~sql
SELECT * FROM `t_index` WHERE key1 = '1' AND key2 = '2' AND key3 = '3';
~~~

1. **使用不等式：** 查询脚本，只要有一个条件含有不等式，都不会走索引

   ~~~sql
   SELECT * FROM `t_index` WHERE key1 <> '1' AND key2 = '2' AND key3 = '3';
   SELECT * FROM `t_index` WHERE key1 = '1' AND key2 <> '2' AND key3 = '3';
   SELECT * FROM `t_index` WHERE key1 = '1' AND key2 = '2' AND key3 <> '3';
   ~~~

   查看执行计划，结果显示全表扫描，三种情况结果一样。

2. **查询条件类型不一致：** 

   ~~~sql
   SELECT * FROM `t_index` WHERE key1 = 1 AND key2 = '2' AND key3 = '3';
   SELECT * FROM `t_index` WHERE key1 = '1' AND key2 = 2 AND key3 = '3';
   SELECT * FROM `t_index` WHERE key1 = '1' AND key2 = '2' AND key3 = 3;
   ~~~

   查看执行计划，结果显示（第一个参数类型不一致走全表扫描，第二个参数类型不一致，索引仅仅能使用第一列，第三个参数类型不一致，索引能使用前两列。

3. **查询条件使用函数计算**

   ~~~sql
   SELECT * FROM `t_index` WHERE key1 + 1 = '1' AND key2 = '2' AND key3 = '3';
   SELECT * FROM `t_index` WHERE key1 = '1' AND key2 + 1 = '2' AND key3 = '3';
   SELECT * FROM `t_index` WHERE key1 = '1' AND key2 = '2' AND key3 + 1 = '3';
   ~~~

   查看执行计划，结果同上（第一个参数类型不一致走全表扫描，第二个参数类型不一致，索引仅仅能使用第一列，第三个参数类型不一致，索引能使用前两列）

4. **不使用索引首列当查询条件**

   ~~~sql
   SELECT * FROM `t_index` WHERE key2 = '2' AND key3 = '3';
   SELECT * FROM `t_index` WHERE key2 = '2';
   SELECT * FROM `t_index` WHERE key3 = '3';
   ~~~

   查看执行计划，结果显示（都不会走索引），三种情况结果一样

### ⭐重要重要，面试会问的

[美团技术团队文章，里面有索引建立的注意事项](https://tech.meituan.com/2014/06/30/mysql-index.html)

### 建索引的几大原则

1.最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

2.=和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式。

3.尽量选择区分度高的列作为索引，区分度的公式是count(distinct col)/count(*)，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录。

4.索引列不能参与计算，保持列“干净”，比如from_unixtime(create_time) = ’2014-05-29’就不能使用到索引，原因很简单，b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’)。

5.尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

**例子：**

字段a有索引，b没有索引

- where a='1' and b='1'; 走索引a
- where b='1' and a='1'; 走索引a

字段a 和b 组合索引 (a,b)

- where a=‘1’and b='1';
- where b='1' and a='1';
- where a != '1' and b='1';
- where a ='1' or b = '1';

**以下情况说明：**

1. explian执行计划中 key 的值为 null 表示没有使用索引。

2. 数据类型出现隐式转换的时候也不会使用索引，例如，`where 'age' 10=30`；即如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引。

3. 对索引列进行函数运算，原因同上。

4. 正则表达式不会使用索引。

5. 如果 MySQL 估计使用索引比全表扫描更慢，则不使用索引

6. 用 or 分割开的条件，如果 or的两边字段都有索引的话会使用索引，只要有一个没有索引就不会用到索引。

7. 使用负向查询（not ，not in， not like ，<> ,!= ,!> ,!< ） 不会使用索引

8. like查询是以%开头不会使用索引

9. in条件在较新的版本中是走索引的[关于in走不走索引](https://blog.csdn.net/qpc672456416/article/details/80788997?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

# **慢查询怎么优化**

**慢查询优化基本步骤：**

0. 先运行看看是否真的很慢，注意设置SQL_NO_CACHE，不使用缓存

1. where条件单表查，锁定最小返回记录表。这句话的意思是把查询语句的where都应用到表中返回的记录数最小的表开始查起，单表每个字段分别查询，看哪个字段的区分度最高

2. explain查看执行计划，是否与1预期一致（从锁定记录较少的表开始查询）

3. order by limit 形式的sql语句让排序的表优先查

4. 了解业务方使用场景

5. 加索引时参照建索引的几大原则

6. 观察结果，不符合预期继续从0分析

[explain命令的详细介绍](https://segmentfault.com/a/1190000006726948)

explain就是SQL的执行计划，通过执行计划，我们可以了解sql的执行当中的一些细节。
使用方法为在SQL语句前加explain
得到结果如下：

```sql
mysql> explain select id,c1 from t1 where c1=4398825;
+----+-------------+-------+------+---------------+------+---------+------+---------+-------------+
| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+----+-------------+-------+------+---------------+------+---------+------+---------+-------------+
| 1 | SIMPLE | t1 | ALL | NULL | NULL | NULL | NULL | 4992210 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+---------+-------------+
1 row in set (0.00 sec)
```

各列功能如下：

- id: 按照sql语法解析后分层后的编号，可能重复
- select_type：
  - SIMPLE，简单的select查询,不使用union及子查询
  - PRIMARY，最外层的select查询
  - UNION，UNION 中的第二个或随后的 select 查询,不依赖于外部查询的结果集
  - DEPENDENT UNION，UNION 中的第二个或随后的 select 查询,依赖于外部查询的结果集
  - SUBQUERY，子查询中的第一个 select 查询,不依赖于外部查询的结果集
  - DEPENDENT SUBQUERY，子查询中的第一个 select 查询,依赖于外部查询的结果集
  - DERIVED，用于 from子句里有子查询的情况。 MySQL会递归执行这些子查询, 把结果放在临时表里。
  - UNCACHEABLE SUBQUERY，结果集不能被缓存的子查询,必须重新为外层查询的每一行进行评估。
  - UNCACHEABLE UNION，UNION 中的第二个或随后的 select 查询,属于不可缓存的子查询
- table：涉及的表，如果SQL中表有赋别名，这里出现的是别名
- type：
  - system，从系统表读一行。这是const联接类型的一个特例。
  - const，表最多有一个匹配行,它将在查询开始时被读取。因为仅有一行,在这行的列值可被优化器剩余部分认为是常数。const表很快,因为它们只读取一次!
  - eq_ref，查询条件为等于
  - ref，条件查询不等于
  - ref_or_null，同ref(条件查询),包含NULL值的行。
  - index_merge，索引联合查询
  - unique_subquery，利用唯一索引进行子查询
  - index_subquery，用非唯一索引进行子查询
  - range，索引范围扫描
  - index，索引全扫描
  - ALL，全表扫描。
- possible_keys：可能使用的索引
- key：sql中使用的索引
- key_len：索引长度
- ref：使用哪个列或常数与key一起从表中选择行。
- rows：显示MYSQL执行查询的行数，简单且重要，数值越大越不好，说明没有用好索引
- Extra：该列包含MySQL解决查询的详细信息。
  - Distinct,去重，返回第一个满足条件的值
  - Not exists 使用not exists查询
  - Range checked for each record,有索引，但索引选择率很低
  - Using filesort,有序查询
  - Using index,索引全扫描
  - Using index condition,索引查询
  - Using temporary,临表表检索
  - Using where,where条件查询
  - Using sort_union,有序合并查询
  - Using union,合并查询
  - Using intersect,索引交叉合并
  - Impossible WHERE noticed after reading const tables,读取const tables,查询结果为空
  - No tables used,没有使用表
  - Using join buffer (Block Nested Loop),使用join buffer(BNL算法)
  - Using MRR(Multi-Range Read ) 使用辅助索引进行多范围读

# 日志

[看这里](https://blog.csdn.net/u010002184/article/details/88526708?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

[日志发生时机](https://blog.csdn.net/zhaoliang831214/article/details/91125749?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

## redo日志

redo log是InnoDB存储引擎层的日志，又称重做日志文件，用于记录事务操作的变化，记录的是数据修改之后的值，不管事务是否提交都会记录下来。在实例和介质失败（media failure）时，redo log文件就能派上用场，如数据库掉电，InnoDB存储引擎会使用redo log恢复到掉电前的时刻，以此来保证数据的完整性。



在一条更新语句进行执行的时候，InnoDB引擎会把更新记录写到redo log日志中，然后更新内存，此时算是语句执行完了，然后在空闲的时候或者是按照设定的更新策略将redo log中的内容更新到磁盘中，

**关键点是先写日志，再写磁盘** （知道这个就够了）

有了redo log日志，那么在数据库进行异常重启的时候，可以根据redo log日志进行恢复，

redo log日志的大小是固定的，即记录满了以后就从头循环写。

![img](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/181121105137361.jpg)

该图展示了一组4个文件的redo log日志，checkpoint之前表示擦除完了的，即可以进行写的，擦除之前会更新到磁盘中，write pos是指写的位置，当write pos和checkpoint相遇的时候表明redo log已经满了，这个时候数据库停止进行数据库更新语句的执行，转而进行redo log日志同步到磁盘中。



**作用：**

**确保事务的持久性。**

防止在发生故障的时间点，尚有脏页未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的持久性这一特性。

**什么时候产生：**

事务开始之后就产生redo log，redo log的落盘并不是随着事务的提交才写入的，而是在事务的执行过程中，便开始写入redo log文件中。



## undo日志

undo log是回滚日志，提供回滚操作

**作用：**

保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读

**什么时候产生：**

事务开始之前，将当前是的版本生成undo log，undo 也会产生 redo 来保证undo log的可靠性

**什么时候释放：**

当事务提交之后，undo log并不能立马被删除，
而是放入待清理的链表，由purge线程判断是否由其他事务在使用undo段中表的上一个事务之前的版本信息，决定是否可以清理undo log的日志空间。

## binlog日志

binlog是属于MySQL Server层面的，又称为归档日志，属于逻辑日志，是以二进制的形式记录的是这个语句的原始逻辑。

**作用：**

用于复制，在主从复制中，从库利用主库上的binlog进行重播，实现主从同步。 
用于数据库的基于时间点的还原。

**什么时候产生：**

事务提交的时候，一次性将事务中的sql语句（一个事物可能对应多个sql语句）按照一定的格式记录到binlog中。

### redo log和binlog区别

- redo log是属于innoDB层面，binlog属于MySQL Server层面的，这样在数据库用别的存储引擎时可以达到一致性的要求。
- redo log是物理日志，记录该数据页更新的内容；binlog是逻辑日志，记录的是这个更新语句的原始逻辑
- redo log是循环写，日志空间大小固定；binlog是追加写，是指一份写到一定大小的时候会更换下一个文件，不会覆盖。
- binlog可以作为恢复数据使用，主从复制搭建，redo log作为异常宕机或者介质故障后的数据恢复使用。



# 大表优化

## 1. 限定数据的范围

务必禁止不带任何限制数据范围条件的查询语句。

## 2. 读/写分离

经典的数据库拆分方案，主库负责写，从库负责读；

## 3. 垂直分区

**简单来说垂直拆分是指数据表列的拆分，把一张列比较多的表拆分为多张表。**

![数据库垂直分区](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/数据库垂直分区.png)

- **垂直拆分的优点：** 可以使得列数据变小，在查询时减少读取的Block数，减少I/O次数。此外，垂直分区可以简化表的结构，易于维护。
- **垂直拆分的缺点：** 主键会出现冗余，需要管理冗余列，并会引起Join操作，可以通过在应用层进行Join来解决。此外，垂直分区会让事务变得更加复杂；

## 4. 水平分区

**保持数据表结构不变，通过某种策略存储数据分片。这样每一片数据分散到不同的表或者库中，达到了分布式的目的。 水平拆分可以支撑非常大的数据量。**

把一张的表的数据拆成多张表来存放

![数据库水平拆分](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/数据库水平拆分.png)

# 数据库连接池

### 解释一下什么是池化设计思想。什么是数据库连接池?为什么需要数据库连接池?

数据库连接本质就是一个 socket 的连接。数据库服务端还要维护一些缓存和用户权限信息之类的 所以占用了一些内存。我们可以把数据库连接池是看做是维护的数据库连接的缓存，以便将来需要对数据库的请求时可以重用这些连接。为每个用户打开和维护数据库连接，尤其是对动态数据库驱动的网站应用程序的请求，既昂贵又浪费资源。**在连接池中，创建连接后，将其放置在池中，并再次使用它，因此不必建立新的连接。如果使用了所有连接，则会建立一个新连接并将其添加到池中**。 连接池还减少了用户必须等待建立与数据库的连接的时间。

**三个重要参数：**

- 最大链接数
- 核心链接数
- 最大等待时间

# 数据库语法知识

## 自然联结，内联结，外联结

[文章说明](https://blog.csdn.net/weixin_40673608/article/details/88675051?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param)

### 1. 自然连接

定义：无论何时对表进行联结，应该至少有一个列出现不止一个表中（被联结的列）。标准的联结返回所有数据，甚至相同的列多次出现。自然联结排除多次出现，使每个列只返回一次；

例如有表R和表S，表的内容如下：

~~~xml
  R表                	       S表  
A  B  C                      D  B  E  
1  a  3                      2  c  7  
2  b  6                      3  d  5  
3  c  7                      1  a  3
~~~

现在R和S要进行自然联结。

自然连接不用指定连接列，也不能使用ON语句，它默认比较两张表里相同的列。

SQL语句如下：

~~~sql
 `SELECT * FROM A NATURAL JOIN S;`
~~~

自然联结步骤

1.就是用R表中的每一项乘以S表中的每一项，得到的结果如下

~~~xml
A   B     C      D     B       E  
1   a     3      2     c       7  
1   a     3      3     d       5  
1   a     3      1     a       3  
2   b     6      2     c       7  
2   b     6      3     d       5  
2   b     6      1     a       3  
3   c     7      2     c       7  
3   c     7      3     d       5   
3   c     7      1     a       3 
~~~

2.过滤出R.B=S.B的行，结果如下

~~~xml
R.A    R.B     R.C       S.D     S.B    S.E  
1       a       3         1       a       3   
3       c       7         2       c       7 
~~~

3.最后去除一个相同的列，即R.B和S.B其中一个，最后自然联结的结果为

~~~xml
A     B      C      D       E  
1     a      3      1        3  
3     c      7      2        7
~~~

### 2. 内联结

内联结最简单，两个表进行内联结，匹配符合过滤条件的行就可以了

例如SQL：

~~~sql
select * from A inner join B where A.a = B.a
~~~

**把表A和表B中的列A相等的所有行都显示出来。**

**和自然连接的区别：**

1. 使用 inner join 
2. 需要使用 on 指定联结条件
3. 不需要要求两个表的用于连接的列名相同

### 3. 外联结

#### 3.1 左外联结（left outer join)

左外连接是将两表进行自然连接，然后把左表全部保留在结果集中，右表不满足条件的对应的列上填null
sql语句：

~~~sql
Select …… from 表1 left outer join 表2 on 表1.C=表2.C
~~~

#### 3.2 右外联结(rignt outer join)

右外连接是将两表进行自然连接，把右表全部保留在结果集中，左表不满足条件的对应的列上填null。

~~~sql
Select …… from 表1 rignt outer join 表2 on 表1.C=表2.C
~~~

#### 3.3 全外联结(full join)

全外连接是在两表进行自然连接，只把左表和右表满足相等条件的都保留在结果集中，不满足相等条件的相对应的列上填null。

~~~sql
Select …… from 表1 full join 表2 on 表1.C=表2.C
~~~



# 经典面试题

## 一条SQL语句执行得很慢的原因有哪些？

[原文在这里](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485185&idx=1&sn=66ef08b4ab6af5757792223a83fc0d45&chksm=cea248caf9d5c1dc72ec8a281ec16aa3ec3e8066dbb252e27362438a26c33fbe842b0e0adf47&token=79317275&lang=zh_CN#rd)

一个 SQL 执行的很慢，我们要分两种情况讨论：

1、大多数情况下很正常，偶尔很慢，则有如下原因

(1)、数据库在刷新脏页，例如 redo log 写满了需要同步到磁盘。

(2)、执行的时候，遇到锁，如表锁、行锁。

2、这条 SQL 语句一直执行的很慢，则有如下原因。

(1)、没有用上索引：例如该字段没有索引；由于对字段进行运算、函数操作导致无法用索引。

(2)、数据库选错了索引。

## having与where的区别？

[自己的总结](面经.md)

