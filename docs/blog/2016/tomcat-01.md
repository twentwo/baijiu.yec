---
tags:
- Tomcat
---

# Tomcat（一）-重定向Web应用程序目录

:material-clock-time-three-outline: 2016-07-02 20:46:00

**说明**：

- 使用Tomcat二进制发行版，版本6.0.45，且已安装
- 安装目录为`/usr/server/apache-tomcat-6.0.45`($CATALINA_HOME)
- 以tomcat用户启动，tomcat登陆shell设为/sbin/nologin
- 系统环境Linux Mint 16

---

在安装一份Tomcat发行版的情况下，怎么同时运行两个以上的不同配置的Tomcat实例？一般在使用tomcat时，服务器会从conf及webapps目录中读取配置文件，并将文件写入logs、temp、work目录，当然，一些jar文件和class文件需要从服务器公共目录树中予以加载。为了让多个实例都能运行，每个tomcat实例必须都有自己的目录集，且他们不能共享两个不同的已配置的Tomcat JVM实例。做法就是将`CATALINA_BASE`的环境变量设置成与新存储JVM实例文件（自己生成的网站文件目录集）相同的路径，这样启动时就会使用`CATALINA_BASE`中定义的文件运行。如此一来，我们就做到了网站文件和Tomcat发行版文件分开，即重定向了Web应用程序的目录。

**1. Tomcat目录结构**

Tomcat根目录在叫`$CATALINA_HOME`, 里面有7个目录

1. `$CATALINA_HOME/bin`:        存放各种平台下启动和关闭tomcat的脚本文件
2. `$CATALINA_HOME/lib`:         在lib目录下的lib目录,存放tomcat服务器和所有web应用都能访问的jar
3. `$CATALINA_HOME/work`:      tomcat把各种由jsp生成的servlet文件放在这个目录下
4. `$CATALINA_HOME/temp`:      临时活页夹，tomcat运行时候存放临时文件用的
5. `$CATALINA_HOME/logs`:       存放tomcat的日志文件
6. `$CATALINA_HOME/conf`:       tomcat的各种配置文件,最重要的是server.xml
7. `$CATALINA_HOME/webapps`:     tomcat的主要web发布目录，默认情况下把Web应用文件放于此目录


**2. 重定向**

创建放置实例文件的目录
```
twen@twen-thinkpad $ cd /opt
twen@twen-thinkpad /opt $ sudo mkdir tomcat-instance
twen@twen-thinkpad /opt $ cd tomcat-instance
```
为新实例创建目录（将网站存入其中后最好对其命名）
```
twen@twen-thinkpad /opt/tomcat-instance $ sudo mkdir twen.cn
twen@twen-thinkpad /opt/tomcat-instance $ cd twen.cn
```
将Tomcat发行版的config目录复制到新目录中，然后创建Tomcat其他的实例全部目录
```
twen@twen-thinkpad /opt/tomcat-instance/twen.cn $ sudo cp -a $CATALINA_HOME/conf ./
twen@twen-thinkpad /opt/tomcat-instance/twen.cn $ sudo mkdir logs temp webapps work
```

最后，将此实例的Web应用程序内容放入webapps子目录中。编辑conf/server.xml文件为指定该实例文件，并将文件修改为只包含运行该实例所需的参数。
将停止端口换成不同的端口号
```xml
<Server port="8007" shutdown="SHUTDOWN">
```
连接器的端口号
```xml
<Connector port="8081" protocol="HTTP/1.1" 
           connectionTimeout="20000" 
           redirectPort="8443" />
```

删除其中所有的Context元素


**3. 启动实例**

为了方便启动，写一个简单启动脚本
```
twen@twen-thinkpad /opt/tomcat-instance/twen.cn $ sudo mkdir bin
twen@twen-thinkpad /opt/tomcat-instance/twen.cn $ cd bin
twen@twen-thinkpad /opt/tomcat-instance/twen.cn/bin $ sudo touch start
```

编写start文件，加入下面内容
```
#!/bin/sh
export CATALINA_BASE=/opt/tomcat-instance/twen.cn
export CATALINA_HOME=/usr/server/apache-tomcat-6.0.45
$CATALINA_HOME/bin/catalina.sh start
```

将`$CATALINA_HOME`子目录webapps下的ROOT拷贝到此刻的`$CATALINA_BASE`下
```
twen@twen-thinkpad /opt/tomcat-instance/twen.cn/bin $ sudo cp -a $CATALINA_HOME/webapps/ROOT /opt/tomcat-instance/twen.cn/webapps
```

改变文件拥有者为tomcat， 改变模式使之有执行权限
```
twen@twen-thinkpad /opt/tomcat-instance/twen.cn/bin $ sudo chown tomcat stop
twen@twen-thinkpad /opt/tomcat-instance/twen.cn/bin $ sudo chmod u+x stop
```

用户tomcat启动
```
twen@twen-thinkpad /opt/tomcat-instance/twen.cn/bin $ sudo -u tomcat ./start
```

log如下
```
Using CATALINA_BASE:   /opt/tomcat-instance/twen.cn
Using CATALINA_HOME:   /usr/server/apache-tomcat-6.0.45
Using CATALINA_TMPDIR: /opt/tomcat-instance/twen.cn/temp
Using JRE_HOME:        /usr
Using CLASSPATH:       /usr/server/apache-tomcat-6.0.45/bin/bootstrap.jar
```

访问`127.0.0.1:8081`，显示TOM猫出现了成功。

现在同时启动`$CATALINA_HOME`中的实例
```
twen@twen-thinkpad /opt/tomcat-instance/twen.cn $ sudo -u tomcat $CATALINA_HOME/bin/catalina.sh start
```

同样log如下
```
Using CATALINA_BASE:   /usr/server/apache-tomcat-6.0.45
Using CATALINA_HOME:   /usr/server/apache-tomcat-6.0.45
Using CATALINA_TMPDIR: /usr/server/apache-tomcat-6.0.45/temp
Using JRE_HOME:        /usr
Using CLASSPATH:       /usr/server/apache-tomcat-6.0.45/bin/bootstrap.jar
```

访问`127.0.0.1:8080`，显示TOM猫出现了，也成功。

再看一眼实例
```
twen@twen-thinkpad /opt/tomcat-instance/twen.cn $ ps -ef | grep tomcat
```

显示如下
```
tomcat    3190     1  0 17:33 pts/0    00:00:25 /usr/bin/java -Djava.util.logging.config.file=/usr/server/apache-tomcat-6.0.45/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.endorsed.dirs=/usr/server/apache-tomcat-6.0.45/endorsed -classpath /usr/server/apache-tomcat-6.0.45/bin/bootstrap.jar -Dcatalina.base=/usr/server/apache-tomcat-6.0.45 -Dcatalina.home=/usr/server/apache-tomcat-6.0.45 -Djava.io.tmpdir=/usr/server/apache-tomcat-6.0.45/temp org.apache.catalina.startup.Bootstrap start
tomcat    3226     1  0 17:34 pts/0    00:00:24 /usr/bin/java -Djava.util.logging.config.file=/opt/tomcat-instance/twen.cn/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.endorsed.dirs=/usr/server/apache-tomcat-6.0.45/endorsed -classpath /usr/server/apache-tomcat-6.0.45/bin/bootstrap.jar -Dcatalina.base=/opt/tomcat-instance/twen.cn -Dcatalina.home=/usr/server/apache-tomcat-6.0.45 -Djava.io.tmpdir=/opt/tomcat-instance/twen.cn/temp org.apache.catalina.startup.Bootstrap start
```

可以看出不同
  


  

