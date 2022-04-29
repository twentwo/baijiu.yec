---
tags:
- XMemcached
- Kestrel
- Memcached
---

# Kestrel（二）

:material-clock-time-three-outline: 2015-09-08 16:03

## Kestrel Java Client
---
XMemcached,看名字好像它和Memcached有关系。对了，Memcached是一个开源的，C写的分布式key-value缓存，XMemcached是它的一个访问客户端。

XMemcached为Memcached而生，它支持如下特性：

- 支持所有的文本协议和二进制协议，支持连接**Kestrel**和TokyoTyrant等Memcached协议兼容的系统并作特殊处理。
- 支持动态添加和删除Memcached节点。
- 支持客户端统计。
- 支持JMX监控和统计，可以通过JMX增删节点。
- 高性能。
- 支持节点的权重设置。
- 支持nio的连接池，在高负载环境下提高吞吐量。

正因为Kestrel采用的协议是Memcached的文本协议，所以XMemcached支持了Kestrel！

`推荐`：
> [初识Kestrel][1]
> [Xmemcached的FAQ和性能调整建议][2]
> ------------- killme2008（XMemcached作者）

## XMemcached与Spring集成
---
目录结构

```
Kestrel
  |-main 
      |-resources
          |-applicationContext.xml
          |-kestrel.properties
          |-kestrel.xml          
  |-test
      |-java 
          |-包名
             |-KestrelTest.java
```

1. pom文件中加入XMemcached依赖

```xml
  <dependency>
  	<groupId>com.googlecode.xmemcached</groupId>
  	<artifactId>xmemcached</artifactId>
  	<version>1.3.7</version>
  	<type>jar</type>
  	<scope>compile</scope>
  </dependency>
```

2. kestrel.propertis

```properties
#\u8fde\u63a5\u6c60\u5927\u5c0f\u5373\u5ba2\u6237\u7aef\u4e2a\u6570
kestrel.connectionPoolSize=50
kestrel.failureMode=true
#server1
#kestrel.server1.host=192.168.56.101
kestrel.server1.host=172.16.128.248
kestrel.server1.port=22133
kestrel.server1.weight=1
#server2
#kestrel.server2.host=10.11.155.41
#kestrel.server2.port=22133
#kestrel.server2.weight=1				
```
多个server配置只需打开server2，以此类推。

2. Spring配置

- kestrel.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
	xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	<!-- http://code.google.com/p/xmemcached/wiki/Spring_Integration -->
	<bean
		id="memcachedClientBuilder"
		class="net.rubyeye.xmemcached.XMemcachedClientBuilder"
		p:connectionPoolSize="${kestrel.connectionPoolSize}"
		p:failureMode="${kestrel.failureMode}">
		<!-- XMemcachedClientBuilder have two arguments.First is server list,and 
			second is weights array. -->
		<constructor-arg>
			<list>
				<bean class="java.net.InetSocketAddress">
					<constructor-arg>
						<value>${kestrel.server1.host}</value>
					</constructor-arg>
					<constructor-arg>
						<value>${kestrel.server1.port}</value>
					</constructor-arg>
				</bean>
				<!-- <bean class="java.net.InetSocketAddress"> -->
				<!-- <constructor-arg> -->
				<!-- <value>${kestrel.server2.host}</value> -->
				<!-- </constructor-arg> -->
				<!-- <constructor-arg> -->
				<!-- <value>${kestrel.server2.port}</value> -->
				<!-- </constructor-arg> -->
				<!-- </bean> -->
			</list>
		</constructor-arg>
		<constructor-arg>
			<list>
				<value>${kestrel.server1.weight}</value>
				<!-- <value>${kestrel.server2.weight}</value> -->
			</list>
		</constructor-arg>
		<property name="commandFactory">
			<bean class="net.rubyeye.xmemcached.command.KestrelCommandFactory" />
		</property>
		<property name="sessionLocator">
			<bean class="net.rubyeye.xmemcached.impl.KetamaMemcachedSessionLocator" />
		</property>
		<property name="transcoder">
			<bean class="net.rubyeye.xmemcached.transcoders.SerializingTranscoder" />
		</property>
	</bean>
	<!-- Use factory bean to build memcached client -->
	<bean
		id="memcachedClient"
		factory-bean="memcachedClientBuilder"
		factory-method="build"
		destroy-method="shutdown">
		<!-- 如果收发双方使用Client不同，开启该选项 -->
		<!-- <property -->
		<!-- name="primitiveAsString" -->
		<!-- value="true" /> -->
	</bean>
</beans>
```

`注`：这里要使用KestrelCommandFactory

```xml
<property name="commandFactory">  
    <bean class="net.rubyeye.xmemcached.command.KestrelCommandFactory" />  
</property>  
```

KestrelCommandFactory会在数据头加一个4字节的flag，支持存储任意可序列化类型。如果收发双方Client不同，为确保移植性，需要将数据转为String格式，舍弃这个flag。方式是开启如下选项，缺点是开启之后，数据就不能支持序列化操作，仅支持String类型。

```xml
<bean  
    id="memcachedClient"  
    factory-bean="memcachedClientBuilder"  
    factory-method="build"  
    destroy-method="shutdown">  
    <!-- 如果收发双方使用Client不同，开启该选项 -->  
    <property  
        name="primitiveAsString"  
        value="true" />  
</bean>  
```






[1]: http://www.blogjava.net/killme2008/archive/2009/09/15/295119.html
[2]: http://www.blogjava.net/killme2008/archive/2010/07/08/325564.html