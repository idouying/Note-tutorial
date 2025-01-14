# 关系型数据库面试题

<!-- TOC depthFrom:2 depthTo:2 -->

- [1. 概念](#1-概念)
- [2. SQL](#2-sql)
- [3. 索引和约束](#3-索引和约束)
- [4. 事务](#4-事务)
- [5. 锁](#5-锁)
- [6. 分库分表](#6-分库分表)
- [7. 数据库架构设计](#7-数据库架构设计)
- [8. 参考资料](#8-参考资料)
- [9. :door: 传送门](#9-door-传送门)

<!-- /TOC -->

## 1. 概念

### 1.1.1. 什么是存储过程？有哪些优缺点？

**存储过程就像我们编程语言中的函数一样，封装了我们的代码(PLSQL、T-SQL)**。

优点：

- **能够将代码封装起来**
- **保存在数据库之中**
- **让编程语言进行调用**
- **存储过程是一个预编译的代码块，执行效率比较高**
- **一个存储过程替代大量 T_SQL 语句 ，可以降低网络通信量，提高通信速率**

缺点：

- **每个数据库的存储过程语法几乎都不一样，十分难以维护（不通用）**
- **业务逻辑放在数据库上，难以迭代**

示例：

```sql
DELIMITER //
CREATE PROCEDURE phelloword()
BEGIN
SELECT * FROM admin;
END//
CALL phelloword()
```

### 1.1.2. 什么是视图？以及视图的使用场景有哪些？

视图是一种基于数据表的一种**虚表**

- 视图是一种虚表
- 视图建立在已有表的基础上, 视图赖以建立的这些表称为基表
- **向视图提供数据内容的语句为 SELECT 语句,可以将视图理解为存储起来的 SELECT 语句**
- 视图向用户提供基表数据的另一种表现形式
- 视图没有存储真正的数据，真正的数据还是存储在基表中
- 程序员虽然操作的是视图，但最终视图还会转成操作基表
- 一个基表可以有 0 个或多个视图

有的时候，我们可能只关系一张数据表中的某些字段，而另外的一些人只关系同一张数据表的某些字段...

那么把全部的字段都都显示给他们看，这是不合理的。

我们应该做到：**他们想看到什么样的数据，我们就给他们什么样的数据...一方面就能够让他们只关注自己的数据，另一方面，我们也保证数据表一些保密的数据不会泄露出来...**

<div align="center"><img src="https://user-gold-cdn.xitu.io/2018/3/5/161f3de9b3092439?imageView2/0/w/1280/h/960/format/webp/ignore-error/1"/></div>

我们在查询数据的时候，常常需要编写非常长的 SQL 语句，几乎每次都要写很长很长....上面已经说了，**视图就是基于查询的一种虚表，也就是说，视图可以将查询出来的数据进行封装。。。那么我们在使用的时候就会变得非常方便**...

值得注意的是：**使用视图可以让我们专注与逻辑，但不提高查询效率**

## 2. SQL

### 1.2.1. drop、delete 与 truncate 分别在什么场景之下使用？

- drop table

  - 属于 DDL
  - 不可回滚
  - 不可带 where
  - 表内容和结构删除
  - 删除速度快

- truncate table

  - 属于 DDL
  - 不可回滚
  - 不可带 where
  - 表内容删除
  - 删除速度快

- delete from
  - 属于 DML
  - 可回滚
  - 可带 where
  - 表结构在，表内容要看 where 执行的情况
  - 删除速度慢,需要逐行删除
  - **不再需要一张表的时候，用 drop**
  - **想删除部分数据行时候，用 delete，并且带上 where 子句**
  - **保留表而删除所有数据的时候用 truncate**

## 3. 索引和约束

### SQL 约束有哪几种？

- NOT NULL: 用于控制字段的内容一定不能为空（NULL）。
- UNIQUE: 控件字段内容不能重复，一个表允许有多个 Unique 约束。
- PRIMARY KEY: 也是用于控件字段内容不能重复，但它在一个表只允许出现一个。
- FOREIGN KEY: 用于预防破坏表之间连接的动作，也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。
- CHECK: 用于控制字段的值范围。

### 超键、候选键、主键、外键分别是什么？

- 超键：**在关系中能唯一标识元组的属性集称为关系模式的超键**。一个属性可以为作为一个超键，多个属性组合在一起也可以作为一个超键。**超键包含候选键和主键**。
- **候选键(候选码)：是最小超键，即没有冗余元素的超键**。
- **主键(主码)：数据库表中对储存数据对象予以唯一和完整标识的数据列或属性的组合**。一个数据列只能有一个主键，且主键的取值不能缺失，即不能为空值（Null）。
- **外键：在一个表中存在的另一个表的主键称此表的外键**。

### 1.3.1. 数据库索引有哪些数据结构？

- B-Tree
- B+Tree
- Hash

#### 1.3.1.1. B-Tree

一棵 M 阶的 B-Tree 满足以下条件：

- 每个结点至多有 M 个孩子；
- 除根结点和叶结点外，其它每个结点至少有 M/2 个孩子；
- 根结点至少有两个孩子（除非该树仅包含一个结点）；
- 所有叶结点在同一层，叶结点不包含任何关键字信息；
- 有 K 个关键字的非叶结点恰好包含 K+1 个孩子；

对于任意结点，其内部的关键字 Key 是升序排列的。每个节点中都包含了 data。

<div align="center">
<img src="http://dunwu.test.upcdn.net/cs/database/RDB/B-TREE.png!zp" />
</div>

对于每个结点，主要包含一个关键字数组 Key[]，一个指针数组（指向儿子）Son[]。

在 B-Tree 内，查找的流程是：

1.  使用顺序查找（数组长度较短时）或折半查找方法查找 Key[]数组，若找到关键字 K，则返回该结点的地址及 K 在 Key[]中的位置；
2.  否则，可确定 K 在某个 Key[i]和 Key[i+1]之间，则从 Son[i]所指的子结点继续查找，直到在某结点中查找成功；
3.  或直至找到叶结点且叶结点中的查找仍不成功时，查找过程失败。

#### 1.3.1.2. B+Tree

B+Tree 是 B-Tree 的变种：

- 每个节点的指针上限为 2d 而不是 2d+1（d 为节点的出度）。
- 非叶子节点不存储 data，只存储 key；叶子节点不存储指针。

<div align="center">
<img src="http://dunwu.test.upcdn.net/cs/database/RDB/B+TREE.png!zp" />
</div>

由于并不是所有节点都具有相同的域，因此 B+Tree 中叶节点和内节点一般大小不同。这点与 B-Tree 不同，虽然 B-Tree 中不同节点存放的 key 和指针可能数量不一致，但是每个节点的域和上限是一致的，所以在实现中 B-Tree 往往对每个节点申请同等大小的空间。

##### 1.3.1.2.1. 带有顺序访问指针的 B+Tree

一般在数据库系统或文件系统中使用的 B+Tree 结构都在经典 B+Tree 的基础上进行了优化，增加了顺序访问指针。

<div align="center">
<img src="http://dunwu.test.upcdn.net/cs/database/RDB/带有顺序访问指针的B+Tree.png!zp" />
</div>

在 B+Tree 的每个叶子节点增加一个指向相邻叶子节点的指针，就形成了带有顺序访问指针的 B+Tree。

这个优化的目的是为了提高区间访问的性能，例如上图中如果要查询 key 为从 18 到 49 的所有数据记录，当找到 18 后，只需顺着节点和指针顺序遍历就可以一次性访问到所有数据节点，极大提到了区间查询效率。

#### 1.3.1.3. Hash

Hash 索引只有精确匹配索引所有列的查询才有效。

对于每一行数据，对所有的索引列计算一个 hashcode。哈希索引将所有的 hashcode 存储在索引中，同时在 Hash 表中保存指向每个数据行的指针。

哈希索引的优点：

- 因为索引数据结构紧凑，所以查询速度非常快。

哈希索引的缺点：

- 哈希索引数据不是按照索引值顺序存储的，所以无法用于排序。
- 哈希索引不支持部分索引匹配查找。如，在数据列 (A,B) 上建立哈希索引，如果查询只有数据列 A，无法使用该索引。
- 哈希索引只支持等值比较查询，不支持任何范围查询，如 WHERE price > 100。
- 哈希索引有可能出现哈希冲突，出现哈希冲突时，必须遍历链表中所有的行指针，逐行比较，直到找到符合条件的行。

### 1.3.2. B-Tree 和 B+Tree 有什么区别？

- B+Tree 更适合外部存储(一般指磁盘存储)，由于内节点(非叶子节点)不存储 data，所以一个节点可以存储更多的内节点，每个节点能索引的范围更大更精确。也就是说使用 B+Tree 单次磁盘 IO 的信息量相比较 B-Tree 更大，IO 效率更高。
- mysql 是关系型数据库，经常会按照区间来访问某个索引列，B+Tree 的叶子节点间按顺序建立了链指针，加强了区间访问性，所以 B+Tree 对索引列上的区间范围查询很友好。而 B-Tree 每个节点的 key 和 data 在一起，无法进行区间查找。

### 1.3.3. 索引原则有哪些？

#### 1.3.3.1. 独立的列

如果查询中的列不是独立的列，则数据库不会使用索引。

“独立的列” 是指索引列不能是表达式的一部分，也不能是函数的参数。

:x: 错误示例：

```sql
SELECT actor_id FROM sakila.actor WHERE actor_id + 1 = 5;
SELECT ... WHERE TO_DAYS(CURRENT_DAT) - TO_DAYS(date_col) <= 10;
```

#### 1.3.3.2. 前缀索引和索引选择性

有时候需要索引很长的字符列，这会让索引变得大且慢。

解决方法是：可以索引开始的部分字符，这样可以大大节约索引空间，从而提高索引效率。但这样也会降低索引的选择性。

索引的选择性是指：不重复的索引值和数据表记录总数的比值。最大值为 1，此时每个记录都有唯一的索引与其对应。选择性越高，查询效率也越高。

对于 BLOB/TEXT/VARCHAR 这种文本类型的列，必须使用前缀索引，因为数据库往往不允许索引这些列的完整长度。

要选择足够长的前缀以保证较高的选择性，同时又不能太长（节约空间）。

#### 1.3.3.3. 多列索引

不要为每个列创建独立的索引。

#### 1.3.3.4. 选择合适的索引列顺序

经验法则：将选择性高的列或基数大的列优先排在多列索引最前列。

但有时，也需要考虑 WHERE 子句中的排序、分组和范围条件等因素，这些因素也会对查询性能造成较大影响。

#### 1.3.3.5. 聚簇索引

聚簇索引不是一种单独的索引类型，而是一种数据存储方式。

聚簇表示数据行和相邻的键值紧凑地存储在一起。因为无法同时把数据行存放在两个不同的地方，所以一个表只能有一个聚簇索引。

#### 1.3.3.6. 覆盖索引

索引包含所有需要查询的字段的值。

具有以下优点：

- 因为索引条目通常远小于数据行的大小，所以若只读取索引，能大大减少数据访问量。
- 一些存储引擎（例如 MyISAM）在内存中只缓存索引，而数据依赖于操作系统来缓存。因此，只访问索引可以不使用系统调用（通常比较费时）。
- 对于 InnoDB 引擎，若辅助索引能够覆盖查询，则无需访问主索引。

#### 1.3.3.7. 使用索引扫描来做排序

索引最好既满足排序，又用于查找行。这样，就可以使用索引来对结果排序。

#### 1.3.3.8. = 和 in 可以乱序

比如 a = 1 and b = 2 and c = 3 建立（a,b,c）索引可以任意顺序，mysql 的查询优化器会帮你优化成索引可以识别的形式。

#### 1.3.3.9. 尽量的扩展索引，不要新建索引

比如表中已经有 a 的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

## 4. 事务

### 1.4.1. 什么是事务？

事务简单来说：**一个 Session 中所进行所有的操作，要么同时成功，要么同时失败**

**ACID — 数据库事务正确执行的四个基本要素**

- 原子性（Atomicity）
- 一致性（Consistency）
- 隔离性（Isolation）
- 持久性（Durability）

**一个支持事务（Transaction）中的数据库系统，必需要具有这四种特性，否则在事务过程（Transaction processing）当中无法保证数据的正确性，交易过程极可能达不到交易。**

### 1.4.2. 数据库事务隔离级别？事务隔离级别分别解决什么问题？

- `未提交读（READ UNCOMMITTED）` - 事务中的修改，即使没有提交，对其它事务也是可见的。
- `提交读（READ COMMITTED）` - 一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其它事务是不可见的。
- `可重复读（REPEATABLE READ）` - 保证在同一个事务中多次读取同样数据的结果是一样的。
- `可串行化（SERIALIXABLE）` - 强制事务串行执行。

| 隔离级别 | 脏读 | 不可重复读 | 幻影读 |
| :------: | :--: | :--------: | :----: |
| 未提交读 | YES  |    YES     |  YES   |
|  提交读  |  NO  |    YES     |  YES   |
| 可重复读 |  NO  |     NO     |  YES   |
| 可串行化 |  NO  |     NO     |   NO   |

- `脏读` - **一个事务读取到另外一个事务未提交的数据**
- `不可重复读` - **一个事务读取到另外一个事务已经提交的数据，也就是说一个事务可以看到其他事务所做的修改**
- `虚读(幻读)` - **是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。**

### 1.4.3. 如何解决分布式事务？若出现网络问题或宕机问题，如何解决？

## 5. 锁

### 1.5.1. 数据库的乐观锁和悲观锁是什么？

确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性以及数据库的统一性，**乐观锁和悲观锁是并发控制主要采用的技术手段。**

- **`悲观锁`** - 假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作
  - **在查询完数据的时候就把事务锁起来，直到提交事务**
  - 实现方式：使用数据库中的锁机制
- **`乐观锁`** - 假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。
  - **在修改数据的时候把事务锁起来，通过 version 的方式来进行锁定**
  - 实现方式：使用 version 版本或者时间戳

### 1.5.2. 数据库锁有哪些类型？如何实现？

#### 1.5.2.1. 锁粒度

- **表级锁（table lock）** - 锁定整张表。用户对表进行写操作前，需要先获得写锁，这会阻塞其他用户对该表的所有读写操作。只有没有写锁时，其他用户才能获得读锁，读锁之间不会相互阻塞。
- **行级锁（row lock）** - 仅对指定的行记录进行加锁，这样其它进程还是可以对同一个表中的其它记录进行操作。

InnoDB 行锁是通过给索引上的索引项加锁来实现的。只有通过索引条件检索数据，InnoDB 才使用行级锁；否则，InnoDB 将使用表锁！

索引分为主键索引和非主键索引两种，如果一条 sql 语句操作了主键索引，MySQL 就会锁定这条主键索引；如果一条语句操作了非主键索引，MySQL 会先锁定该非主键索引，再锁定相关的主键索引。在 UPDATE、DELETE 操作时，MySQL 不仅锁定 WHERE 条件扫描过的所有索引记录，而且会锁定相邻的键值，即所谓的 next-key locking。

当两个事务同时执行，一个锁住了主键索引，在等待其他相关索引。另一个锁定了非主键索引，在等待主键索引。这样就会发生死锁。发生死锁后，InnoDB 一般都可以检测到，并使一个事务释放锁回退，另一个获取锁完成事务。

#### 1.5.2.2. 读写锁

- 排它锁（Exclusive），简写为 X 锁，又称写锁。
- 共享锁（Shared），简写为 S 锁，又称读锁。

有以下两个规定：

- 一个事务对数据对象 A 加了 X 锁，就可以对 A 进行读取和更新。加锁期间其它事务不能对 A 加任何锁。
- 一个事务对数据对象 A 加了 S 锁，可以对 A 进行读取操作，但是不能进行更新操作。加锁期间其它事务能对 A 加 S 锁，但是不能加 X 锁。

锁的兼容关系如下：

|  -  |  X  |  S  |
| :-: | :-: | :-: |
|  X  | NO  | NO  |
|  S  | NO  | YES |

使用：

- 排他锁：`SELECT ... FOR UPDATE;`
- 共享锁：`SELECT ... LOCK IN SHARE MODE;`

innodb 下的记录锁（也叫行锁），间隙锁，next-key 锁统统属于排他锁。

在 InnoDB 中，行锁是通过给索引上的索引项加锁来实现的。如果没有索引，InnoDB 将会通过隐藏的聚簇索引来对记录加锁。另外，根据针对 sql 语句检索条件的不同，加锁又有以下三种情形需要我们掌握。

1.  Record lock：对索引项加锁。若没有索引项则使用表锁。
2.  Gap lock：对索引项之间的间隙加锁。
3.  Next-key lock：1+2，锁定一个范围，并且锁定记录本身。对于行的查询，都是采用该方法，主要目的是解决幻读的问题。当利用范围条件而不是相等条件获取排他锁时，innoDB 会给符合条件的所有数据加锁。对于在条件范围内但是不存在的记录，叫做间隙。innoDB 也会对这个间隙进行加锁。另外，使用相等的检索条件时，若指定了本身不存在的记录作为检索条件的值的话，则此值对应的索引项也会加锁。

#### 1.5.2.3. 意向锁

使用意向锁（Intention Locks）可以更容易地支持多粒度封锁。

在存在行级锁和表级锁的情况下，事务 T 想要对表 A 加 X 锁，就需要先检测是否有其它事务对表 A 或者表 A 中的任意一行加了锁，那么就需要对表 A 的每一行都检测一次，这是非常耗时的。

意向锁在原来的 X/S 锁之上引入了 IX/IS，IX/IS 都是表锁，用来表示一个事务想要在表中的某个数据行上加 X 锁或 S 锁。有以下两个规定：

- 一个事务在获得某个数据行对象的 S 锁之前，必须先获得表的 IS 锁或者更强的锁；
- 一个事务在获得某个数据行对象的 X 锁之前，必须先获得表的 IX 锁。

通过引入意向锁，事务 T 想要对表 A 加 X 锁，只需要先检测是否有其它事务对表 A 加了 X/IX/S/IS 锁，如果加了就表示有其它事务正在使用这个表或者表中某一行的锁，因此事务 T 加 X 锁失败。

各种锁的兼容关系如下：

|  -  |  X  | IX  |  S  | IS  |
| :-: | :-: | :-: | :-: | :-: |
|  X  | NO  | NO  | NO  | NO  |
| IX  | NO  | YES | NO  | YES |
|  S  | NO  | NO  | YES | YES |
| IS  | NO  | YES | YES | YES |

解释如下：

- 任意 IS/IX 锁之间都是兼容的，因为它们只是表示想要对表加锁，而不是真正加锁；
- S 锁只与 S 锁和 IS 锁兼容，也就是说事务 T 想要对数据行加 S 锁，其它事务可以已经获得对表或者表中的行的 S 锁。

意向锁是 InnoDB 自动加的，不需要用户干预。

## 6. 分库分表

### 1.6.1. 为什么要分库分表？

分库分表的基本思想就要把一个数据库切分成多个部分放到不同的数据库(server)上，从而缓解单一数据库的性能问题。

分库分表一定是为了**支撑高并发、数据量大**两个问题的。

#### 1.6.1.1. 分表

比如你单表都几千万数据了，你确定你能扛住么？绝对不行，**单表数据量太大**，会极大影响你的 sql **执行的性能**，到了后面你的 sql 可能就跑的很慢了。一般来说，就以我的经验来看，单表到几百万的时候，性能就会相对差一些了，你就得分表了。

分表是啥意思？就是把一个表的数据放到多个表中，然后查询的时候你就查一个表。比如按照用户 id 来分表，将一个用户的数据就放在一个表中。然后操作的时候你对一个用户就操作那个表就好了。这样可以控制每个表的数据量在可控的范围内，比如每个表就固定在 200 万以内。

#### 1.6.1.2. 分库

分库是啥意思？就是你一个库一般我们经验而言，最多支撑到并发 2000，一定要扩容了，而且一个健康的单库并发值你最好保持在每秒 1000 左右，不要太大。那么你可以将一个库的数据拆分到多个库中，访问的时候就访问一个库好了。

这就是所谓的**分库分表**，为啥要分库分表？你明白了吧。

| #            | 分库分表前                   | 分库分表后                                   |
| ------------ | ---------------------------- | -------------------------------------------- |
| 并发支撑情况 | MySQL 单机部署，扛不住高并发 | MySQL 从单机到多机，能承受的并发增加了多倍   |
| 磁盘使用情况 | MySQL 单机磁盘容量几乎撑满   | 拆分为多个库，数据库服务器磁盘使用率大大降低 |
| SQL 执行性能 | 单表数据量太大，SQL 越跑越慢 | 单表数据量减少，SQL 执行效率明显提升         |

### 1.6.2. 用过哪些分库分表中间件？不同的分库分表中间件都有什么优点和缺点？

这个其实就是看看你了解哪些分库分表的中间件，各个中间件的优缺点是啥？然后你用过哪些分库分表的中间件。

比较常见的包括：

- cobar
- TDDL
- atlas
- sharding-jdbc
- mycat

#### 1.6.2.1. cobar

阿里 b2b 团队开发和开源的，属于 proxy 层方案，就是介于应用服务器和数据库服务器之间。应用程序通过 JDBC 驱动访问 cobar 集群，cobar 根据 SQL 和分库规则对 SQL 做分解，然后分发到 MySQL 集群不同的数据库实例上执行。早些年还可以用，但是最近几年都没更新了，基本没啥人用，差不多算是被抛弃的状态吧。而且不支持读写分离、存储过程、跨库 join 和分页等操作。

#### 1.6.2.2. TDDL

淘宝团队开发的，属于 client 层方案。支持基本的 crud 语法和读写分离，但不支持 join、多表查询等语法。目前使用的也不多，因为还依赖淘宝的 diamond 配置管理系统。

#### 1.6.2.3. atlas

360 开源的，属于 proxy 层方案，以前是有一些公司在用的，但是确实有一个很大的问题就是社区最新的维护都在 5 年前了。所以，现在用的公司基本也很少了。

#### 1.6.2.4. sharding-jdbc

当当开源的，属于 client 层方案。确实之前用的还比较多一些，因为 SQL 语法支持也比较多，没有太多限制，而且目前推出到了 2.0 版本，支持分库分表、读写分离、分布式 id 生成、柔性事务（最大努力送达型事务、TCC 事务）。而且确实之前使用的公司会比较多一些（这个在官网有登记使用的公司，可以看到从 2017 年一直到现在，是有不少公司在用的），目前社区也还一直在开发和维护，还算是比较活跃，个人认为算是一个现在也**可以选择的方案**。

#### 1.6.2.5. mycat

基于 cobar 改造的，属于 proxy 层方案，支持的功能非常完善，而且目前应该是非常火的而且不断流行的数据库中间件，社区很活跃，也有一些公司开始在用了。但是确实相比于 sharding jdbc 来说，年轻一些，经历的锤炼少一些。

#### 1.6.2.6. 总结

综上，现在其实建议考量的，就是 sharding-jdbc 和 mycat，这两个都可以去考虑使用。

sharding-jdbc 这种 client 层方案的**优点在于不用部署，运维成本低，不需要代理层的二次转发请求，性能很高**，但是如果遇到升级啥的需要各个系统都重新升级版本再发布，各个系统都需要**耦合** sharding-jdbc 的依赖；

mycat 这种 proxy 层方案的**缺点在于需要部署**，自己运维一套中间件，运维成本高，但是**好处在于对于各个项目是透明的**，如果遇到升级之类的都是自己中间件那里搞就行了。

通常来说，这两个方案其实都可以选用，但是我个人建议中小型公司选用 sharding-jdbc，client 层方案轻便，而且维护成本低，不需要额外增派人手，而且中小型公司系统复杂度会低一些，项目也没那么多；但是中大型公司最好还是选用 mycat 这类 proxy 层方案，因为可能大公司系统和项目非常多，团队很大，人员充足，那么最好是专门弄个人来研究和维护 mycat，然后大量项目直接透明使用即可。

### 1.6.3. 你们具体是如何对数据库如何进行垂直拆分或水平拆分的？

**水平拆分**的意思，就是把一个表的数据给弄到多个库的多个表里去，但是每个库的表结构都一样，只不过每个库表放的数据是不同的，所有库表的数据加起来就是全部数据。水平拆分的意义，就是将数据均匀放更多的库里，然后用多个库来扛更高的并发，还有就是用多个库的存储容量来进行扩容。

<div align="center"><img src="https://github.com/doocs/advanced-java/blob/master/images/database-split-horizon.png"/></div>

**垂直拆分**的意思，就是**把一个有很多字段的表给拆分成多个表**，**或者是多个库上去**。每个库表的结构都不一样，每个库表都包含部分字段。一般来说，会**将较少的访问频率很高的字段放到一个表里去**，然后**将较多的访问频率很低的字段放到另外一个表里去**。因为数据库是有缓存的，你访问频率高的行字段越少，就可以在缓存里缓存更多的行，性能就越好。这个一般在表层面做的较多一些。

<div align="center"><img src="https://github.com/doocs/advanced-java/blob/master/images/database-split-vertically.png"/></div>

这个其实挺常见的，不一定我说，大家很多同学可能自己都做过，把一个大表拆开，订单表、订单支付表、订单商品表。

还有**表层面的拆分**，就是分表，将一个表变成 N 个表，就是**让每个表的数据量控制在一定范围内**，保证 SQL 的性能。否则单表数据量越大，SQL 性能就越差。一般是 200 万行左右，不要太多，但是也得看具体你怎么操作，也可能是 500 万，或者是 100 万。你的 SQL 越复杂，就最好让单表行数越少。

好了，无论分库还是分表，上面说的那些数据库中间件都是可以支持的。就是基本上那些中间件可以做到你分库分表之后，**中间件可以根据你指定的某个字段值**，比如说 userid，**自动路由到对应的库上去，然后再自动路由到对应的表里去**。

你就得考虑一下，你的项目里该如何分库分表？一般来说，垂直拆分，你可以在表层面来做，对一些字段特别多的表做一下拆分；水平拆分，你可以说是并发承载不了，或者是数据量太大，容量承载不了，你给拆了，按什么字段来拆，你自己想好；分表，你考虑一下，你如果哪怕是拆到每个库里去，并发和容量都 ok 了，但是每个库的表还是太大了，那么你就分表，将这个表分开，保证每个表的数据量并不是很大。

而且这儿还有两种**分库分表的方式**：

- 一种是按照 range 来分，就是每个库一段连续的数据，这个一般是按比如**时间范围**来的，但是这种一般较少用，因为很容易产生热点问题，大量的流量都打在最新的数据上了。
- 或者是按照某个字段 hash 一下均匀分散，这个较为常用。

range 来分，好处在于说，扩容的时候很简单，因为你只要预备好，给每个月都准备一个库就可以了，到了一个新的月份的时候，自然而然，就会写新的库了；缺点，但是大部分的请求，都是访问最新的数据。实际生产用 range，要看场景。

hash 分发，好处在于说，可以平均分配每个库的数据量和请求压力；坏处在于说扩容起来比较麻烦，会有一个数据迁移的过程，之前的数据需要重新计算 hash 值重新分配到不同的库或表。

### 1.6.4. 分库分表的常见问题以及解决方案？

#### 1.6.4.1. 事务问题

方案一：使用分布式事务

- 优点：交由数据库管理，简单有效
- 缺点：性能代价高，特别是 shard 越来越多时

方案二：由应用程序和数据库共同控制

- 原理：将一个跨多个数据库的分布式事务分拆成多个仅处于单个数据库上面的小事务，并通过应用程序来总控各个小事务。
- 优点：性能上有优势
- 缺点：需要应用程序在事务控制上做灵活设计。如果使用了 spring 的事务管理，改动起来会面临一定的困难。

#### 1.6.4.2. 跨节点 Join 的问题

只要是进行切分，跨节点 Join 的问题是不可避免的。但是良好的设计和切分却可以减少此类情况的发生。解决这一问题的普遍做法是分两次查询实现。在第一次查询的结果集中找出关联数据的 id，根据这些 id 发起第二次请求得到关联数据。

#### 1.6.4.3. 跨节点的 count,order by,group by 以及聚合函数问题

这些是一类问题，因为它们都需要基于全部数据集合进行计算。多数的代理都不会自动处理合并工作。

解决方案：与解决跨节点 join 问题的类似，分别在各个节点上得到结果后在应用程序端进行合并。和 join 不同的是每个节点的查询可以并行执行，因此很多时候它的速度要比单一大表快很多。但如果结果集很大，对应用程序内存的消耗是一个问题。

#### 1.6.4.4. ID 唯一性

一旦数据库被切分到多个物理节点上，我们将不能再依赖数据库自身的主键生成机制。一方面，某个分区数据库自生成的 ID 无法保证在全局上是唯一的；另一方面，应用程序在插入数据之前需要先获得 ID，以便进行 SQL 路由。

一些常见的主键生成策略：

- 使用全局唯一 ID：GUID。
- 为每个分片指定一个 ID 范围。
- 分布式 ID 生成器 (如 Twitter 的 Snowflake 算法)。

#### 1.6.4.5. 数据迁移，容量规划，扩容等问题

来自淘宝综合业务平台团队，它利用对 2 的倍数取余具有向前兼容的特性（如对 4 取余得 1 的数对 2 取余也是 1）来分配数据，避免了行级别的数据迁移，但是依然需要进行表级别的迁移，同时对扩容规模和分表数量都有限制。总得来说，这些方案都不是十分的理想，多多少少都存在一些缺点，这也从一个侧面反映出了 Sharding 扩容的难度。

#### 1.6.4.6. 分库数量

分库数量首先和单库能处理的记录数有关，一般来说，Mysql 单库超过 5000 万条记录，Oracle 单库超过 1 亿条记录，DB 压力就很大(当然处理能力和字段数量/访问模式/记录长度有进一步关系)。

#### 1.6.4.7. 跨分片的排序分页

- 如果是在前台应用提供分页，则限定用户只能看前面 n 页，这个限制在业务上也是合理的，一般看后面的分页意义不大（如果一定要看，可以要求用户缩小范围重新查询）。
- 如果是后台批处理任务要求分批获取数据，则可以加大 page size，比如每次获取 5000 条记录，有效减少分页数（当然离线访问一般走备库，避免冲击主库）。
- 分库设计时，一般还有配套大数据平台汇总所有分库的记录，有些分页查询可以考虑走大数据平台。

### 1.6.5. 如何设计可以动态扩容缩容的分库分表方案？

### 1.6.6. 有哪些分库分表中间件？各自有什么优缺点？底层实现原理？

#### 1.6.6.1. 简单易用的组件：

- [当当 sharding-jdbc](https://github.com/dangdangdotcom/sharding-jdbc)
- [蘑菇街 TSharding](https://github.com/baihui212/tsharding)

#### 1.6.6.2. 强悍重量级的中间件：

- [sharding ](https://github.com/go-pg/sharding)
- [TDDL Smart Client 的方式（淘宝）](https://github.com/alibaba/tb_tddl)
- [Atlas(Qihoo 360)](https://github.com/Qihoo360/Atlas)
- [alibaba.cobar(是阿里巴巴（B2B）部门开发)](https://github.com/alibaba/cobar)
- [MyCAT（基于阿里开源的 Cobar 产品而研发）](http://www.mycat.org.cn/)
- [Oceanus(58 同城数据库中间件)](https://github.com/58code/Oceanus)
- [OneProxy(支付宝首席架构师楼方鑫开发)](http://www.cnblogs.com/youge-OneSQL/articles/4208583.html)
- [vitess（谷歌开发的数据库中间件）](https://github.com/youtube/vitess)

## 7. 数据库架构设计

### 1.7.1. 高并发系统数据层面如何设计？

#### 1.7.1.1. 读写分离的原理

主服务器用来处理写操作以及实时性要求比较高的读操作，而从服务器用来处理读操作。

读写分离常用代理方式来实现，代理服务器接收应用层传来的读写请求，然后决定转发到哪个服务器。

MySQL 读写分离能提高性能的原因在于：

- 主从服务器负责各自的读和写，极大程度缓解了锁的争用；
- 从服务器可以配置 MyISAM 引擎，提升查询性能以及节约系统开销；
- 增加冗余，提高可用性。

<div align="center">
<img src="http://dunwu.test.upcdn.net/cs/database/mysql/master-slave-proxy.png!zp" />
</div>

#### 1.7.1.2. 垂直切分

按照业务线或功能模块拆分为不同数据库。

更进一步是服务化改造，将强耦合的系统拆分为多个服务。

#### 1.7.1.3. 水平切分

- 哈希取模：hash(key) % NUM_DB
- 范围：可以是 ID 范围也可以是时间范围
- 映射表：使用单独的一个数据库来存储映射关系

## 8. 参考资料

[数据库面试题(开发者必看)](https://juejin.im/post/5a9ca0d6518825555c1d1acd)

## 9. :door: 传送门

| [我的 Github 博客](https://github.com/dunwu/blog) | [db-tutorial 首页](https://github.com/dunwu/db-tutorial) |
