# Nginx On Centos6（一）

:material-clock-time-three-outline: 2016-02-09 23:46:00

## 安装说明

 - 服务安装路径：/usr/servers
 - 源码安装路径：/usr/servers/env
 - tar包及解压源码路径：/usr/local/src

---

## 安装必要环境

C的编译器（已安装的可以省略）
```bash
 yum install gcc
```

C++编译器（已安装的可以省略）
```bash
yum install gcc+c++
```

下载并`copy`以下包到源码安装路径
 - pcre-8.38
 - zlib-1.2.8
 - openssl-1.0.1r

源码安装pcre-8.38
```bash
cd /usr/local/src/pcre-8.38
./configure --prefix= /usr/servers/env/pcre   //指向安装路径
make && make install
```

源码安装zlib-1.2.8
```bash
cd /usr/local/src/zlib-1.2.8
./configure --prefix= /usr/servers/env/zlib   //指向安装路径
make && make install
```

源码安装openssl-1.0.1r
```bash
cd /usr/local/src/openssl-1.0.1r
./configure --prefix= /usr/servers/env/openssl --openssldir=/usr/servers/env/openssl/conf   //指向安装路径
make && make install
```

## 安装nginx
下载并copy`nginx-1.8.1`到源码安装路径
配置nginx
```bash
cd /usr/local/src/nginx-1.8.1
./configure
--prefix=/usr/servers/nginx
--sbin-path=/usr/servers/nginx/nginx
--conf-path=/usr/servers/nginx/nginx.conf
--pid-path=/usr/servers/nginx/nginx.pid
--with-http_ssl_module
--with-pcre=/usr/local/src/pcre-8.38    //指向源码
--with-zlib=/usr/local/src/zlib-1.2.8      //指向源码
--with-openssl=/usr/local/src/openssl-1.0.1r      //指向源码
```

配置结果
```bash
Configuration summary
 + using PCRE library: /usr/local/src/pcre-8.38
 + using OpenSSL library: /usr/local/src/openssl-1.0.1r
 + md5: using OpenSSL library
 + sha1: using OpenSSL library
 + using zlib library: /usr/local/src/zlib-1.2.8

 nginx path prefix: "/usr/servers/nginx"
 nginx binary file: "/usr/servers/nginx/nginx"
 nginx configuration prefix: "/usr/servers/nginx"
 nginx configuration file: "/usr/servers/nginx/nginx.conf"
 nginx pid file: "/usr/servers/nginx/nginx.pid"
 nginx error log file: "/usr/servers/nginx/logs/error.log"
 nginx http access log file: "/usr/servers/nginx/logs/access.log"
 nginx http client request body temporary files: "client_body_temp"
 nginx http proxy temporary files: "proxy_temp"
 nginx http fastcgi temporary files: "fastcgi_temp"
 nginx http uwsgi temporary files: "uwsgi_temp"
 nginx http scgi temporary files: "scgi_temp"
```

编译安装
```bash
make && make install
```

安装完成

## 测试安装结果
执行
```bash
cd /usr/servers/nginx
./nginx
```

浏览器访问`server_ip:80`，出现

![搜狗截图20160217140600.png][1]

即安装成功

`注：`如果出现不能访问很有可能是防火墙的原因:)


  [1]: nginx-on-centos6-01/4048615500.png