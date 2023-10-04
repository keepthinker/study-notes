# Spring Boot

## 标准starter模式开发

`@SpringBootApplication` is a convenience annotation that adds all of the following:

- `@Configuration`: Tags the class as a source of bean definitions for the application context.

- `@EnableAutoConfiguration`: Tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings. For example, if `spring-webmvc` is on the classpath, this annotation flags the application as a web application and activates key behaviors, such as setting up a `DispatcherServlet`.

- `@ComponentScan`: Tells Spring to look for other components, configurations, and services in the `com/example` package, letting it find the controllers.

## Customizing the Banner

The banner that is printed on start up can be changed by adding a `banner.txt` file to your classpath or by setting the `spring.banner.location` property to the location of such a file. If the file has an encoding other than UTF-8, you can set `spring.banner.charset`.

Inside your `banner.txt` file, you can use any key available in the `Environment`.

### Application Availability

When deployed on platforms, applications can provide information about their availability to the platform using infrastructure such as Kubernetes Probes. Spring Boot includes out-of-the box support for the commonly used “liveness” and “readiness” availability states. If you are using Spring Boot’s “actuator” support then these states are exposed as health endpoint groups.

## Application Events and Listeners

In addition to the usual Spring Framework events, such as [`ContextRefreshedEvent`](https://docs.spring.io/spring-framework/docs/6.0.11/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html), a `SpringApplication` sends some additional application events.

Some events are actually triggered before the `ApplicationContext` is created, so you cannot register a listener on those as a `@Bean`. You can register them with the `SpringApplication.addListeners(…​)` method or the `SpringApplicationBuilder.listeners(…​)` method.<br><br>If you want those listeners to be registered automatically, regardless of the way the application is created, you can add a `META-INF/spring.factories` file to your project and reference your listener(s) by using the `org.springframework.context.ApplicationListener` key, as shown in the following example: 

```bash
org.springframework.context.ApplicationListener=com.example.project.MyListener |
```

Application events are sent in the following order, as your application runs:

1. An `ApplicationStartingEvent` is sent at the start of a run but before any processing, except for the registration of listeners and initializers.

2. An `ApplicationEnvironmentPreparedEvent` is sent when the `Environment` to be used in the context is known but before the context is created.

3. An `ApplicationContextInitializedEvent` is sent when the `ApplicationContext` is prepared and ApplicationContextInitializers have been called but before any bean definitions are loaded.

4. An `ApplicationPreparedEvent` is sent just before the refresh is started but after bean definitions have been loaded.

5. An `ApplicationStartedEvent` is sent after the context has been refreshed but before any application and command-line runners have been called.

6. An `AvailabilityChangeEvent` is sent right after with `LivenessState.CORRECT` to indicate that the application is considered as live.

7. An `ApplicationReadyEvent` is sent after any application and command-line runners have been called.

8. An `AvailabilityChangeEvent` is sent right after with `ReadinessState.ACCEPTING_TRAFFIC` to indicate that the application is ready to service requests.

9. An `ApplicationFailedEvent` is sent if there is an exception on startup.

The above list only includes `SpringApplicationEvent`s that are tied to a `SpringApplication`. In addition to these, the following events are also published after `ApplicationPreparedEvent` and before `ApplicationStartedEvent`:

- A `WebServerInitializedEvent` is sent after the `WebServer` is ready. `ServletWebServerInitializedEvent` and `ReactiveWebServerInitializedEvent` are the servlet and reactive variants respectively.

- A `ContextRefreshedEvent` is sent when an `ApplicationContext` is refreshed.

```java
 c.k.e.s.listener.MyApplicationListener   : application event:org.springframework.boot.web.servlet.context.ServletWebServerInitializedEvent[source=org.springframework.boot.web.embedded.tomcat.TomcatWebServer@4833eff3]
 c.k.e.s.listener.MyApplicationListener   : application event:org.springframework.context.event.ContextRefreshedEvent[source=org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@4af0df05, started on Thu Sep 28 21:47:12 CST 2023]
 c.k.example.simple.SimpleApplication     : Started SimpleApplication in 1.009 seconds (process running for 1.384)
 c.k.e.s.listener.MyApplicationListener   : application event:org.springframework.boot.context.event.ApplicationStartedEvent[source=org.springframework.boot.SpringApplication@4f8d86e4]
 c.k.e.s.listener.MyApplicationListener   : application event:org.springframework.boot.availability.AvailabilityChangeEvent[source=org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@4af0df05, started on Thu Sep 28 21:47:12 CST 2023]
 -2 my command line runner
 1 my command line runner
 c.k.e.s.listener.MyApplicationListener   : application event:org.springframework.boot.context.event.ApplicationReadyEvent[source=org.springframework.boot.SpringApplication@4f8d86e4]
 c.k.e.s.listener.MyApplicationListener   : application event:org.springframework.boot.availability.AvailabilityChangeEvent[source=org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@4af0df05, started on Thu Sep 28 21:47:12 CST 2023]
```

## Using the ApplicationRunner or CommandLineRunner

If you need to run some specific code once the `SpringApplication` has started, you can implement the `ApplicationRunner` or `CommandLineRunner` interfaces. Both interfaces work in the same way and offer a single `run` method, which is called just before `SpringApplication.run(…​)` completes.

```java
@Component
public class MyCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {
        // Do something...
    }

}
```

## Accessing Application Arguments

If you need to access the application arguments that were passed to `SpringApplication.run(…​)`, you can inject a `org.springframework.boot.ApplicationArguments` bean. The `ApplicationArguments` interface provides access to both the raw `String[]` arguments as well as parsed `option` and `non-option` arguments, as shown in the following example:

```java
import java.util.List;

import org.springframework.boot.ApplicationArguments;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        if (debug) {
            System.out.println(files);
        }
        // if run with "--debug logfile.txt" prints ["logfile.txt"]
    }

}
```

## Application Exit

Each `SpringApplication` registers a shutdown hook with the JVM to ensure that the `ApplicationContext` closes gracefully on exit. All the standard Spring lifecycle callbacks (such as the `DisposableBean` interface or the `@PreDestroy` annotation) can be used.

## Externalized Configuration

Spring Boot lets you externalize your configuration so that you can work with the same application code in different environments. You can use a variety of external configuration sources including Java properties files, YAML files, environment variables, and command-line arguments.

Property values can be injected directly into your beans by using the `@Value` annotation, accessed through Spring’s `Environment` abstraction, or be [bound to structured objects](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties) through `@ConfigurationProperties`.

Spring Boot uses a very particular `PropertySource` order that is designed to allow sensible overriding of values. Later property sources can override the values defined in earlier ones. . Sources are considered in the following order:

1. Default properties (specified by setting `SpringApplication.setDefaultProperties`).

2. [`@PropertySource`](https://docs.spring.io/spring-framework/docs/6.0.12/javadoc-api/org/springframework/context/annotation/PropertySource.html) annotations on your `@Configuration` classes. Please note that such property sources are not added to the `Environment` until the application context is being refreshed. This is too late to configure certain properties such as `logging.*` and `spring.main.*` which are read before refresh begins.

3. Config data (such as `application.properties` files).

4. A `RandomValuePropertySource` that has properties only in `random.*`.

5. OS environment variables.

6. Java System properties (`System.getProperties()`).

7. JNDI attributes from `java:comp/env`.

8. `ServletContext` init parameters.

9. `ServletConfig` init parameters.

10. Properties from `SPRING_APPLICATION_JSON` (inline JSON embedded in an environment variable or system property).

11. Command line arguments.

12. `properties` attribute on your tests. Available on [`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/3.1.4/api/org/springframework/boot/test/context/SpringBootTest.html) and the [test annotations for testing a particular slice of your application](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.autoconfigured-tests).

13. [`@DynamicPropertySource`](https://docs.spring.io/spring-framework/docs/6.0.12/javadoc-api/org/springframework/test/context/DynamicPropertySource.html) annotations in your tests.

14. [`@TestPropertySource`](https://docs.spring.io/spring-framework/docs/6.0.12/javadoc-api/org/springframework/test/context/TestPropertySource.html) annotations on your tests.

15. [Devtools global settings properties](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools.globalsettings) in the `$HOME/.config/spring-boot` directory when devtools is active.

### Example

```bash
0 = {ConfigurationPropertySourcesPropertySource@3705} "ConfigurationPropertySourcesPropertySource@743778731 {name='configurationProperties', properties=org.springframework.boot.context.properties.source.SpringConfigurationPropertySources@57ea113a}"
1 = {SimpleCommandLinePropertySource@3706} "SimpleCommandLinePropertySource {name='commandLineArgs'}"
2 = {PropertySource$StubPropertySource@3707} "StubPropertySource {name='servletConfigInitParams'}"
3 = {PropertySource$StubPropertySource@3708} "StubPropertySource {name='servletContextInitParams'}"
4 = {PropertiesPropertySource@3709} "PropertiesPropertySource {name='systemProperties'}"
5 = {SystemEnvironmentPropertySourceEnvironmentPostProcessor$OriginAwareSystemEnvironmentPropertySource@3710} "OriginAwareSystemEnvironmentPropertySource@181252244 {name='systemEnvironment', properties={USERDOMAIN_ROAMINGPROFILE=keepthinker, PROCESSOR_LEVEL=6, SESSIONNAME=Console, ALLUSERSPROFILE=C:\ProgramData, PROCESSOR_ARCHITECTURE=AMD64, PSModulePath=C:\Program Files\WindowsPowerShell\Modules;C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules, SystemDrive=C:, iGame=D:\iGame\, MAVEN_HOME=D:\Program Files\apache-maven-3.8.6, ROCKETMQ_HOME=D:\Program Files\rocketmq-all-5.1.3-bin-release, MOZ_PLUGIN_PATH=C:\Program Files (x86)\Foxit Software\Foxit PDF Reader\plugins\, USERNAME=shengkai.ke, ProgramFiles(x86)=C:\Program Files (x86), FPS_BROWSER_USER_PROFILE_STRING=Default, PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC, DriverData=C:\Windows\System32\Drivers\DriverData, OneDriveConsumer=C:\Users\shengkai.ke\OneDrive, ProgramData=C:\ProgramData, ProgramW6432=C:\Program Files, HOMEPATH=\Users\shengkai.ke, PROCESSOR_IDENTIFIER=Intel64 Family 6 Model 154 Stepping 3, GenuineIntel, Progr"
6 = {RandomValuePropertySource@3711} "RandomValuePropertySource@1733022752 {name='random', properties=java.util.Random@2b0f373b}"
7 = {OriginTrackedMapPropertySource@3712} "OriginTrackedMapPropertySource@753631393 {name='Config resource 'file [application.yaml]' via location 'optional:file:./'', properties={pay.orderIds[0]=oi1024823, pay.orderIds[1]=oi20h9f28}}"
8 = {OriginTrackedMapPropertySource@3713} "OriginTrackedMapPropertySource@1262869688 {name='Config resource 'class path resource [application.yaml]' via location 'optional:classpath:/'', properties={logging.level.root=info, pay.merchantIds[0]=102380, pay.merchantIds[1]=234809, pay.merchantIds[2]=238400, pay.orderIds=or1304,or12309,or2301}}"
```

Config data files are considered in the following order:

1. [Application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files) packaged inside your jar (`application.properties` and YAML variants).

2. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific) packaged inside your jar (`application-{profile}.properties` and YAML variants).

3. [Application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files) outside of your packaged jar (`application.properties` and YAML variants).

4. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific) outside of your packaged jar (`application-{profile}.properties` and YAML variants).

加载顺序：

[SpringBoot2.x基础篇：配置文件的加载顺序以及优先级覆盖-阿里云开发者社区](https://developer.aliyun.com/article/1080100)

## Accessing Command Line Properties

By default, `SpringApplication` converts any command line option arguments (that is, arguments starting with `--`, such as `--server.port=9000`) to a `property` and adds them to the Spring `Environment`. As mentioned previously, **command line properties always take precedence over file-based property sources.**

## External Application Properties

Spring Boot will automatically find and load `application.properties` and `application.yaml` files from the following locations when your application starts:

1. From the classpath
   
   1. The classpath root
   
   2. The classpath `/config` package

2. From the current directory
   
   1. The current directory
   
   2. The `config/` subdirectory in the current directory
   
   3. Immediate child directories of the `config/` subdirectory

The list is ordered by precedence (with values from lower items overriding earlier ones). Documents from the loaded files are added as `PropertySources` to the Spring `Environment`.

### Profile Specific Files

As well as `application` property files, Spring Boot will also attempt to load profile-specific files using the naming convention `application-{profile}`. For example, if your application activates a profile named `prod` and uses YAML files, then both `application.yaml` and `application-prod.yaml` will be considered.

Profile-specific properties are loaded from the same locations as standard `application.properties`, with profile-specific files always overriding the non-specific ones. If several profiles are specified, a last-wins strategy applies. For example, if profiles `prod,live` are specified by the `spring.profiles.active` property, values in `application-prod.properties` can be overridden by those in `application-live.properties`.

### Importing Additional Data

Application properties may import further config data from other locations using the `spring.config.import` property. Imports are processed as they are discovered, and are treated as additional documents inserted immediately below the one that declares the import.

For example, you might have the following in your classpath `application.properties` file:

Properties

Yaml

```properties
spring.application.name=myapp
spring.config.import=optional:file:./dev.properties
```

This will trigger the import of a `dev.properties` file in current directory (if such a file exists). Values from the imported `dev.properties` will take precedence over the file that triggered the import. In the above example, the `dev.properties` could redefine `spring.application.name` to a different value.

### Property Placeholders

The values in `application.properties` and `application.yaml` are filtered through the existing `Environment` when they are used, so you can refer back to previously defined values (for example, from System properties or environment variables). The standard `${name}` property-placeholder syntax can be used anywhere within a value. Property placeholders can also specify a default value using a `:` to separate the default value from the property name, for example `${name:default}`.

The use of placeholders with and without defaults is shown in the following example:

Properties

Yaml

```properties
app.name=MyApp
app.description=${app.name} is a Spring Boot application written by ${username:Unknown}
```

Assuming that the `username` property has not been set elsewhere, `app.description` will have the value `MyApp is a Spring Boot application written by Unknown`.

### Configuring Random Values

The `RandomValuePropertySource` is useful for injecting random values (for example, into secrets or test cases). It can produce integers, longs, uuids, or strings, as shown in the following example:

Properties

Yaml

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number-less-than-ten=${random.int(10)}
my.number-in-range=${random.int[1024,65536]}
```

The `random.int*` syntax is `OPEN value (,max) CLOSE` where the `OPEN,CLOSE` are any character and `value,max` are integers. If `max` is provided, then `value` is the minimum value and `max` is the maximum value (exclusive).

#### JavaBean Properties Binding

It is possible to bind a bean declaring standard JavaBean properties as shown in the following example:

Java

Kotlin

```java
@ConfigurationProperties("my.service")
public class MyProperties {    private boolean enabled;    private InetAddress remoteAddress;    private final Security security = new Security();

    // getters / setters...

    public static class Security {        private String username;        private String password;        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

        // getters / setters...

    }}
```

The preceding POJO defines the following properties:

- `my.service.enabled`, with a value of `false` by default.

- `my.service.remote-address`, with a type that can be coerced from `String`.

- `my.service.security.username`, with a nested "security" object whose name is determined by the name of the property. In particular, the type is not used at all there and could have been `SecurityProperties`.

- `my.service.security.password`.

- `my.service.security.roles`, with a collection of `String` that defaults to `USER`.

### Enabling @ConfigurationProperties-annotated Types

Spring Boot provides infrastructure to bind `@ConfigurationProperties` types and register them as beans. You can either enable configuration properties on a class-by-class basis or enable configuration property scanning that works in a similar manner to component scanning.

Sometimes, classes annotated with `@ConfigurationProperties` might not be suitable for scanning, for example, if you’re developing your own auto-configuration or you want to enable them conditionally. In these cases, specify the list of types to process using the `@EnableConfigurationProperties` annotation. This can be done on any `@Configuration` class, as shown in the following example:

Java

Kotlin

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(SomeProperties.class)
public class MyConfiguration {}
```

Java

Kotlin

```java
@ConfigurationProperties("some.properties")
public class SomeProperties {}
```

To use configuration property scanning, add the `@ConfigurationPropertiesScan` annotation to your application. Typically, it is added to the main application class that is annotated with `@SpringBootApplication` but it can be added to any `@Configuration` class. By default, scanning will occur from the package of the class that declares the annotation. If you want to define specific packages to scan, you can do so as shown in the following example:

Java

Kotlin

```java
@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "com.example.another" })
public class MyApplication {}
```

#### Relaxed Binding

Spring Boot uses some relaxed rules for binding `Environment` properties to `@ConfigurationProperties` beans, so there does not need to be an exact match between the `Environment` property name and the bean property name. Common examples where this is useful include dash-separated environment properties (for example, `context-path` binds to `contextPath`), and capitalized environment properties (for example, `PORT` binds to `port`).

As an example, consider the following `@ConfigurationProperties` class:

```java
@ConfigurationProperties(prefix = "my.main-project.person")
public class MyPersonProperties {   
  private String firstName;    
  public String getFirstName() {        
     return this.firstName;    
  }    

  public void setFirstName(String firstName) {        
   this.firstName = firstName;    
  }
}
```

With the preceding code, the following properties names can all be used:

| Property                            | Note                                                                                         |
| ----------------------------------- | -------------------------------------------------------------------------------------------- |
| `my.main-project.person.first-name` | Kebab case, which is recommended for use in `.properties` and YAML files.                    |
| `my.main-project.person.firstName`  | Standard camel case syntax.                                                                  |
| `my.main-project.person.first_name` | Underscore notation, which is an alternative format for use in `.properties` and YAML files. |
| `MY_MAINPROJECT_PERSON_FIRSTNAME`   | Upper case format, which is recommended when using system environment variables.             |

| Property Source       | Simple                                                                                                                                                                                                                                                                        | List                                                                                                                                                                                                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Properties Files      | Camel case, kebab case, or underscore notation                                                                                                                                                                                                                                | Standard list syntax using `[ ]` or comma-separated values                                                                                                                                                                                                         |
| YAML Files            | Camel case, kebab case, or underscore notation                                                                                                                                                                                                                                | Standard YAML list syntax or comma-separated values                                                                                                                                                                                                                |
| Environment Variables | Upper case format with underscore as the delimiter (see [Binding From Environment Variables](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables)). | Numeric values surrounded by underscores (see [Binding From Environment Variables](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables)) |
| System properties     | Camel case, kebab case, or underscore notation                                                                                                                                                                                                                                | Standard list syntax using `[ ]` or comma-separated values                                                                                                                                                                                                         |

##### Binding Maps

When binding to `Map` properties you may need to use a special bracket notation so that the original `key` value is preserved. If the key is not surrounded by `[]`, any characters that are not alpha-numeric, `-` or `.` are removed.

For example, consider binding the following properties to a `Map<String,String>`:

Properties

Yaml

```properties
my.map.[/key1]=value1
my.map.[/key2]=value2
my.map./key3=value3
```

|     | For YAML files, the brackets need to be surrounded by quotes for the keys to be parsed properly. |
| --- | ------------------------------------------------------------------------------------------------ |

The properties above will bind to a `Map` with `/key1`, `/key2` and `key3` as the keys in the map. The slash has been removed from `key3` because it was not surrounded by square brackets.
