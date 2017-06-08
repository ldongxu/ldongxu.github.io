---
title: Java中System.getProperties()和System.getenv()的区别
date: 2017-06-08 14:36:09
categories: Java
tags: Java
---
System.getenv()是获取系统环境变量，System.getProperty()是获取当前系统相关属性信息。
<!--more-->
## System.getenv()方法
JDK中的说明：
>Returns an unmodifiable string map view of the current system environment.
The environment is a system-dependent mapping from names to values which is passed from parent to child processes.

>返回当前系统环境的字符串Map。
环境是名称对从父进程传递到子进程的值的系统相关映射。

等于在Linux系统下执行：`evn`命令。

    xxxMacBook-Air:~ xxx$ env
    TERM_PROGRAM=Apple_Terminal
    SHELL=/bin/bash
    TERM=xterm-256color
    TMPDIR=/var/folders/4l/lmg0v7813rl7k731njjt3tlh0000gn/T/
    Apple_PubSub_Socket_Render=/private/tmp/com.apple.launchd.8uZ86VHyIp/Render
    TERM_PROGRAM_VERSION=388.1
    TERM_SESSION_ID=2521885E-908A-4F63-8096-A42727205AA8
    USER=ldongxu
    SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.BVRB0zu2xV/Listeners
    __CF_USER_TEXT_ENCODING=0x1F5:0x19:0x34
    PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/apache-maven-3.5.0/bin:/usr/local/zookeeper-3.4.10/bin
    JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/
    LANG=zh_CN.UTF-8
    XPC_FLAGS=0x0
    XPC_SERVICE_NAME=0
    M2_HOME=/usr/local/apache-maven-3.5.0
    SHLVL=1
    HOME=/Users/xxx
    Z_HOME=/usr/local/zookeeper-3.4.10
    LOGNAME=xxx
    DISPLAY=/private/tmp/com.apple.launchd.QX7X8x0y77/org.macosforge.xquartz:0
在windows中‘系统属性’->‘高级’->‘环境变量’；
 
## System.getProperty()方法
获取系统相关属性，比如：Java版本、操作系统信息、用户名等，这些跟jvm和操作系统相关属性。
JDK中说明：
>Determines the current system properties.

它可获取的一些属性如下：

| key     |  说明  |
| -------- | :----  |
| java.version    |   Java 运行时环境版本    |
| java.vendor      |   Java 运行时环境供应商   |
| java.vendor.url  |  Java 供应商的 URL  |
| java.home      |  Java 安装目录  |
| java.vm.specification.version  |  Java 虚拟机规范版本  |
| java.vm.specification.vendor |  Java 虚拟机规范供应商  |
| java.vm.specification.name   |  Java 虚拟机规范名称 |
| java.vm.version      |  Java 虚拟机实现版本  |
| java.vm.vendor      |  Java 虚拟机实现供应商  |
| java.vm.name      |  Java 虚拟机实现名称  |
| java.specification.version      |  Java 运行时环境规范版本  |
| java.specification.vendor    |  Java 运行时环境规范供应商  |
| java.specification.name   |  Java 运行时环境规范名称 |
| java.class.version     |  Java 类格式版本号  |
| java.class.path    |  Java 类路径  |
| java.library.path    |  加载库时搜索的路径列表  |
| java.io.tmpdir    |  默认的临时文件路径  |
| java.compiler    |  要使用的 JIT 编译器的名称  |
| java.ext.dirs    |  一个或多个扩展目录的路径  |
| os.name    | 操作系统的名称  |
| os.arch    | 操作系统的架构  |
| file.separator  | 文件分隔符（在 UNIX 系统中是“/”）  |
| path.separator    | 路径分隔符（在 UNIX 系统中是“:”）  |
| line.separator   | 行分隔符（在 UNIX 系统中是“/n”）  |
| user.name   | 用户的账户名称 |
| user.home   | 用户的主目录  |
| user.dir  | 用户的当前工作目录 |
