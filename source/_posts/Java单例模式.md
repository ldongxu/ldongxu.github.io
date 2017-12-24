---
title: Java单例模式
date: 2017-07-04 16:06:34
catagories: Java设计模式
tags: [单例模式,Java]
---
单例类可以是没有状态的，一般仅用做提供工具性函数的对象。既然是提供工具性函数，也就没有必要创建多个实例。
<!--more-->
## Singleton的几种实现方式：

#### 一、懒汉式（双重检验锁）

```java
public class Singleton{
	private volatile static Singleton uniqueInstance=null;
	private Singleton(){}
	public static Singleton getInstance(){
		if(uniqueInstance==null){
			synchronized(Singleton.class){
				if(uniqueInstance==null){
					uniqueInstance=new Singleton();
				}
			}
		}
		return uniqueInstance;
	}
	//other useful methods here
}
```
这种写法能够在多线程中很好的工作，而且看起来它也具备很好的lazy loading，但是，遗憾的是，效率很低，99%情况下不需要同步。

#### 二、饿汉式

```java
public class Singleton{
	private static Singleton uniqueInstance=new Singleton();
	private Singleton(){}
	public static Singleton getInstance(){
		return uniqueInstance;
	}
}
```
种方式基于classloder机制避免了多线程的同步问题，不过，instance在类装载时就实例化，虽然导致类装载的原因有很多种，在单例模式中大多数都是调用getInstance方法，但是也不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化instance显然没有达到lazy loading的效果。

#### 三、静态内部类（线程安全）

```java
public class Singleton {
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    private Singleton() {}

    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
这种方式是Singleton类被装载了，instance不一定被初始化。因为SingletonHolder类没有被主动使用，只有显示通过调用getInstance方法时，才会显式装载SingletonHolder类，实例化instance，从而达到lazy loading效果，同时根据JVM本身机制，静态内部类的加载能够保证线程安全。

#### 四、枚举法

以上方式存在两种创建新实例的风险，一是通过反射机制调用私有构造器，二是通过反序列化一个序列化的实例。

在《Effective Java》推荐了这样一个写法，它更加简洁，而且保证了线程安全。此方法无偿提供了序列化机制，绝对防止多次实例化，即使面对复杂的序列化或者反射攻击。单元素枚举类型已经成为实现Singleton的最佳方法。
```java
public enum Singleton {
    INSTANCE;
    
    public void execute(){...}
}
```

很多人会对枚举法实现的单例模式很不理解。这里需要深入理解的是两个点：

+ 枚举类实现其实省略了private类型的构造函数
+ 枚举类的域(field)其实是相应的enum类型的一个实例对象

对于第一点实际上enum内部是如下代码:
```
public enum Singleton {
    INSTANCE;
    // 这里隐藏了一个空的私有构造方法
    private Singleton () {}
}
```

>单元素的枚举类型实现Singleton.
>Java枚举类型背后的基本想法：通过公有的静态final域为每个枚举常量导出实例的类。因为没有可访问的构造器，枚举类型是真正的final。因为客户端既不能创建枚举类型实例，也不能对它进行扩张，因此很可能没有实例，而只有声明过的枚举常量。换句话说，枚举类型是实例受控的，是单例的泛型化，本质上是单元素的枚举
 -《Effective Java》
 
对于一个标准的enum单例模式，更好的写法还是实现接口的形式:
```
// 定义单例模式中需要完成的代码逻辑
public interface MySingleton {
    void doSomething();
}

public enum Singleton implements MySingleton {
    INSTANCE {
        @Override
        public void doSomething() {
            System.out.println("complete singleton");
        }
    };

    public static MySingleton getInstance() {
        return Singleton.INSTANCE;
    }
}
```