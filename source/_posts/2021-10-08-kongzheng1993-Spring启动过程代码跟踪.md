---
title: Spring启动过程源码跟踪！
excerpt: ''
tags: [Spring]
categories: [Spring]
comments: true
date: 2020-10-08 00:30:52
---


    跟踪Spring源码org.springframework.context.support.ClassPathXmlApplicationContextTests#testSingleConfigLocation记录Spring容器启动过程，整个过程全部在ClassPathXmlApplicationContext的构造方法中
## 1. 根据传入的xml配置文件创建ApplicationContext
测试用例中传入的配置文件为/org/springframework/context/support/simpleContext.xml。org.springframework.context.support.AbstractRefreshableConfigApplicationContext#setConfigLocations方法中会有配置文件的判空和文件路径格式的调整，以方便后面读取文件。
## 2. 容器初始化 refresh()
### 1. prepareRefresh()
容器刷新前的准备，设置上下文状态，获取属性，验证必要的属性等
### 2. obtainFreshBeanFactory()
获取新的beanFactory，销毁原有beanFactory、为每个bean生成BeanDefinition等
BeanFactory，顾名思义Bean工厂，用于实例化和保存对象。
FactoryBean，是一个Bean，用于实例化创建过程比较复杂的对象。使用工厂方法模式，由一个特定的工厂来生产特定的java类的实例化对象。
ObjectFactory，是某个特定的工厂，用于在项目启动时，延迟实例化对象，解决循环依赖的问题。
public interface FactoryBean<T> { 
    //返回的对象实例 
    T getObject() throws Exception; 
    //Bean的类型 
    Class<?> getObjectType(); 
    //true是单例，false是非单例 在Spring5.0中此方法利用了JDK1.8的新特性变成了default方法，返回true 
    boolean isSingleton(); 
}

FactoryBean的好处：正常情况下，Spring在实例化对象的时候，都是由BeanFactory从上下文获取BeanDefinition信息，然后通过反射，调用java类的构造方法进行实例化，而通过FactoryBean的形式，相当于将实例化的功能交给了这个类对应的FactoryBean来实现，可以更加灵活的去做一些解析、判断和逻辑处理。
Spring中由两种Bean，一种是普通的Bean，另一种是实现了FactoryBean的工厂Bean。如果共BeanFactory中getBean的时候，获取到的Bean是工厂Bean，会自动调用这个工厂Bean的getObject方法返回真实的实例化对象。如果就是要获取工厂Bean对象，需要在getBean的时候加上前缀'&'。
ObjectFactory只有一个方法getObject()，可以借助Scope接口来自定义scope控制对象的创建时机。
执行过程：
  - 创建DefaultListableBeanFactory
  - 载入beanDefinition
    - 获取spring相关的xml文件，并转换为Document对象
    - 解析Document，判断node是否属于beans(http://www.springframework.org/schema/beans)这个命名空间（Namespace）。针对默认命名空间(beans）和非默认空间，有不同的处理。
        - 处理默认命名空间相关node:import、alias、bean、beans4种node。
        - 处理非默认命名空间相关node，如：<context:annotation-config/>、<context:component-scan base-package="xx"/>、<mongo:mongo-client/>
    - 注册beanDefinition
### 3. prepareBeanFactory(beanFactory)
配置标准的beanFactory，设置ClassLoader，设置SpEL表达式解析器，添加忽略注入的接口，添加bean，添加bean后置处理器等
执行过程：
  - 设置类加载器
  - 设置EL表达式解析器（Bean初始化完成后填充属性时会用到）
  - 利用BeanPostProcessor的特性给各种Aware接口的实现类注入ApplicationContext中对应的属性
  - 设置各种Aware接口的实现类为忽略自动配置ignoreDependencyInterface
  - 设置自动装配的类registerResolvableDependency（BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext）
  - 如果BeanFactory中存在loadTimeWeaver的bean，那么需要添加动态织入功能
  - 注册各种可用组件（environment、systemProperties、systemEnvironment）
### 4. postProcessBeanFactory(beanFactory)
模板方法，此时，所有的beanDefinition已经加载，但是还没有实例化。允许在子类中对beanFactory进行扩展处理。比如添加ware相关接口自动装配设置，添加后置处理器等，是子类扩展prepareBeanFactory(beanFactory)的方法
### 5. invokeBeanFactoryPostProcessors(beanFactory)
实例化并调用所有注册的beanFactory后置处理器（实现接口BeanFactoryPostProcessor的bean，在beanFactory标准初始化之后执行）。例如: PropertyPlaceholderConfigurer(处理占位符)。BeanFactoryPostProcessor 接口是 Spring 初始化 BeanFactory 时对外暴露的扩展点，Spring IoC 容器允许 BeanFactoryPostProcessor 在容器实例化任何 bean 之前读取 bean 的定义，并可以修改它。
BeanDefinitionRegistryPostProcessor 继承自 BeanFactoryPostProcessor，比 BeanFactoryPostProcessor 具有更高的优先级，主要用来在常规的 BeanFactoryPostProcessor 检测开始之前注册其他 bean 定义。特别是，你可以通过 BeanDefinitionRegistryPostProcessor 来注册一些常规的 BeanFactoryPostProcessor，因为此时所有常规的 BeanFactoryPostProcessor 都还没开始被处理。
执行过程：
  - 拿到当前应用上下文的beanFactoryPostProcessors
  - 实例化并调用所有已注册的BeanFactoryPostProcessor
  - 
### 6. registerBeanPostProcessors(beanFactory)
实例化和注册beanFactory中扩展了BeanPostProcessor的bean
例如：
 AutowiredAnnotationBeanPostProcessor(处理被@Autowired注解修饰的bean并注入)
 RequiredAnnotationBeanPostProcessor(处理被@Required注解修饰的方法)
 CommonAnnotationBeanPostProcessor(处理@PreDestroy、@PostConstruct、@Resource等多个注解的作用)等。
### 7. initMessageSource()
初始化国际化工具类MessageSource。MessageSource定义的Bean名字必须是 messageSource，如果找不到就会默认注册DelegatingMessageSource 作为messageSource的Bean。
### 8. initApplicationEventMulticaster()
初始化事件广播器。先判断有没有applicationEventMulticaster的bean，有的话就使用，没有的话就new一个SimpleApplicationEventMulticaster
### 9. onRefresh()
模板方法，在容器刷新的时候可以自定义逻辑，不同的Spring容器做不同的事情。
### 10. registerListeners()
注册监听器，如果有earlyEventsToProcess广播earlyEvent
### 11. finishBeanFactoryInitialization(beanFactory)
实例化所有剩余的（非懒加载）单例
比如invokeBeanFactoryPostProcessors方法中根据各种注解解析出来的类，在这个时候都会被初始化。实例化的过程各种BeanPostProcessor开始起作用。
### 12. finishRefresh()
refresh做完之后需要做的其他事情。
  - 清除上下文资源缓存（如扫描中的ASM元数据）
  - 初始化上下文的生命周期处理器
  - 刷新Spring容器中实现了Lifecycle接口的bean并执行start()方法
  - 发布ContextRefreshedEvent事件告知对应的ApplicationListener进行响应的操作
