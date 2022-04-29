---
tags:
- BeanCopier
- MapStruct
---

# BeanCopier之MapStruct（一）

:material-clock-time-three-outline: 2018-04-12 10:46:11

无意中见同事在比较BeanCopier的效率，MapStruct的使用者很牛皮的说我的效率是你的XX倍，今天认识了一下MapStrut，毫无疑问反射的效率绝对输给setter/getter

## 引入

```xml
<!-- OR use this with Java 8 and beyond: <artifactId>mapstruct-jdk8</artifactId> -->
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-jdk8</artifactId>
    <version>${org.mapstruct.version}</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>${org.mapstruct.version}</version>
    <scope>provided</scope>
</dependency>
```  

maven编译更新成1.8结合`lombok`，同时更新`maven-compiler-plugin`

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.6.2</version>
    <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
        <encoding>${project.encoding}</encoding>
        <annotationProcessorPaths>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </path>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>${org.mapstruct.version}</version>
            </path>
        </annotationProcessorPaths>
        <compilerArgs>
            <compilerArg>
                -Amapstruct.defaultComponentModel=spring
            </compilerArg>
        </compilerArgs>
    </configuration>
</plugin>
```  
加`-Amapstruct.defaultComponentModel=spring`编译配置的目的是指定mapstruct编译生成实现类的时候支持spring的扫描

## 编译
忽略具体bean，copier如下
```java
@Mapper
public interface OrderFundsLiteMapper {

    WjsOrderFundsLiteToValidBean toWjsOrderFundsLiteToValidBean(OrderFundsLiteDO orderFundsLiteDO);

}
```  
maven编译，实现类如下
```java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2018-04-12T10:58:53+0800",
    comments = "version: 1.2.0.Final, compiler: javac, environment: Java 1.8.0_111 (Oracle Corporation)"
)
@Component
public class OrderFundsLiteMapperImpl implements OrderFundsLiteMapper {

    @Override
    public WjsOrderFundsLiteToValidBean toWjsOrderFundsLiteToValidBean(OrderFundsLiteDO orderFundsLiteDO) {
        if ( orderFundsLiteDO == null ) {
            return null;
        }

        WjsOrderFundsLiteToValidBean wjsOrderFundsLiteToValidBean = new WjsOrderFundsLiteToValidBean();

        wjsOrderFundsLiteToValidBean.setOrderFundsId( orderFundsLiteDO.getOrderFundsId() );
        wjsOrderFundsLiteToValidBean.setAgencyCode( orderFundsLiteDO.getAgencyCode() );
        wjsOrderFundsLiteToValidBean.setOrderFundsAmt( orderFundsLiteDO.getOrderFundsAmt() );
        wjsOrderFundsLiteToValidBean.setGmtContractUpdate( orderFundsLiteDO.getGmtContractUpdate() );

        return wjsOrderFundsLiteToValidBean;
    }
}
```  
可见MapStruct在编译期生成实现类，同时作为spring的`@Component`，可以直接注入使用。

## 运行
```
WjsOrderFundsLiteToValidBean(orderFundsId=1, agencyCode=twen, orderFundsAmt=1, gmtContractUpdate=Thu Apr 12 11:05:13 CST 2018)
```  

## 总结
MapStruct在编译时，自动生成具体的setter/getter，减少了代码量，同时避免反射带来的效率牺牲。

## 后
具体学习文档 参见
MapStruct 1.2.0.Final参考指南
[http://mapstruct.org/documentation/stable/reference/html/](http://mapstruct.org/documentation/stable/reference/html/)