# MySQL

[TOC]

## 1 MySQL基础知识

### 1.1 数据库和数据库实例

- **数据库**：是一种操作系统文件或者其他形式的二进制文件的集合。
- **数据库实例**：数据库实例是一种程序，是位于用户和操作系统之间的数据管理软件，所有对数据可数据的操作都是在数据库实例下进行的（借助数据库实例来队数据库文件进行CRUD）

通常情况下，数据库和数据库实例是一对一的关系，在集群情况下也可能存在数据库和数据库实例一对多的情况。

### 1.2 MySQL体系结构

![image-20230414203959260](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230414203959260.png)

数据库结构依次分为：**连接器**、**服务层**、**存储引擎层和文件系统**。

### 1.3 数据可的四大范式

- **第一范式**：数据库表中的字段都是单一属性的，不可再分。即每一列满足原子性。
- **第二范式**：在第一范式的基础上，非主键列完全依赖于主键列，而不能是依赖于主键的一部分
- **第三范式**：在第二范式的基础上，非主键列只能依赖于主键列，不能依赖于其他非主键，即确保每个列和主键直接相关，而不能是间接相关。
- **第四范式**：在第三范式的基础上，消除多值依赖，即如果存在复合关键字，则关键字之间不能存在依赖，即主键列之间不存在依赖关系。

### 1.4 MySQL基本语法

SQL语句主要分为四大类：

- **DDL**：数据定义语句，创建/定义数据库和表
- **DML**：数据操纵语言，主要队数据进行插入/删除/更新
- **DQL**：数据查询语言，主要队数据进行查询
- **DCL**：数据控制语言，主要是账号/数据库/表授权/事务等相关操作

#### 1.4.1 DDL数据定义语法

![image-20210601203526275](E:/面经/JavaNotesForInterview/images/image-20210601203526275.png)

#### 1.4.2 DML数据操纵语法

![image-20210601203547046](E:/面经/JavaNotesForInterview/images/image-20210601203547046.png)

> <font color="red">一般在线上环境不直接删除数据，一般是加一个字段做删除标记（软删除？）！</font>

#### 1.4.3 DQL语法数据查询

![image-20210601203634836](E:/面经/JavaNotesForInterview/images/image-20210601203634836.png)

>- 使用group by子句时，**除了聚合语句外**，select中要查询的每一个列都必须出现在group by后面，比如
>
>```sql
># 不合法语句
>select gender, count(*), name from user where id > 10 group by gender
># 合法语句
>select gender, count(*) from user where id > 10 group by gender
>```
>
>- 当group by 后有2个字段时，当成一个整体的字段进行分组；

#### 1.4.4 DCL数据控制语法

```sql
GRANT
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user [auth_option] [, user [auth_option]] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH {GRANT OPTION | resource_option} ...]
```

个人比较常用的是给一个新用户授权制定的数据库。可以使用以下语句：

```sql
 GRANT ALL PRIVILEGES ON database_name.* TO 'user_name'@'%' IDENTIFIED BY 'user_password' WITH GRANT OPTION;   
 FLUSH PRIVILEGES;
```

事务相关操作主要使用：

```sql
begin;
// ... 一系列增删改查SQL操作
// roolback;  // 回滚之前的操作
// ... 一系列增删改查SQL操作
commit;
```

## 2 InnoDB存储结构

### 2.1 InnoDB的体系结构

![image-20230415225950178](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230415225950178.png)

InnoDB的体系结构主要分为后台线程和内存池（内部包含多个内存块）。

- 后台线程：主要负责刷新内存池中的数据，保证磁盘中的数据的一致性（读写），同时保证数据可发生异常时可以恢复到正常运行的状态。主要包含四类线程：
  - **Matser Thread**：核心后台线程，主要负责将缓存池中的数据异步刷新到磁盘，保证数据的一致性。
  - **IO Thread**：包含多个write/read和一个log/insert——thread来负责队数据可的Iod请求，提高读写性能。
  - **Pruge Thread**：事务被提交后，负责回收undolog（为了完成十五回滚做的一种备份日志）。
  - **Page Cleaner Thread**：脏页（缓存中和磁盘中内存不一致的页）刷新，减轻Master Thread负担。
- **内存池**：协调CPU与磁盘速度的不匹配，主要负责以下工作：
  - 维护所有进程/线程需要的内部数据结构
  - 缓存磁盘书（LRU缓存算法）、方便快速读取，同时缓存队磁盘文件的数据修改
  - 重做日志缓存

### 2.2 MySQL存储引擎

#### 2.2.1 什么是存储引擎

- **InnoDB**：默认存储引擎，支持事务、MVCC、外键、行锁、表锁、按照主键顺序存放数据（聚集索引）；性能强大；
- **MyISAM**：不支持事务；不支持行锁、只支持表锁，索引和数据分开存储（非聚集索引）；查询速度较快（适合用在读写分离的从机上）；
- **Memory**：将数据存储在内存中，适合保存临时表，速度快，默认使用哈希索引；只支持表锁，并发性能差。
- **Archive**：只支持**insert**，**select**操作，适合存储归档数据（压缩数据）。

#### 2.2.2 InnoDB和MyISAM的区别

**MyISAM是MySQL5.5之前的默认数据库引擎**，虽然性能良好，且提供了全文索引、压缩、空间函数等特性，但是MyISAM不支持事务和行级锁，且崩溃后无法安全恢复，因此在MySQL5.5之后采用InnoDB（事务性数据库引擎）为默认存储引擎。

- **事务**：InnoDB支持事务，MyISAM不支持事务；
- **安全恢复**：
  - InnoDB提供事务支持，外键等高级数据库功能。且具有事务（commit）、回滚（rollback）和崩溃修复能力（crash recovery capabilities)的事务安全(transaction-safe (ACID compliant))型表。
  - MyISAM更注重性能，其每次查询都具有原子性，执行速度较InnoDB更快，但崩溃后无法安全修复。
- **外键**：InnoDB支持外键，MyISAM不支持外键；
- **MVCC（多版本并发控制）**：InnoDB支持MVCC，MyISAM不支持MVCC；
  - 在应对高并发事务时，MVCC比单纯的加锁更加高效；MVCC只在READ CONNITTED和REPEATABLE READ两个隔离级别下工作；MVCC使用乐观锁和悲观锁来实现。
- **全文索引**：InnoDB不支持全文索引（5.7之后InnoDB也支持全文索引），MyISAM支持全文索引；
- **锁**：InnoDB支持表级锁，行级锁，MyISAM支持表级锁；
- **主键**：
  - InnoDB必须有主键，MyISAM可以没有主键；
  - InnoDB按主键大小有序插入，MyISAM按记录插入顺序保存；
- **存储**：InnoDB需要更多的内存和存储，而MyISAM可以被压缩，存储空间更小；
- **索引组织表**：InnoDB属于索引组织表，使用共享表空间和多表空间存储数据。而MyISAM使用.frm、.MYD、.MTI来存储表定义，数据和索引。
- **索引实现**：InnoDB的主键索引一般采用聚集索引的形式（叶子节点存储完整的行数据），MyISAM无论是主键索引还是辅助索引都在用非聚集索引（叶子结点只存储数据地址）。

**`select count(*) from table`时InnoDB和MyISAM哪个更快——MyISAM**

因为，MyISAM使用了变量来保存整张表的总行数，所以可以直接读取出，InnoDB则需要遍历整张表才可以得出表内数据行数。

## 3 文件

### 3.1 参数文件

MySQL数据库实例启动时会首先读取配置文件（可以通过`mysql --help| grep my.cnf`来查询配置文件的位置），里面包含了数据库支持的各种配置参数。

### 3.2 日志文件

主要包含以下几类日志文件：

- 错误日志（error log）：记录MySQL运行期间产生的错误信息，通过`show variables like "log_error";`命令可查看错误日志文件的存储位置。

- 二进制日志（binlog）：记录对数据库的所有修改操作（不记录`select`和`show`操作），方便对数据库进行 **恢复**、**复制**、**审计**。

  > 注意把binlog和undo log已经redo log进行区分！
  >
  > binlog是数据库产生的（任何存储引擎对数据库的修改都会产生二进制日志），binlog记录的是SQL语句；而后两个是存储引擎产生的，记录的是行/页的修改！

- 慢查询日志（slow query log）：记录执行时间较长的SQL语句的日志，以便对数据库进行SQL语句层面的优化。

- 查询日志（log）：记录对MySQL数据库请求的信息，无论该请求是否得到了正确的执行。

### 3.3 套接字文件

​	Unix系统中本地连接MYSQL可使用unix域套接字的方式，这种方式需要一个套接字文件。

### 3.4 pid文件

MySQL启动后会将自己的进程ID写入到pid文件中。

### 3.5 表结构定义文件

MySQL数据的存储是**根据表来进行**的，每个表/视图无论使用何种存储引擎都会有一个`.frm`结尾的文件来记录表/视图结构定义！

### 3.6 InnoDB存储引擎文件

前面几类文件都是数据库本身的文件、和存储引擎无关。除了这些文件之外，每个表的存储引擎还有其独有的文件！InnoDB存储引擎主要有**重做日志文件**和**表空间文件**两种。

#### 3.6.1 表空间文件

用于存储存储引擎为InnoDB的表相关的各种数据。

如果启用了`innodb_file_per_table`参数，则将每个表的*数据、索引和插入缓存BITMAP* 等信息存入对应的独立表空间文件中（一种`.idb`文件），其它的如 *回滚信息、插入缓存信息、事务信息* 等依旧存储在共享表空间中！

![image-20210124162436993](E:/面经/JavaNotesForInterview/images/image-20210124162436993.png)

#### 3.6.3 重做日志文件

用于记录InnoDb存储于引擎的事务日志？没怎么看懂！

## 4 索引与算法

### 4.1 索引

#### 4.1.1 索引是什么

索引是为了加速对表数据行的检索和创建的一种分散存储的数据结构。

在MySQL中，索引是在存储引擎层实现的，而不是在服务层实现的，所以不同存储引擎具有不同的索引类型和索引实现。

索引太少会影响查询速度，索引太多会加重系统负担，因为频繁的插入/修改数据需要频繁的更新索引，同时索引文件也会大量占用存储空间。

索引的核心就是减少查找IO的次数，使用索引后可以不用扫描全表来定位某行的数据，而是先通过索引表找到该行数据对应的物理地址然后访问相应的数据。

#### 4.1.2 为什么需要索引

- 索引可以极大地减少存储引擎需要扫描的数据量（减少全表扫描）；
- 索引可以把随机IO变成有序IO；
- 索引可以帮助我们在进行分组、排序等操作时避免使用临时表；

#### 4.1.3 MySQL支持哪些索引类型

InnoDB支持以下索引：

- **B+树索引**：用于快速查找数据所在的页，数据库通过把页读入内存，在内存中继续查找（二分查找）最终定位到目标数据；
- **哈希索引**：InnoDB存储引擎根据表的使用情况自动生成哈希索引，无法人工干预。适合单条记录的查询，不适合范围查询；
- **全文索引**：通过建立倒排索引，可以极大地提升检索效率，解决字段是否包含的问题。

##### B+树索引

B+树是B树的一种改进：

- **将数据都存储到叶子节点处**，非叶子节点只记录key的信息，减少IO查询次数（单词Page载入可以读取更多索引节点信息）；
- 同时叶子节点之间使用双向链表进行连接，适合范围查询。

![image-20230417133621594](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230417133621594.png)

根据B+树索引在屋里存储层次的实现方式的不用，可以将其分为聚集索引和非聚集索引，连着最大的区别在于B+树的叶子节点是否包含完整的行数据信息。**聚集索引的叶子节点存储完整的行数据信息非聚集索引叶子节点只存储数据中的部分字段信息或地址。**

##### **为什么选用B+树而不是二叉树**

主要分析维度：查询是否够快、效率是否程度、存储数据多少、查找磁盘次数来分析。

1. **为什么不用一般二叉树**
   - 数据量越大→树的高度越高→IO操作次数越多→IO操作耗时越大→查询速度越慢
   - 每个磁盘块（节点/页）保存的数据太小（每次IO只能读取到一个关键字），没有很好地利用操作磁盘IO的数据交换特性，也没有利用好磁盘IO的预读能力（空间局部性原理），从而带来频繁的IO操作。
2. **为什么不用平衡二叉树**
   - 平衡二叉树的每个节点值存储一个键值和数据，而B树则可以存储更多的数据节点，树的高度也会降低，从而导致读取磁盘次数的减小，查询速度会加快。
3. **为什么不用B树**
   - B+树是范围查找，定位min与max之后，中间叶子节点就是结果集，不需要中序便利回溯；
   - B+树的磁盘读写能力更强（叶子节点不保存真实数据，因此一个磁盘块能保存的关键字更多，因此每次加载的关键字更多）；
   - B+树的扫表和扫库能力更强（B树需要扫描整棵树，B+树只需要扫描叶子子节点）。

##### **为什么采用B+树作索引？B+树和B树作为索引的区别**

> - 局部性原理：一个数据被用到时，它附近的数据也可能马上被用到。具有局部性的程序可以大大提高IO效率。
> - 磁盘预读原理：页时计算机管理存储器的逻辑块，磁盘预读一般时连续载入多个连续的页。

###### **B+树的优点**

- **减少磁盘IO次数**：非叶子节点只存储索引值，叶子节点存储数据（和物理页相对应，内存与磁盘以页为单位交换数据），适合文件系统，依次IO即可载入更多的索引key信息，减少磁盘IO次数。
- **适合范围查询**：logN的时间复杂度定位到页，利用页之间的双向链表可以直接找到指定范围的数据。B树进行范围查询必须进行中序遍历。
- **查询效率稳定**：对于所有关键词的查询次数，都是相同的，**查询次数取决于B+树的高度**。

###### **B+树的缺点**

- **产生大量随机IO**：随着新数据的插入，叶子节点会慢慢分裂，逻辑上连续的叶子节点往往在物理上不连续，甚至间隔较远，但在做范围查询时，会产生大量的随机IO。对于大量的随机写页一样。
- 如果热点数据距离根节点很近的话，B树的效率会更高一些。

###### B树的优点

- 热点数据距离根节点较近时查询速度会更快一些，可以把热点数据放在距离根节点较近的节点。再对一些热点、特定数据进行多次重复查询时B树效果会更好（但InnoDB可以自适应再B+索引上增加哈希索引）。

###### B树的缺点

- **查询效率不稳定**：非叶子节点数据查询快，叶子节点数据查询慢；
- **不适合范围查询**：需要对树进行中序遍历；
- **IO次数较多**；

###### AVL、红黑树、二叉树作索引时的缺点

- AVL和红黑树是一种二叉搜索树，深度相对较大，磁盘IO次数较多（适合比较扁平的，层数较少的情况）
- 二叉树的每个节点只存储一个键值和数据，这极大的浪费了每次IO获取的信息数量；
- 都不适合作范围查找（无法中序遍历）。

AVL 树和红黑树基本都是存储在内存中才会使用的数据结构。<span style = "color:red">**在大规模数据存储的时候，红黑树往往出现由于树深度过大而造成磁盘IO读写过于频繁，进而导致效率低下的情况。**</span>为什么会出现这样的情况，我们知道要获取盘上数据，必须先通过磁盘移动臂移动到数据所在的**柱面**，然后找到指定**盘面**，接着旋转盘面找到数据所在的**道**，最后对数据进行读写。磁盘IO代价主要花费在查找所需的柱面上，树的深度过大会造成磁盘IO频繁读写。<span style = "color:red">**根据磁盘查找存取的次数往往由树的高度所决定**</span>

> tips：红黑树这种二叉搜索树结构，h明显要深的多。由于逻辑上很近的节点(父子)物理上可能很远，无法利用局部性原理。所以把数据结构设计的更为‘矮胖’一点就可以减少访问的层数。

### 4.2 聚集索引和非聚集索引

#### 4.2.1 聚集索引

聚集索引是根据表的主键构造一棵B+树，**每张表只有一个聚集索引**。聚集索引的叶子节点是一个数据页，里面存放了完整的行记录数据（即每一行的所有字段信息），叶子节点/页之间通过双向链表进行连接且按照逐渐排序，页和页之间逻辑上是连续的，但是物理上不一定连续。

**主键索引不等于聚集索引，在InnoDB中主键索引就是聚集索引，在MyISAM中主键索引不是聚集索引**

在InnoDB中：

- 如果定义了主键，主键就是聚集索引；
- 如果没有定义主键目的一个非空（not null）且唯一（unique）列就是聚集索引；
- 如果没有符合条件的列，会自动创建一个隐藏的row-id作为聚集索引；

上述三种情况下聚集索引就是主键索引

在MyISAM中：

- 主键索引是非聚集索引（数据和索引文件时分开存储的）；
- 辅助索引也是非聚集索引；

###### MyISAM 和 InnoBD 实现 B+Tree 索引方式的区别

- **MyISAM**：主键索引和辅助索引均采用**非聚集索引**的方式，B+树的叶子节点只存储只想数据的地址指针而不存储数据。数据文件和索引文件分离。
- **InnoDB**：主键索引采用聚集索引的方式，B+树的叶子节点存储完整的行数据，其数据文件本身就是索引文件；辅助索引都需要使用主键作为data域（叶子节点存储主键值），再使用主键值进行回表操作，从表中获取全部数据。

#### 4.2.2 非聚集索引

非聚集索引和聚集索引类似，**但一张表可以有多个非聚集索引**（不同的字段可以设置非聚集索引），它的叶子节点不存储完整的行数据（InnoDB中存储列值 + 主键；MyISAM存储行数据的物理地址）。

**当InnoDB在使用非聚集索引来查找数据时：**

- 首先通过key值找到非聚集索引对应的B+树的叶子节点；
- 然后通过叶子节点记录的书签（主键值）到聚集索引对应的B+树进行进一步查找（回表）；

![image-20230417135745674](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230417135745674.png)

##### **聚集索引和非聚集索引的区别**

- 一个表中只能有一个聚集索引，而一个表中可以存在多个非聚集索引；
- 聚集索引中键值的逻辑顺序决定了表中相应行的排列顺序；非聚集索引，索引中的逻辑顺序与表中相应行的排列顺序不同。
- 聚集索引的叶子节点存储的就是数据本身；非聚集索引的叶子节点存储的是指向数据的地址指针。

![聚集索引](https://img-blog.csdnimg.cn/img_convert/07596c40e8f22b104fd0ce20dd52fa6b.png)

### 4.3 哈希索引

哈希索引底层的数据结构就是哈希表（将key映射从数字后利用哈希函数计算下标值），所以多个数据在存储关系上是完全没有任何顺序关系的：

- 无法用于排序和分组；
- 只支持精确查找，无法用于部分查找和范围查找；

**在绝大多数需求为单条记录查询的时候，可以选择哈希索引**，查询性能更快；其余大部分场景应该选择B+树索引。

InnoDB存储引擎可以自适应创建哈希索引，当某个索引值被使用的非常频繁时，会在B+索引之上再创建一个哈希索引，这样对一些热点数据，可以使用哈希索引优化查找速度。

### 4.4 全文索引

再MySQL5.6之前仅MyISAM支持全文索引，再MySQL5.6之后InnoDB也开始支持全文索引。

全文索引底层采用倒排索引技术。只支持空格分词，不支持中文。

### 4.5 索引优化

#### 4.5.1 覆盖索引减少回表操作

如果一个索引包含所有需要查询的字段的值，我们就称之为覆盖索引。通过创建覆盖索引，可以减少查询时的回表操作。例如创建了索引(username， age)后，可以执行如下语句：

```sql
select username , age from user where username = 'Java' and age = 22
```

此时叶子节点中已经包含了全部需要信息username和age，我们便可以直接返回结果而不必再回表。

#### 4.5.2 最左匹配原则/索引匹配原则

创建联合索引时注意最左匹配原则，联合索引是指对表中的队列进行索引。而最左前缀原则指的是，如果查询的时候查询条件精确匹配索引的左边连续一列或几列，则可命中索引，**范围查询之后的索引无法命中**。

- **字段出现顺序不同不影响索引使用**：查询的时候如果所有的条件都用上了，但是顺序不同，则查询引擎会自动将查询优化为匹配联合索引的顺序。

  - 例如：`user` 表中有两个字段 `name` 和 `city`，并将两个字段建立了联合索引`(name, city)`，但是再查询语句中的顺序为：

    ```sql
    select name, city from user where city = xx and name == xx
    ```

    实际执行时时时

    ```sql
    select name, city from user where name = xx and city == xx
    ```

- **某个字段使用范围查询**：范围查询的后续字段不会使用索引。

  - 例如：字段 **a** 和字典 **b** 建立了联合索引 **(a, b)**，其中 **a** 是全局有序的，**b** 是全局无序但局部有序（即只有再 **a** 值唯一确定的情况下 **b** 才会使用索引）。![image-20230417152429762](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230417152429762.png)

- **order by 之后的字段如果出现在 where 中会使用索引，否则不会使用索引。**

- **or会使用索引（但必须所有由or连接的条件都是独立索引）。**

#### 4.5.3 联合索引中区分度高的字段放在前面

**由于最左前缀原则，再创建联合索引时，索引字段的顺序需要考虑字段值去重之后的个数。较多的放在前面。order by 子句同样遵循这条规则**

例如，将`name`和`city`建立联合索引，因为`name`相较于`city`可以更快的将数据区分开，分为更小的组，即`name`可以迅速缩减查找范围。所以应该将`name`放在左边，即`(name, city)`。

### 4.6 关于索引的其他面试题

#### (1)索引有哪些类型

- **普通索引：**最基本的索引，没有任何限制，只是为了提高查询速度；

  ```sql
  ALTER TABLE table_name ADD INDEX index_name(column); --创建普通索引
  ```

- **唯一索引**：数据列不允许重复，允许为NULL值，一个表允许多个列创建唯一索引；

  ```sql
  ALTER tABLE table_name ADD UNIQUE(column) --唯一索引
  ```

- **主键索引**：数据列不允许重复，不允许为NULL值，一个表只能有一个主键；

  ```sql
  ALTER TABLE table_name ADD PRIMARY KEY(column)
  ```

- **复合索引**：也被称为组合索引，再多个字段上联合建立一个索引，次啊用最左前缀原则；

  ```sql
  ALTER TABLE table_name ADD INDEX index_name(column1, column2); --组合索引
  ```

- **全文索引**：MySQL5.6之前仅MyISAM支持，5.6之后InnoDB也支持。

  ```sql
  ALTER TABLE table_name ADD FULLTEXT(column); --全文索引
  ```

- **覆盖索引**：如果一个索引包含（覆盖）了所有我们需要查询的字段，我们就称之为“覆盖索引”。再InnoDB引擎中，如果不是主键索引，叶子节点存储的就是主键 + 列值，最终还要回表，而覆盖索引则无需回表。

#### (2)索引什么情况下会失效

- **查询条件包含or，且由or连接的条件有未加索引的字段；**

  例如: `a or b or c`，如果 a、b、c 中有一个字段没有索引，便会导致其他字段的索引也失效，因为这样需要全表扫描一遍+索引扫描+结果聚合，实际上全表扫描一遍便已经可以完成查询，索引扫描是冗余的。如果 a、b、c 三个字段都有索引，那么仍然会使用索引查询。

- **对索引列进行运算（+ - × /）；**

- **对索引字段进行隐式转换；**

- **在索引字段上使用（!= <> not in）；**

- **在索引字段上使用is null， is not null 会导致索引失效；**

- **like通配符以%开头会导致索引失效；**

- **相互join的两个表的字符编码不同，不能命中索引，会导致笛卡尔积的循环运算；**

- **MySQL搜索引擎估计使用全表扫描比索引快时会不使用索引。**

#### (3)哪些场景不适用索引

- 数据量少的不适合加索引；
- 更新比较频繁的也不适合加索引；
- 离散性较低的字段不适合加索引。

#### (4)如何判断使用聚集索引还是非聚集索引

- 聚集索引和非聚集索引的本质区别在于表内的行记录数据的排列顺序（逻辑存储顺序）是否和索引顺序一致。聚集索引是一致，因此查找速度较快，插入速度较慢，因为可能需要重新排页。

具体使用场景见下表：

| 动作描述                 | 使用聚集索引 | 使用非聚集索引 |
| ------------------------ | ------------ | -------------- |
| 1.列经常被分组排序       | 应           | 应             |
| **2.返回某范围内的数据** | 应           | 不应           |
| 3.一个或极少不同值       | 不应         | 不应           |
| 4.小数目的不同值         | 应           | 不应           |
| **5.大数目的不同值**     | 不应         | 应             |
| **6.频繁更新的列**       | 不应         | 应             |
| 7.外键列                 | 应           | 应             |
| 8.主键列                 | 应           | 应             |
| **9.频繁修改索引列**     | 不应         | 应             |

#### (5)联合索引的底层原理

B+树，但是与普通索引相比，树上节点的键的数量大于1，按照联合索引中从左到右的顺序对进行排序。

![image-20230417165819759](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230417165819759.png)

## 5 锁

锁时数据库区别于文件系统的一个重要特性，为数据库对共享资源进行并发访问提供数据完整性和一致性。

- InnoDB既支持行级别的锁，也支持表级别的锁，默认使用行锁；
- MyISAM支持表级别的锁。

在MySQL中引入锁是为了支持对共享资源的并发访问，提供数据的完整性和一致性。

### 6.1 InnoDB锁类型+当前/快照读

#### 6.1.1 锁的类型

**行级锁**

- **行级锁（S锁）**：允许事务读取一行数据，多个事务可以同时获取该锁（**InnoDB默认使用一致性非锁定读**，除非显示加锁或者将事务等级隔离级别为SERIALIABLE踩死使用S锁）。
- **排他锁（X锁）**：允许事务删除或更新一行数据，只能被一个事务获取且该行的S锁处于释放状态。

```sql
SELECT ... FOR UPDATE;			# 加X行锁
SELECT ... LOCK IN SHARE MODE;	# 加S行锁
```

**表级锁**

- **意向共享锁（IS锁）**：事务想要获取一张表的某几行数据的共享锁。
- **意向排他锁（IX锁）**：事务想要获取一张表的某几行数据的排他锁。
- **表级X/S锁**：

```sql
LOCK TABLE my_tabl_name READ; # 用读锁锁表，会阻塞其他事务修改表数据
LOCK TABLE my_table_name WRITE; # 用写锁锁表，会阻塞其他事务读和写
```

**表级锁的触发条件：**

- 如果没有索引，更新数据时会锁住整张表。
- 事务隔离级别为串行化时，读写数据都会锁住整张表（因此只能有一个连接来操作表）。

**事务在给行加X/S锁之前，需要先给行所在的表加对应的意向锁。意向锁是一种快读判断表锁与之前可能存储在的行锁冲突的机制，时InnoDB为了提高加锁的兼容性判断效率而自动添加的粗粒度的锁，用户无法操作。**

> 举例
>
> 当Session A需要修改某一行的数据：
>
> - **InnoDB会献给该行所在的表加X锁（表锁）**
> - 然后对改行加X锁（行锁）
>
> 当Session B想对该表加X锁（表锁）时：
>
> - 如果没有意向锁存在，则需要进行下面两步判断：
>   - 判断该表是否有标记S锁或表级X锁
>   - 判断该表的每一行是否有行级X锁或行级S锁（遍历很耗时间）
> - 如果存在意向锁，则只需要：
>   - 根据兼容性表，如果表级X锁和IX锁冲突，则加锁失败，无需遍历所有行，效率比较高。

表级锁兼容性：

<font color="red">下面的X、S、IX、IS锁均为**表级锁**！不是行锁！行锁和意向锁并不冲突</font>

|      | `X`           | `IX`            | `S`             | `IS`            |
| :--- | :------------ | :-------------- | :-------------- | --------------- |
| `X`  | Conflict 冲突 | Conflict 冲突   | Conflict 冲突   | Conflict 冲突   |
| `IX` | Conflict 冲突 | Compatible 兼容 | Conflict 冲突   | Compatible 兼容 |
| `S`  | Conflict 冲突 | Conflict 冲突   | Compatible 兼容 | Compatible 兼容 |
| `IS` | Conflict 冲突 | Compatible 兼容 | Compatible 兼容 | Compatible 兼容 |

#### 6.1.2 一致性非锁定读/快照读

一致性非锁定读是指InnoDB使用更多版本控制（MVCC）的方式来读取当前执行时间数据库中行的数据（MySQL默认使用的方式）。

如读取正在`UPDATE`或`DELETE`的行时（已经被加了X锁），读操作无需等待并加锁（无需等待X锁释放后加S锁），直接读取改行的快照数据（在undo记录中有，undo记录由于事务回滚）。这种方式可以极大的提高数据库的并发性。

在不同事务隔离登记下读取的方式有所不同：

- READ COMMiTTED级别：读取最新一份快照数据；
- REPEATABLE READ级别（默认级别）：读取事务开始时的行数据，可能不是最新commit的数据。

#### 6.1.3 一致性锁定读/当前读

用户显示的对读操作进行加锁以保证数据逻辑的一致性。InnoDB支持以下两种方式对SELECT实现一致性锁定读：

```sql
BEGIN 							# 首先开启事务
SELECT ... FOR UPDATE			# 方式1：加X锁(意义在于可以利用嵌套语句修改查询到的数据)
SELECT ... LOCK IN SHARE MODE	# 方式2：加S锁
insert/update/delete操作都是当前读
```

### 6.2 行锁的算法

#### 6.2.1 InnoDB行锁的三种算法

InnoDB行锁是**通过给索引上的索引项加锁**来实现的，如果没有索引，InnoDB将用过**隐藏的聚集索引**来给记录加锁。

- **Record Lock**：对单条索引记录上的锁（锁住索引key为X的记录）；
- **Cap Lock**：间隙锁，锁定一个索引范围，但不包含索引记录本身（锁住索引key在（A，X）范围内的记录）；
- **Next-Key Lock**：Recod Lock 和 Gap Lock的结合，锁定一个范围且包含该范围内的记录本身。  

![image-20230417203252440](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230417203252440.png)

假设有`id`分别为1、3或7的数据库记录；现在执行SQL查询 `select id from test_table where id > 2 for update`, 此时 next-key lock 将会对区间(2, 5], (5, 7], (7, +inf)加锁，任何试图在这个区间插入的记录都会被阻塞。

Gap Lock 是针对事务隔离级别为重复可读或以上级别。

Next-key Lock 是为了解决幻读问题。

#### 6.2.2 幻读

幻读问题是指在同一十五下，两次执行SQL查询语句的得到的结果不一致的问题，第二次SQL语句可能会返回之前不存在的行（区别与不可重复读）。即一个事务读取到了另一个事务中的数据，违反了隔离性。

```sql
# Session A执行以下查询语句
SELECT * FROM t WHERE id > 99 and id < 102 FOR UPDATE;	# 此时不会查询到id为101的记录(不存在)
# Session B执行插入语句并提交
INSERT INTO t select 101	# 插入一条id为101的记录(这是被允许的)
# Session A再次执行以下查询语句
SELECT * FROM t WHERE id > 100 FOR UPDATE;	# 此时会查询到id=101的记录
```

幻读在REPEAT READ及以上的事务隔离级别中不存在

##### InnoDB如何解决幻读问题

InnoDB使用Next-Key-Lock来避免幻读问题（RR级别下），利用Next-Key-Lock将行记录和行记录之间的数据都锁住，其他事务无法在这个范围内插入新的数据。如上述例子中Session A执行查询时，将对(99, 102] 加X锁，此时其他事务无法插入id = 100/101/102的