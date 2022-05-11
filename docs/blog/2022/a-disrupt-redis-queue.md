---
tags:
- Disrupt
- Redis
- Queue
---

# Disrupt+Redis队列

:material-clock-time-three-outline: 2022-05-02 16:07:27

## 架构图
![git-model@2x.png][1]

## 源码
移步至 [https://github.com/twentwo/fresno][2]

## 用法

spring-boot application support

### Maven

引入Maven包

```xml
<dependency>
  <groupId>io.yec</groupId>
  <artifactId>fresno-spring-boot-starter</artifactId>
  <version>${fresno.version}</version>
</dependency>
```

### EventHandler

定义 EventHandler

```java
@EventHandler
public class OrderEventListener implements EventListener<Order> {

    @Resource
    private FooService fooService;

    @Override
    public void onEvent(EventQueue<Order> eventQueue, Event<Order> event) {
        try {
            fooService.getOrder(event.getObj());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

### EventQueueReference

引用 EventQueue

```java
@Service
public class FooServiceImpl implements FooService {

    @EventQueueReference(belongTo = OrderEventListener.class)
    private EventQueue<Order> orderEventQueue;

    @Override
    public void enQueueOrder(Order order) {
        log.info("enQueueOrder :: {}", order);
        orderEventQueue.enqueueToBack(order);
    }
}
```


[1]: a-disrupt-redis-queue/Disrupt+Redis%20Queue.png
[2]: https://github.com/twentwo/fresno