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

```shell

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

```shell

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

3. Purge Thread: 事务被提交后，其所使用的undolog可能不再需要，因此需要Purge Thread来回收已经使用并分配的undo页。在InnoDB1.1之前，purge操作仅在Master Thread中完成。而从InnoDB1.1版本开始，purge操作可以独立到单独的线程中进行，以此来减轻Master Thread的工作，从而提高CPU的使用率以及提升存储引擎的性能。用户可以在配置文件中添加配置来启用独立的Purge Thread：
    ```conf

    [mysqld]
    innodb_purge_threads=1

    ```
    在InnoDB1.1中，即使将purge线程数设置大于1，启动时也会将其设置为1，从1.2版本开始，InnoDB开始支持多个Purge Thread，这样可以加快undo页的回收，由于Purge Thread需要离散的随机读取undo页，这样也能
