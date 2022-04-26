# 用Gogs替代Gitlab（二）

:material-clock-time-three-outline: 2016-02-09 23:49:00

## nginx反向代理gogs
---

**1.先导说明**
- 已安装nginx
- nginx安装在本机

**2.配置nginx**
在`nginx.conf`文件中，将下面的`server`部分增加至`http`分区内并重载配置
```nginx
server {
        listen       80;
        server_name  localhost;

        access_log  logs/gogs.access.log;

        location / {
            proxy_pass http://localhost:3000/;
            keepalive_timeout  120; 
            expires 5m; 
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header connection Keep-Alive;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

    }
```

**3.至此完成反向代理**