## Spring中Bean的作用域

| 类别            | 说明                                                                       |
| ------------- | ------------------------------------------------------------------------ |
| singleton     | 在Spring IoC容器中仅存在一个实例，也就是单例。                                             |
| prototype     | 每次从容器中调用getBean()，都会生成一个新的实例。                                            |
| request(web)  | 每次HTTP请求都会创建一个新的Bean，不同Session使用不同的Bean，仅适用于WebApplicationContext环境    。 |
| session(web)  | 同一个HTTP Session共享一个Bean，不同Session使用不同的Bean。                              |
| globalSession | 表示在一个全局的HTTP Session中，一个bean定义对应一个实例。典型情况下，仅在使用portlet context的时候有效。     |

## Spring IOC原理

Spring 通过一个配置文件描述 Bean 及 Bean 之间的依赖关系，利用 Java 语言的反射功能实例化  Bean 并建立 Bean 之间的依赖关系。 Spring 的 IoC 容器在完成这些底层工作的基础上，还提供  了 Bean 实例缓存、生命周期管理、 Bean 实例代理、事件发布、资源装载等高级服务。

## Bean的生命周期

Bean实例生命周期的执行过程如下：

<img title="" src="bean-life-circle.png" alt="" width="718">

1. Spring对bean进行实例化，默认bean是单例；

2. Spring对bean进行依赖注入；

3. 如果bean实现了BeanNameAware接口，Spring将bean的名称传给setBeanName()方法；

4. 如果bean实现了BeanFactoryAware接口，Spring将调用setBeanFactory()方法，将BeanFactory实例传进来；

5. 如果bean实现了<mark>ApplicationContextAware</mark>接口，它的setApplicationContext()方法将被调用，将应用上下文的引用传入到bean中；

6. 如果bean实现了BeanPostProcessor接口，它的postProcessBeforeInitialization()方法将被调用；

7. 如果bean中有方法添加了@PostConstruct注解，那么该方法将被调用；

8. 如果bean实现了InitializingBean接口，spring将调用它的afterPropertiesSet()接口方法。

9. 如果在xml文件中通过<bean>标签的init-method元素指定了初始化方法，那么该方法将被调用；

10. 如果bean实现了BeanPostProcessor接口，它的postProcessAfterInitialization()接口方法将被调用；

11. 此时bean已经准备就绪，可以被应用程序使用了，他们将一直驻留在应用上下文中，直到该应用上下文被销毁；

12. 如果bean中有方法添加了@PreDestroy注解，那么该方法将被调用；

13. 若bean实现了DisposableBean接口，spring将调用它的distroy()接口方法。同样的，如果bean使用了destroy-method属性声明了销毁方法，则该方法被调用；

[Spring中bean的作用域与生命周期](https://blog.csdn.net/fuzhongmin05/article/details/73389779)

```java
package com.keepthinker.example.spring.ioc;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

/**
 2022-02-12 17:38:48,400 [main] DEBUG [org.springframework.context.support.ClassPathXmlApplicationContext]: Bean factory for org.springframework.context.support.ClassPathXmlApplicationContext@28ba21f3: org.springframework.beans.factory.support.DefaultListableBeanFactory@6500df86: defining beans [annotationConfig,beanInsideComponent,beanInsideConfiguration,configA,configB,cglibBean,person,xmlMain,org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,org.springframework.context.event.internalEventListenerProcessor,org.springframework.context.event.internalEventListenerFactory,abstractAnimal,animal,animalDuplicate,inheritedAnimal,overrideAnimal,earth,earthFromFactory,inner,main,com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor#0,com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor2#0,com.keepthinker.example.spring.ioc.postprocessor.MyInstantiationAwareBeanPostProcessor#0,com.keepthinker.example.spring.ioc.postprocessor.ObscenityRemovingBeanFactoryPostProcessor#0,com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0]; root of factory hierarchy
 2022-02-12 17:38:48,587 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0'
 2022-02-12 17:38:48,587 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0'
 2022-02-12 17:38:48,587 [main] DEBUG [org.springframework.context.annotation.CommonAnnotationBeanPostProcessor]: Found init method on class [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: public void com.keepthinker.example.spring.ioc.BeanLifeCircleObserver.PostConstructMethod()
 2022-02-12 17:38:48,587 [main] DEBUG [org.springframework.context.annotation.CommonAnnotationBeanPostProcessor]: Registered init method on class [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor$LifecycleElement@8c636b8
 2022-02-12 17:38:48,587 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0' to allow for resolving potential circular references
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------setBeanName|com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------setBeanFactory|org.springframework.beans.factory.support.DefaultListableBeanFactory@6500df86: defining beans [annotationConfig,beanInsideComponent,beanInsideConfiguration,configA,configB,cglibBean,person,xmlMain,org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,org.springframework.context.event.internalEventListenerProcessor,org.springframework.context.event.internalEventListenerFactory,abstractAnimal,animal,animalDuplicate,inheritedAnimal,overrideAnimal,earth,earthFromFactory,inner,main,com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor#0,com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor2#0,com.keepthinker.example.spring.ioc.postprocessor.MyInstantiationAwareBeanPostProcessor#0,com.keepthinker.example.spring.ioc.postprocessor.ObscenityRemovingBeanFactoryPostProcessor#0,com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0,panda,compTiger,confTiger,a,b,myTiger,tiger,propertyConfigInDev]; root of factory hierarchy
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------setApplicationContext|org.springframework.context.support.ClassPathXmlApplicationContext@28ba21f3: startup date [Sat Feb 12 17:38:48 CST 2022]; root of context hierarchy
 2022-02-12 17:38:48,587 [main] DEBUG [org.springframework.context.annotation.CommonAnnotationBeanPostProcessor]: Invoking init method on bean 'com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0': public void com.keepthinker.example.spring.ioc.BeanLifeCircleObserver.PostConstructMethod()
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------PostConstructMethod
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor]: postProcessBeforeInitialization before init0 : com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor]: postProcessBeforeInitialization before init1 : com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0
 2022-02-12 17:38:48,587 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Invoking afterPropertiesSet() on bean with name 'com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0'
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------afterPropertiesSet
 2022-02-12 17:38:48,587 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Invoking init method  'xmlInitMethod' on bean with name 'com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0'
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------XmlInitMethod
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor]: postProcessAfterInitialization after init0 : com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor]: postProcessAfterInitialization after init1 : com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0
 2022-02-12 17:38:48,587 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0'
 2022-02-12 17:38:48,587 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@6500df86: defining beans [annotationConfig,beanInsideComponent,beanInsideConfiguration,configA,configB,cglibBean,person,xmlMain,org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,org.springframework.context.event.internalEventListenerProcessor,org.springframework.context.event.internalEventListenerFactory,abstractAnimal,animal,animalDuplicate,inheritedAnimal,overrideAnimal,earth,earthFromFactory,inner,main,com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor#0,com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor2#0,com.keepthinker.example.spring.ioc.postprocessor.MyInstantiationAwareBeanPostProcessor#0,com.keepthinker.example.spring.ioc.postprocessor.ObscenityRemovingBeanFactoryPostProcessor#0,com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0,panda,compTiger,confTiger,a,b,myTiger,tiger,propertyConfigInDev]; root of factory hierarchy
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------postProcessBeforeInitialization|bean:com.keepthinker.example.spring.ioc.AnnotationConfig$$EnhancerBySpringCGLIB$$ec1b4486@78b1cc93|beanName:annotationConfig
 2022-02-12 17:38:48,587 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------postProcessAfterInitialization|bean:com.keepthinker.example.spring.ioc.AnnotationConfig$$EnhancerBySpringCGLIB$$ec1b4486@78b1cc93|beanName:annotationConfig
 2022-02-12 17:38:48,603 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------postProcessBeforeInitialization|bean:com.keepthinker.example.spring.ioc.AtBean.BeanInsideComponent@6646153|beanName:beanInsideComponent
 2022-02-12 17:38:48,603 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------postProcessAfterInitialization|bean:com.keepthinker.example.spring.ioc.AtBean.BeanInsideComponent@6646153|beanName:beanInsideComponent
 2022-02-12 17:38:48,697 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------postProcessBeforeInitialization|bean:com.keepthinker.example.spring.ioc.XmlMain@6af93788|beanName:main
 2022-02-12 17:38:48,697 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------postProcessAfterInitialization|bean:com.keepthinker.example.spring.ioc.XmlMain@6af93788|beanName:main
 2022-02-12 17:38:48,697 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0'
 2022-02-12 17:38:48,697 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------postProcessBeforeInitialization|bean:com.keepthinker.example.spring.ioc.atimport.A@ef9296d|beanName:a
 2022-02-12 17:38:48,697 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------postProcessAfterInitialization|bean:com.keepthinker.example.spring.ioc.atimport.A@ef9296d|beanName:a
 2022-02-12 17:38:49,743 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Destroying singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@6500df86: defining beans [annotationConfig,beanInsideComponent,beanInsideConfiguration,configA,configB,cglibBean,person,xmlMain,org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,org.springframework.context.event.internalEventListenerProcessor,org.springframework.context.event.internalEventListenerFactory,abstractAnimal,animal,animalDuplicate,inheritedAnimal,overrideAnimal,earth,earthFromFactory,inner,main,com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor#0,com.keepthinker.example.spring.ioc.postprocessor.InstantiationTracingBeanPostProcessor2#0,com.keepthinker.example.spring.ioc.postprocessor.MyInstantiationAwareBeanPostProcessor#0,com.keepthinker.example.spring.ioc.postprocessor.ObscenityRemovingBeanFactoryPostProcessor#0,com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0,panda,compTiger,confTiger,a,b,myTiger,tiger,propertyConfigInDev]; root of factory hierarchy
 2022-02-12 17:38:49,743 [main] DEBUG [org.springframework.beans.factory.support.DisposableBeanAdapter]: Invoking destroy() on bean with name 'com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0'
 2022-02-12 17:38:49,743 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------destroy
 2022-02-12 17:38:49,743 [main] DEBUG [org.springframework.beans.factory.support.DisposableBeanAdapter]: Invoking destroy method 'xmlDestroyMethod' on bean with name 'com.keepthinker.example.spring.ioc.BeanLifeCircleObserver#0'
 2022-02-12 17:38:49,743 [main] INFO  [com.keepthinker.example.spring.ioc.BeanLifeCircleObserver]: -------xmlDestroyMethod
 */
public class BeanLifeCircleObserver implements BeanNameAware, BeanFactoryAware, ApplicationContextAware, InitializingBean, BeanPostProcessor, DisposableBean {
    private static Logger logger = LoggerFactory.getLogger(BeanLifeCircleObserver.class);

    /**
     * 如果这个 Bean 已经实现了 BeanNameAware 接口，会调用它实现的 setBeanName(String)
     * 方法，此处传递的就是 Spring 配置文件中 Bean 的 id 值
     */
    @Override
    public void setBeanName(String name) {
        logger.info("-------setBeanName|{}", name);
    }

    /**
     * 如果这个 Bean 已经实现了 BeanFactoryAware 接口，会调用它实现的 setBeanFactory，
     * setBeanFactory(BeanFactory)传递的是 Spring 工厂自身（可以用这个方式来获取其它 Bean，
     * 只需在 Spring 配置文件中配置一个普通的 Bean 就可以）。
     */
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        logger.info("-------setBeanFactory|{}", beanFactory);
    }

    /**
     *如果这个 Bean 已经实现了 ApplicationContextAware 接口，会调用
     * setApplicationContext(ApplicationContext)方法，传入 Spring 上下文（同样这个方式也
     * 可以实现步骤 4 的内容，但比 4 更好，因为 ApplicationContext 是 BeanFactory 的子接
     * 口，有更多的实现方法）
     */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        logger.info("-------setApplicationContext|{}", applicationContext);
    }

    @PostConstruct
    public void PostConstructMethod() {
        logger.info("-------PostConstructMethod");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        logger.info("-------afterPropertiesSet");
    }

    public void xmlInitMethod() {
        logger.info("-------XmlInitMethod");
    }

    /**
     * 如果这个 Bean 关联了 BeanPostProcessor 接口，将会调用
     * postProcessBeforeInitialization(Object obj, String s)方法， BeanPostProcessor 经常被用
     * 作是 Bean 内容的更改，并且由于这个是在 Bean 初始化结束时调用那个的方法，也可以被应
     * 用于内存或缓存技术。
     */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        logger.info("-------postProcessBeforeInitialization|bean:{}|beanName:{}", bean, beanName);
        return bean;
    }

    /*
    init-method
    如果 Bean 在 Spring 配置文件中配置了 init-method 属性会自动调用其配置的初始化方法。
    */

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        logger.info("-------postProcessAfterInitialization|bean:{}|beanName:{}", bean, beanName);
        return bean;
    }

    /**
     * Destroy 过期自动清理阶段
     * 当 Bean 不再需要时，会经过清理阶段，如果 Bean 实现了 DisposableBean 这个接口，会调
     * 用那个其实现的 destroy()方法；
     * @throws Exception
     */
    @Override
    public void destroy() throws Exception {
        logger.info("-------destroy");
    }

    public void xmlDestroyMethod() {
        logger.info("-------xmlDestroyMethod");
    }

    /*
    destroy-method
    自配置清理，如果这个 Bean 的 Spring 配置中配置了 destroy-method 属性，会自动调用其配置的
    销毁方法。
     */

    /*
    bean 标签有两个重要的属性（init-method 和 destroy-method）。用它们你可以自己定制
    初始化和注销方法。它们也有相应的注解（@PostConstruct 和@PreDestroy） 。
    <bean id="" class="" init-method="初始化方法" destroy-method="销毁方法">
     */
}
```

## BeanFactoryPostProcessor

### BeanFactoryPostProcessor

```java
public interface BeanFactoryPostProcessor {
   void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

实现该接口，可以在 Spring 创建 bean 之前修改 bean 的定义属性。也就是说，Spring 允许 BeanFactoryPostProcessor 在容器实例化 bean 之前读取配置元数据，并可以根据需要进行修改。例如可以把 bean 的 Scope 从 singleton 改为 prototype ，也可以把 **property 的值给修改掉**。另外可以同时配置多个 BeanFactoryPostProcessor，并通过 order 属性来控制 BeanFactoryPostProcessor 的执行顺序 （ 在实现 BeanFactoryPostProcessor 时应该考虑实现 Ordered 接口 ）。比如PropertyPlaceholderConfigurer就是在postProcessBeanFactory方法里实现property值的替换。

BeanFactoryPostProcessor 是在 Spring 容器加载了定义 bean 的 XML 文件之后，在 bean 实例化之前执行的。接口方法的入参是 ConfigurrableListableBeanFactory 类型，使用该参数可以获取到相关的 bean 的定义信息。

## PostProcessor

### Bean post processors

Spring 框架提供了几种 PostProcessor接口用于建模对容器或者bean的后置处理器，它们定义了一些方法，这些方法在特定的时机会被调用。通过这种机制，框架自身或者应用开发人员有机会在不侵入容器或者bean核心逻辑的情况下为容器或者bean做针对某些特定方面的定制或者扩展：能力增强，属性设置，内容修改，对象代理，甚至直接替换整个bean。Spring 提供的 PostProcessor 接口有如下几种 ：

1. BeanDefinitionRegistryPostProcessor– BeanDefinitionRegistry后置处理器 – 容器级别

2. BeanFactoryPostProcessor–BeanFactory后置处理器 – 容器级别

3. BeanPostProcessor–Bean后置处理器 – bean实例级别
   实际应用中又可细分为如下几类 :
   
   1. InstantiationAwareBeanPostProcessor
   
   2. MergedBeanDefinitionPostProcessor
   
   3. DestructionAwareBeanPostProcessor
   
   4. SmartInstantiationAwareBeanPostProcessor
   
   5. 一般BeanPostProcessor

Spring框架自身提供了很多这些PostProcessor的实现类，每个PostProcessor实现类分别有不同的关注点,Spring利用这些PostProcessor实现类完成了很多框架自身的任务，主要在容器启动和bean获取阶段。另外，开发人员也可以实现自己的PostProcessor来扩展Spring容器或者bean的能力。这里面尤其是通过自定义实现BeanPostProcessor,开发人员有机会对容器中所有的bean做定制。

[Spring的各种PostProcessor_Details Inside Spring-CSDN博客](https://blog.csdn.net/andy_zhang2007/article/details/78595558)

[Spring之BeanFactoryPostProcessor和BeanPostProcessor - 掘金](https://juejin.cn/post/6844903708745007112)

## Spring AOP原理

"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共 同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

使用"横切"技术，AOP把软件系统分为两个部分：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生 在核心关注点的多处，而各处基本相似，比如权限认证、日志、事物。AOP的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。  

AOP 主要应用场景有：  

1. Authentication 权限  

2. Caching 缓存  

3. Context passing 内容传递  

4. Error handling 错误处理  

5. Lazy loading 懒加载  

6. Debugging 调试  

7. logging, tracing, profiling and monitoring 记录跟踪 优化 校准  

8. Performance optimization 性能优化  

9. Persistence 持久化  

10. Resource pooling 资源池  

11. Synchronization 同步  

12. Transactions 事务

### AOP 核心概念

1. 切面（aspect） ： 类是对物体特征的抽象，切面就是对横切关注点的抽象。

2. 横切关注点： 对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点。  

3. 连接点（joinpoint） ： 被拦截到的点，因为 Spring 只支持方法类型的连接点，`所以在 Spring  中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器。`  

4. 切入点（pointcut） ： 对连接点进行拦截的定义。

5. 通知（advice） ： 所谓通知指的就是指拦截到连接点之后要执行的代码， 通知分为前置、后置、 异常、最终、环绕通知五类。

6. 目标对象： 代理的目标对象。

7. 织入（weave） ： 将切面应用到目标对象并导致代理对象创建的过程。

8. 引入（introduction） ： 在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段。

Spring 提供了两种方式来生成代理对象: JDKProxy 和 Cglib，具体使用哪种方式 生成由  AopProxyFactory 根据 AdvisedSupport 对象的配置来决定。 默认的策略是如果目标类是接口， 则使用 JDK 动态代理技术，否则使用 Cglib 来生成代理。

## Resource

Resource 接口是 Spring 资源访问策略的抽象，它本身并不提供任何资源访问实现，具体的资源访问由该接口的实现类完成——每个实现类代表一种资源访问策略。 Spring 为 Resource 接口提供了如下实现类：

- UrlResource：访问网络资源的实现类。
- ClassPathResource：访问类加载路径里资源的实现类。
- FileSystemResource：访问文件系统里资源的实现类。
- ServletContextResource：访问相对于 ServletContext 路径里的资源的实现类：
- InputStreamResource：访问输入流资源的实现类。
- ByteArrayResource：访问字节数组资源的实现类。 这些 Resource 实现类，针对不同的的底层资源，提供了相应的资源访问逻辑，并提供便捷的包装，以利于客户端程序的资源访问。

## AbstractApplicationContext

最常被使用的 ApplicationContext 接口实现：

- FileSystemXmlApplicationContext：该容器从 XML 文件中加载已被定义的 bean。在这里，你需要提供给构造器 XML 文件的完整路径

- ClassPathXmlApplicationContext：该容器从 XML 文件中加载已被定义的 bean。在这里，你不需要提供 XML 文件的完整路径，只需正确配置 CLASSPATH 环境变量即可，因为，容器会从 CLASSPATH 中搜索 bean 配置文件。

- WebXmlApplicationContext：该容器会在一个 web 应用程序的范围内加载在 XML 文件中已被定义的 bean。

参考：[Homiss/Java-interview-questions · GitHub](https://github.com/Homiss/Java-interview-questions/blob/master/%E6%A1%86%E6%9E%B6/Spring%20%E9%9D%A2%E8%AF%95%E9%A2%98.md)
