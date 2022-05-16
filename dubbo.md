## Dubbo

Apache Dubbo 是一款微服务开发框架，它提供了 RPC通信 与 微服务治理 两大关键能力。这意味着，使用 Dubbo 开发的微服务，将具备相互之间的远程发现与通信能力， 同时利用 Dubbo 提供的丰富服务治理能力，可以实现诸如服务发现、负载均衡、流量调度等服务治理诉求。同时 Dubbo 是高度可扩展的，用户几乎可以在任意功能点去定制自己的实现，以改变框架的默认行为来满足自己的业务需求。

![dubbo-relation](dubbo-architecture.png)

## Loadbalance

### Random LoadBalance

### RoundRobin LoadBalance

### LeastActive LoadBalance

- **LeastActive**, a random mechanism based on actives, `actives` means the num of requests a consumer have sent but not return yet。

- Slower providers will receive fewer requests, cause slower provider have higher `actives`

### ConsistentHash LoadBalance

### ShortestResponse LoadBalance

- Give priority to the shorter response time of the current interval of time. If there are multiple invokers and the same weight, then randomly is called.

- Providers with faster response times can handle more requests.

- Disadvantages: It may cause traffic to be too concentrated on high-performance nodes.

- The response time is the average response time of providers in the interval of time. The interval of time is 30 seconds by default.

## Dubbo支持的协议及其特点

- **dubbo://**（推荐）
- rmi://
- **hessian://**
- **http://**
- webservice://
- thrift://
- memcached://
- **redis://**
- rest://

## Dubbo与Spring Cloud的区别

SpringCloud 与 Dubbo 的区别

两者都是现在主流的微服务框架，但却存在不少差异：

初始定位不同：SpringCloud 定位为微服务架构下的一站式解决方案；Dubbo 是 SOA 时代的产物，它的关注点主要在于服务的调用和治理
生态环境不同：SpringCloud 依托于 Spring 平台，具备更加完善的生态体系；而 Dubbo 一开始只是做 RPC 远程调用，生态相对匮乏，现在逐渐丰富起来。
调用方式：SpringCloud 是采用 Http 协议做远程调用，接口一般是 Rest 风格，比较灵活；Dubbo 是采用 Dubbo 协议，接口一般是 Java 的 Service 接口，格式固定。但调用时采用 Netty 的 NIO 方式，性能较好。
组件差异比较多，例如 SpringCloud 注册中心一般用 Eureka，而 Dubbo 用的是 Zookeeper
SpringCloud 生态丰富，功能完善，更像是品牌机，Dubbo 则相对灵活，可定制性强，更像是组装机。相关资料：

SpringCloud：Spring 公司开源的微服务框架，SpirngCloud 定位为微服务架构下的一站式解决方案。
Dubbo：阿里巴巴开源的 RPC 框架，Dubbo 是 SOA 时代的产物，它的关注点主要在于服务的调用，流量分发、流量监控和熔断。

[SpringCloud与Dubbo的区别 | Oldman](https://oldman.run/posts/42d9c690/)

## Dubbo XML配置

## provider.xml 示例

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-provider"/>
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <dubbo:protocol name="dubbo" port="20890"/>
    <bean id="demoService" class="org.apache.dubbo.samples.basic.impl.DemoServiceImpl"/>
    <dubbo:service interface="org.apache.dubbo.samples.basic.api.DemoService" ref="demoService"/>
</beans>
```

## consumer.xml示例

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-consumer"/>
    <dubbo:registry group="aaa" address="zookeeper://127.0.0.1:2181"/>
    <dubbo:reference id="demoService" check="false" interface="org.apache.dubbo.samples.basic.api.DemoService"/>
</beans>
```

## Dubbo SPI

SPI 全称为 Service Provider Interface，是一种服务发现机制。SPI 的本质是将接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类。这样可以在运行时，动态为接口替换实现类。正因此特性，我们可以很容易的通过 SPI 机制为我们的程序提供拓展功能。

SPI 机制在第三方框架中也有所应用，比如 Dubbo 就是通过 SPI 机制加载所有的组件。不过，**Dubbo 并未使用 Java 原生的 SPI 机制，而是对其进行了增强**(dubbo的org.apache.dubbo.common.extension.ExtensionLoader使用了java.util.ServiceLoader.load方法)，使其能够更好的满足需求。在 Dubbo 中，SPI 是一个非常重要的模块。基于 SPI，我们可以很容易的对 Dubbo 进行拓展。 Dubbo 中，SPI 主要有两种用法，一种是加载固定的扩展类，另一种是加载自适应扩展类。这两种方式会在下面详细的介绍。 需要特别注意的是: 在 Dubbo 中，基于 SPI 扩展加载的类是单例的。

### 扩展例子

#### 协议扩展

RPC 协议扩展，封装远程调用细节。

##### 扩展接口

- `org.apache.dubbo.rpc.Protocol`
- `org.apache.dubbo.rpc.Exporter`
- `org.apache.dubbo.rpc.Invoker`

## 已知扩展

- `org.apache.dubbo.rpc.protocol.injvm.InjvmProtocol`
- `org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol`
- `org.apache.dubbo.rpc.protocol.rmi.RmiProtocol`
- `org.apache.dubbo.rpc.protocol.http.HttpProtocol`
- `org.apache.dubbo.rpc.protocol.http.hessian.HessianProtocol`
- `org.apache.dubbo.rpc.support.MockProtocol`

## 扩展示例

Maven项目结构：

```properties
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxProtocol.java (实现Protocol接口)
                |-XxxExporter.java (实现Exporter接口)
                |-XxxInvoker.java (实现Invoker接口)
    |-resources
        |-META-INF
            |-dubbo
                |-org.apache.dubbo.rpc.Protocol (纯文本文件，内容为：xxx=com.xxx.XxxProtocol)
```

META-INF/dubbo/org.apache.dubbo.rpc.Protocol：

```properties
xxx=com.xxx.XxxProtocol
```

## 集群容错方案

| 方案                | 说明                                                                      |
| ----------------- | ----------------------------------------------------------------------- |
| Failover Cluster  | 失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 retries="2" 来设置重试次数(不含第一次)。 |
| Failfast Cluster  | 快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。                                |
| Failsafe Cluster  | 失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。                                          |
| Failback Cluster  | 失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。                                        |
| Forking Cluster   | 并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks="2" 来设置最大并行数。 |
| Broadcast Cluster | 广播调用所有提供者，逐个调用，任意一台报错则报错。通常用于通知所有提供者更新缓存或日志等本地资源信息。                     |

## 代理对象的创建

Dubbo实现代理对象的方式有两种，一种是使用JDK动态代理，使用的是JDKProxyFactory；另外一种是使用Javassist字节码来实现，使用JavassistProxyFactory来实现。Dubbo默认使用的是JavassistProxyFactory。

## 参考

[Dubbo3 简介 | Apache Dubbo](https://dubbo.apache.org/zh/docs/introduction/)

https://www.zhihu.com/collection/31769803
