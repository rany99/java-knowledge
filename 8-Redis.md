# Redis

[TOC]

## NoSQL概述

### 1.1 发展过程

#### 1.1.1 单机MySQL

![image-20230414205129095](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230414205129095.png)

**缺点**

- 数据量过大时，单台机器无法处理；
- 数据索引（B+树）太多，单台机器难以存储；
- 访问量剧增（读写混合），单台机器处理压力大；

#### 1.1.2 缓存 + MySQL + 读写分离

![image-20230414205316179](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230414205316179.png)

使用多台MySQL服务器，读写分离（MySQL2负责写数据，其他负责读数据并和MySQL2数据库进行同步），同时增加**MemCached**（高速缓存系统）来加速数据查询速度（因为网站大多数请求都是读数据而不是写数据库）。

#### 1.1.3 分库分表 + 集群

![image-20230414205537545](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230414205537545.png)

分库分表，使用服务器集群（主从服务器），缓解读写，存储压力。

##### **Redis和MemCached相比，有哪些优势？**

- **Redis数据结构更加丰富，支持String、List、Set、Hash、Zset等数据结构存储，MemCach仅支持String类型的操作。**
- **Redis支持数据的持久化，可以把内存中的数据持久化到硬盘中，而MemCached不支持持久化，数据只能储存在内存中，重启后数据就消失了。**
- **MecCached没有原生的集群模式，需要依赖客户端自己实现集群分片，而Redis原生支持集群模式。**
- **MemCached是多线程，非阻塞IO复用的网络模型；Redis使用单线程的多路IO复用模型。**

##### **Redis为什么要把数据放到缓存中**

**Redis为了达到最快的读写速度将数据都读到内存中，并通过异步的方式将数据刷回磁盘，所以Redis具有性能优越和数据持久化的特征。如果不讲数据存放在内存中，磁盘I/O速度会严重限制Redis的性能。而且随着内存价格的降低，Redis会越来越受欢迎。**

### 1.2 NoSQL

#### 1.2.1 NoSQL定义

**Not Only SQL** 非关系型数据库，主要有四大分类：

- KV键值对：查找速度快、但是数据无结构。如Redis、Tail、MemCache；
- 文档行数据库：K-V存储，但是Value是结构化数据。如MongoDB；
- 列存储数据库：以一列来作为一条数据进行存储。如HBase、分布式文件系统；
- 图关系数据库：存储关系的数据库（如社交网络），如Graph。

#### 1.2.2 NoSQL的优点

- 目前，很多数据信息难以完全固定格式；
- NoSQL可以更加方便的横向扩展（在数据量巨大的数据库中增加一个字段十分困难）；

#### 1.2.3 关系型数据库和非关系型数据库

- 关系型数据库通过外键关联来建立表与表之间的联系；
- 非关系型数据库通常指数据以**对象/键值对（k-V）**的形式存储在数据库中，而对象之间的关系由每个对象自身的属性来决定

#### 1.2.4 非关系型数据库的优势

- 以键值对的形式存储，无需经过sql解析，性能高；
- 数据之间没有耦合，很容易实现水平扩展

## 2 Redis基础

Redis（Remote Dictionary Service）即为远程字典服务，是速度非常宽（QPS10W+）的非关系型（NoSQL）内存键值数据库，可以存储键和物种不同类型的值之间的映射。支持多语言API，支持网络。可以基于内存进行持久化存储。

**Redis是单线程的**，但是严格来讲Redis Server是多线程，只是它的请求处理整个流程是单线程处理的。

##### **Redis的速度为什么快**

1. **内存数据库：大部分请求都是内存操作（除了持久化机制会涉及到I/O操作），执行效率高。**
   - Redis是一个内存数据库，它的所有数据都储存在内存中，这也就意味着我们读写数据都是在内存中完成，这个速度是非常快的。
2. **数据结构简单：数据结构比较简单，操作也相对简单，所以速度比较快。**
   - Redis中只有String、List、Set、Hash、ZSet物种简单数据结构；
   - Redis底层采用了相对较为高效的数据库结构，例如SDS、哈希表和跳表。
3. **单线程操作：I/O处理部分采用单线程，避免了不必要的线程上下文切换和资源竞争、死锁问题。**
   - 单线程避免了多线程上下文切换导致的性能损耗
   - 避免了多线程访问共享资源加锁导致的性能损耗
4. **IO多路复用：采用IO多路复用机制（Reactor模式、事件分发和处理机制）。**
   - Redis基于Reactor模式开发了自己的网络事件处理器：这个处理器被称为文件事件处理器（file event handler）。文件事件处理器使用I/O多路复用（multiplexing）程序来同时监听多个套接字，并根据套接字当前执行的任务来为套接字关联不同的事件处理器。
   - 当被监听的台阶子准备好执行连接应答（accept）、读取（read）、写入（write）、关闭（close）等操作时，与操作相对应的文件时间就会产生，这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件；
   - 虽然文件事件处理器以单线程方式运行，但通过使用I/O多路复用程序来监听多个套接字，文件事件处理器既实现了高性能的网络通信模型，又可以很好地与Redis服务器中其他同样以单线程反射光hi运行的模块进行对接，这保持了Redis内部单线程设计的简单性。
5. **非CPU密集型任务**
   - 采用单线程的缺点便是无法使用多核CPU，因此Redis的性能瓶颈在于内存和网络带宽；
   - 在高并发请求下，Redis需要更多的内存和更高的网络带宽，否则很容易出现内存不够用和网络延迟等待的情况；
   - 同时在单个Redis实例的性能不足以支撑业务时，可以通过部署多个Redis节点，组成集群的方式来利用CPU的多喝能力，而不是在单个实例上使用多线程来处理。

## 3 Redis的基本数据类型

Redis有5中基础数据结构，分别为**String**、**List**、**Set**、**Hash**、**Zset**。

Redis中的所有数据结构都是一个key对应一个value，不同的数据结构之间的差异在于value的结构不同。

### 3.1 String-字符串

value可以是单纯的字符串或者数字组成的字符串

**底层结构**

底层采用SDS（Simple Dynamic String）来存储，每次追加字符串前都会检查空间是否充足，不够会重新申请更大的空间。

```java
struct sdshrd {
    // 记录已经使用长度
    int len;
    // 记录空闲未使用的长度
    int free;
    // 字符数组
    char[] buf;
}
```

**SDS相对C语言中字符串的优势**

- **多增加len来表示当前字符串的长度**：这样可以直接获取字符串长度，复杂度为O(1)，而C中每次需要便利到\0才能计算得出char *长度。
- **自动拓展空间**：当SDS需要对字符串进行修改是，首先借助于len和allow检查空间是否满足修改所需的需求，如果空间不够的话，SDS会自动扩展空间，避免了像C字符串操作中的覆盖情况；
- **有效降低内存分配次数**：C字符串在涉及增加或者清除操作时会改变底层数组的大小重新分配，SDS使用了**空间预分配**和**惰性空间释放**机制，即在每次扩展时是城北的多分配的，在缩容时也是先留着并不正式归还给OS；
- **二进制安全**：C语言字符串只能保存ASCII码，对于图片、音频等信息无法保存，SDS是二进制安全的，写入什么读取出来就是什么，不会进行任何的过滤和限制。

**应用**

- **计数器**：对于文章浏览量暴增的情况，使用Redis来快速更新浏览量而不是每次增加一个浏览量就更新一次数据库数据，而且Redis的单线程性还保证了数据的一致性。

### 3.2 List-链表

**底层结构**

List-链表底层采用双向链表结构进行存储，**当列表弹出了最后一个元素之后，该数据结构自动被删除，内存被回收。**

**优点**

两端插入性能高，但中间的插入性能较低

**缺点**

随机读取性能差，例如读取第10个节点时，需要从头开始遍历这个链表，直到找到第10个元素为止。

**应用**

- 异步队列：如消息队列，使用PUSH和POP吧消息加入队列或者从队列中取出
- 任务轮询
- 文章列表

### 3.3 Hash-哈希表

K-V形式，Key（哈希表名）对应的value是一个Map（哈希表），Map中存储着K-V对。**当最后一对键值对被移除后，该数据结构自动被删除，内存被回收。**

**底层结构**

![image-20230415152456214](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230415152456214.png)

Redis的Hash实现和Java的HashMap是类似的，**底层采用Redis中的字典结构（Dict）来实现**，字典内部的哈希表都是采用**数组 + 链表**的方式来存储数据，**一个字典内部包含两个哈希表(dicht)**，正常情况下（未进行rehash操作）下字典结构如下：

![image-20230415152717763](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230415152717763.png)

哈希表h[0]存储数据，h[1]为空以备rehash操作。

**Rehash操作**

当存储数据的那个哈希表h[0]中的数据不断变多或变少，需要执行rehash（重新散列）以提升哈希表的效率。

- 首先会在h[1]上申请合适的空间（收缩则为大于等于且最接近h[0].used的2^n，扩展则为大于等于且最接近h[0].used × 2 的 2^n）；
- 将h[0]中的数据**渐进式转移**入h[1]中；
- 转移完成后交换h[0]和h[1]的指向。

**渐进式Rehash**

在rehash时，如果一次性将h[0]中的数据转移到h[1]中，如果数据量较大可能会导致服务器停止的时间较长。所以rehash操作选择**分批次、渐进式（在对字典进行增删改查时顺便进行数据的转移）**地完成，这是为了避免一次性执行过多的rehash操作给服务器带来过大的负担。

在进行渐进式地转移时，**数据同时存在于两个字典中**，如果在h[0]中未找到合适的，复合要求的key时会去h[1]中查找。

**应用**

- 经常变动的用户信息保存，一个用户的各类字段信息作为一个K-V对进行保存，用户id‘则作为key。
- 适合经常变动的对象的信息存储。

### 3.4 Set-无序集合

Set中的值是**唯一的、无序的**。**当集合中的最后一个元素被移除之后，数据结构被自动删除，内存被回收。**

**底层实现**

Redis的set集合相当于Java的HashSet。Redis中的set类型是一种无序集合，集合中的元素没有先后顺序（即插入顺序和迭代遍历顺序是不一致的）。

HashSet实际上就是基于HashMap实现的。

**应用**

- **抽奖**：如果数据量不是特别大的时候，可以使用spop（移除并返回集合中的一个随机元素）或srandmember（返回集合中一个或多个随机数）
- 关注集合、好友集合

### 3.5 ZSet-有序集合

ZSet中的值是唯一的、有序的。ZSet在Set的基础上，增加了一个权重参数score，使得集合中的元素能够按照score进行有序排列，还可以通过score的范围来获取元素的列表。使得它类似于Java的TreeSet和HashMap的结合体。

**当Zset中的最后一个元素被移除后，数据结构自动删除，内存被回收。**

**底层结构**

Zset的底层结构是使用**字典+跳跃表**来实现的：

- 字典的建保存元素的值，字典的值则保存元素的分值；（保证value的唯一性，同时以O(1)的时间复杂度查找value对应的score）
- 跳跃表节点的object属性保存元素的值，跳跃表节点的score属性保存元素的分值。（维护value的有序性支持范围查询）。

![image-20230415161352207](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230415161352207.png)

**跳表**

跳表是一种对传统单链表的改造，通过给非相邻节点之间增加连接指针来实现类似二分查找的机制。

![image-20230415163001205](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230415163001205.png)

例如查找19时会现在head最高层开始，向后遍历比较，如果发现目标值不在两个节点之间，则继续像狗查找，如果发现目标值在两个节点之间则到下一层急性更加精细化地朝招。

**为什么采用跳表**

因为ZSet支持随机插入和删除，所以他不宜使用数组来实现，由于ZSet中的数据结构有序，因此我们可以联想到用**红黑树/平衡树**这样的属性结构来实现，但是最终Redis还是选择了使用跳表来实现ZSet，主要有以下几个原因。

- **性能考虑**：插入速度非常快速，因为不需要进行旋转等操作来维护平衡性；查找/插入/删除的平均复杂度均为O(logN)。
- **实现考虑**：在复杂度与红黑树相同的情况下，采用跳表实现更加简单，开起来也更加直观。
- **支持无锁操作**。

**应用**

- 排行榜等有序数据场景。
- 订单支付超时（下单时插入，member为订单号，score为订单超时截止时间戳，然后使用定时任务每隔一段时间执行zrange）

## 4 Redis事务

事务指一组操作，事务是回复和并发控制的基本单位。

###操作命令

主要分为：开启事务、命令入队、执行事务。

事务执行过程中，如果服务端收到有EXEC、DISCAED、WATCH、MULTI之外的请求，会把请求放入队列中排队。

```bash
127.0.0.1:6379> MULTI	# 标记一个事务块开始
OK
127.0.0.1:6379> set name rany	# 命令入队
QUEUED
127.0.0.1:6379> set age 23	# 命令入队
QUEUED
127.0.0.1:6379> EXEC	# 执行事务
1) OK
2) OK
127.0.0.1:6379> keys *
1) "age"
2) "name"

127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> LSET names 0 Bob
QUEUED
127.0.0.1:6379> LSET names 1 Halen
QUEUED
127.0.0.1:6379> get names
QUEUED
127.0.0.1:6379> DISCARD	# 取消事务(队列中的命令全部丢弃)
OK
127.0.0.1:6379> EXEC	
(error) ERR EXEC without MULTI
127.0.0.1:6379> get names
(nil)
```

编译异常

```bash
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set xxx	# 语法出错！事务中的所有命令均无法执行
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379> EXEC
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k1
(nil)
```

运行异常

```bash
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> INCR k1	# 错误操作：对一个字符串自增1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> EXEC	# 第二条命令执行失败，但是其它命令执行成功
1) OK
2) (error) ERR value is not an integer or out of range
3) OK
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
```

> **Redis的事务单条命令具有原子性，但是多条命令保证执行顺序，不保证执行的原子性。**

### Redis事务特性

- **原子性**
  - 单条命令是原子执行；
  - 事务中命令入队时就报错，会放弃整个事务执行，保证原子性；
  - 事务中命令入队时没有报错，实际执行时报错，则部分命令执行成功部分执行失败，不保证整个事务的原子性。
- **隔离性**
  - 并发操作在EXEC命令前执行，此时，隔离性的保证要使用WATCH机制来实现，否则被隔离性无法保证；![image-20230415174318312](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230415174318312.png)![image-20210714102729452](E:/面经/JavaNotesForInterview/images/image-20210714102729452.png)
  - 并发操作在EXEC命令后执行，此时，隔离性可以保证。![image-20210714103257982](E:/面经/JavaNotesForInterview/images/image-20210714103257982.png)
- **持久性**
  - 支持RDB/AOF对数据进行持久化，AOF持久化的always策略下，事务具有持久性，每一条修改都会被保存，其它形式的持久化机制不支持事务持久性。
- **一致性**
  - 不支持，事务执行失败时，由于不支持原子性和回滚，数据库会出现不一致的状态。
  - Redis事务没有隔离级别的概念，因为Redis事务是单线程的且事务中的命令在顺序执行时不会被其他命令打断。
  - Redis不支持回滚，因为Redis事务执行出错都是因为编程错误产生的，实际生产环境中中很少出现，因此Redis没有复杂的事务回滚机制。**所以含有多条指令的Redis事务不具有原子性，事务只能保证命令的执行顺序而不能回滚**。

## 5 锁

Redis使用的时乐观锁。只有在事务提交的时候才会判断元数据是否已经被修改，如果已经被修改则提交失败。

```java
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> set in 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> WATCH in out	# 监控变量，实现乐观锁【类似先存储变量的原始值】
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> INCR in
QUEUED
127.0.0.1:6379> DECR out
QUEUED
127.0.0.1:6379> EXEC	
# 如果在执行这命令前有别的线程修改了in/out则实物执行失败！此时需要UNWATCH再进行操作(类似CAS自旋)
(nil)
```

### 悲观锁

每次读取数据的时候都默认其他线程会更改数据，因此需要进行加锁操作，当其他县城逍遥访问数据时都会被阻塞挂起导致程序性能下降。

> 当要对数据库中的一条数据进行修改的时候，为了避免同时被其他人修改，最好的办法就是直接对数据进行加锁以防止并发。这种借助数据库锁机制，在修改数据之前先锁定，再修改的方式被称之为悲观并发控制，简称悲观锁。

####分类

**共享锁[shared locks]**

又称为读锁，S锁。加了共享锁的数据可以被多个事务同时访问，但是都只能读取数据不能修改数据。

**排他锁[exclusive locks]**

又称为写锁，X锁。排他锁不能与其他锁并存，如果一个事务获取了一个数据行的排他锁，其他食物就不能再获取改行的其他所，包括共享锁和排他锁，但是获取排他锁的事务是可以队数据行读取和修改。

#### 应用

- 传统的关系型数据库使用这种锁机制，比如行锁、表锁、读锁、写锁等，都是在进行数据操作之前先上锁。
- java里的同步 [synchronized](https://www.jianshu.com/p/c8f997e7f75c) 关键字。

### 乐观锁

相对于悲观锁而言，乐观锁假设数据在一般情况下不会产生冲突，所以数据进行提交更新的时候，才会正式地队数据是否发生冲突进行检测，如果发现冲突了，则返回给用户错误地信息，让用户决定如何去做。乐观锁适用于读操作多地场景，这样可以提高程序地吞吐量。

#### 应用

- CAS实现：Java中`java.util.concurrent.atomic`包下的原子变量使用了乐观锁的一种CAS实现方式。
- 版本号控制：一般是在数据表中加上一个数据版本号version字段，白哦是数据被修改的次数。当数据被修改时，version会+1。当线程A要更新数据值时，在读取数据地同时也会读取version值，在提交更新时，若刚才读取到地version值与当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。

#### 实现

Redis地`WATCH`和`UNWATCH`命令实际上是一种乐观锁，他可以在执行EXEC指令前监视任意数量的键，如果在事务提交前被监视地键至少有一个被修改，则事务执行失败。

![image-20230415201513459](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230415201513459.png)

#### Redis实现分布式锁

通过`SETNX key value`和`DEL key`来实现加锁和解锁（如果key已经存在则加锁失败SETNX返回0，否则加锁成功给返回1，但是该命令不支持超时时间）；

或者使用 `SET key value [EX seconds | PX milliseconds] [NX]` 加锁，使用LUA脚本解锁。

![image-20230415201836296](C:\Users\a3781\AppData\Roaming\Typora\typora-user-images\image-20230415201836296.png)

- 为了防止客户端宕机导致死锁，可以为锁设置超时时间（利用守护线程来不断刷新定时器直到释放锁；或者由Redis删除已经超时地锁）；
- 为了防止加速失败的客户端一直轮询尝试获取加锁，可以利用Redis地发布订阅机制，当锁释放时主动推送消息给正在阻塞等待地客户端；
- 为了防止自己家的锁被别的客户端释放，可以在value中加入唯一标识；
- 在Redis集群环境下还可能会面临更多的问题：如客户端1在Master节点上加锁成功了，Slaver还未及时同步数据时Master宕机、Slaver称为新的Master，则客户端1地加锁记录消失了。[可以使用RedLock算法来解决]

**RedLock算法**

> RedLock算法地基本思路，是让客户端和多个独立地Redis实例一次请求加锁，如果客户端能够和半数以上的实例成功地完成加锁操作，那么我们就认为，客户端成功地获得分布式锁了，否则加锁失败。这样一来，即使有单个Redis实例发生故障，因为锁变量在其他实例上也有保存，所以客户端仍然可以正常地进行锁操作，锁变量并不会丢失。

1. 客户端获取当前时间；
2. 客户端按照顺序依次向N个Redis实例执行加锁操作；
   - 这里的加锁操作和单例上执行的加锁操作是一致的
3. 一旦客户端完成了所有Redis实例地加锁操作，客户端就要计算整个加锁过程的总耗时；