---
title: "Linux下安装FoundationDB"
date: 2018-04-23T21:20:03+08:00
draft: false
categories: ["db", "foundationdb"]
keywords: ["db", "nosql", "foundationdb"]
tags: ["db", "nosql", "foundationdb"]
---



<!-- # Linux下安装FoundationDB -->
### 简介
2015 年苹果公司收购了数据提供商 FoundationDB，目的是为了提升旗下 App Store、iTunes Connect、 iTunes 服务在云端的服务器技术。FoundationDB 随之从开源变为闭源，而三年后的现在，它又重新开源了。
FoundationDB是”一个能在多集群服务器上存放大规模结构化数据的分布式数据库“。该数据库系统专注于高性能、高可扩展性、优秀的容错能力。
FoundationDB是由戴夫·罗森塔尔（Dave Rosenthal）、戴夫·谢勒（Dave Scherer）、和尼克拉维泽（Nick Lavezzo）于2009年开发的，旨在建立一个符合ACID约束的NoSQL数据库，ACID是一种即使在发生错误时也保证数据完整性的数据库机制。



-  [下载地址][https://apple.github.io/foundationdb/downloads.html]

`系统要求: `

- 64位操作系统
- RHEL/CentOS 6.x或者7.x
- Ubuntu 12.04或更高版本
- 不支持的Linux发行版包含：
内核版本介于2.6.33和3.0.x之间（含）或3.7或更高,适用于.deb或.rpm包 或者，macOS 10.7或更高版本


>      警告： macOS下FoundationDB仅适用于本地访问。


### Ubuntu
>      google@localhost$ sudo dpkg -i foundationdb-clients_5.1.5-1_amd64.deb foundationdb-server_5.1.5-1_amd64.deb

### RHEL/CentOS 6
>      google@localhost$ sudo rpm -Uvh foundationdb-clients-5.1.5-1.el7.x86_64.rpm foundationdb-server-5.1.5-1.el7.x86_64.rpm

### RHEL/CentOS 7
>      google@localhost$ sudo rpm -Uvh foundationdb-clients-5.1.5-1.el7.x86_64.rpm  foundationdb-server-5.1.5-1.el7.x86_64.rpm

### 测试

``` sql
google@localhost$ fdbcli
Using cluster file `/etc/foundationdb/fdb.cluster'.

The database is available.

Welcome to the fdbcli. For help, type `help'.
fdb> status

Configuration:
  Redundancy mode        - single
  Storage engine         - memory
  Coordinators           - 1

Cluster:
  FoundationDB processes - 1
  Machines               - 1
  Memory availability    - 4.1 GB per process on machine with least available
  Fault Tolerance        - 0 machines
  Server time            - Thu Mar 15 14:41:34 2018

Data:
  Replication health     - Healthy
  Moving data            - 0.000 GB
  Sum of key-value sizes - 8 MB
  Disk space used        - 103 MB

Operating space:
  Storage server         - 1.0 GB free on most full server
  Transaction log        - 1.0 GB free on most full server

Workload:
  Read rate              - 2 Hz
  Write rate             - 0 Hz
  Transactions started   - 2 Hz
  Transactions committed - 0 Hz
  Conflict rate          - 0 Hz

Backup and DR:
  Running backups        - 0
  Running DRs            - 0

Client time: Thu Mar 15 14:41:34 2018
```
如果显示上述代表成功。


-------------------

[1] https://apple.github.io/foundationdb/downloads.html