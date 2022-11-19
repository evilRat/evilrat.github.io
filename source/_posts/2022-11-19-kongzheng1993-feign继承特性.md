---
title: Spring Cloud Feign继承特性
excerpt: 'feign'
tags: [feign]
categories: [feign]
comments: true
date: 2022-11-19 10:30:52
---

由于FeignClient的编写和其对应的服务提供方Controller及其相似，很多情况我们都是找服务提供方要过来Controller改造一下，其实我们可以利用其继承特性来减少代码的复制操作。

以下一个demo来演示一下：

## 1. 制定可复用的dto和接口定义
创建一个工程user-service-api，里面包含同时可复用于服务端和客户端的接口和dto。
<img src="1.png">

```java
package com.evil.user.dto;


public class UserDto {

    private String userName;

    private Integer age;

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

```

```java
package com.evil.user.service;

import com.evil.user.dto.UserDto;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

@RequestMapping("/user")
public interface UserService {

    @RequestMapping("get/{id}")
    UserDto getById(@PathVariable("id") Integer id);

}

```
定义好了dto和接口，打包user-service-api给服务提供者和消费者公用。

## 2. 服务提供者使用

服务提供者引入依赖
```xml
    <dependency>
      <groupId>com.evil</groupId>
      <artifactId>user-service-api</artifactId>
      <version>1.0.0</version>
    </dependency>
```

controller直接实现一把

```java

package com.evil.user.controller;

import com.evil.user.dto.UserDto;
import com.evil.user.service.UserService;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController implements UserService {

    public UserDto getById(Integer integer) {
        UserDto userDto = new UserDto();
        userDto.setUserName("小明");
        userDto.setAge(28);
        return userDto;
    }
}

```

启动后测试。

<img src="2.png">

## 3. 服务消费方使用

一样的，服务消费者引入依赖，另外还要引入feign的依赖

```xml
    <dependency>
      <groupId>com.evil</groupId>
      <artifactId>user-service-api</artifactId>
      <version>1.0.0</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-feign</artifactId>
      <version>1.4.7.RELEASE</version>
    </dependency>
```

FienClient直接继承api包里的UserService

```java
package com.evil.feign;

import com.evil.user.dto.UserDto;
import com.evil.user.service.UserService;
import org.springframework.cloud.netflix.feign.FeignClient;

@FeignClient
public interface UserFeign extends UserService {

}
```

## 总结：

同一个接口定义，服务提供者实现UserService，增加@RestController注解即可；服务消费者只需要继承UserService，增加@FeignClient，即可完成对服务提供者的调用。
