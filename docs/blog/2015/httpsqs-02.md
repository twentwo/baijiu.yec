---
tags:
- HTTPSQS
---

# HTTPSQS（二）

:material-clock-time-three-outline: 2015-09-23 19:46

## Test
---
1. 新建工程，添加jar包`httpsq4j.jar`

2. 新建JUnit测试

```java
import com.daguu.lib.httpsqs4j.Httpsqs4j;
import com.daguu.lib.httpsqs4j.HttpsqsClient;
import com.daguu.lib.httpsqs4j.HttpsqsException;
import com.daguu.lib.httpsqs4j.HttpsqsStatus;
import org.junit.Before;
import org.junit.Test;
import org.springframework.util.StopWatch;
import static junit.framework.Assert.fail;

/**
 * Created by congye on 9/23/2015.
 */
public class HttpsqsTest {

    private final static String QUEUE_NAME = "KQ";
    private static final int COUNT = 5000;
    HttpsqsClient client;

    /**
     * @throws Exception
     */
    @Before
    public void before() throws Exception {
        Httpsqs4j.setConnectionInfo("192.168.56.101", 1218, "UTF-8");
        client = Httpsqs4j.createNewClient();
        HttpsqsStatus status = client.getStatus(QUEUE_NAME);
        System.out.println(status.version);
        System.out.println(status.queueName);
        System.out.println(status.maxNumber);
        System.out.println(status.getLap);
        System.out.println(status.getPosition);
        System.out.println(status.putLap);
        System.out.println(status.putPosition);
        System.out.println(status.unreadNumber);
    }

    /**
     *发包测试
     */
    @Test
    public void send() {
        StopWatch stopWatch = new StopWatch("send");
        stopWatch.start("set");
        for (int i = 0; i < COUNT; i++) {
            try {
                client.putString(QUEUE_NAME, "http://" + i + "_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo_twentwo");
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
        String value = "";
        StopWatch stopWatch = new StopWatch("receive");
        stopWatch.start("get");
        try {
            while (true) {
                value = client.getString(QUEUE_NAME);
                //System.out.println(value);
            }
        } catch (HttpsqsException e) {
            System.out.println(e.getMessage());
            //e.printStackTrace();
        } finally {
            stopWatch.stop();
            System.out.println(stopWatch.prettyPrint());
        }
    }

}
```

3. 测试环境（和之前Kestrel环境相同）

```
- kestrel server服务环境，本地虚拟机，centOS6.6，只分配了650M内存，1核CPU
- jdk7，单线程，单client
- 消息个数5000，消息长度约256
```

4. 发送测试结果

```
StopWatch 'send': running time (millis) = 3730
-----------------------------------------
ms     %     Task name
-----------------------------------------
03730  100%  set
```

5. 接收测试结果

```
There's no data in queue [KQ].
StopWatch 'receive': running time (millis) = 3644
-----------------------------------------
ms     %     Task name
-----------------------------------------
03644  100%  get
```