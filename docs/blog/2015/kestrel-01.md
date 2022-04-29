---
tags:
- Kestrel
---

# Kestrel（一）

:material-clock-time-three-outline: 2015-09-07 14:22

## 简单介绍

Kestrel是twitter开源的基于scale语言的开源消息中间件。它有如下特性：

```
 - 快，运行在JVM上，利用java性能优势。
 - 小巧，大约2500行scale代码。
 - 持久，不但快速存储在内存，还记录journal到硬盘，防止数据丢失。
 - 可靠，支持可靠获取。
```

Kestrel采用的协议是memcached的文本协议，但是并不完全支持所有memcached协议，也不是完全兼容现有协议。

可支持的标准协议：

``` 
 - SET          存
 - GET          取
 - FLUSH_ALL    清理
 - STATS        状态
```

扩展协议：

``` 
 - SHUTDOWN        关闭kestrel server，如果执行该操作，需强制重启Kestrel
 - RELOAD          动态重新加载配置文件 
 - DUMP_CONFIG     dump配置文件 
 - FLUSH queueName    flush某个队列
```

`推荐`：* [Kernel官方][1] * [征服 Kestrel][2]

## Kestrel安装

1. 安装依赖（本机是centos6.6-DVD1 miniDeskTop版本，已安装yum工具）
- sbt

```
curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
yum install sbt
```

- daemon

```
wget http://libslack.org/daemon/download/daemon-0.6.4.tar.gz 
tar -xzvf daemon-0.6.4.tar.gz 
./configure
make & make install
```

2. 安装Kestrel

```
cd /usr/servers
wget http://robey.github.com/kestrel/download/kestrel-2.4.1.zip
unzip kestrel-2.4.1.zip
```

3. 启动

进入`${kestrel_path}/kestrel/script`

```
cd /usr/servers/kestrel-2.4.1/script
```

修改权限

```
chmod -777 devel.sh
```

后台启动服务

```
nohup ./devel/sh &
```

查看`nohup.out`

```
Starting kestrel in development mode...
```

说明Kestrel已经以开发模式启动。

## Kestrel脚本&配置说明

我们只需关心以下文件：

```
Kestrel-2.4.1
  |-kestrel_2.9.2-2.4.1.jar 
  |-config 
      |-development.scale
      |-production.scale
  |-libs 
  |-scripts 
      |-devel.sh 
      |-kestrel.sh 
```

1. 适用于开发环境：

`script/devel.sh` 用于验证服务配置是否可用
`config/development.scale` 配合devel.sh进行操作的配置文件

- devel.sh

```
#!/bin/bash
echo "Starting kestrel in development mode..."

# find jar no matter what the root dir name
SCRIPT_DIR=$(cd `dirname "$0"`; pwd)
ROOT_DIR=`dirname "$SCRIPT_DIR"`

java -server -Xmx800m -Dstage=development -jar "$ROOT_DIR"/kestrel_2.9.2-2.4.1.jar
```

- development.scale

```
import com.twitter.conversions.storage._
import com.twitter.conversions.time._
import com.twitter.logging.config._
import com.twitter.ostrich.admin.config._
import net.lag.kestrel.config._

new KestrelConfig {
  listenAddress = "0.0.0.0"
  memcacheListenPort = 22133
  textListenPort = 2222
  thriftListenPort = 2229

  queuePath = "/var/spool/kestrel"

  clientTimeout = 30.seconds

  expirationTimerFrequency = 1.second

  maxOpenTransactions = 100

  // default queue settings:
  default.defaultJournalSize = 16.megabytes
  default.maxMemorySize = 128.megabytes
  default.maxJournalSize = 1.gigabyte

  admin.httpPort = 2223

  admin.statsNodes = new StatsConfig {
    reporters = new TimeSeriesCollectorConfig
  }

  queues = new QueueBuilder {
    // keep items for no longer than a half hour, and don't accept any more if
    // the queue reaches 1.5M items.
    name = "weather_updates"
    maxAge = 1800.seconds
    maxItems = 1500000
  } :: new QueueBuilder {
    // don't keep a journal file for this queue. when kestrel exits, any
    // remaining contents will be lost.
    name = "transient_events"
    keepJournal = false
  } :: new QueueBuilder {
    name = "jobs_pending"
    expireToQueue = "jobs_ready"
    maxAge = 30.seconds
  } :: new QueueBuilder {
    name = "jobs_ready"
    syncJournal = 0.seconds
  } :: new QueueBuilder {
    name = "spam"
  } :: new QueueBuilder {
    name = "spam0"
  } :: new QueueBuilder {
    name = "hello"
    fanoutOnly = true
  } :: new QueueBuilder {
    name = "small"
    maxSize = 128.megabytes
    maxMemorySize = 16.megabytes
    maxJournalSize = 128.megabytes
    discardOldWhenFull = true
  } :: new QueueBuilder {
    name = "slow"
    syncJournal = 10.milliseconds
  }

  aliases = new AliasBuilder {
    name = "wx_updates"
    destinationQueues = List("weather_updates")
  } :: new AliasBuilder {
    name = "spam_all"
    destinationQueues = List("spam", "spam0")
  }

  loggers = new LoggerConfig {
    level = Level.INFO
    handlers = new FileHandlerConfig {
      filename = "/var/log/kestrel/kestrel.log"
      roll = Policy.Never
    }
  }
}
```

自己可以根据需要修改日志等配置项。


2. 适用于生产环境：

`scripts/kestrel.sh` 核心执行文件
`config/production.scale` 核心配置文件

- kestrel.sh

```
#!/bin/bash
#
# kestrel init.d script.
#
# All java services require the same directory structure:
#   /usr/local/$APP_NAME
#   /var/log/$APP_NAME
#   /var/run/$APP_NAME

APP_NAME="kestrel"
ADMIN_PORT="2223"
VERSION="2.4.1"
SCALA_VERSION="2.9.2"
APP_HOME="/usr/local/$APP_NAME/current"
INITIAL_SLEEP=15

JAR_NAME="${APP_NAME}_${SCALA_VERSION}-${VERSION}.jar"
STAGE="production"
FD_LIMIT="262144"

HEAP_OPTS="-Xmx4096m -Xms4096m -XX:NewSize=768m"
GC_OPTS="-XX:+UseConcMarkSweepGC -XX:+UseParNewGC"
GC_TRACE="-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC"
GC_LOG="-Xloggc:/var/log/$APP_NAME/gc.log"
DEBUG_OPTS="-XX:ErrorFile=/var/log/$APP_NAME/java_error%p.log"

# allow a separate file to override settings.
test -f /etc/sysconfig/kestrel && . /etc/sysconfig/kestrel

JAVA_OPTS="-server -Dstage=$STAGE $GC_OPTS $GC_TRACE $GC_LOG $HEAP_OPTS $DEBUG_OPTS"

pidfile="/var/run/$APP_NAME/$APP_NAME.pid"
# This second pidfile exists for legacy purposes, from the days when kestrel
# was started by daemon(1)
daemon_pidfile="/var/run/$APP_NAME/$APP_NAME-daemon.pid"


running() {
  kill -0 `cat $pidfile`
}

find_java() {
  if [ ! -z "$JAVA_HOME" ]; then
    return
  fi
  for dir in /opt/jdk /System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK/Home /usr/java/default; do
    if [ -x $dir/bin/java ]; then
      JAVA_HOME=$dir
      break
    fi
  done
}

find_java


case "$1" in
  start)
    echo -n "Starting $APP_NAME... "

    if [ ! -r $APP_HOME/$JAR_NAME ]; then
      echo "FAIL"
      echo "*** $APP_NAME jar missing: $APP_HOME/$JAR_NAME - not starting"
      exit 1
    fi
    if [ ! -x $JAVA_HOME/bin/java ]; then
      echo "FAIL"
      echo "*** $JAVA_HOME/bin/java doesn't exist -- check JAVA_HOME?"
      exit 1
    fi
    if running; then
      echo "already running."
      exit 0
    fi

    TIMESTAMP=$(date +%Y%m%d%H%M%S);
    # Move the existing gc log to a timestamped file in case we want to examine it.
    # We must do this here because we have no option to append this via the JVM's
    # command line args.
    if [ -f /var/log/$APP_NAME/gc.log ]; then
      mv /var/log/$APP_NAME/gc.log /var/log/$APP_NAME/gc_$TIMESTAMP.log;
    fi

    ulimit -n $FD_LIMIT || echo -n " (no ulimit)"
    ulimit -c unlimited || echo -n " (no coredump)"

    sh -c "echo "'$$'" > $pidfile; echo "'$$'" > $daemon_pidfile; exec ${JAVA_HOME}/bin/java ${JAVA_OPTS} -jar ${APP_HOME}/${JAR_NAME} >> /var/log/$APP_NAME/stdout 2>> /var/log/$APP_NAME/error" &
    disown %-
    sleep $INITIAL_SLEEP

    tries=0
    while ! running; do
      tries=$((tries + 1))
      if [ $tries -ge 5 ]; then
        echo "FAIL"
        exit 1
      fi
      sleep 1
    done
    echo "done."
  ;;

  stop)
    echo -n "Stopping $APP_NAME... "
    if ! running; then
      echo "wasn't running."
      exit 0
    fi

    curl -m 5 -s http://localhost:${ADMIN_PORT}/shutdown.txt > /dev/null
    tries=0
    while running; do
      tries=$((tries + 1))
      if [ $tries -ge 15 ]; then
        echo "FAILED SOFT SHUTDOWN, TRYING HARDER"
        if [ -f $daemon_pidfile ]; then
          kill $(cat $daemon_pidfile)
        else
          echo "CAN'T FIND PID, TRY KILL MANUALLY"
          exit 1
        fi
        hardtries=0
        while running; do
          hardtries=$((hardtries + 1))
          if [ $hardtries -ge 5 ]; then
            echo "FAILED HARD SHUTDOWN, TRY KILL -9 MANUALLY"
            kill -9 $(cat $daemon_pidfile)
          fi
          sleep 1
        done
      fi
      sleep 1
    done
    echo "done."
  ;;

  status)
    if running; then
      echo "$APP_NAME is running."
    else
      echo "$APP_NAME is NOT running."
    fi
  ;;

  restart)
    $0 stop
    sleep 2
    $0 start
  ;;

  *)
    echo "Usage: /etc/init.d/${APP_NAME}.sh {start|stop|restart|status}"
    exit 1
  ;;
esac

exit 0
```
自己可能要修改`APP_NAME`，`APP_HOME`，`JAVA_OPTS`等相关配置。

- production.scale

```
import com.twitter.conversions.storage._
import com.twitter.conversions.time._
import com.twitter.logging.config._
import com.twitter.ostrich.admin.config._
import net.lag.kestrel.config._

new KestrelConfig {
  listenAddress = "0.0.0.0"
  memcacheListenPort = 22133
  textListenPort = 2222
  thriftListenPort = 2229

  queuePath = "/var/spool/kestrel"

  clientTimeout = None

  expirationTimerFrequency = 1.second

  maxOpenTransactions = 100

  // default queue settings:
  default.defaultJournalSize = 16.megabytes
  default.maxMemorySize = 128.megabytes
  default.maxJournalSize = 1.gigabyte
  default.syncJournal = 100.milliseconds

  admin.httpPort = 2223

  admin.statsNodes = new StatsConfig {
    reporters = new TimeSeriesCollectorConfig
  }

  queues = new QueueBuilder {
    // keep items for no longer than a half hour, and don't accept any more if
    // the queue reaches 1.5M items.
    name = "weather_updates"
    maxAge = 1800.seconds
    maxItems = 1500000
  } :: new QueueBuilder {
    // don't keep a journal file for this queue. when kestrel exits, any
    // remaining contents will be lost.
    name = "transient_events"
    keepJournal = false
  }

  loggers = new LoggerConfig {
    level = Level.INFO
    handlers = new FileHandlerConfig {
      filename = "/var/log/kestrel/kestrel.log"
      roll = Policy.SigHup
    }
  }
}
```

自己可以根据需要修改日志等配置项。

[1]: http://robey.github.io/kestrel/readme.html
[2]: http://snowolf.iteye.com/blog/1604531