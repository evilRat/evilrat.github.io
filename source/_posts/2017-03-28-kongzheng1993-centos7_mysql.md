---
layout: post
title: "centos 7 安装mysql遇到的问题"
date: 2016-07-26
excerpt: "getRequestDispatcher,forword,sendRedirect"
tags: [re]
comments: true
---


## cent os 7安装mysql遇到的问题

1.在centos 7上装mysql，但是运行的话会报错，服务未启动

```

[evilrat@evilRat_desktop ~]$ mysql status
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)


```

2.尝试启动服务：

```

[evilrat@evilRat_desktop ~]$ systemctl enable mysql.service
Failed to execute operation: Access denied


```
这样也不行……

```

[evilrat@evilRat_desktop ~]$ service mysql start
Redirecting to /bin/systemctl start  mysql.service



```

3.通过百度找到这个
mariaDB


MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可 MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品。在存储引擎方面，使用XtraDB（英语：XtraDB）来代替MySQL的InnoDB。 MariaDB由MySQL的创始人Michael Widenius（英语：Michael Widenius）主导开发，他早前曾以10亿美元的价格，将自己创建的公司MySQL AB卖给了SUN，此后，随着SUN被甲骨文收购，MySQL的所有权也落入Oracle的手中。MariaDB名称来自Michael Widenius的女儿Maria的名字。
MariaDB基于事务的Maria存储引擎，替换了MySQL的MyISAM存储引擎，它使用了Percona的 XtraDB，InnoDB的变体，分支的开发者希望提供访问即将到来的MySQL 5.4 InnoDB性能。这个版本还包括了 PrimeBase XT (PBXT) 和 FederatedX存储引擎。


4.于是我尝试安装了一下


```

[root@evilRat_desktop evilrat]# yum install mariadb-server -y
Loaded plugins: fastestmirror, langpacks
Repository epel is listed more than once in the configuration
Repository epel-debuginfo is listed more than once in the configuration
Repository epel-source is listed more than once in the configuration
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
adobe-linux-x86_64                                       |  951 B     00:00     
base                                                     | 3.6 kB     00:00     
epel                                                     | 4.3 kB     00:00     
extras                                                   | 3.4 kB     00:00     
nux-dextop                                               | 2.9 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/5): extras/7/x86_64/primary_db                          | 139 kB   00:00     
(2/5): epel/x86_64/updateinfo                              | 765 kB   00:01     
(3/5): epel/x86_64/primary_db                              | 4.6 MB   00:10     
(4/5): updates/7/x86_64/primary_db                         | 3.8 MB   00:11     
(5/5): nux-dextop/x86_64/primary_db                        | 1.7 MB   00:29     
adobe-linux-x86_64/primary                                 | 1.3 kB   00:00     
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * epel: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * nux-dextop: li.nux.ro
 * updates: mirrors.aliyun.com
adobe-linux-x86_64                                                          3/3
Resolving Dependencies
--> Running transaction check
---> Package mariadb-server.x86_64 1:5.5.52-1.el7 will be installed
--> Processing Dependency: perl-DBI for package: 1:mariadb-server-5.5.52-1.el7.x86_64
--> Processing Dependency: perl-DBD-MySQL for package: 1:mariadb-server-5.5.52-1.el7.x86_64
--> Processing Dependency: perl(DBI) for package: 1:mariadb-server-5.5.52-1.el7.x86_64
--> Running transaction check
---> Package perl-DBD-MySQL.x86_64 0:4.023-5.el7 will be installed
---> Package perl-DBI.x86_64 0:1.627-4.el7 will be installed
--> Processing Dependency: perl(RPC::PlServer) >= 0.2001 for package: perl-DBI-1.627-4.el7.x86_64
--> Processing Dependency: perl(RPC::PlClient) >= 0.2000 for package: perl-DBI-1.627-4.el7.x86_64
--> Running transaction check
---> Package perl-PlRPC.noarch 0:0.2020-14.el7 will be installed
--> Processing Dependency: perl(Net::Daemon) >= 0.13 for package: perl-PlRPC-0.2020-14.el7.noarch
--> Processing Dependency: perl(Net::Daemon::Test) for package: perl-PlRPC-0.2020-14.el7.noarch
--> Processing Dependency: perl(Net::Daemon::Log) for package: perl-PlRPC-0.2020-14.el7.noarch
--> Processing Dependency: perl(Compress::Zlib) for package: perl-PlRPC-0.2020-14.el7.noarch
--> Running transaction check
---> Package perl-IO-Compress.noarch 0:2.061-2.el7 will be installed
--> Processing Dependency: perl(Compress::Raw::Zlib) >= 2.061 for package: perl-IO-Compress-2.061-2.el7.noarch
--> Processing Dependency: perl(Compress::Raw::Bzip2) >= 2.061 for package: perl-IO-Compress-2.061-2.el7.noarch
---> Package perl-Net-Daemon.noarch 0:0.48-5.el7 will be installed
--> Running transaction check
---> Package perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7 will be installed
---> Package perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                      Arch        Version               Repository
                                                                           Size
================================================================================
Installing:
 mariadb-server               x86_64      1:5.5.52-1.el7        base       11 M
Installing for dependencies:
 perl-Compress-Raw-Bzip2      x86_64      2.061-3.el7           base       32 k
 perl-Compress-Raw-Zlib       x86_64      1:2.061-4.el7         base       57 k
 perl-DBD-MySQL               x86_64      4.023-5.el7           base      140 k
 perl-DBI                     x86_64      1.627-4.el7           base      802 k
 perl-IO-Compress             noarch      2.061-2.el7           base      260 k
 perl-Net-Daemon              noarch      0.48-5.el7            base       51 k
 perl-PlRPC                   noarch      0.2020-14.el7         base       36 k

Transaction Summary
================================================================================
Install  1 Package (+7 Dependent packages)

Total download size: 12 M
Installed size: 59 M
Downloading packages:
(1/8): perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64.rpm      |  32 kB   00:00     
(2/8): perl-Compress-Raw-Zlib-2.061-4.el7.x86_64.rpm       |  57 kB   00:00     
(3/8): perl-DBD-MySQL-4.023-5.el7.x86_64.rpm               | 140 kB   00:00     
(4/8): perl-DBI-1.627-4.el7.x86_64.rpm                     | 802 kB   00:00     
(5/8): perl-IO-Compress-2.061-2.el7.noarch.rpm             | 260 kB   00:00     
(6/8): perl-Net-Daemon-0.48-5.el7.noarch.rpm               |  51 kB   00:00     
(7/8): perl-PlRPC-0.2020-14.el7.noarch.rpm                 |  36 kB   00:00     
(8/8): mariadb-server-5.5.52-1.el7.x86_64.rpm              |  11 MB   00:12     
--------------------------------------------------------------------------------
Total                                              1.0 MB/s |  12 MB  00:12     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64                   1/8 
  Installing : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64                  2/8 
  Installing : perl-IO-Compress-2.061-2.el7.noarch                          3/8 
  Installing : perl-Net-Daemon-0.48-5.el7.noarch                            4/8 
  Installing : perl-PlRPC-0.2020-14.el7.noarch                              5/8 
  Installing : perl-DBI-1.627-4.el7.x86_64                                  6/8 
  Installing : perl-DBD-MySQL-4.023-5.el7.x86_64                            7/8 
  Installing : 1:mariadb-server-5.5.52-1.el7.x86_64                         8/8 
  Verifying  : perl-Net-Daemon-0.48-5.el7.noarch                            1/8 
  Verifying  : 1:mariadb-server-5.5.52-1.el7.x86_64                         2/8 
  Verifying  : perl-IO-Compress-2.061-2.el7.noarch                          3/8 
  Verifying  : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64                  4/8 
  Verifying  : perl-PlRPC-0.2020-14.el7.noarch                              5/8 
  Verifying  : perl-DBI-1.627-4.el7.x86_64                                  6/8 
  Verifying  : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64                   7/8 
  Verifying  : perl-DBD-MySQL-4.023-5.el7.x86_64                            8/8 

Installed:
  mariadb-server.x86_64 1:5.5.52-1.el7                                          

Dependency Installed:
  perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7                                  
  perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7                                   
  perl-DBD-MySQL.x86_64 0:4.023-5.el7                                           
  perl-DBI.x86_64 0:1.627-4.el7                                                 
  perl-IO-Compress.noarch 0:2.061-2.el7                                         
  perl-Net-Daemon.noarch 0:0.48-5.el7                                           
  perl-PlRPC.noarch 0:0.2020-14.el7                                             

Complete!




```

5.然后我启动服务，尝试启动mysql

```

[root@evilRat_desktop evilrat]# systemctl start mariadb.service
[root@evilRat_desktop evilrat]# systemctl enable mariadb.service
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@evilRat_desktop evilrat]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.52-MariaDB MariaDB Server

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases
    -> ;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> exit
Bye



```




<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-centos7_mysql/" data-title="centos7_mysql" data-url="http://kongzheng1993.github.io/kongzheng1993-centos7_mysql/"></div>
<!-- 多说评论框 end -->
<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
<script type="text/javascript">
var duoshuoQuery = {short_name:"kongzheng1993"};
    (function() {
        var ds = document.createElement('script');
        ds.type = 'text/javascript';ds.async = true;
        ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
        ds.charset = 'UTF-8';
        (document.getElementsByTagName('head')[0] 
         || document.getElementsByTagName('body')[0]).appendChild(ds);
    })();
</script>
</html>

