---
title: 静态工具类获取Spring IoC管理的bean组件
date: 2017-04-25 14:05:03
categories: Java
tags: [Java,Spring]
---

一般情况我们通过@Resource或者@Autowired或者applicationContext.xml配置来注入所需的bean组件。
但是，有的时候我们在一些静态工具类中或者组件里要通过名称去获得spring管理的bean。
<!--more-->
### 1、实现ApplicationContextAware接口
```java
package com.ldongxu.common;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class SpringUtil implements ApplicationContextAware {
    private static ApplicationContext applicationContext = null;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if (SpringUtil.applicationContext == null) {
            SpringUtil.applicationContext = applicationContext;
            System.out.println("========ApplicationContext配置成功,在普通类中可以通过调用SpringUtil.getApplicationContext()获取applicationContext对象,applicationContext=" + applicationContext + "========");
        }

    }

    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    public static Object getBean(String name) {
        return applicationContext.getBean(name);
    }
}

```
### 2、在Spring的配置文件中声明添加此工具类
```
<bean class="com.ldongxu.common.SpringUtil"></bean>
```
>注意：springUtil只能获取到和它定义在同一个上下文里的bean，换句话说，如果想用springUtil获取某个bean，那么这个bean必须和springUtil同在一个上下文中，即声明放在同一个xml文件里。
