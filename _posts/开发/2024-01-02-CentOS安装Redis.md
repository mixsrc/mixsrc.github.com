---
title: CentOS Stream 9 编译安装 Redis 7.2.5
categories: 开发
toc: true
permalink: /centos-install-redis.html
---

以 CentOS 9 安装 Redis-7.2.5 为例，官方文档如下：

https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-from-source/

系统信息

```shell
[lighthouse@VM-24-16-centos ~]$ cat /etc/centos-release 
CentOS Stream release 9
```

## 1、下载 Redis 源码

```shell
[lighthouse@VM-24-16-centos ~]$ wget https://download.redis.io/releases/redis-7.2.5.tar.gz
--2024-06-07 09:57:24--  https://download.redis.io/releases/redis-7.2.5.tar.gz
Resolving download.redis.io (download.redis.io)... 45.60.125.1
Connecting to download.redis.io (download.redis.io)|45.60.125.1|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3386454 (3.2M) [application/octet-stream]
Saving to: ‘redis-7.2.5.tar.gz’

redis-7.2.5.tar.gz                          100%[==========================================================================================>]   3.23M  7.73MB/s    in 0.4s    

2024-06-07 09:57:25 (7.73 MB/s) - ‘redis-7.2.5.tar.gz’ saved [3386454/3386454]
```

## 2、解压源码，进入源码目录

```shell
[lighthouse@VM-24-16-centos ~]$ tar xzf redis-7.2.5.tar.gz 
[lighthouse@VM-24-16-centos ~]$ ls
mysql84-community-release-el9-1.noarch.rpm  redis-7.2.5  redis-7.2.5.tar.gz
[lighthouse@VM-24-16-centos ~]$ cd redis-7.2.5
[lighthouse@VM-24-16-centos redis-7.2.5]$ ls
00-RELEASENOTES  CODE_OF_CONDUCT.md  COPYING  INSTALL   MANIFESTO  redis.conf  runtest-cluster    runtest-sentinel  sentinel.conf  tests   utils
BUGS             CONTRIBUTING.md     deps     Makefile  README.md  runtest     runtest-moduleapi  SECURITY.md       src            TLS.md
```

## 3、执行 make 命令

```shell
[lighthouse@VM-24-16-centos redis-7.2.5]$ make
cd src && make all
make[1]: Entering directory '/home/lighthouse/redis-7.2.5/src'
    CC Makefile.dep
    CC logreqres.o
......
......
......
    LINK redis-server
    INSTALL redis-sentinel
    CC redis-cli.o
    CC redisassert.o
    CC cli_common.o
    CC cli_commands.o
    LINK redis-cli
    CC redis-benchmark.o
    LINK redis-benchmark
    INSTALL redis-check-rdb
    INSTALL redis-check-aof

Hint: It's a good idea to run 'make test' ;)

make[1]: Leaving directory '/home/lighthouse/redis-7.2.5/src'
```

## 4、执行测试

```shell
[lighthouse@VM-24-16-centos redis-7.2.5]$ make test
cd src && make test
make[1]: Entering directory '/home/lighthouse/redis-7.2.5/src'
    CC Makefile.dep
You need tcl 8.5 or newer in order to run the Redis test
make[1]: *** [Makefile:462: test] Error 1
make[1]: Leaving directory '/home/lighthouse/redis-7.2.5/src'
make: *** [Makefile:6: test] Error 2

[lighthouse@VM-24-16-centos redis-7.2.5]$ sudo yum install tcl
Last metadata expiration check: 1:00:37 ago on Fri 07 Jun 2024 09:03:04 AM CST.
Dependencies resolved.
===============================================================================================================================================================================
 Package                              Architecture                            Version                                            Repository                               Size
===============================================================================================================================================================================
Installing:
 tcl                                  x86_64                                  1:8.6.10-7.el9                                     baseos                                  1.1 M

Transaction Summary
===============================================================================================================================================================================
Install  1 Package

Total download size: 1.1 M
Installed size: 4.1 M
Is this ok [y/N]: y
Downloading Packages:
tcl-8.6.10-7.el9.x86_64.rpm                                                                                                                    678 kB/s | 1.1 MB     00:01    
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                          678 kB/s | 1.1 MB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                       1/1 
  Installing       : tcl-1:8.6.10-7.el9.x86_64                                                                                                                             1/1 
  Running scriptlet: tcl-1:8.6.10-7.el9.x86_64                                                                                                                             1/1 
  Verifying        : tcl-1:8.6.10-7.el9.x86_64                                                                                                                             1/1 

Installed:
  tcl-1:8.6.10-7.el9.x86_64                                                                                                                                                    

Complete!

[lighthouse@VM-24-16-centos redis-7.2.5]$ make test
cd src && make test
make[1]: Entering directory '/home/lighthouse/redis-7.2.5/src'
Cleanup: may take some time... OK
Starting test server at port 21079
[ready]: 58547
Testing unit/printver
......
......
......
  1 seconds - violations
  152 seconds - defrag
  0 seconds - set-large-memory

\o/ All tests passed without errors!

Cleanup: may take some time... OK
make[1]: Leaving directory '/home/lighthouse/redis-7.2.5/src'
```

## 5、执行 make install

```shell
[lighthouse@VM-24-16-centos redis-7.2.5]$ sudo make install
cd src && make install
make[1]: Entering directory '/home/lighthouse/redis-7.2.5/src'

Hint: It's a good idea to run 'make test' ;)

    INSTALL redis-server
    INSTALL redis-benchmark
    INSTALL redis-cli
make[1]: Leaving directory '/home/lighthouse/redis-7.2.5/src'
```

## 6、查看版本，验证安装

```shell
[lighthouse@VM-24-16-centos redis-7.2.5]$ redis-server --version
Redis server v=7.2.5 sha=00000000:0 malloc=jemalloc-5.3.0 bits=64 build=e39b33f702c1c9f0
```

## 7、设置配置文件

复制配置文件到/etc/目录下，设置好权限，绑定所有端口，设置密码，开启后台运行

```shell
[lighthouse@VM-24-16-centos redis-7.2.5]$ sudo cp redis.conf /etc/
[lighthouse@VM-24-16-centos redis-7.2.5]$ ll /etc/redis.conf 
-rw-r----- 1 root root 107514 Jun  7 10:30 /etc/redis.conf
[lighthouse@VM-24-16-centos redis-7.2.5]$ sudo chmod o+r /etc/redis.conf 
[lighthouse@VM-24-16-centos redis-7.2.5]$ ll /etc/redis.conf 
-rw-r--r-- 1 root root 107514 Jun  7 10:30 /etc/redis.conf
[lighthouse@VM-24-16-centos redis-7.2.5]$ sudo vim /etc/redis.conf 
[lighthouse@VM-24-16-centos redis-7.2.5]$ diff redis.conf /etc/redis.conf
87c87
< bind 127.0.0.1 -::1
---
> # bind 127.0.0.1 -::1
309c309
< daemonize no
---
> daemonize yes
1044c1044
< # requirepass foobared
---
> requirepass ********
```

## 8、启动 Redis

```shell
[lighthouse@VM-24-16-centos redis-7.2.5]$ redis-server /etc/redis.conf
115821:C 07 Jun 2024 11:00:48.600 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
[lighthouse@VM-24-16-centos redis-7.2.5]$ redis-cli -a ******** shutdown
[lighthouse@VM-24-16-centos redis-7.2.5]$ sudo sysctl vm.overcommit_memory=1
[lighthouse@VM-24-16-centos redis-7.2.5]$ redis-server /etc/redis.conf
[lighthouse@VM-24-16-centos ~]$ sudo vim /etc/sysctl.conf
添加 vm.overcommit_memory = 1
```

## 9、下载 Windows 客户端

https://github.com/redis-windows/redis-windows

```shell
PS D:\APP\Redis-7.2.5-Windows-x64-msys2> .\redis-cli.exe -h 101.**.***.**
101.**.***.**:6379> ping
(error) NOAUTH Authentication required.
101.**.***.**:6379> AUTH ********
OK
101.**.***.**:6379> ping
PONG
```


