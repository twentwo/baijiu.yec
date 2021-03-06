---
tags:
- Strategy
- Router
---

# Wula: A Strategy Route Framework

:material-clock-time-three-outline: 2022-07-11 11:18:27

[https://github.com/twentwo/wula][1]

## Usage

spring-boot application support

### Maven

maven dependency import

```xml

<dependency>
    <groupId>io.yec</groupId>
    <artifactId>wula-spring-boot-starter</artifactId>
    <version>${wula.version}</version>
</dependency>
```

### Strategy Bean

create different strategy bean with a logical component name

```java
@Slf4j
@Component("newOrderRouter")
public class NewOrderRouter implements IOrderRouter {

    @Override
    public void wula(OrderEntity orderEntity) {
        log.info("NewOrderRouter = {}", orderEntity);
    }

}
// ...
```

### bizRulesConfig

define strategy route rules, support both:
- bizRulesConfig*.json
- bizRulesConfig*.yml

```json
[
  {
    "group": "io.yec.wula.example.extpoint.IOrderRouter",
    "def": [
      {
        "beanName": "newOrderRouter",
        "extEl": "['businessType'] == 'NEW' && ['discounted'] == false && ['sellerId'] == '618'",
        "desc": "新品"
      },
      {
        "beanName": "normalOrderRouter",
        "extEl": "['businessType'] == 'NORMAL' && ['discounted'] == true ",
        "desc": "普品"
      },
      {
        "beanName": "defaultOrderRouter",
        "extEl": "['businessType'] == null",
        "desc": "默认"
      }
    ]
  }
]
```

or

```yaml
---
- group: io.yec.wula.example.extpoint.IOrderRouter
  def:
    - beanName: newOrderRouter
      extEl: "['businessType'] == 'NEW' && ['discounted'] == false && ['sellerId'] == '618'"
      desc: 新品
    - beanName: normalOrderRouter
      extEl: "['businessType'] == 'NORMAL' && ['discounted'] == true"
      desc: 普品
    - beanName: defaultOrderRouter
      extEl: "['businessType'] == null"
      desc: 默认
- group: io.yec.wula.example.extpoint.IPersonRouter
  def:
    - beanName: yellowRouter
      extEl: "['raceEnum'] == 'YELLOW'"
      desc: 黄种人
```

### ExtensionExecutor

strategy route executor usage

```java
@SpringBootApplication
public class WulaApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(WulaApplication.class, args);
        ExtensionExecutor extensionExecutor = applicationContext.getBean(ExtensionExecutor.class);
        OrderEntity newOrderEntity = OrderEntity.createNewOrderEntity();
        extensionExecutor.executeVoid(IOrderRouter.class,
                // match rule param
                IdentityParam.builder()
                        .businessType(newOrderEntity.getBusinessType())
                        .discounted(false)
                        .sellerId("618")
                        .build(),
                orderRouter -> orderRouter.wula(newOrderEntity));
    }

}
```

[1]: https://github.com/twentwo/wula