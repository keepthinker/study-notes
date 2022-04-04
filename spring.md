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

## Spring容器和Bean的生命周期

Bean实例生命周期的执行过程如下：

<img title="" src="bean-life-circle.png" alt="" width="1280">

1. 如果一个Bean实现了BeanFactoryPostProcessor，那么调用postProcessBeanFactory方法。该接口主要用于在生产Bean之前对容器一些配置进行修改。常见的BeanFactoryPostProcessor实现有PropertyPlaceholderConfigurer。
2. 准备开始创建Bean之前，先调用postProcessBeforeInstantiation，传入参数有即将创建Bean的Class和beanName，如果该函数返回不为null的对象，那么下一步回调直接到postProcessAfterInitialization方法，中间的postProcessAfterInstantiation，postProcessPropertyValues，postProcessBeforeInitialization将都不会执行。。
3. 开始调用类的构造函数，也就是constructor方法。
4. 调用postProcessAfterInstantiation方法。传入参数有Bean的实例对象和beanName。
5. 调用postProcessPropertyValues方法。可以对Bean的设置的property进行修改，比如可以对<property name="lifetime" value="#{3600*24}"/>这个标签的值进行修改。
6. Spring对bean进行依赖注入。比如使用了Autowired，那么会将Autowired的对象注入到注解对应的地方。
7. 如果bean实现了BeanNameAware接口，Spring将bean的名称传给setBeanName()方法。
8. 如果bean实现了BeanClassLoaderAware接口，Spring将调用etBeanClassLoader()方法，把Classloader传递进来。
9. 如果bean实现了BeanFactoryAware接口，Spring将调用setBeanFactory()方法，把BeanFactory实例传进来。
10. 如果Bean实现了EnvironmentAware接口，Spring将调用setEnvironment()方法，把Environment实例传进来。
11. 如果bean实现了ApplicationContextAware接口，它的setApplicationContext()方法将被调用，将应用上下文的引用传入到bean中。
12. 如果一个Bean实现了BeanPostProcessor接口，postProcessBeforeInitialization()方法将被调用。
13. 如果bean中有方法添加了@PostConstruct注解，那么该方法将被调用。
14. 如果bean实现了InitializingBean接口，spring将调用它的afterPropertiesSet()接口方法。
15. 如果在xml文件中通过<bean>标签的init-method元素指定了初始化方法，那么该方法将被调用。
16. 如果bean实现了BeanPostProcessor接口，它的postProcessAfterInitialization()接口方法将被调用。在这个时候Bean的属性如前所述基本准备完毕，在这个方法里一般用Java Proxy或CgLIB做代理实现一些额外的功能，比如spring aop使用这个机制做数据库事务功能。
17. 此时bean已经准备就绪，可以被应用程序使用了，他们将一直驻留在应用上下文中，直到该应用上下文被销毁。
18. 如果bean中有方法添加了@PreDestroy注解，那么该方法将被调用。
19. 若bean实现了DisposableBean接口，spring将调用它的distroy()接口方法。
20. 如果bean使用了destroy-method属性声明了销毁方法，则该方法被调用。

[Spring中bean的作用域与生命周期](https://blog.csdn.net/fuzhongmin05/article/details/73389779)

```java
package com.keepthinker.example.spring.ioc;


public class MyInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {
    private static Logger logger = LoggerFactory.getLogger(MyInstantiationAwareBeanPostProcessor.class);
    /*** 执行顺序1
     * 实例化前置处理方法，也就是在Bean没有生成之前执行。（注意：这里所说的是Bean未生成指的是Bean没有走spring定义创建Bean的流程，也就是doCreateBean()方法。）*/
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        logger.info("postProcessBeforeInstantiation|beanClass:{}|beanName:{}", beanClass, beanName);
        return null;
    }
    /** 执行顺序2 */
    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        logger.info("postProcessAfterInstantiation|bean:{}|beanName:{}", bean, beanName);
        return true;
    }
    /** 执行顺序3 */
    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {
        logger.info("postProcessPropertyValues|PropertyValues:{}|PropertyDescriptor:{}|bean:{}|beanName:{}",
                pvs, pds, bean, beanName);
        return pvs;
    }
    /** 执行顺序4 */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        logger.info("postProcessBeforeInitialization|bean:{}|beanName:{}", bean, beanName);
        return bean;
    }
    /*** 执行顺序5 */
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        logger.info("postProcessPropertyValues|bean:{}|beanName:{}", bean, beanName);
        return bean;
    }
}

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

    /**
     * 如果这个Bean 关联了 BeanPostProcessor 接口，将会调用postProcessBeforeInitialization(Object obj, String s)
     */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        logger.info("-------postProcessBeforeInitialization|bean:{}|beanName:{}", bean, beanName);
        return bean;
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
    如果这个Bean实现了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法；
    由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术； 以上几个步骤完成后，
    Bean就已经被正确创建了，之后就可以使用这个Bean了。
    */
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        logger.info("-------postProcessAfterInitialization|bean:{}|beanName:{}", bean, beanName);
        return bean;
    }

    @PreDestroy
    public void preDestroyMethod() throws Exception {
        logger.info("-------@PreDestroyMethod");
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

## 上述代码运行后的日志

```java
2022-03-12 23:20:05,899 [main] DEBUG [org.springframework.core.env.StandardEnvironment]: Adding PropertySource 'systemProperties' with lowest search precedence
2022-03-12 23:20:05,902 [main] DEBUG [org.springframework.core.env.StandardEnvironment]: Adding PropertySource 'systemEnvironment' with lowest search precedence
2022-03-12 23:20:05,902 [main] DEBUG [org.springframework.core.env.StandardEnvironment]: Initialized StandardEnvironment with PropertySources [MapPropertySource@114935352 {name='systemProperties', properties={java.runtime.name=Java(TM) SE...
2022-03-12 23:20:05,905 [main] INFO  [org.springframework.context.support.ClassPathXmlApplicationContext]: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@28ba21f3: startup date [Sat Mar 12 23:20:05 CST 2022]; root of context hierarchy
2022-03-12 23:20:05,951 [main] DEBUG [org.springframework.core.env.StandardEnvironment]: Adding PropertySource 'systemProperties' with lowest search precedence
2022-03-12 23:20:05,952 [main] DEBUG [org.springframework.core.env.StandardEnvironment]: Adding PropertySource 'systemEnvironment' with lowest search precedence
2022-03-12 23:20:05,952 [main] DEBUG [org.springframework.core.env.StandardEnvironment]: Initialized StandardEnvironment with PropertySources [MapPropertySource@1989972246 {name='systemProperties', properties={java.runtime.name=Java(TM) SE Runtime Environment, sun.boot.library.path=D:\Program Files\Java\jdk1.8.0_251\jre\bin, java.vm.version=25.251-b08, ...
2022-03-12 23:20:05,961 [main] INFO  [org.springframework.beans.factory.xml.XmlBeanDefinitionReader]: Loading XML bean definitions from class path resource [applicationContext.xml]
2022-03-12 23:20:05,976 [main] DEBUG [org.springframework.beans.factory.xml.DefaultDocumentLoader]: Using JAXP provider [com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl]
2022-03-12 23:20:06,003 [main] DEBUG [org.springframework.beans.factory.xml.PluggableSchemaResolver]: Loading schema mappings from [META-INF/spring.schemas]
2022-03-12 23:20:06,007 [main] DEBUG [org.springframework.beans.factory.xml.PluggableSchemaResolver]: Loaded schema mappings: {https://www.springframework.org/schema/aop/spring-aop.xsd=org/springframework/aop/config/spring-aop-4.3.xsd...
2022-03-12 23:20:06,008 [main] DEBUG [org.springframework.beans.factory.xml.PluggableSchemaResolver]: Found XML schema [http://www.springframework.org/schema/beans/spring-beans.xsd] in classpath: org/springframework/beans/factory/xml/spring-beans-4.3.xsd
2022-03-12 23:20:06,039 [main] DEBUG [org.springframework.beans.factory.xml.PluggableSchemaResolver]: Found XML schema [http://www.springframework.org/schema/context/spring-context.xsd] in classpath: org/springframework/context/config/spring-context-4.3.xsd
2022-03-12 23:20:06,044 [main] DEBUG [org.springframework.beans.factory.xml.PluggableSchemaResolver]: Found XML schema [https://www.springframework.org/schema/tool/spring-tool-4.3.xsd] in classpath: org/springframework/beans/factory/xml/spring-tool-4.3.xsd
2022-03-12 23:20:06,051 [main] DEBUG [org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader]: Loading bean definitions
2022-03-12 23:20:06,058 [main] DEBUG [org.springframework.beans.factory.xml.DefaultNamespaceHandlerResolver]: Loading NamespaceHandler mappings from [META-INF/spring.handlers]
2022-03-12 23:20:06,059 [main] DEBUG [org.springframework.beans.factory.xml.DefaultNamespaceHandlerResolver]: Loaded NamespaceHandler mappings: {http://www.springframework.org/schema/p=org.springframework.beans.factory.xml.SimplePropertyNamespaceHandler...
2022-03-12 23:20:06,079 [main] DEBUG [org.springframework.core.io.support.PathMatchingResourcePatternResolver]: Resolved classpath location [com/keepthinker/example/spring/lifecircle/] to resources [URL [file:/D:/git/hotchportch-example-code/example-spring-lifecircle/target/classes/com/keepthinker/example/spring/lifecircle/]]
2022-03-12 23:20:06,079 [main] DEBUG [org.springframework.core.io.support.PathMatchingResourcePatternResolver]: Looking for matching resources in directory tree [D:\git\hotchportch-example-code\example-spring-lifecircle\target\classes\com\keepthinker\example\spring\lifecircle]
2022-03-12 23:20:06,079 [main] DEBUG [org.springframework.core.io.support.PathMatchingResourcePatternResolver]: Searching directory [D:\git\hotchportch-example-code\example-spring-lifecircle\target\classes\com\keepthinker\example\spring\lifecircle] for files matching pattern [D:/git/hotchportch-example-code/example-spring-lifecircle/target/classes/com/keepthinker/example/spring/lifecircle/**/*.class]
2022-03-12 23:20:06,082 [main] DEBUG [org.springframework.core.io.support.PathMatchingResourcePatternResolver]: Searching directory [D:\git\hotchportch-example-code\example-spring-lifecircle\target\classes\com\keepthinker\example\spring\lifecircle\postprocessor] for files matching pattern [D:/git/hotchportch-example-code/example-spring-lifecircle/target/classes/com/keepthinker/example/spring/lifecircle/**/*.class]
2022-03-12 23:20:06,083 [main] DEBUG [org.springframework.core.io.support.PathMatchingResourcePatternResolver]: Searching directory [D:\git\hotchportch-example-code\example-spring-lifecircle\target\classes\com\keepthinker\example\spring\lifecircle\zoo] for files matching pattern [D:/git/hotchportch-example-code/example-spring-lifecircle/target/classes/com/keepthinker/example/spring/lifecircle/**/*.class]
2022-03-12 23:20:06,084 [main] DEBUG [org.springframework.core.io.support.PathMatchingResourcePatternResolver]: Resolved location pattern [classpath*:com/keepthinker/example/spring/lifecircle/**/*.class] to resources [file [D:\git\hotchportch-example-code\example-spring-lifecircle\target\classes\com\keepthinker\example\spring\lifecircle\BeanLifeCircleObserver.class]...
2022-03-12 23:20:06,121 [main] DEBUG [org.springframework.context.annotation.ClassPathBeanDefinitionScanner]: Identified candidate component class: file [D:\git\hotchportch-example-code\example-spring-lifecircle\target\classes\com\keepthinker\example\spring\lifecircle\postprocessor\MyBeanFactoryPostProcessor.class]
2022-03-12 23:20:06,122 [main] DEBUG [org.springframework.context.annotation.ClassPathBeanDefinitionScanner]: Identified candidate component class: file [D:\git\hotchportch-example-code\example-spring-lifecircle\target\classes\com\keepthinker\example\spring\lifecircle\postprocessor\MyInstantiationAwareBeanPostProcessor.class]
2022-03-12 23:20:06,122 [main] DEBUG [org.springframework.context.annotation.ClassPathBeanDefinitionScanner]: Identified candidate component class: file [D:\git\hotchportch-example-code\example-spring-lifecircle\target\classes\com\keepthinker\example\spring\lifecircle\zoo\Tiger.class]
2022-03-12 23:20:06,125 [main] DEBUG [org.springframework.context.annotation.ClassPathBeanDefinitionScanner]: Identified candidate component class: file [D:\git\hotchportch-example-code\example-spring-lifecircle\target\classes\com\keepthinker\example\spring\lifecircle\zoo\Zookeeper.class]
2022-03-12 23:20:06,141 [main] DEBUG [org.springframework.beans.factory.xml.BeanDefinitionParserDelegate]: Neither XML 'id' nor 'name' specified - using generated bean name [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0]
2022-03-12 23:20:06,141 [main] DEBUG [org.springframework.beans.factory.xml.XmlBeanDefinitionReader]: Loaded 11 bean definitions from location pattern [applicationContext.xml]
2022-03-12 23:20:06,141 [main] DEBUG [org.springframework.context.support.ClassPathXmlApplicationContext]: Bean factory for org.springframework.context.support.ClassPathXmlApplicationContext@28ba21f3: org.springframework.beans.factory.support.DefaultListableBeanFactory@e25b2fe: defining beans [myBeanFactoryPostProcessor...
2022-03-12 23:20:06,159 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
2022-03-12 23:20:06,159 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
2022-03-12 23:20:06,171 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor' to allow for resolving potential circular references
2022-03-12 23:20:06,173 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
2022-03-12 23:20:06,194 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'myBeanFactoryPostProcessor'
2022-03-12 23:20:06,194 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'myBeanFactoryPostProcessor'
2022-03-12 23:20:06,194 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'myBeanFactoryPostProcessor' to allow for resolving potential circular references
2022-03-12 23:20:06,206 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'myBeanFactoryPostProcessor'
2022-03-12 23:20:06,206 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyBeanFactoryPostProcessor]: postProcessBeanFactory invoked
2022-03-12 23:20:06,207 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
2022-03-12 23:20:06,207 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
2022-03-12 23:20:06,208 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor' to allow for resolving potential circular references
2022-03-12 23:20:06,212 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
2022-03-12 23:20:06,212 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'org.springframework.context.annotation.internalRequiredAnnotationProcessor'
2022-03-12 23:20:06,212 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'org.springframework.context.annotation.internalRequiredAnnotationProcessor'
2022-03-12 23:20:06,212 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'org.springframework.context.annotation.internalRequiredAnnotationProcessor' to allow for resolving potential circular references
2022-03-12 23:20:06,215 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'org.springframework.context.annotation.internalRequiredAnnotationProcessor'
2022-03-12 23:20:06,215 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
2022-03-12 23:20:06,215 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
2022-03-12 23:20:06,217 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor' to allow for resolving potential circular references
2022-03-12 23:20:06,221 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
2022-03-12 23:20:06,221 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'myInstantiationAwareBeanPostProcessor'
2022-03-12 23:20:06,221 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'myInstantiationAwareBeanPostProcessor'
2022-03-12 23:20:06,224 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'myInstantiationAwareBeanPostProcessor' to allow for resolving potential circular references
2022-03-12 23:20:06,226 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'myInstantiationAwareBeanPostProcessor'
2022-03-12 23:20:06,227 [main] DEBUG [org.springframework.context.support.ClassPathXmlApplicationContext]: Unable to locate MessageSource with name 'messageSource': using default [org.springframework.context.support.DelegatingMessageSource@75881071]
2022-03-12 23:20:06,229 [main] DEBUG [org.springframework.context.support.ClassPathXmlApplicationContext]: Unable to locate ApplicationEventMulticaster with name 'applicationEventMulticaster': using default [org.springframework.context.event.SimpleApplicationEventMulticaster@a74868d]
2022-03-12 23:20:06,230 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@e25b2fe: defining beans [myBeanFactoryPostProcessor,myInstantiationAwareBeanPostProcessor,...
2022-03-12 23:20:06,230 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'myBeanFactoryPostProcessor'
2022-03-12 23:20:06,230 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'myInstantiationAwareBeanPostProcessor'
2022-03-12 23:20:06,230 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'tiger'
2022-03-12 23:20:06,230 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'tiger'
2022-03-12 23:20:06,231 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessBeforeInstantiation|beanClass:class com.keepthinker.example.spring.lifecircle.zoo.Tiger|beanName:tiger
2022-03-12 23:20:06,231 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'tiger' to allow for resolving potential circular references
2022-03-12 23:20:06,231 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessAfterInstantiation|bean:Tiger{id=1, name='null'}|beanName:tiger
2022-03-12 23:20:06,233 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessPropertyValues|PropertyValues:PropertyValues: length=0|PropertyDescriptor:[org.springframework.beans.GenericTypeAwarePropertyDescriptor[name=class], org.springframework.beans.GenericTypeAwarePropertyDescriptor[name=name]]|bean:Tiger{id=1, name='null'}|beanName:tiger
2022-03-12 23:20:06,233 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessBeforeInitialization|bean:Tiger{id=1, name='null'}|beanName:tiger
2022-03-12 23:20:06,233 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessAfterInitialization|bean:Tiger{id=1, name='null'}|beanName:tiger
2022-03-12 23:20:06,233 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'tiger'
2022-03-12 23:20:06,233 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'zookeeper'
2022-03-12 23:20:06,233 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'zookeeper'
2022-03-12 23:20:06,233 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessBeforeInstantiation|beanClass:class com.keepthinker.example.spring.lifecircle.zoo.Zookeeper|beanName:zookeeper
2022-03-12 23:20:06,240 [main] DEBUG [org.springframework.beans.factory.annotation.InjectionMetadata]: Registered injected element on class [com.keepthinker.example.spring.lifecircle.zoo.Zookeeper]: AutowiredMethodElement for private void com.keepthinker.example.spring.lifecircle.zoo.Zookeeper.setTiger(com.keepthinker.example.spring.lifecircle.zoo.Tiger)
2022-03-12 23:20:06,240 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'zookeeper' to allow for resolving potential circular references
2022-03-12 23:20:06,240 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessAfterInstantiation|bean:Zookeeper{, tiger=null}|beanName:zookeeper
2022-03-12 23:20:06,240 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessPropertyValues|PropertyValues:PropertyValues: length=0|PropertyDescriptor:[org.springframework.beans.GenericTypeAwarePropertyDescriptor[name=class]]|bean:Zookeeper{, tiger=null}|beanName:zookeeper
2022-03-12 23:20:06,240 [main] DEBUG [org.springframework.beans.factory.annotation.InjectionMetadata]: Processing injected element of bean 'zookeeper': AutowiredMethodElement for private void com.keepthinker.example.spring.lifecircle.zoo.Zookeeper.setTiger(com.keepthinker.example.spring.lifecircle.zoo.Tiger)
2022-03-12 23:20:06,244 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'tiger'
2022-03-12 23:20:06,244 [main] DEBUG [org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor]: Autowiring by type from bean name 'zookeeper' to bean named 'tiger'
2022-03-12 23:20:06,244 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessBeforeInitialization|bean:Zookeeper{, tiger=Tiger{id=1, name='null'}}|beanName:zookeeper
2022-03-12 23:20:06,244 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessAfterInitialization|bean:Zookeeper{, tiger=Tiger{id=1, name='null'}}|beanName:zookeeper
2022-03-12 23:20:06,244 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'zookeeper'
2022-03-12 23:20:06,244 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
2022-03-12 23:20:06,244 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
2022-03-12 23:20:06,244 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'org.springframework.context.annotation.internalRequiredAnnotationProcessor'
2022-03-12 23:20:06,244 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
2022-03-12 23:20:06,244 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
2022-03-12 23:20:06,244 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'org.springframework.context.event.internalEventListenerProcessor'
2022-03-12 23:20:06,244 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessBeforeInstantiation|beanClass:class org.springframework.context.event.EventListenerMethodProcessor|beanName:org.springframework.context.event.internalEventListenerProcessor
2022-03-12 23:20:06,245 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'org.springframework.context.event.internalEventListenerProcessor' to allow for resolving potential circular references
2022-03-12 23:20:06,245 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessAfterInstantiation|bean:org.springframework.context.event.EventListenerMethodProcessor@319b92f3|beanName:org.springframework.context.event.internalEventListenerProcessor
2022-03-12 23:20:06,246 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessPropertyValues|PropertyValues:PropertyValues: length=0|PropertyDescriptor:[org.springframework.beans.GenericTypeAwarePropertyDescriptor[name=class]]|bean:org.springframework.context.event.EventListenerMethodProcessor@319b92f3|beanName:org.springframework.context.event.internalEventListenerProcessor
2022-03-12 23:20:06,247 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessBeforeInitialization|bean:org.springframework.context.event.EventListenerMethodProcessor@319b92f3|beanName:org.springframework.context.event.internalEventListenerProcessor
2022-03-12 23:20:06,247 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessAfterInitialization|bean:org.springframework.context.event.EventListenerMethodProcessor@319b92f3|beanName:org.springframework.context.event.internalEventListenerProcessor
2022-03-12 23:20:06,247 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'org.springframework.context.event.internalEventListenerProcessor'
2022-03-12 23:20:06,247 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
2022-03-12 23:20:06,247 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'org.springframework.context.event.internalEventListenerFactory'
2022-03-12 23:20:06,247 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessBeforeInstantiation|beanClass:class org.springframework.context.event.DefaultEventListenerFactory|beanName:org.springframework.context.event.internalEventListenerFactory
2022-03-12 23:20:06,247 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'org.springframework.context.event.internalEventListenerFactory' to allow for resolving potential circular references
2022-03-12 23:20:06,247 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessAfterInstantiation|bean:org.springframework.context.event.DefaultEventListenerFactory@5c18298f|beanName:org.springframework.context.event.internalEventListenerFactory
2022-03-12 23:20:06,249 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessPropertyValues|PropertyValues:PropertyValues: length=0|PropertyDescriptor:[org.springframework.beans.GenericTypeAwarePropertyDescriptor[name=class], org.springframework.beans.GenericTypeAwarePropertyDescriptor[name=order]]|bean:org.springframework.context.event.DefaultEventListenerFactory@5c18298f|beanName:org.springframework.context.event.internalEventListenerFactory
2022-03-12 23:20:06,249 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessBeforeInitialization|bean:org.springframework.context.event.DefaultEventListenerFactory@5c18298f|beanName:org.springframework.context.event.internalEventListenerFactory
2022-03-12 23:20:06,249 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessAfterInitialization|bean:org.springframework.context.event.DefaultEventListenerFactory@5c18298f|beanName:org.springframework.context.event.internalEventListenerFactory
2022-03-12 23:20:06,249 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'org.springframework.context.event.internalEventListenerFactory'
2022-03-12 23:20:06,249 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating shared instance of singleton bean 'com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0'
2022-03-12 23:20:06,249 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Creating instance of bean 'com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0'
2022-03-12 23:20:06,249 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessBeforeInstantiation|beanClass:class com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver|beanName:com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0
2022-03-12 23:20:06,250 [main] INFO  [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: java constructor invoked
2022-03-12 23:20:06,251 [main] DEBUG [org.springframework.context.annotation.CommonAnnotationBeanPostProcessor]: Found init method on class [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: public void com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver.PostConstructMethod()
2022-03-12 23:20:06,251 [main] DEBUG [org.springframework.context.annotation.CommonAnnotationBeanPostProcessor]: Found destroy method on class [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: public void com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver.preDestroyMethod() throws java.lang.Exception
2022-03-12 23:20:06,251 [main] DEBUG [org.springframework.context.annotation.CommonAnnotationBeanPostProcessor]: Registered init method on class [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor$LifecycleElement@8c636b8
2022-03-12 23:20:06,251 [main] DEBUG [org.springframework.context.annotation.CommonAnnotationBeanPostProcessor]: Registered destroy method on class [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor$LifecycleElement@63071798
2022-03-12 23:20:06,254 [main] DEBUG [org.springframework.beans.factory.annotation.InjectionMetadata]: Registered injected element on class [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: AutowiredMethodElement for private void com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver.setZookeeper(com.keepthinker.example.spring.lifecircle.zoo.Zookeeper)
2022-03-12 23:20:06,254 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Eagerly caching bean 'com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0' to allow for resolving potential circular references
2022-03-12 23:20:06,254 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessAfterInstantiation|bean:com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver@7a36aefa|beanName:com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0
2022-03-12 23:20:06,255 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessPropertyValues|PropertyValues:PropertyValues: length=1; bean property 'lifetime'|PropertyDescriptor:[org.springframework.beans.GenericTypeAwarePropertyDescriptor[name=class], org.springframework.beans.GenericTypeAwarePropertyDescriptor[name=lifetime]]|bean:com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver@7a36aefa|beanName:com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0
2022-03-12 23:20:06,255 [main] DEBUG [org.springframework.beans.factory.annotation.InjectionMetadata]: Processing injected element of bean 'com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0': AutowiredMethodElement for private void com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver.setZookeeper(com.keepthinker.example.spring.lifecircle.zoo.Zookeeper)
2022-03-12 23:20:06,255 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'zookeeper'
2022-03-12 23:20:06,255 [main] DEBUG [org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor]: Autowiring by type from bean name 'com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0' to bean named 'zookeeper'
2022-03-12 23:20:06,255 [main] INFO  [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: property zookeeper set
2022-03-12 23:20:06,280 [main] INFO  [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: -------setBeanName|com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0
2022-03-12 23:20:06,280 [main] INFO  [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: -------setBeanClassLoader|sun.misc.Launcher$AppClassLoader@18b4aac2
2022-03-12 23:20:06,280 [main] INFO  [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: -------setBeanFactory|org.springframework.beans.factory.support.DefaultListableBeanFactory@e25b2fe: defining beans [myBeanFactoryPostProcessor,myInstantiationAwareBeanPostProcessor,tiger,zookeeper,...
2022-03-12 23:20:06,280 [main] INFO  [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: -------setEnvironment|StandardEnvironment {activeProfiles=[], defaultProfiles=[default], propertySources=[MapPropertySource@114935352 {name='systemProperties', properties={java.runtime.name=Java(TM) SE Runtime Environment, ...
2022-03-12 23:20:06,280 [main] INFO  [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: -------setApplicationContext|org.springframework.context.support.ClassPathXmlApplicationContext@28ba21f3: startup date [Sat Mar 12 23:20:05 CST 2022]; root of context hierarchy
2022-03-12 23:20:06,283 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessBeforeInitialization|bean:com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver@7a36aefa|beanName:com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0
2022-03-12 23:20:06,283 [main] DEBUG [org.springframework.context.annotation.CommonAnnotationBeanPostProcessor]: Invoking init method on bean 'com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0': public void com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver.PostConstructMethod()
2022-03-12 23:20:06,283 [main] INFO  [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: -------@PostConstructMethod
2022-03-12 23:20:06,283 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Invoking afterPropertiesSet() on bean with name 'com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0'
2022-03-12 23:20:06,283 [main] INFO  [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: -------afterPropertiesSet
2022-03-12 23:20:06,283 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Invoking init method  'xmlInitMethod' on bean with name 'com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0'
2022-03-12 23:20:06,283 [main] INFO  [com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver]: -------XmlInitMethod
2022-03-12 23:20:06,283 [main] INFO  [com.keepthinker.example.spring.lifecircle.postprocessor.MyInstantiationAwareBeanPostProcessor]: postProcessAfterInitialization|bean:com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver@7a36aefa|beanName:com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0
2022-03-12 23:20:06,283 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Finished creating instance of bean 'com.keepthinker.example.spring.lifecircle.BeanLifeCircleObserver#0'
2022-03-12 23:20:06,283 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
2022-03-12 23:20:06,296 [main] DEBUG [org.springframework.context.support.ClassPathXmlApplicationContext]: Unable to locate LifecycleProcessor with name 'lifecycleProcessor': using default [org.springframework.context.support.DefaultLifecycleProcessor@64d7f7e0]
2022-03-12 23:20:06,296 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'lifecycleProcessor'
2022-03-12 23:20:06,297 [main] DEBUG [org.springframework.core.env.PropertySourcesPropertyResolver]: Could not find key 'spring.liveBeansView.mbeanDomain' in any property source
2022-03-12 23:20:06,299 [main] DEBUG [org.springframework.beans.factory.support.DefaultListableBeanFactory]: Returning cached instance of singleton bean 'zookeeper'
```

## PostProcessor

### BeanFactoryPostProcessor

```java
public interface BeanFactoryPostProcessor {
   void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

实现该接口，可以在 Spring 创建 bean 之前修改 bean 的定义属性。也就是说，Spring 允许 BeanFactoryPostProcessor 在容器实例化 bean 之前读取配置元数据，并可以根据需要进行修改。例如可以把 bean 的 Scope 从 singleton 改为 prototype ，也可以把 **property 的值给修改掉**。另外可以同时配置多个 BeanFactoryPostProcessor，并通过 order 属性来控制 BeanFactoryPostProcessor 的执行顺序 （ 在实现 BeanFactoryPostProcessor 时应该考虑实现 Ordered 接口 ）。比如PropertyPlaceholderConfigurer就是在postProcessBeanFactory方法里实现property值的替换。

BeanFactoryPostProcessor 是在 Spring 容器加载了定义 bean 的 XML 文件之后，在 bean 实例化之前执行的，**也就是一定会在postProcessBeforeInstantiation之前执行**。接口方法的入参是 ConfigurrableListableBeanFactory 类型，使用该参数可以获取到相关的 bean 的定义信息。

使用BeanFactoryPostProcessor可以修改Spring Bean的全限定类名，scope，是否懒加载，所依赖的类名等。

[Spring扩展点之BeanFactoryPostProcessor：彻底搞懂原理以及使用场景【源码分析】_CoderOu-CSDN博客_beanfactorypostprocessor使用场景](https://blog.csdn.net/qq_42154259/article/details/108305938)

## 

### Bean post processors

Spring 框架提供了几种 PostProcessor接口，用于对容器或者bean进行后置处理，它们定义了一些方法，这些方法在特定的时机会被调用。通过这种机制，框架自身或者应用开发人员有机会在不侵入容器或者bean核心逻辑的情况下为容器或者bean做针对某些特定方面的定制或者扩展：能力增强，属性设置，内容修改，对象代理，甚至直接替换整个bean。Spring 提供的 PostProcessor 接口有如下几种 ：

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

### 例子

```java
@Configuration
@EnableAspectJAutoProxy
public class AspectJContext {

    @Bean
    LoggingHandler loggingHandler(){
        return new LoggingHandler();
    }    
    @Bean
    public StringService stringService(){
        return new StringService();
    }
}


@Aspect
public class LoggingHandler {

    @Pointcut("execution(* com.keepthinker.example.spring.aop.aspectj.StringService.*(..))")
    private void log(){}

    @Before("log()")
    public void beforeAdvice(JoinPoint joinpoint){
        System.out.println("beforeAdvice");
        String targetClassName=joinpoint.getTarget().getClass().getName();

        String signature=joinpoint.getSignature().toString();
        StringBuilder argsInfo = new StringBuilder();
        for(Object obj:joinpoint.getArgs()){
            argsInfo.append(obj).append(":").append(obj.getClass().getName()).append("  ");
        }
        Logger logger = Logger.getLogger(targetClassName);
        logger.info("get into via " + signature + (argsInfo.length() > 0 ? 
                (" with args: " + argsInfo) : ""));
    }

    @After("log()")
    public void afterAdvice(){
        System.out.println("afterAdvice");
    }

    @AfterReturning(pointcut = "log()", returning="retVal")
    public void afterReturningAdvice(Object retVal){
        System.out.println("afterReturningAdvice : Returning:" + retVal.toString() );
    }

    @AfterThrowing(pointcut = "log()", throwing = "ex")
    public void AfterThrowingAdvice(IllegalArgumentException ex){
        System.out.println("AfterThrowingAdvice: There has been an exception: " + ex.toString());   
    }
}

public class StringService {
    private String str = "string";

    public String getStr(String suffix) {

        return str + ":" + suffix;
    }
    public void setStr(String str) {
        this.str = str;
    }    
}

public class Main {
    public static void main(String[] args){
        AbstractApplicationContext context = new AnnotationConfigApplicationContext(AspectJContext.class);
        StringService service = context.getBean(StringService.class);

        System.out.println("main string:" + service.getStr("suffix"));

        context.registerShutdownHook();
        context.close();
    }
}


/**
输出如下：
beforeAdvice
get into via String com.keepthinker.example.spring.aop.aspectj.StringService.getStr(String) with args: suffix:java.lang.String  
execute getStr method with arg:suffix
afterAdvice
afterReturningAdvice : Returning:string:suffix
main string:string:suffix
*/
```

### AOP 核心概念

1. 切面（aspect） ： 类是对物体特征的抽象，切面就是对横切关注点的抽象。

2. 横切关注点： 对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点。  

3. 连接点（joinpoint） ： 被拦截到的点，因为 Spring 只支持方法类型的连接点，`所以在 Spring  中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器。`  

4. 切入点（pointcut） ： 对连接点进行拦截的定义。

5. 通知（advice） ： 所谓通知指的就是指拦截到连接点之后要执行的代码， 通知分为**前置**、**后置**、 **异常**、**最终**、**环绕**通知五类。

6. 目标对象： 代理的目标对象。

7. 织入（weave） ： 将切面应用到目标对象并导致代理对象创建的过程。

8. 引入（introduction） ： 在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段。

Spring 提供了两种方式来生成代理对象: JDKProxy 和 Cglib，具体使用哪种方式 生成由  AopProxyFactory 根据 AdvisedSupport 对象的配置来决定。 默认的策略是如果目标类是接口， 则使用 JDK 动态代理技术，否则使用 Cglib 来生成代理。

AOP实现的核心类AnnotationAwareAspectJAutoProxyCreator实现了BeanPostProcessor接口，当Spring加载了这个Bean时会在实例化前调用其postProcessAfterInitialization方法实现创建代理逻辑。

## Spring AOP的JDK与Cglib代理总结

如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP。

如果目标对象实现了接口，可以强制使用CGLIB实现AOP。

如果目标对象没有实现接口，必须采用CGLIB库，Spring会自动在JDK动态代理和CGLIB之间转换。

#### 如何强制使用CGLIB实现AOP

添加CGLIB库，Spring_HOME、cglib/*.jar

在Spring配置文件中加入<aop:aspectj-autoproxy proxy-target-class="true"/>

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

## Spring MVC

Spring 的模型-视图-控制器（MVC）框架是围绕一个 DispatcherServlet (继承javax.servlet.http.HttpServlet)来设计的，这个 Servlet  会把请求分发给各个处理器，并支持可配置的处理器映射、视图渲染、本地化、时区与主题渲染等，甚至还能支持文件上传。

![](C:\Users\Admin\AppData\Roaming\marktext\images\2022-02-14-22-39-25-image.png)

### Http 请求到 DispatcherServlet

    (1) 客户端请求提交到 DispatcherServlet。

### HandlerMapping 寻找处理器

    (2) 由 DispatcherServlet 控制器查询一个或多个 HandlerMapping，找到处理请求的Controller。

### 调用处理器 Controller

    (3) DispatcherServlet 将请求提交到 Controller。

### Controller 调用业务逻辑处理后，返回 ModelAndView

    (4)(5)调用业务处理和返回结果： Controller 调用业务逻辑处理后，返回 ModelAndView。

### DispatcherServlet 查询 ModelAndView

    (6)(7)处理视图映射并返回模型： DispatcherServlet 查询一个或多个 ViewResoler 视图解析器，找到 ModelAndView 指定的视图。

### ModelAndView 反馈浏览器 HTTP

    (8) Http 响应：视图负责将结果显示到客户端。

### MVC常用注解

![mvc-annotation](spring-mvc-annotation.png)

### web.xml配置

```xml
    <!--使用ContextLoaderListener配置时，需要告诉它Spring配置文件的位置-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring.xml,classpath:spring-mvc.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>wm-web-api</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
```

## Spring事务的传播机制

- **PROPAGATION_REQUIRED**：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，这也是通常我们的默认选择。
- **PROPAGATION_REQUIRES_NEW**：创建新事务，无论当前存不存在事务，都创建新事务。
- **PROPAGATION_NESTED**：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。
- **PROPAGATION_NOT_SUPPORTED**：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- **PROPAGATION_NEVER**：以非事务方式执行，如果当前存在事务，则抛出异常。
- **PROPAGATION_MANDATORY**：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
- **PROPAGATION_SUPPORTS**：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。

参考：https://zhuanlan.zhihu.com/p/368769721 

## ApplicationContext

```java
ClassPathXmlApplicationContext extends AbstractXmlApplicationContext 
extends AbstractRefreshableConfigApplicationContext 
extends AbstractRefreshableApplicationContext 
extends AbstractApplicationContext extends DefaultResourceLoader

/** 
管理spring上下文信息和整个生命周期。其中refresh方法完成spring上下文的初始化，bean的初始化等。
*/
AbstractXmlApplicationContext
    /** Unique id for this context, if any */
    id String
    beanFactoryPostProcessors List<BeanFactoryPostProcessor> 
    applicationListeners Set<ApplicationListener<?>>
    resourcePatternResolver ResourcePatternResolver using PathMatchingResourcePatternResolver
    environment StandardEnvironment 
    beanFactory DefaultListableBeanFactory
    loadBeanDefinitions(DefaultListableBeanFactory beanFactory)

    public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            //也包含执行initPropertySources，比如web程序获取web.xml配置进行处理。
            // Prepare this context for refreshing. 如果执行了environment.setRequiredProperties("my-required-config"); setRequiredProperties要求需要在jvm启动参数设置（java -Dmy-required-config=my-required-value）或者操作系统环境变量设置export my-required-config=my-required-value
            prepareRefresh();

            // Tell the subclass to refresh the internal bean factory. 解析xml配置文件，然后将bean definition放入一个map对象中。
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

            // Prepare the bean factory for use in this context.
            prepareBeanFactory(beanFactory);

            try {
                // Allows post-processing of the bean factory in context subclasses. 是一个protected方法，如果子类实现该方法将调用。
                postProcessBeanFactory(beanFactory);

                // Invoke factory processors registered as beans in the context. 比如某个Bean实现了BeanFactoryPostProcessor
                invokeBeanFactoryPostProcessors(beanFactory);

                // Register bean processors that intercept bean creation.
                registerBeanPostProcessors(beanFactory);

                // Initialize message source for this context.
                initMessageSource();

                // Initialize event multicaster for this context.
                initApplicationEventMulticaster();

                // Initialize other special beans in specific context subclasses.
                onRefresh();

                // Check for listener beans and register them.
                registerListeners();

                // Instantiate all remaining (non-lazy-init) singletons. 开始执行BeanPostProcessor的方法。
                finishBeanFactoryInitialization(beanFactory);

                // Last step: publish corresponding event.
                finishRefresh();
            }

            catch (BeansException ex) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
                }

                // Destroy already created singletons to avoid dangling resources.
                destroyBeans();

                // Reset 'active' flag.
                cancelRefresh(ex);

                // Propagate exception to caller.
                throw ex;
            }

            finally {
                // Reset common introspection caches in Spring's core, since we
                // might not ever need metadata for singleton beans anymore...
                resetCommonCaches();
            }
        }
    }
```

## BeanFactory

```java
/**
  保存bean定义，bean对象等数据。管理Bean对象生命周期。
*/
DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory 
extends AbstractBeanFactory extends FactoryBeanRegistrySupport 
extends DefaultSingletonBeanRegistry
    /** Cache of singleton objects: bean name --> bean instance 保存实际Bean对象映射*/
    singletonObjects Map<String, Object>
    /** Cache of early singleton objects: bean name --> bean instance */
    earlySingletonObjects Map<String, Object>
    /** Custom PropertyEditors to apply to the beans of this factory */
    customEditors Map<Class<?>, Class<? extends PropertyEditor>>
    /** ClassLoader to resolve bean class names with, if necessary */
     beanClassLoader ClassLoade
    /** BeanPostProcessors to apply in createBean */
     beanPostProcessorsr List<BeanPostProcessor>
    /** Strategy for creating bean instances CglibSubclassingInstantiationStrategy*/
     instantiationStrategy 
    /** Cache of unfinished FactoryBean instances: FactoryBean name --> BeanWrapper */
     factoryBeanInstanceCache ConcurrentMap<String, BeanWrapper>
    /** Map of bean definition objects, keyed by bean name */
     beanDefinitionMap Map<String, BeanDefinition>
     beanDefinitionNames List<String>
     /** Add the given singleton object to the singleton cache of this factory.*/
     addSingleton(String beanName, Object singletonObject)
    /**
     * Add the given bean to the list of disposable beans in this factory,
     * registering its DisposableBean interface and/or the given destroy method
     * to be called on factory shutdown (if applicable). Only applies to singletons.
     */
     registerDisposableBeanIfNecessary(String beanName, Object bean, RootBeanDefinition mbd)

     Object createBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)
         resolveBeforeInstantiation(beanName, mbd);
         /**     Actually create the specified bean. Pre-creation processing has already happened
         * at this point, e.g. checking {@code postProcessBeforeInstantiation} callbacks.
         */
         doCreateBean(beanName, mbd, args);
             populateBean(beanName, mbd, instanceWrapper);
            /**    Initialize the given bean instance, applying factory callbacks
             * as well as init methods and bean post processors. **/
             initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd)
                 invokeAwareMethods(beanName, bean);
                 applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
     populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw)
        postProcessAfterInstantiation(bw.getWrappedInstance(), beanName))


    private void invokeAwareMethods(final String beanName, final Object bean) {
        if (bean instanceof Aware) {
            if (bean instanceof BeanNameAware) {
                ((BeanNameAware) bean).setBeanName(beanName);
            }
            if (bean instanceof BeanClassLoaderAware) {
                ((BeanClassLoaderAware) bean).setBeanClassLoader(getBeanClassLoader());
            }
            if (bean instanceof BeanFactoryAware) {
                ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
            }
        }
    }

    public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
        processor.postProcessBeforeInitialization(result, beanName);
```
