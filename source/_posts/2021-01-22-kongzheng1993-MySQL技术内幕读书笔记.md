---
title: MySQL技术内幕读书笔记（持续更新）
excerpt: 'MySQL'
tags: [MySQL]
categories: [MySQL]
comments: true
date: 2021-01-22 10:30:52
---

由于对MySQL的了解不够透彻，虽然用起来没问题，也知道一些常见的知识点，但是一直把它当作一个黑盒来使用，不免有些心里没底，所以下定决心，通过<strong>姜承尧</strong>大佬的书[MySQL技术内幕](https://book.douban.com/subject/24708143/)，认真了解MySQL技术细节。我会将一些我认为重要的知识，记录在这篇博客中，希望看到这篇文章的你，也能有所收获。

# 第一章 MySQL体系结构和存储引擎

## 1. Mysql体系结构

1. 数据库是文件的集合，是依照某种数据模型组织起来并存放于二级存储器中的数据集合；数据库实例是程序，是位于用户与操作系统之间的一层数据管理软件，用户对数据库数据的任何操作，包括数据库定义、数据查询、数据维护、数据库运行控制等都是在数据库实例下进行的，应用程序只有通过数据库实例才能和数据库打交道。
2. MySQL由以下几个部分组成：
   - 连接池组件
   - 管理服务和工具组件
   - SQL接口组件
   - 查询分析器组件
   - 优化器组件
   - 缓存（cache）组件
   - 插件式存储引擎
   - 物理文件
3. MySQL区别于其他数据库的最重要的一个特点就是它的插件式的表存储引擎。
4. 存储引擎是基于表的，而不是数据库。

## 2. MySQL存储引擎

1. MySQL是开源的，用户可以基于MySQL预定义的存储引擎接口编写自己的存储引擎。当然也可以通过修改某一存储引擎的源码来得到想要的特性。
2. InnoDB存储引擎最早是第三方存储引擎，后来被Oracle收购。

### 1. InnoDB存储引擎
1. InnoDB存储引擎支持事务。其设计目标主要面向在线事务处理（OLTP）的应用。其特点是：行锁设计、支持外键，并支持类似Oracle的非锁定读，即默认读取操作不会产生锁。
2. 从5.5.8版本开始，InnoDB是MySQL默认的存储引擎。
3. InnoDB存储引擎将数据放在一个逻辑的表空间中。表空间由InnoDB自身进行管理。从MySQL4.1开始，它可以将每个InnoDB存储引擎的表单独存放到一个独立的ibd文件中。InnoDB支持用裸设备（raw disk）建立表空间。<strong>裸设备：裸设备(raw device)，也叫裸分区（原始分区），是一种没有经过格式化，不被Unix通过文件系统来读取的特殊块设备文件。由应用程序负责对它进行读写操作。不经过文件系统的缓冲。它是不被操作系统直接管理的设备。这种设备少了操作系统这一层，I/O效率更高。不少数据库都能通过使用裸设备作为存储介质来提高I/O效率。</strong>
4. InnoDB通过使用多版本并发控制（MVCC）来获得高并发性，并实现了SQL的4种隔离级别，默认Repeatable级别。
5. InnoDB使用一种next-key locking的策略来避免幻读的产生。
6. InnoDB提供了插入缓存（insert buffer）、二次写（double write）、自适应哈希索引（adaptive hash index）、预读（read ahead）等高性能和高可用功能。
7. InnoDB表存储数据采用了聚集（clustered）的方式，因此每张表的存储都是按照主键的顺序进行存放。如果没有显式地定义主键，InnoDB会为每一行数据生成一个6字节的ROWID，并以此作为主键。

### 2. MyISAM存储引擎
1. MyISAM存储引擎不支持事务、锁表设计，支持全文索引，主要面向一些OLAP数据库应用。
2. MySQL5.5.8之前默认存储引擎是MyISAM（Windows版本除外）。
3. MyISAM存储引擎的缓冲池只缓存索引文件，而不缓存数据文件。这点和大多数数据库都不同。
4. MyISAM存储引擎表由MYD和MYI组成，MYD用来存放数据文件，MYI用来存放索引文件。
5. MySQL5.0之前MyISAM默认支持的表大小是4G，如果需要更大的MyISAM表的话，就要制定MAX_ROWS和AVG_ROW_LENGTH属性。从5.0版本开始，MyISAM默认支持256TB的单表数据，这足够一般应用的需求。
6. MyISAM存储引擎表，Mysql数据库只缓存其索引文件，数据文件的缓存交给操作系统本身完成，这与LRU算法缓存数据的大部分数据库都不同。MySQL 5.1.23之前，缓存索引的缓冲区最大只能设置为4GB，在之后的版本中，64位系统可以支持大于4GB的索引缓冲区。

### 3. 其他存储引擎

1. NDB： 一个集群存储引擎。数据全部放在内存中（从MySQL 5.1之后，可以将非索引数据放在磁盘上），所以主键查找速度极快，通过添加NDB数据存储节点，可以线性的提高数据库性能，是高可用、高性能的集群系统。复杂的连接操作网络开销很大，因为NDB的连接操作（JOIN）是在数据库层完成的，而不是在存储引擎层完成的。
2. Memory: 之前被称为HEAP存储引擎。将表中的数据存放在内存中，如果数据库重启或发生崩溃，表中的数据全部消失。非常适合非常适合存储临时数据。默认使用hash哈希索引，而不是B+树索引。只支持表锁，并法性能差，不支持TEXT、BLOB列类型。存储变长字段varchar是按照定长char方式存储的，因此会浪费空间。MySQL数据库使用Memory存储引擎作为临时表来存放查询的中间结果集，如果中间结果集大于Memory存储引擎表的容量设置，又或者中间结果含有TEXT或BLOB字段，则MySQL会把其转换成MyISAM存储引擎表存放到磁盘中，因为MyISAM不缓存数据文件，所以这时产生的临时表的性能对于查询会有损失。
3. Archive存储引擎： Archive引擎只支持INSERT和SELECT操作，从MySQL5.1开始支持索引。Archive引擎使用zlib算法将数据行（row）进行压缩后存储，压缩比可达到1：10。正如其名，Archive存储引擎非常适合存储归档数据，如日志信息。通过行锁来实现高并发的插入操作。但是不是事务安全的，目的主要是提供高速的插入和压缩。
4. Federated存储引擎： Federated存储引擎并不存储数据，他只是指向一台远程MySQL数据库服务器上的表。类似SQL Server的链接服务器和Oracle的透明网关。不同的是，Federated只支持MySQL数据库表，不支持异构数据库表。
5. Maria存储引擎： Maria当初是为了取代原有的MyISAM而设计的，从而成为MySQL默认存储引擎。支持缓存数据和索引文件，应用了行锁设计，提供了MVCC功能，支持事务和非事务安全的选项，以及更好的BLOB字符类型的处理性能。


### 4. 其它
1. 查看当前使用MySQL版本支持的引擎：

```sql

show engines\G;

```

## 3. 连接MySQL

### 1. TCP/IP

```bash

mysql -h 192.168.0.101 -u root -p

```

通过TCP/IP连接到数据库实例的时候，MySQL数据库会先检查一张权限视图，用来判断此请求是否允许连接。该视图在MySQL架构下，表名为user。

```sql

use mysql;
select host, user, password from user;

```

### 2. 命名管道和共享内存

如果在MySQL服务器本机连接，可以通过命名管道，但是需要MySQL数据库在配置文件中启用`--enable-named-pipe`选项。在MySQL 4.1之后，还提供了共享内存的连接方式，需要在配置文件中添加`--shared-memory`实现，在连接时，MySQL客户端还需要使用`--protocol=memory`选项。

### 3. UNIX域套接字

在Linux/UNIX环境下，可以使用UNIX域套接字。UNIX域套接字并不是一个网络协议，所以只能在MySQL客户端和数据库实例在一台服务器上时使用。用户可以在配置文件中执行套接字文件的路径。如`--socket=/tmp/mysql.sock`。可以通过命令`show variables like 'socket';`来查找套接字文件。知道了套接字文件的路径后，就可以通过下面的命令连接了：

```bash

mysql -u root -S /tmp/mysql.sock

```

# 第二章 InnoDB存储引擎

## 1. InnoDB存储引擎概述

InnoDB存储引擎是第一个完整支持ACID事务的MySQL存储引擎，其特点是行锁设计、支持MVCC、支持外键、提供一致性非锁定读，同时被设计用来最有效地利用以及使用内存和CPU。

## 2. InnoDB存储引擎的版本

早期InnoDB随MySQL数据库的更新而更新，从MySQL5.1开始，MySQL允许存储引擎开发商以动态方式加载引擎，这样存储引擎可以不受MySQL数据库版本的限制。

## 3. InnoDB体系架构

### 1. 后台线程

InnoDB存储引擎是多线程的模型，后台有多个不同的线程，负责处理不同的任务。

1. Master Thread： 这是一个非常核心的后台线程，主要负责将缓冲池中的数据异步刷新到磁盘，保证数据一致性，包括脏页的刷新、合并插入缓存（INSERT BUFFER）、UNDO页的回收等。
2. IO Thread： 在InnoDB存储引擎中大量的使用了AIO（Async IO）来处理写IO请求，这样极大地提高了数据库的性能。IO Thread主要负责这些IO请求的回调（call back）处理。InnoDB 1.0之前共有4个IO Thread，分别是write、read、insert buffer和log IO Thread。在Linux平台下，IO Thread的数量不能进行调整，但是在Windows平台下，可以通过参数`innodb_file_io_threads`来增大IO Thread。从InnoDB 1.0.x开始，read thread和write thread分别增大到4个，并且不再使用`innodb_file_io_threads`参数，而是分别使用`innodb_read_io_threads`和`innodb_write_io_threads`参数进行设置。

```sql

#查看innodb引擎版本
show variables like 'innodb_version'\G;

#output
*************************** 1. row ***************************
Variable_name: innodb_version
        Value: 8.0.22
1 row in set (1.18 sec)


#查看innodb读写IO线程
show variables like 'innodb_%io_threads'\G;

#output
*************************** 1. row ***************************
Variable_name: innodb_read_io_threads
        Value: 4
*************************** 2. row ***************************
Variable_name: innodb_write_io_threads
        Value: 4
2 rows in set (0.00 sec)


#查看InnoDB中的IO Threads
show engine innodb status\G;

#output
...
--------
FILE I/O
--------
I/O thread 0 state: waiting for completed aio requests (insert buffer thread)
I/O thread 1 state: waiting for completed aio requests (log thread)
I/O thread 2 state: waiting for completed aio requests (read thread)
I/O thread 3 state: waiting for completed aio requests (read thread)
I/O thread 4 state: waiting for completed aio requests (read thread)
I/O thread 5 state: waiting for completed aio requests (read thread)
I/O thread 6 state: waiting for completed aio requests (write thread)
I/O thread 7 state: waiting for completed aio requests (write thread)
I/O thread 8 state: waiting for completed aio requests (write thread)
I/O thread 9 state: waiting for completed aio requests (write thread)
...

```

3. Purge Thread: 事务被提交后，其所使用的undolog可能不再需要，因此需要Purge Thread来回收已经使用并分配的undo页。在InnoDB1.1之前，purge操作仅在Master Thread中完成。而从InnoDB1.1版本开始，purge操作可以独立到单独的线程中进行，以此来减轻Master Thread的工作，从而提高CPU的使用率以及提升存储引擎的性能。用户可以在配置文件中添加配置来启用独立的Purge Thread（见下面的代码），在InnoDB1.1中，即使将purge线程数设置大于1，启动时也会将其设置为1，从1.2版本开始，InnoDB开始支持多个Purge Thread，这样可以加快undo页的回收，由于Purge Thread需要离散的随机读取undo页，这样也能进一步利用磁盘的随机读取性能。

```conf

[mysqld]
innodb_purge_threads=1

```

4. Page Cleaner Thread: InnoDB 1.2.x引入，作用是将之前版本中的脏页的刷新操作都放入到单独的线程中来完成。目的是减轻Master Thread的工作，以及用户查询线程的阻塞，进一步提高InnoDB存储引擎的性能。

### 2. 内存

#### 1. 缓冲池

InnoDB存储引擎是基于磁盘存储的，并将其中的记录按照页的方式进行管理。因此可以将其视为基于磁盘的数据库系统（Disk-base Database）。CPU速度与磁盘速度差距很大，基于磁盘的数据库系统通常采用缓冲池技术来提高数据库的整体性能。

缓冲池就是一块内存区域，通过内存的速度来弥补磁盘速度对数据库性能的影响。

1. 数据库操作时，缓冲池的使用
   - 当数据库进行读取页当操作当时候，首先将从磁盘读到当页缓存在缓冲池中，这个过程称为将页“FIX”在缓冲池中。下一次读取相同的页的时候，首先判断该页是否在缓冲池中，如果在，称该页在缓冲池中被命中，直接读取该页，否则读取磁盘上的页。
   - 对于数据库中页的修改操作，首先修改在缓冲池中的页，然后再以一定的频率刷新到磁盘上，这里要注意，页从缓冲池刷新回磁盘的操作并不是在每次页发生更新时触发，而是通过一种成为`Checkpoint`的机制刷新回磁盘。这样也提高了数据库的整体性能。

2. 缓冲池的大小： 缓冲池的大小直接影响着数据库的整体性能
   1. 系统限制：32位操作系统的限制，最多将该值设置为3G。用户可以打开操作系统的PAE选项来获得32位系统下最大64GB内存的支持。
   2. 强烈建议采用64位操作系统，让数据库使用更多的内存。
   3. 对于InnoDB来说，缓冲池配置通过参数`innodb_buffer_pool_size`来设置

3. 缓冲池的类型： 索引页、数据页、undo页、插入缓冲（insert buffer）、自适应哈希索引（adaptive hash index）、InnoDB存储的锁信息（lock info）、数据字典信息（data dictionary）等。从InnoDB 1.0.x开始，允许有多个缓冲池实例。每个页根据哈希值平均分配到不同到缓冲池实例中，增加了数据库的并发处理能力。可以通过参数`innodb_buffer_pool_instances`来进行配置，默认为1。

#### 2. LRU List、Free List和Flush List

1. LRU List
   - 数据库中的缓冲池是通过LRU（Last Recent Used，最近最少使用）算法来进行管理的。即最频繁使用的页在LRU列表的前端，而最少使用的页在LRU列表的尾端。当缓冲池不能存放新读取到的页时，将首先释放LRU列表中尾端的页。
   - InnoDB对LRU算法做了一些优化，在LRU列表中加入了midpoint位置。新读取到到页，虽然是最新访问到数据，但是也不会直接放到LRU列表到首部，而是放到midpoint的位置。这个算法在InnoDB存储引擎下称为`midpoint insertion strategy`。在默认配置下，midpoint在LRU列表长度的5/8处，也就是LRU列表尾端的3/8（37%）的位置。midpoint位置可以由参数`innodb_old_blocks_pct`控制。在InnoDB中，把midpoint之后的列表称为old列表，之前的列表称为new列表。可以简单的理解为new列表中的页都是做活跃的热点数据。
   - 为什么不用最常见的LRU算法（新数据直接放到首部）？这是因为某些SQL结果集可能超大，可能回将整个LRU列表中的数据刷出，导致真正的热点数据也被清除。加入midpoint可以保护热点数据。
   - InnoDB还有另一个参数`innodb_old_blocks_time`（以毫秒为单位）用于表示页读到mid位置的后需要等待多久才会被加入到LRU列表的热端。页插入到mid位置后的`innodb_old_blocks_time`时间内，可以通过LRU列表访问页，但是无论多少次查询都不会将其移动到new列表，`innodb_old_blocks_time`时间后，如果再次被访问，就会被移动到new列表。

```sql

#为了LRU列表中的热点数据不被刷出啊，可以先设置innodb_old_blocks_time

mysql> SET GLOBAL innodb_old_blocks_time=1000;
Query OK, 0 rows affected.

# data or index scan operation
......

mysql> SET GLOBAL innodb_old_blocks_time=0;
Query OK, 0 rows affected.

```

如果用户预估自己活跃的热点数据不止63%，那么再执行SQL语句前可以通过下面的语句来减少热点页被刷出的概率。

```sql
mysql> SET GLOBAL innodb_old_blocks_pct=20;
Query OK, 0 rows affected.
```


2. Free List

LRU列表是用来管理管理已经读取到的页的，但是当数据库刚启动时，LRU列表是空的，即没有任何的页，这时页都存放于Free列表中。当需要从缓冲池中分页时，首先从Free列表中查找到是否可用的空闲页，若有则将该页从Free列表中删除，放入到LRU列表中。否则，根据LRU算法，淘汰LRU列表末尾的页，将该内存空间分配给新的页。

        这里感觉有点难理解，数据库刚启动的时候，缓冲池里是没有缓存数据的，但是相应的内存空间已经开辟了，当我们查询数据后，磁盘返回数据，这时候，缓冲池要缓存数据，LRU列表是空的，要去Free List里取出空闲页，这里的空闲页其实是一个没有存储数据的页的描述（或者说地址），然后将磁盘返回的数据缓存到这个页，并将这个页交由LRU列表管理。也就是FreeList就是记录空闲页描述的双向列表，当LRU列表需要一个新的页的时候，就来找Free列表要，然后新的数据就被缓存了。

- 这里感觉上LRU列表管理已经读取到的页，Free列表管理未使用的页，他们的size的和就应该是缓冲池里所有页的数量了。但是通过`SHOW ENGINE INNODB STATUS`可以看到，free buffers（Free列表页数量）和Database pages（LRU列表中页的数量）之和并不是缓冲池页的总数（Buffer pool size）。这是因为缓冲池中的页可能会被分配给自适应哈希索引、Lock信息、Insert Buffer等页，而这部分页不需要LRU算法进行维护，因此不在LRU列表中。
- 从InnoDB1.2版本开始，还可以通过`INNODB_BUFFER_POOL_STATUS`来观察缓冲池的运行状态。
   ```sql
        select pool_id, hit_rate, pages_made_young, pages_not_made_young from information_schema.INNODB_BUFFER_POOL_STATUS;
   ```
- 可以通过`INNODB_BUFFER_PAGE_LRU`来观察每个LRU列表中每个页的具体信息，例如通过下面的语句可以看到缓冲池LRU列表中SPACE为1的表的页类型：
    ```sql
        select table_name, space, page_number, page_type from innodb_buffer_page_lru where space = 1;
    ```
- InnoDB存储引擎从1.0.x版本开始支持压缩页的功能，即将原本16k的页压缩为1kb、2kb、4kb和8kb。而由于页的大小发生了变化，LRU列表也有了些许的改动。对于非16kb的页，是通过unzip_LRU列表来进行管理的。通过命令`SHOW ENGINE INNODB STATUS\G;`可以看到LRU列表和unzip_LRU列表的页的数量。LRU中的页包含了unzip_LRU列表中的页。
- unzip_LRU列表中对不同压缩页大小的页进行分别管理，并通过`伙伴算法`进行内存的分配。以需要从缓冲池中申请页为4kb的大小为例，其过程如下：
  
   1. 检查4kb的unzip_LRU列表，检查是否有可用的空闲页；
   2. 如果有，则直接使用；
   3. 否则，检查8kb的unzip_LRU列表；
   4. 如果能够得到空闲页，将页分成2个4KB页，存放到4KB的unzip_LRU列表；
   5. 如果不能得到空闲页，从LRU列表中申请一个16KB的页，将页分为1个8KB的页、2个4KB的页，分别存放到对应的unzip_LRU列表中。

   可以通过information_schema架构下的表INNODB_BUFFER_PAGE_LRU来观察unzip_LRU列表中的页

   ```sql
      select table_name, space, page_number, compressed_size from innodb_buffer_page_lru where compressed_size <> 0;
   ```

   ```
   伙伴算法，简而言之，就是将内存分成若干块，然后尽可能以最适合的方式满足程序内存需求的一种内存管理算法，伙伴算法的一大优势是它能够完全避免外部碎片的产生。什么是外部碎片以及内部碎片，前面博文slab分配器后面已有介绍。申请时，伙伴算法会给程序分配一个较大的内存空间，即保证所有大块内存都能得到满足。很明显分配比需求还大的内存空间，会产生内部碎片。所以伙伴算法虽然能够完全避免外部碎片的产生，但这恰恰是以产生内部碎片为代价的。
   ```
3. Flush List
- 在LRU列表中的页被修改后，称该页为脏页（dirty page），即缓冲池中的页和磁盘上的页的数据产生了不一致。这时数据库会通过`checkpoint`机制将脏页刷新回磁盘，而Flush列表中的页即为`脏页列表`。需要注意的是，脏页既存在于LRU列表，也存在于Flush列表中。LRU列表用于来管理缓冲池中也的可用性，Flush列表用来管理将页刷回磁盘，二者互不影响。
- 同LRU列表一样，Flush列表也能通过`SHOW ENGINE INNODB STATUS`来查看，`Modified db pages 24673`就显示了脏页的数量。
- 查看脏页的数量和类型（table_name为null说明该页属于系统表空间）：
   ```sql
      select table_name, space, page_number, page_type 
      from innodb_buffer_page_lru where oldest_modification > 0;
   ```

### 3. 重做日志缓冲

重做日志缓冲（redo log buffer）。InnoDB存储引擎首先将重做日志信息放入到这个缓冲区，然后按照一定频率将其刷新到重做日志文件。重做日志缓冲不一定要设置的很大，因为一般情况下每一秒会将重做日志缓存刷新到日志文件，因此只需要保证每秒产生的事务量在在这个缓存大小之内即可。该值由配置参数`innodb_log_buffer_size`控制，默认为8M：

```sql
mysql> show variables like 'innodb_log_buffer_size'\G;
*************************** 1. row ***************************
Variable_name: innodb_log_buffer_size
        Value: 16777216
1 row in set (0.00 sec)
```

重做日志在下面三种情况下会将重做日志缓冲中的内容刷新到外部磁盘的重做日志文件中：

- Master Thread每一秒将重做日志缓冲刷新到重做日志文件；
- 每个事务提交时会将重做日志缓存刷新到重做日志文件；
- 当重做日志缓冲池剩余空间小于1/2时，重做日志缓冲会刷新到缓存日志文件。

### 4. 额外的内存池

在InnoDB中，对内存的管理是通过一种称为内存堆（heap）的方式进行的。在对一些数据结构本身的内存进行分配时，需要从额外的内存池中进行申请，当该区域的内存不够时，会从缓冲池中进行申请。这里的数据结构本身是指每个缓冲池中的帧缓存（frame buffer）和对应的缓冲控制对象（buffer control block），这些对象记录了一些诸如LRU、锁、等待等信息，而这些对象的内存需要从额外的内存池中申请。<strong>因此在申请了很大的InnoDB缓冲池时，应该考虑相应的增加这个值</strong>。

## 4. Checkpoint技术

Mysql缓冲池的设计就是为了跨越cpu速度与磁盘速度之间鸿沟，也就是页的操作都是首先在缓冲池中完成，操作完成时，缓冲池的数据版本比磁盘新，数据库需要将新版本的页刷新到磁盘。但是刷新到磁盘，瓶颈依然在磁盘IO，而且如果在缓存刷新到磁盘的过程中宕机，缓存中的数据时不能恢复的。所以当前事务数据库都普遍采用了`Write Ahead Log`策略，即当事务提交时，先写重做日志，在修改页。当由于宕机而导致数据丢失时，可以通过重做日志来完成数据的恢复。这也是事务ACID中D（Durability持久性）的要求。

思考一下，缓冲池可以让我们读写数据更快速，重做日志可以保证数据库数据不丢失。有了缓冲池和重做日志，是不是就可以不用将缓冲池中的页刷新到磁盘了？好家伙，直接内存数据库了？毕竟mysql是要持久化的，是要存大量数据的。你一直不落磁盘，先不说内存够不够大，就是重新启动时重新应用重做日志的时间也够受的。所以mysql有个Checkpoint（检查点）技术。

### 1. Checkout技术用于解决以下问题：

- 缩短数据库的恢复时间
- 缓冲池不够用时，将脏页刷新到磁盘
- 重做日志不可用时，刷新脏页。

1. 当数据库发生宕机时，数据库不需要重做所有日志，因为Checkpoint之前的页都已经刷新回磁盘了，所以只需要对Checkpoint后的重做日志进行恢复。这样就大大缩短了恢复的时间。
2. 当缓冲池不够用时，根据LRU算法会溢出最近使用最少的页，若此页为脏页，那么需要强制执行Checkpoint，将脏页刷新到磁盘。
3. 重做日志出现不可用的情况时因为当前事务数据库系统对重做日志的设计都是循环使用的，并不是无限增大的。重做日志可以被重用的部分是指这些重做日志已经不再需要，即当数据库发生宕机时，数据库恢复操作不需要这部分重做日志。因此这部分就可以被覆盖重用。若此时重做日志还需要使用，那么必须强制产生Checkpoint，将缓冲区中的页至少刷新到当前重做日志的位置。

### 2. LSN（Log Sequece Number）

InnoDB存储引擎是通过LSN来标记版本的。LSN是8字节的数字，单位是字节。每个页有LSN，重做日志中也有LSN，Checkpoint也有LSN。可以通过`SHOW ENGINE INNODB STATUS`来观察：

```sql
mysql> SHOW ENGINE INNODB STATUS\G;

......
---
LOG
---
Log sequence number 92561351052
Log flushed up to 92561351052
Last checkpoint at 92561351052
```

### 3. InnoDB中有两种Checkpoint

#### 1. Sharp Checkpoint

Sharp Checkpoint发生在数据库关闭时将所有的脏页刷新回磁盘，这是默认的工作方式，即参数`innodb_fast_shutdown=1`。因为它时将所有的脏页都刷回磁盘，所以不可能在数据库运行时来执行，太影响性能。

#### 2. Fuzzy Checkpoint

InnoDB存储引擎内部使用Fuzzy Checkpoint进行页的刷新，即只刷新一部分脏页，而不是刷新所有的脏页到磁盘。

在InnoDB中可能发生如下几种情况的Fuzzy Checkpoint：

1. Master Thread Checkpoint

   Master Thread Checkpoint以每秒或每十秒的速度从缓冲池的脏页列表中刷新一定比例的页回磁盘，这个过程是异步的，不会阻塞用户查询线程。

2. FLUSH_LRU_LIST Checkporint

   InnoDB存储引擎需要保证LRU列表中需要有差不多100个空闲页可供使用。InnoDB1.1.x版本之前，需要检查LRU是否有足够的可用空间操作发生在用户查询线程，这就会阻塞用户的查询操作。如果没有100个可用的空闲页，那么InnoDB存储引擎会将LRU列表尾端的页移除。如果这些页有脏页，那么就要进行Checkpoint。从MySQL5.6版本，也就是InnoDB1.2.x开始，这个检查被放在了一个单独的Page Cleaner线程中进行，并且用户可以通过参数innodb_lru_scan_depth控制LRU列表中可用页的数量，该值默认为1024。

3. Async/Sync Flush Checkpoint

   这里是在重做日志文件不可用时，需要强制将一些页刷回磁盘，而此时脏页是从脏页列表中选取的。若已经写入到重做日志的LSN极为redo_lsn，将已经刷新回磁盘最新页的LSN记为checkpoint_lsn，则可定义:

   `checkpoint_age = redo_lsn - checkpoint_lsn`

   再定义一下变量：

   `async_water_mark = 75% * total_redo_log_file_size`

   `sync_water_mark = 90% * total_redo_log_file_size`

   如果每个重做日志文件大小为1GB，并且定义了两个重做日志文件，则重做日志文件的总大小为2GB，那么`async_water_mark`=1.5GB，`sync_water_mark`=1.8GB。

   那么：

   - 当checkpoint_age < async_water_mark时，不需要刷新任何脏页到磁盘。
   - 当async_water_mark < checkpoint_age < sync_water_mark时，触发Async Flush，从Flush列表中刷新足够的脏页回磁盘，是的刷新后满足checkpoint_age<async_water_mark。
   - checkpoint_age > sync_water_mark这种情况很少发生，除非设置的重做日志文件太小，并且在进行类似LOAD DATA的BULK INSERT操作。此时出发Sync Flush操作，从Flush列表中刷新足够的脏页回磁盘，使得刷新后满足checkpoint_age < async_water_mark。

   可见Async/Sync Flush Checkpoint是为了保证重做日志的循环使用的可用性。InnoDB 1.2.x之前，Async会阻塞发现问题的用户查询线程，而Sync会阻塞所有用户的查询线程。但是后来，这部分的刷新操作同样放入了单独的Page Clener Thread中，不会阻塞用户查询线程。

4. Dirty Page too much Checkpoint

   这种情况就是脏页数量太多，导致InnoDB存储引擎强制进行Checkpoint。目的还是保证缓冲池中有足够可用的页。其可由参数`innodb_max_dirty_pages_pct`来控制。百分比，默认时75（老版本InnoDB是90），表示当缓冲池中脏页的数量占据75%时，强制进行checkpoint，刷新一部分的页到磁盘。


## 5. Master Thread工作方式

InnoDB存储引擎的主要工作都是在一个单独的后台线程Master Thread中完成的。

### 1. InnoDB1.0.x版本之前的Master Thread

Master Thread具有最高的线程优先级别。其内部由多个循环（loop）组成：主循环（loop）、后台循环（background loop）、刷新循环（flush loop）、暂停循环（suspend loop）。Master Thread会根据数据库运行的状态在loop、background loop、flush loop和suspend loop中进行切换。

