# 系统性能监控工具：osquery 和 glances

<!-- toc -->

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [系统性能监控工具：osquery 和 glances](#系统性能监控工具osquery-和-glances)
	- [glances 系统监控](#glances-系统监控)
		- [安装](#安装)
		- [监控截图（单机）](#监控截图单机)
	- [osquery](#osquery)
		- [跨平台](#跨平台)
		- [安装](#安装)
		- [查看体验](#查看体验)
		- [显示更方便](#显示更方便)
		- [更多](#更多)
- [参考资料](#参考资料)

<!-- /TOC -->

- ``glances``: 内存、CPU、IO（磁盘和网络）。它是用``python``写的。
- ``osquery``:  Facebook开源，以SQL的方式查看系统CPU，内存，IO等情况。

- 传统工具：top, iostat, sar

----

## glances 系统监控

不仅支持直接在终端上显示数据，还支持在web上显示，除了监视本机的负载，还可以非常轻易地监控指定服务器（前提是装有Glances，并以server模式启动glances）的负载，甚至提供了REST和XML-RPC的编程接口，支持数据保存在CSV，html或 **InfluxDB** 和Statsd。

### 安装

```
apt-get install python-pip build-essential python-dev
pip install glances bottle     #如果你的linux上缺少bottle这个模块，会无法在web上显示
```

>glances 是python写的，``pip install glances`` 是python的包管理。

### 监控截图（单机）

![glances looking.png](http://upload-images.jianshu.io/upload_images/2915464-5864b2489abf6416.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-----

## osquery

### 跨平台

``osquery`` 做到跨平台了，任何平台的CPU，内存，网络IO，磁盘IO等都以SQL的形式监控。

![osquery 跨平台.png](http://upload-images.jianshu.io/upload_images/2915464-ab8e18c3056e0920.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

甚至连 firefox 浏览器装了哪些插件，它都能查看。

### 安装

mac 下：

``` bash
$ brew install osquery
$ osqueryi
```

安装完成后，如果运行``osqueryi`` 报错：

```
osqueryi
dyld: Library not loaded: /usr/local/opt/libxml2/lib/libxml2.2.dylib
  Referenced from: /usr/local/bin/osqueryi
  Reason: Incompatible library version: osqueryi requires version 12.0.0 or later, but libxml2.2.dylib provides version 10.0.0
[1]    5666 trace trap  osqueryi
```

请执行：
```
brew install libxml2 libxslt
brew link libxml2 libxslt --force
```

其他平台安装，请参考官方资料：
https://osquery.io/downloads/

比如 centos 6 下 rpm 下载安装：

```
$ wget https://osquery-packages.s3.amazonaws.com/centos6/osquery-2.3.2-1.el6.x86_64.rpm
$ rpm -ivh osquery-2.3.2-1.el6.x86_64.rpm
$ osqueryi
```

### 查看体验

- system_info: 机器情况

``` bash
osquery> select * from system_info;
          hostname = MacBook-Pro.local
              uuid = 49258E3F-83F0-55B0-9A5E-XXXXXX
          cpu_type = x86_64h
       cpu_subtype = Intel x86-64h Haswell
         cpu_brand = Intel(R) Core(TM) i5-5257U CPU @ 2.70GHz
cpu_physical_cores = 2
 cpu_logical_cores = 4
   physical_memory = 8589934592
   hardware_vendor = Apple Inc.
    hardware_model = MacBookPro12,1
  hardware_version = 1.0
   hardware_serial = C02PNPL6FVH5
     computer_name =
```


- os_version: 操作系统版本

``` bash
osquery> select * from os_version;
+----------+---------+-------+-------+-------+-------+----------+---------------+----------+
| name     | version | major | minor | patch | build | platform | platform_like | codename |
+----------+---------+-------+-------+-------+-------+----------+---------------+----------+
| Mac OS X | 10.10.4 | 10    | 10    | 4     | 14E46 | darwin   | darwin        |          |
+----------+---------+-------+-------+-------+-------+----------+---------------+----------+
```

如果没有跨平台，不同系统的查看方式不一样：
```
cat /etc/issue
CentOS release 6.5 (Final)
Kernel \r on an \m
```

- arp_cache: 以太网内IP地址到Mac地址的映射关系

比如查操作系统里面，缓存的 IP地址 到mac地址的映射关系。

``` bash
$ osqueryi     # 注意：osqueryi  (命令行)  不是 osquery
osquery> select * from arp_cache;
+--------------+-------------------+-----------+-----------+
| address      | mac               | interface | permanent |
+--------------+-------------------+-----------+-----------+
| 10.1.114.1   | 00:00:5e:00:01:aa | en0       | 0         |
| 10.1.114.19  | 48:45:20:1e:a2:58 | en0       | 0         |
| 10.1.114.29  | ac:bc:32:7e:c1:59 | en0       | 0         |
| 10.1.114.52  | 90:b9:31:21:e1:f1 | en0       | 0         |
```

- routes： 查看系统路由表

当然一般普通的工作台的路由表很简单，无非是“本网段直发，跨网段发gateway（第一条路由器）”。

查看Mac系统的第一条路由器（点击任务栏上的wifi标示 选择“打开网络偏好设置” 点击“高级” 就可看见了）：

![first hop router.png](http://upload-images.jianshu.io/upload_images/2915464-646ffd9a99079035.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看路由表：

```
osquery> select * from routes;
+-----------------------------+---------+------------+--------+-----------+-----------+-------+--------+---------+
| destination                 | netmask | gateway    | source | flags     | interface | mtu   | metric | type    |
+-----------------------------+---------+------------+--------+-----------+-----------+-------+--------+---------+
| 127.0.0.1                   | 32      | 127.0.0.1  |        | 2097157   | lo0       | 16384 | 0      | local   |
| ::1                         | 128     | ::1        |        | 2098181   | lo0       | 16384 | 0      | local   |
| fe80:1::1                   | 128     |            |        | 18875397  | lo0       | 16384 | 0      | local   |
| fe80:4::a65e:60ff:fed6:1c55 | 128     |            |        | 18875397  | lo0       | 16384 | 0      | local   |
| 0.0.0.0                     | 0       | 10.1.114.1 |        | 67587     | en0       | 1500  | 0      | gateway |
| 10.0.5.55                   | 32      | 10.1.114.1 |        | 17170439  | en0       | 1500  | 0      | gateway |
```

- interface_addresses: 网卡情况

``` bash
osquery> select * from interface_addresses;
+-----------+---------------------------+-----------------------------------------+--------------+----------------+
| interface | address                   | mask                                    | broadcast    | point_to_point |
+-----------+---------------------------+-----------------------------------------+--------------+----------------+
| lo0       | ::1                       | ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff |              | ::1            |
| lo0       | 127.0.0.1                 | 255.0.0.0                               |              | 127.0.0.1      |
| lo0       | fe80::1                   | ffff:ffff:ffff:ffff::                   |              |                |
| en0       | fe80::a65e:60ff:fed6:1c55 | ffff:ffff:ffff:ffff::                   |              |                |
| en0       | 10.1.114.72               | 255.255.254.0                           | 10.1.115.255 |                |
+-----------+---------------------------+-----------------------------------------+--------------+----------------+
```

- dns_resolvers:  DNS 域名服务器地址

```
osquery> select * from dns_resolvers;
+----+------------+--------------+---------+---------+
| id | type       | address      | netmask | options |
+----+------------+--------------+---------+---------+
| 0  | nameserver | 10.0.5.55    | 32      | 1729    |
| 1  | nameserver | 10.199.84.13 | 32      | 1729    |
| 2  | nameserver | 10.0.5.56    | 32      | 1729    |
+----+------------+--------------+---------+---------+
```

### 显示更方便

用过mysql命令行的知道，有时候一个列的内容太长，按记录显示特别不方便看。osquery 的设计者想到了这个问题，比如：

```
osquery> .mode line

osquery> select * from processes limit 1;
          pid = 1
         name = launchd
         path = /sbin/launchd
      cmdline =
        state = R
          cwd =
         root =
          uid = 0
          gid = 0
         euid = 0
         egid = 0
         suid = 0
         sgid = 0
      on_disk = 1
   wired_size = -1
resident_size = -1
   total_size = -1
    user_time = -1
  system_time = -1
   start_time = -1
       parent = 0
       pgroup = 1
      threads = -1
         nice = 0

```

更丰富的显示方式，可以查看帮助：

```
osquery> .help
Welcome to the osquery shell. Please explore your OS!
You are connected to a transient 'in-memory' virtual database.

.all [TABLE]     Select all from a table
.help            Show this message
.mode MODE       Set output mode where MODE is one of:
                   csv      Comma-separated values
                   column   Left-aligned columns see .width
                   line     One value per line
                   list     Values delimited by .separator string
                   pretty   Pretty printed SQL results (default)

.schema [TABLE]  Show the CREATE statements
.tables [TABLE]  List names of tables
```

### 更多

关于osquery的更多表，可以查看：
https://osquery.io/docs/tables/

或者 ``osquery> .tables `` 命令帮助查看，而且还支持模糊查询，比如：

```
osquery> .tables list
  => listening_ports

osquery> .tables pro
  => process_envs
  => process_events
  => process_file_events
  => process_memory_map
  => process_open_files
  => process_open_sockets
  => processes

```

接着可以查看表的字段情况（表结构定义）：

```
.schema processes
CREATE TABLE processes(`pid` BIGINT PRIMARY KEY, `name` TEXT, `path` TEXT, `cmdline` TEXT, `state` TEXT, `cwd` TEXT, `root` TEXT, `uid` BIGINT, `gid` BIGINT, `euid` BIGINT, `egid` BIGINT, `suid` BIGINT, `sgid` BIGINT, `on_disk` INTEGER, `wired_size` BIGINT, `resident_size` BIGINT, `total_size` BIGINT, `user_time` BIGINT, `system_time` BIGINT, `start_time` BIGINT, `parent` BIGINT, `pgroup` BIGINT, `threads` INTEGER, `nice` INTEGER, `phys_footprint` BIGINT HIDDEN) WITHOUT ROWID;
```

常见的表还有：
- users:  操作系统里有多少用户账号。
- logged_in_users: 当前登陆了多少用户。
- processes: 进程
- process_open_files: 进程打开的文件句柄。
- process_open_sockets: 进程打开的socket。
- listening_ports： 系统启动的侦听端口（含pid）。



比如：打开文件句柄数最多的进程

``` bash
osquery> select pid, count(fd) as count from process_open_files group by pid order by count desc;
+------+-------+
| pid  | count |
+------+-------+
| 629  | 168   |
| 283  | 142   |
| 287  | 58    |
```

监听端口：

```
osquery> select * from listening_ports;
+------+-------+----------+--------+-----------+
| pid  | port  | protocol | family | address   |
+------+-------+----------+--------+-----------+
| 235  | 0     | 17       | 2      | 0.0.0.0   |
| 235  | 0     | 0        | 0      |           |
| 243  | 61116 | 17       | 2      | 0.0.0.0   |
| 267  | 49153 | 6        | 2      | 127.0.0.1 |
| 477  | 0     | 6        | 2      | 0.0.0.0   |
| 1129 | 4300  | 6        | 2      | 127.0.0.1 |
| 1129 | 4301  | 6        | 2      | 127.0.0.1 |
+------+-------+----------+--------+-----------+
```

- mounts: 挂载的磁盘

```
osquery> select * from mounts;
+---------------+---------------+-------------+--------+-------------+----------+-------------+------------------+----------+-------------+----------+
| device        | device_alias  | path        | type   | blocks_size | blocks   | blocks_free | blocks_available | inodes   | inodes_free | flags    |
+---------------+---------------+-------------+--------+-------------+----------+-------------+------------------+----------+-------------+----------+
| /dev/disk1    | /dev/disk1    | /           | hfs    | 4096        | 60983424 | 29853926    | 29789926         | 60983422 | 29789926    | 75550720 |
| devfs         | devfs         | /dev        | devfs  | 512         | 664      | 0           | 0                | 1149     | 0           | 68161536 |
| map -hosts    | map -hosts    | /net        | autofs | 1024        | 0        | 0           | 0                | 0        | 0           | 72351752 |
| map auto_home | map auto_home | /home       | autofs | 1024        | 0        | 0           | 0                | 0        | 0           | 72351744 |
| /dev/disk2s2  | /dev/disk2s2  | /Volumes/万信 | hfs    | 4096        | 25000    | 5317        | 5317             | 24998    | 5317        | 69243929 |
+---------------+---------------+-------------+--------+-------------+----------+-------------+------------------+----------+-------------+----------+
```

----
# 参考资料

- glances 使用
https://www.ibm.com/developerworks/cn/linux/1304_caoyq_glances/

- 以web形式使用glances
https://my.oschina.net/hochikong/blog/379972

- osquery 安装
https://osquery.readthedocs.io/en/stable/

- osquery tables 列表
https://osquery.io/docs/tables/

- osquery 各平台安装方式
https://osquery.io/downloads/
