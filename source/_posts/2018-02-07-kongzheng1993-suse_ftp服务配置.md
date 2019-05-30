---
layout: post
title: "SUSE server FTP配置"
date: 2018-02-01
excerpt: "suse server FTP"
tags: [suse,server,FTP]
categories: [suse,server,FTP]
comments: true
---

## SUSE FTP配置

建议使用vsftp，如果使用了pure-ftpd，需要屏蔽掉pure-ftpd服务。

1. Root用户执行yast2--->network services-->network services （inetd）

将/usr/sbin/pure-ftpd 和/usr/sbin/vsftpd

分别将pure-ftp的状态置为off，vsftpd的状态置为on，然后单击按钮，修改完成。

2. vi /etc/vsftpd.conf 去掉下面几项的注视：

```bash
#write_enable=YES

#local_enable=YES

#ascii_upload_enable=YES

#ascii_download_enable=YES

#listen=YES

#anonymous_enable=NO
```

(3)vi /etc/ftpuser 将root用户注释掉

(4)service vsftpd restart  重启ftp服务
