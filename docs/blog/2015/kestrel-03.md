---
tags:
- Kestrel
---

# Kestrel（三）

:material-clock-time-three-outline: 2015-09-15 13:38

## Test
---
1. 新建一个spring配置文件`applicationContext.xml`,导入之前的

```
 - kestrel.properties
 - kestrel.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
	xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	<context:property-placeholder location="classpath:kestrel.properties" />
	<import resource="kestrel.xml" />
</beans>
```

2. 新建JUnit测试

```java
import static junit.framework.Assert.*;
import net.rubyeye.xmemcached.MemcachedClient;
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.util.StopWatch;
/**
 * Kestrel 测试
 *
 */
public class KestrelTest {

	private final static String QUEUE_NAME = "KQ";

	private ApplicationContext app;
	private MemcachedClient memcachedClient;

	private static final int COUNT = 5000;

	/**
	 * @throws Exception
	 */
	@Before
	public void before() throws Exception {
		app = new ClassPathXmlApplicationContext("spring/applicationContext.xml");
		memcachedClient = (MemcachedClient) app.getBean("memcachedClient");
	}

	/**
	 * 发包测试
	 */
	@Test
	public void send() {
		StopWatch stopWatch = new StopWatch("send");
		stopWatch.start("set");
		memcachedClient.setOpTimeout(5000L);
		for (int i = 0; i < COUNT; i++) {
			try {
				memcachedClient.set(QUEUE_NAME, 0, "http://"+i+"_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo");
			} catch (Exception e) {
				fail(e.getMessage());
				e.printStackTrace();
			}
		}
		stopWatch.stop();
		System.out.println(stopWatch.prettyPrint());

	}

	/**
	 * 收包测试
	 */
	@Test
	public void receive() {
		String value;
		StopWatch stopWatch = new StopWatch("receive");
		stopWatch.start("get");
		try {
			while (true) {
				memcachedClient.setOpTimeout(5000L);
				value = (String) memcachedClient.get(QUEUE_NAME);
				if (value == null) {
					break;
				}
				System.out.println(value);
			}
		} catch (Exception e) {
			fail(e.getMessage());
			e.printStackTrace();
		}
		stopWatch.stop();
		System.out.println(stopWatch.prettyPrint());
	}
}
```

3. 测试环境

```
 - kestrel server服务环境，本地虚拟机，centOS6.6，只分配了650M内存，1核CPU
 - jdk7，单线程，单client
 - 消息个数5000，消息长度约256
```

4. 发送测试结果

```
StopWatch 'send': running time (millis) = 2248
-----------------------------------------
ms     %     Task name
-----------------------------------------
02248  100%  set
```

5. 接收测试结果

```
StopWatch 'receive': running time (millis) = 2896
-----------------------------------------
ms     %     Task name
-----------------------------------------
02896  100%  get
```

