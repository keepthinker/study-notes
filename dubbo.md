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

### Dubbo热插拔机制

#### SPI的应用及其原理

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
