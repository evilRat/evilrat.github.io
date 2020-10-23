---
title: SpringBoot自定义starter
excerpt: 'SpringBoot'
tags: [SpringBoot]
categories: [SpringBoot]
comments: true
date: 2020-10-23 00:30:52
---

## 什么是SpringBoot starter

Starter可以理解为一个可拔插式的插件，提供一系列便利的依赖描述符，您可以获得所需的所有Spring和相关技术的一站式服务。应用程序只需要在maven中引入starter依赖，SpringBoot就能自动扫描到要加载的信息并启动相应的默认配置。用一句话描述，就是springboot的场景启动器。



## 为什么starter可以热插拔

一个Springboot应用都会有一个`@SpringBootApplication`注解。而`@SpringBootApplication`注解里其实包含三个注解:

- @SpringBootConfiguration 与`@Configuration`作用相同，生命当前类是个配置类
- @EnableAutoConfiguration `@EnableAutoConfiguration`是springboot实现自动化配置的核心注解，通过这个注解把spring应用所需的bean注入容器中。`@EnableAutoConfiguration`源码通过`@Import`注入了一个`ImportSelector`的实现类`AutoConfigurationImportSelector`,这个`ImportSelector`最终实现根据我们的配置，动态加载所需的bean.
- @ComponentScan   扫描各种Component（service，controller，component，repository），默认扫描本类所在包和子包中都类，所以一般会把springboot启动类放在项目源码的根目录。



```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```


```java
@Override
//annotationMetadata 是＠import所用在的注解．这里指定是@EnableAutoConfiguration
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
     //加载XXConfiguration的元数据信息（包含了某些类被生成bean条件），继续跟进这个方法调用，就会发现加载的是：spring-boot-autoconfigure　jar包里面META-INF的spring-autoconfigure-metadata.properties
    AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
            .loadMetadata(this.beanClassLoader);
　　    //获取注解里设置的属性，在＠SpringBootApplication设置的exclude,excludeＮame属性值，其实就是设置＠EnableAutoConfiguration的这两个属性值
       AnnotationAttributes attributes = getAttributes(annotationMetadata);
       //从spring-boot-autoconfigure　jar包里面META-INF/spring.factories加载配置类的名称，打开这个文件发现里面包含了springboot框架提供的所有配置类
    List<String> configurations = getCandidateConfigurations(annotationMetadata,
            attributes);
     //去掉重复项
    configurations = removeDuplicates(configurations);
     //获取自己配置不需要生成bean的class
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    //校验被exclude的类是否都是springboot自动化配置里的类，如果存在抛出异常
    checkExcludedClasses(configurations, exclusions);
   //删除被exclude掉的类
    configurations.removeAll(exclusions);
   //过滤刷选，满足OnClassCondition的类
    configurations = filter(configurations, autoConfigurationMetadata);
    fireAutoConfigurationImportEvents(configurations, exclusions);
//返回需要注入的bean的类路径
    return StringUtils.toStringArray(configurations);
}
```


SpringBoot 在启动时会去依赖的 starter 包中寻找 /META-INF/spring.factories 文件，然后根据文件中配置的路径去扫描项目所依赖的 Jar 包，这类似于 Java 的 SPI 机制。


JavaSPI 实际上是“基于接口的编程＋策略模式＋配置文件”组合实现的动态加载机制。


## 步骤

1. 新建工程

一般starter的命名格式为：spring-boot-starter-hello

2. 新建HelloProperties类，定义一个hello.msg参数（默认值World！）。

```java
@ConfigurationProperties(prefix = "hello")
public class HelloProperties {
    /**
     * 打招呼的内容，默认为“World!”
     */
    private String msg = "World!";
 
    public String getMsg() {
        return msg;
    }
 
    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

3. 编写功能（HelloService）

```java
@Service
public class HelloService {
    @Autowired
    private HelloProperties helloProperties;
 
    /**
     * 打招呼方法
     *
     * @param name 人名，向谁打招呼使用
     * @return
     */
    public String sayHello(String name) {
        return "Hello " + name + " " + helloProperties.getMsg();
    }
```

4. 编写自动配置类

```java
//定义为配置类
@Configuration
//在web工程条件下成立
@ConditionalOnWebApplication
//启用HelloProperties配置功能，并加入到IOC容器中
@EnableConfigurationProperties({HelloProperties.class})
//导入HelloService组件
@Import(HelloService.class)
//@ComponentScan
public class HelloAutoConfiguration {
}
```

5. 在resources目录下新建META-INF目录，并在META-INF下新建spring.factories文件，写入：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.springbootstarterhello.HelloAutoConfiguration
```

6. 收尾工作

    1. 删除自动生成的启动类SpringBootStarterHelloApplication。
    2. 删除resources下的除META-INF目录之外的所有文件目录。
    3. 删除spring-boot-starter-test依赖并且删除test目录。

6. 到目前为止，spring-boot-starter-hello的自动配置功能已实现，并且正确使用了，但还有一点不够完美，如果你也按上面步骤实现了自己的spring-boot-starter-hello自动配置，在application.properties中配置hello.msg属性时，你会发现并没有提示你有关该配置的信息，但是如果你想配置tomcat端口时，输入server.port是有提示的。
新建META-INF/spring-configuration-metadata.json文件，进行配置。

```json
{
  "hints":[{
    "name":"hello.msg",
    "values":[{
      "value":"你好",
      "description":"中文方式打招呼"
    },{
      "value":"Hi",
      "description":"英文方式打招呼"
    }]
  }],
  "groups":[
    {
      "sourceType": "com.seagetech.spring.boot.helloworld.HelloWorldProperties",
      "name": "hello",
      "type": "com.example.springbootstarterhello.HelloProperties"
    }],
  "properties":[
    {
      "sourceType": "com.example.springbootstarterhello.HelloProperties",
      "name": "hello.msg",
      "type": "java.lang.String",
      "description": "打招呼的内容",
      "defaultValue": "Worlds"
    }]
}
```


7. 执行`mvn install`将spring-boot-starter-hello安装到本地


至此，我们的starter就可以使用了。


