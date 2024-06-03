---
title: CentOS Stream 9 安装 JDK8
categories: 开发
toc: true
permalink: /centos-install-jdk8.html
---

以 CentOS 9 安装 JDK8 为例

系统信息

```shell
[lighthouse@VM-24-16-centos ~]$ cat /etc/centos-release 
CentOS Stream release 9
```

## 1、下载 JDK 安装文件

```shell
[lighthouse@VM-24-16-centos ~]$ wget https://download.oracle.com/otn/java/jdk/8u411-b09/43d62d619be4e416215729597d70b8ac/jdk-8u411-linux-x64.tar.gz?AuthParam=1717731248_6cef360cd8c71a3af18be813053193d8 -O jdk-8u411-linux-x64.tar.gz
--2024-06-07 11:32:46--  https://download.oracle.com/otn/java/jdk/8u411-b09/43d62d619be4e416215729597d70b8ac/jdk-8u411-linux-x64.tar.gz?AuthParam=1717731248_6cef360cd8c71a3af18be813053193d8
Resolving download.oracle.com (download.oracle.com)... 104.108.64.113
Connecting to download.oracle.com (download.oracle.com)|104.108.64.113|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 146902735 (140M) [application/x-gzip]
Saving to: ‘jdk-8u411-linux-x64.tar.gz’

jdk-8u411-linux-x64.tar.gz                  100%[==========================================================================================>] 140.10M  12.2MB/s    in 12s     

2024-06-07 11:32:59 (11.8 MB/s) - ‘jdk-8u411-linux-x64.tar.gz’ saved [146902735/146902735]
```

## 2、解压源码，复制源码到 opt 目录

```shell
[lighthouse@VM-24-16-centos ~]$ tar xzf jdk-8u411-linux-x64.tar.gz
[lighthouse@VM-24-16-centos ~]$ sudo mv jdk1.8.0_411 /opt/
```

## 3、配置环境 PATH 

按系统建议，不直接编辑 /etc/profile，而是在 /etc/profile.d 目录下新建一个脚本文件

```shell
[lighthouse@VM-24-16-centos ~]$ sudo vim /etc/profile.d/java.sh
[lighthouse@VM-24-16-centos ~]$ sudo cat /etc/profile.d/java.sh 
export JAVA_HOME=/opt/jdk1.8.0_411
export PATH=$PATH:$JAVA_HOME/bin
[lighthouse@VM-24-16-centos ~]$ sudo chmod a+r /etc/profile.d/java.sh
[lighthouse@VM-24-16-centos ~]$ ll /etc/profile.d/java.sh
-rw-r--r-- 1 root root 68 Jun  7 11:47 /etc/profile.d/java.sh
[lighthouse@VM-24-16-centos ~]$ source /etc/profile
[lighthouse@VM-24-16-centos ~]$ java -version
java version "1.8.0_411"
Java(TM) SE Runtime Environment (build 1.8.0_411-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.411-b09, mixed mode)
```
