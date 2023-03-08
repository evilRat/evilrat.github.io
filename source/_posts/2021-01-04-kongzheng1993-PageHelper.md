---
title: PageHelper
excerpt: '分页'
tags: [分页]
categories: [分页]
comments: true
date: 2020-12-29 10:30:52
---

## 问题

最近遇到了两次PageHelper分页失效的问题，第一次没记住，幸好当时跟同事讲了这个问题，第二次遇到问了他一下，才想起来。其实就是之前有分页查询，后面修改的时候在`PageHelper.startPage(pageRequest.getPageIndex(), pageRequest.getPageSize());`和执行sql之间，加入了其他的数据库查询操作。导致我们前面设置的分页信息，在第一次查询是使用并清空了，第二次真正想使用的时候就没有了。<strong>这就要求我们在真正想使用分页的查询前，设置分页信息，中间不能有其他的数据库操作。</strong>

## 原理

其实PageHelper的原理也简单，就是PageHelper有一个ThreadLocal对象`protected static final ThreadLocal<Page> LOCAL_PAGE = new ThreadLocal();`，每个请求过来的时候，我们手动去增加当前线程的分页信息，然后PageHelper的拦截器会拦截mybatis的query方法，在查询前count一下，然后将分页信息追加到我们到插叙sql中，返回分页数据，然后remove掉threadlocal里的分页信息。

## 使用方法

1. 引入分页插件

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>最新版本</version>
</dependency>
```

2. 配置拦截器插件

    两种方式

    - 在 MyBatis 配置 xml 中配置拦截器插件

        ```xml
        <!-- 
            plugins在配置文件中的位置必须符合要求，否则会报错，顺序如下:
            properties?, settings?, 
            typeAliases?, typeHandlers?, 
            objectFactory?,objectWrapperFactory?, 
            plugins?, 
            environments?, databaseIdProvider?, mappers?
        -->
        <plugins>
            <!-- com.github.pagehelper为PageHelper类所在包名 -->
            <plugin interceptor="com.github.pagehelper.PageInterceptor">
                <!-- 使用下面的方式配置参数，后面会有所有的参数介绍 -->
                <property name="param1" value="value1"/>
            </plugin>
        </plugins>
        ```
    - 在 Spring 配置文件中配置拦截器插件

        ```xml
        <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 注意其他配置 -->
        <property name="plugins">
            <array>
            <bean class="com.github.pagehelper.PageInterceptor">
                <property name="properties">
                <!--使用下面的方式配置参数，一行配置一个 -->
                <value>
                    params=value1
                </value>
                </property>
            </bean>
            </array>
        </property>
        </bean>
        ```

3. 在业务代码中使用

```java
//第一种，RowBounds方式的调用
List<User> list = sqlSession.selectList("x.y.selectIf", null, new RowBounds(0, 10));

//第二种，Mapper接口方式的调用，推荐这种使用方式。
PageHelper.startPage(1, 10);
List<User> list = userMapper.selectIf(1);

//第三种，Mapper接口方式的调用，推荐这种使用方式。
PageHelper.offsetPage(1, 10);
List<User> list = userMapper.selectIf(1);

//第四种，参数方法调用
//存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
public interface CountryMapper {
    List<User> selectByPageNumSize(
            @Param("user") User user,
            @Param("pageNum") int pageNum, 
            @Param("pageSize") int pageSize);
}
//配置supportMethodsArguments=true
//在代码中直接调用：
List<User> list = userMapper.selectByPageNumSize(user, 1, 10);

//第五种，参数对象
//如果 pageNum 和 pageSize 存在于 User 对象中，只要参数有值，也会被分页
//有如下 User 对象
public class User {
    //其他fields
    //下面两个参数名和 params 配置的名字一致
    private Integer pageNum;
    private Integer pageSize;
}
//存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
public interface CountryMapper {
    List<User> selectByPageNumSize(User user);
}
//当 user 中的 pageNum!= null && pageSize!= null 时，会自动分页
List<User> list = userMapper.selectByPageNumSize(user);

//第六种，ISelect 接口方式
//jdk6,7用法，创建接口
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectGroupBy();
    }
});
//jdk8 lambda用法
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(()-> userMapper.selectGroupBy());

//也可以直接返回PageInfo，注意doSelectPageInfo方法和doSelectPage
pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectGroupBy();
    }
});
//对应的lambda用法
pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(() -> userMapper.selectGroupBy());

//count查询，返回一个查询语句的count数
long total = PageHelper.count(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectLike(user);
    }
});
//lambda
total = PageHelper.count(()->userMapper.selectLike(user));
```