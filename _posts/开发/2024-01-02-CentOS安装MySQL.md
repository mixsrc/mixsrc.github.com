---
title: CentOS Stream 9 通过 Yum 安装 MySQL 8.4
categories: 开发
toc: true
permalink: /centos-install-mysql.html
---

以 CentOS 9 安装 MySQL 8.4 为例，官网文档如下

https://dev.mysql.com/doc/refman/8.4/en/linux-installation-yum-repo.html

系统信息

```shell
[lighthouse@VM-24-16-centos ~]$ cat /etc/centos-release 
CentOS Stream release 9
```

## 1、下载 yum 仓库

```shell
[lighthouse@VM-24-16-centos ~]$ wget https://repo.mysql.com//mysql84-community-release-el9-1.noarch.rpm
--2024-06-07 08:11:34--  https://repo.mysql.com//mysql84-community-release-el9-1.noarch.rpm
Resolving repo.mysql.com (repo.mysql.com)... 23.211.42.21, 2600:1417:4400:e84::1d68, 2600:1417:4400:e8c::1d68
Connecting to repo.mysql.com (repo.mysql.com)|23.211.42.21|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13139 (13K) [application/x-redhat-package-manager]
Saving to: ‘mysql84-community-release-el9-1.noarch.rpm’

mysql84-community-release-el9-1.noarch.rpm  100%[==========================================================================================>]  12.83K  --.-KB/s    in 0s      

2024-06-07 08:11:36 (114 MB/s) - ‘mysql84-community-release-el9-1.noarch.rpm’ saved [13139/13139]
```

## 2、添加 yum 仓库

```shell
[lighthouse@VM-24-16-centos ~]$ sudo yum localinstall mysql84-community-release-el9-1.noarch.rpm 
CentOS Stream 9 - AppStream                                                                                                                     29 MB/s |  19 MB     00:00    
CentOS Stream 9 - BaseOS                                                                                                                       5.2 MB/s | 8.1 MB     00:01    
Last metadata expiration check: 0:00:02 ago on Fri 07 Jun 2024 08:14:03 AM CST.
Dependencies resolved.
===============================================================================================================================================================================
 Package                                                Architecture                        Version                            Repository                                 Size
===============================================================================================================================================================================
Installing:
 mysql84-community-release                              noarch                              el9-1                              @commandline                               13 k

Transaction Summary
===============================================================================================================================================================================
Install  1 Package

Total size: 13 k
Installed size: 14 k
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                       1/1 
  Installing       : mysql84-community-release-el9-1.noarch                                                                                                                1/1 
  Verifying        : mysql84-community-release-el9-1.noarch                                                                                                                1/1 

Installed:
  mysql84-community-release-el9-1.noarch                                                                                                                                       

Complete!
```

## 3、查看MySQL版本及安装配置

可以看到默认就是安装的 mysql-8.4-lts-community

```shell
[lighthouse@VM-24-16-centos ~]$ yum repolist all | grep mysql
mysql-8.4-lts-community                      MySQL 8.4 LTS Community Se enabled
mysql-8.4-lts-community-debuginfo            MySQL 8.4 LTS Community Se disabled
mysql-8.4-lts-community-source               MySQL 8.4 LTS Community Se disabled
mysql-cluster-8.0-community                  MySQL Cluster 8.0 Communit disabled
mysql-cluster-8.0-community-debuginfo        MySQL Cluster 8.0 Communit disabled
mysql-cluster-8.0-community-source           MySQL Cluster 8.0 Communit disabled
mysql-cluster-8.4-lts-community              MySQL Cluster 8.4 LTS Comm disabled
mysql-cluster-8.4-lts-community-debuginfo    MySQL Cluster 8.4 LTS Comm disabled
mysql-cluster-8.4-lts-community-source       MySQL Cluster 8.4 LTS Comm disabled
mysql-cluster-innovation-community           MySQL Cluster Innovation R disabled
mysql-cluster-innovation-community-debuginfo MySQL Cluster Innovation R disabled
mysql-cluster-innovation-community-source    MySQL Cluster Innovation R disabled
mysql-connectors-community                   MySQL Connectors Community enabled
mysql-connectors-community-debuginfo         MySQL Connectors Community disabled
mysql-connectors-community-source            MySQL Connectors Community disabled
mysql-innovation-community                   MySQL Innovation Release C disabled
mysql-innovation-community-debuginfo         MySQL Innovation Release C disabled
mysql-innovation-community-source            MySQL Innovation Release C disabled
mysql-tools-8.4-lts-community                MySQL Tools 8.4 LTS Commun enabled
mysql-tools-8.4-lts-community-debuginfo      MySQL Tools 8.4 LTS Commun disabled
mysql-tools-8.4-lts-community-source         MySQL Tools 8.4 LTS Commun disabled
mysql-tools-community                        MySQL Tools Community      disabled
mysql-tools-community-debuginfo              MySQL Tools Community - De disabled
mysql-tools-community-source                 MySQL Tools Community - So disabled
mysql-tools-innovation-community             MySQL Tools Innovation Com disabled
mysql-tools-innovation-community-debuginfo   MySQL Tools Innovation Com disabled
mysql-tools-innovation-community-source      MySQL Tools Innovation Com disabled
mysql80-community                            MySQL 8.0 Community Server disabled
mysql80-community-debuginfo                  MySQL 8.0 Community Server disabled
mysql80-community-source                     MySQL 8.0 Community Server disabled
```

## 4、安装 MySQL

```sehll
[lighthouse@VM-24-16-centos ~]$ sudo yum install mysql-community-server
MySQL 8.4 LTS Community Server                                                                                                                 162 kB/s | 226 kB     00:01    
MySQL Connectors Community                                                                                                                      61 kB/s |  53 kB     00:00    
MySQL Tools 8.4 LTS Community                                                                                                                   89 kB/s |  97 kB     00:01    
Dependencies resolved.
===============================================================================================================================================================================
 Package                                              Architecture                 Version                                 Repository                                     Size
===============================================================================================================================================================================
Installing:
 mysql-community-server                               x86_64                       8.4.0-1.el9                             mysql-8.4-lts-community                        50 M
Installing dependencies:
 libaio                                               x86_64                       0.3.111-13.el9                          baseos                                         24 k
 mysql-community-client                               x86_64                       8.4.0-1.el9                             mysql-8.4-lts-community                       3.1 M
 mysql-community-client-plugins                       x86_64                       8.4.0-1.el9                             mysql-8.4-lts-community                       1.4 M
 mysql-community-common                               x86_64                       8.4.0-1.el9                             mysql-8.4-lts-community                       576 k
 mysql-community-icu-data-files                       x86_64                       8.4.0-1.el9                             mysql-8.4-lts-community                       2.3 M
 mysql-community-libs                                 x86_64                       8.4.0-1.el9                             mysql-8.4-lts-community                       1.5 M

Transaction Summary
===============================================================================================================================================================================
Install  7 Packages

Total download size: 59 M
Installed size: 330 M
Is this ok [y/N]: y
Downloading Packages:
(1/7): libaio-0.3.111-13.el9.x86_64.rpm                                                                                                        176 kB/s |  24 kB     00:00    
(2/7): mysql-community-common-8.4.0-1.el9.x86_64.rpm                                                                                           479 kB/s | 576 kB     00:01    
(3/7): mysql-community-client-plugins-8.4.0-1.el9.x86_64.rpm                                                                                   1.0 MB/s | 1.4 MB     00:01    
(4/7): mysql-community-client-8.4.0-1.el9.x86_64.rpm                                                                                           1.9 MB/s | 3.1 MB     00:01    
(5/7): mysql-community-libs-8.4.0-1.el9.x86_64.rpm                                                                                             4.5 MB/s | 1.5 MB     00:00    
(6/7): mysql-community-icu-data-files-8.4.0-1.el9.x86_64.rpm                                                                                   3.9 MB/s | 2.3 MB     00:00    
(7/7): mysql-community-server-8.4.0-1.el9.x86_64.rpm                                                                                            12 MB/s |  50 MB     00:04    
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                           10 MB/s |  59 MB     00:05     
MySQL 8.4 LTS Community Server                                                                                                                 3.0 MB/s | 3.1 kB     00:00    
Importing GPG key 0xA8D3785C:
 Userid     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 Fingerprint: BCA4 3417 C3B4 85DD 128E C6D4 B7B3 B788 A8D3 785C
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2023
Is this ok [y/N]: y
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                       1/1 
  Installing       : mysql-community-common-8.4.0-1.el9.x86_64                                                                                                             1/7 
  Installing       : mysql-community-client-plugins-8.4.0-1.el9.x86_64                                                                                                     2/7 
  Installing       : mysql-community-libs-8.4.0-1.el9.x86_64                                                                                                               3/7 
  Running scriptlet: mysql-community-libs-8.4.0-1.el9.x86_64                                                                                                               3/7 
  Installing       : mysql-community-client-8.4.0-1.el9.x86_64                                                                                                             4/7 
  Installing       : mysql-community-icu-data-files-8.4.0-1.el9.x86_64                                                                                                     5/7 
  Installing       : libaio-0.3.111-13.el9.x86_64                                                                                                                          6/7 
  Running scriptlet: mysql-community-server-8.4.0-1.el9.x86_64                                                                                                             7/7 
  Installing       : mysql-community-server-8.4.0-1.el9.x86_64                                                                                                             7/7 
  Running scriptlet: mysql-community-server-8.4.0-1.el9.x86_64                                                                                                             7/7 
  Verifying        : libaio-0.3.111-13.el9.x86_64                                                                                                                          1/7 
  Verifying        : mysql-community-client-8.4.0-1.el9.x86_64                                                                                                             2/7 
  Verifying        : mysql-community-client-plugins-8.4.0-1.el9.x86_64                                                                                                     3/7 
  Verifying        : mysql-community-common-8.4.0-1.el9.x86_64                                                                                                             4/7 
  Verifying        : mysql-community-icu-data-files-8.4.0-1.el9.x86_64                                                                                                     5/7 
  Verifying        : mysql-community-libs-8.4.0-1.el9.x86_64                                                                                                               6/7 
  Verifying        : mysql-community-server-8.4.0-1.el9.x86_64                                                                                                             7/7 

Installed:
  libaio-0.3.111-13.el9.x86_64                        mysql-community-client-8.4.0-1.el9.x86_64                   mysql-community-client-plugins-8.4.0-1.el9.x86_64          
  mysql-community-common-8.4.0-1.el9.x86_64           mysql-community-icu-data-files-8.4.0-1.el9.x86_64           mysql-community-libs-8.4.0-1.el9.x86_64                    
  mysql-community-server-8.4.0-1.el9.x86_64          

Complete!
```

## 5、启动MySQL服务

```shell
[lighthouse@VM-24-16-centos ~]$ sudo systemctl start mysqld
```

## 6、从日志查查找 root 的登陆密码

```shell
[lighthouse@VM-24-16-centos ~]$ sudo grep 'temporary password' /var/log/mysqld.log
2024-06-07T01:50:08.087272Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: .egikTfW711,
```

## 7、登陆 MySQL，重置 root 的登陆密码

```shell
[lighthouse@VM-24-16-centos ~]$ mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.4.0

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '********';
Query OK, 0 rows affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)

mysql> exit
Bye
```