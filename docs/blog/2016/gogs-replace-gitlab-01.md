---
tags:
- Gogs
- Gitlab
---

# 用Gogs替代Gitlab（一）

:material-clock-time-three-outline: 2016-02-09 23:49:00

## 安装说明
- 已安装git（yum -y install git）
- 已安装mysql（[yum install mysql][1]）
- 服务安装路径：/usr/servers
- gogs版本号： gogs_v0.8.25_linux_amd64.tar.gz
- 以用户git（非root）启动

---

**1.下载并解压**
将gogs_v0.8.25_linux_amd64.tar.gz拷贝到/usr/servers,解压
```bash
tar -zxvf gogs_v0.8.25_linux_amd64.tar.gz
```
解压文件夹为gogs

**2.新建系统用户**
```bash
adduser --system --shell /bin/bash --comment 'GitLab' --create-home --home-dir /home/git/ git
```
为了包含`/usr/local/bin`到git用户的`$PATH`，方法是编辑超级用户文件。

以管理员身份运行：
```bash
visudo
```
然后搜索：
```
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
```
将其改成：
```
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
```

**3.修改目录权限**
修改`/usr/servers/gogs`目录下所有文件、所有子文件、文件夹的所有用户
```bash
chown -R git /usr/servers/gogs
```

**4.启动**
以用户git启动
```
[git@vdlmobile16 gogs]$ ./gogs web
2016/02/22 15:16:02 [W] Custom config '/usr/servers/gogs/custom/conf/app.ini' not found, ignore this if you're running first time
2016/02/22 15:16:02 [T] Custom path: /usr/servers/gogs/custom
2016/02/22 15:16:02 [T] Log path: /usr/servers/gogs/log
2016/02/22 15:16:02 [I] codeRepo 0.8.25.0129
2016/02/22 15:16:02 [I] Log Mode: File(Info)
2016/02/22 15:16:02 [I] Cache Service Enabled
2016/02/22 15:16:02 [I] Session Service Enabled
2016/02/22 15:16:02 [I] Git Version: 1.7.1
2016/02/22 15:16:03 [T] Doing: CheckRepoStats
2016/02/22 15:16:03 [I] SQLite3 Supported
2016/02/22 15:16:03 [I] Run Mode: Production
[mysql] 2016/02/22 15:16:03 statement.go:27: invalid connection
2016/02/22 15:16:03 [I] Listen: http://0.0.0.0:3000
```

**5.访问浏览器**
初始化配置并安装

**6.配置SSH**
安装完成并注册，增加公钥，运行`ssh -T git@ip_addr`时报错Permission denied (publickey,gssapi-keyex,gssapi-with-mic)，解决如下：
1)修改sshd_config文件
```bash
vi /etc/ssh/sshd_config
```
开启以下内容
```
HostKey /etc/ssh/ssh_host_rsa_key
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
```
重启
```bash
/etc/init.d/sshd restart
```
2)权限设置
```bash
chown -R git /home/git //如果以git启动，owner默认为git
chmod 700 /home/git
chmod 700 /home/git/.ssh
chmod 644 /home/git/.ssh/authorized_keys  //公钥文件的所有权限
```

至此应该完成，如果在客户端执行`ssh -T git@ip_addr`依然报错
```
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).  
```
继续下文。

关闭SELinux解决问题：
1)暂时关闭（重启后恢复）：
``` 
setenforce 0  
```
2)永久关闭（需要重启）：
```bash
vi /etc/selinux/config  
SELINUX=disabled
```

最后执行`ssh -T git@ip_addr`，结果：
```
Hi there, You've successfully authenticated, but Gogs does not provide shell access.
If this is unexpected, please log in with password and setup Gogs under another user.
```

**7.至此成功**

[1]: /baijiu.yec/blog/2016/yum-install-mysql/