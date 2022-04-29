---
tags:
- HTTPSQS
---

# HTTPSQS（一）

:material-clock-time-three-outline: 2015-09-15 13:40

## 简单介绍
---
HTTPSQS（HTTP Simple Queue Service）是一款基于 HTTP GET/POST 协议的轻量级开源简单消息队列服务，使用 Tokyo Cabinet 的 B+Tree Key/Value 数据库来做数据的持久化存储。

HTTPSQS 具有以下特征：

```
- 非常简单，基于 HTTP GET/POST 协议。PHP、Java、Perl、Shell、Python、Ruby等支持HTTP协议的编程语言均可调用。
- 非常快速，入队列、出队列速度超过10000次/秒。
- 高并发，支持上万的并发连接，C10K不成问题。
- 支持多队列。
- 单个队列支持的最大队列数量高达10亿条。
- 低内存消耗，海量数据存储，存储几十GB的数据只需不到100MB的物理内存缓冲区。
- 可以在不停止服务的情况下便捷地修改单个队列的最大队列数量。
- 可以实时查看队列状态（入队列位置、出队列位置、未读队列数量、最大队列数量）。
- 可以查看指定队列ID（队列点）的内容，包括未出、已出的队列内容。
- 查看队列内容时，支持多字符集编码。
- 源代码不超过800行，适合二次开发。
```

`推荐`: [基于HTTP协议的轻量级开源简单队列服务：HTTPSQS\[原创\]][1]  -------------HTTPSQS作者张宴

## HTTPSQS安装
---
1. 安装依赖（本机是centos6.6-DVD1 miniDeskTop版本，已安装yum工具）
- libevent

```bash
cd /usr/servers
wget http://httpsqs.googlecode.com/files/libevent-2.0.12-stable.tar.gz
tar -zxvf libevent-2.0.12-stable.tar.gz
cd libevent-2.0.12-stable/
./configure --prefix=/usr/local/libevent-2.0.12-stable/
```
如果过程中出现提示缺少zlib.h或者bzlib.h，利用yum安装zlib-devel或者bzip2-devel

```bash
yum install zlib-devel
yum install bzip2-devel
```

```bash
make
make install
cd ../
```

- tokyocabinet

```bash
wget http://httpsqs.googlecode.com/files/tokyocabinet-1.4.47.tar.gz
tar zxvf tokyocabinet-1.4.47.tar.gz
cd tokyocabinet-1.4.47/
./configure --prefix=/usr/local/tokyocabinet-1.4.47/
#注：在32位Linux操作系统上编译Tokyo cabinet，请使用./configure --enable-off64代替./configure，可以使数据库文件突破2GB的限制。
#./configure --enable-off64 --prefix=/usr/local/tokyocabinet-1.4.47/
make
make install
cd ../
```
至此，依赖安装完成。

2. 安装httpsqs-1.7

```bash
wget http://httpsqs.googlecode.com/files/httpsqs-1.7.tar.gz
tar zxvf httpsqs-1.7.tar.gz
cd httpsqs-1.7/
make
------------------------------------------------------------------------
gcc -o httpsqs httpsqs.c prename.c -Wl,-rpath,/usr/local/libevent-2.0.12-stable/lib/:/usr/local/tokyocabinet-1.4.47/lib/ -L/usr/local/libevent-2.0.12-stable/lib/ -levent -L/usr/local/tokyocabinet-1.4.47/lib/ -ltokyocabinet -I/usr/local/libevent-2.0.12-stable/include/ -I/usr/local/tokyocabinet-1.4.47/include/ -lz -lbz2 -lrt -lpthread -lm -lc -O2 -g 

httpsqs build complete.
------------------------------------------------------------------------
make install
------------------------------------------------------------------------
install -m 4755 -o root httpsqs /usr/bin
------------------------------------------------------------------------
```

执行`httpsqs -h`，出现帮助文档

```bash
[root@localhost httpsqs-1.7]# httpsqs -h
------------------------------------------------------------------------
HTTP Simple Queue Service - httpsqs v1.7 (April 14, 2011)

Author: Zhang Yan (http://blog.s135.com), E-mail: net@s135.com
This is free software, and you are welcome to modify and redistribute it under the New BSD License

-l <ip_addr> interface to listen on, default is 0.0.0.0
-p <num> TCP port number to listen on (default: 1218)
-x <path> database directory (example: /opt/httpsqs/data)
-t <second> keep-alive timeout for an http request (default: 60)
-s <second> the interval to sync updated contents to the disk (default: 5)
-c <num> the maximum number of non-leaf nodes to be cached (default: 1024)
-m <size> database memory cache size in MB (default: 100)
-i <file> save PID in <file> (default: /tmp/httpsqs.pid)
-a <auth> the auth password to access httpsqs (example: mypass123)
-d run as a daemon
-h print this help and exit

Use command "killall httpsqs", "pkill httpsqs" and "kill `cat /tmp/httpsqs.pid`" to stop httpsqs.
Please note that don't use the command "pkill -9 httpsqs" and "kill -9 PID of httpsqs"!

Please visit "http://code.google.com/p/httpsqs" for more help information.

------------------------------------------------------------------------
```
至此，安装完成。

`注`：不要将依赖包`libevent`和`tokyocabinet`解压路径不要在`/usr/local/`下，否则`./configure`的时候会报冲突。

[1]: http://blog.zyan.cc/httpsqs/