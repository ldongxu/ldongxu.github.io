---
title: Java 集合细节（二）：asList 的缺陷
date: 2017-06-02 18:22:00
categories: Java
tags: Java
---
在实际开发过程中我们经常使用 asList 讲数组转换为 List，这个方法使用起来非常方便，但是 asList 方法存在几个缺陷：
<!--more-->
## 一、避免使用基本数据类型数组转换为列表
使用 8 个基本类型数组转换为列表时会存在一个比较有味的缺陷。先看如下程序：
```java
public static void main(String[] args) {
        int[] ints = {1,2,3,4,5};
        List list = Arrays.asList(ints);
        System.out.println("list'size：" + list.size());
}
```
        outPut：
        list'size：1
        
程序的运行结果并没有像我们预期的那样是 5 而是逆天的 1，这是什么情况？先看源码：
```java
public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
}
```
asList 接受的参数是一个泛型的变长参数，我们知道基本数据类型是无法发型化的，也就是说 8 个基本类型是无法作为 asList 的参数的， 要想作为泛型参数就必须使用其所对应的包装类型。但是这个这个实例中为什么没有出错呢？因为该实例是将 int 类型的数组当做其参数，而在 Java 中数组是一个对象，它是可以泛型化的。所以该例子是不会产生错误的。既然例子是将整个 int 类型的数组当做泛型参数，那么经过 asList 转换就只有一个 int 的列表了。如下：
```
public static void main(String[] args) {
    int[] ints = {1,2,3,4,5};
    List list = Arrays.asList(ints);
    System.out.println("list 的类型:" + list.get(0).getClass());
    System.out.println("list.get(0) == ints：" + list.get(0).equals(ints));
}
```
    outPut:
    list 的类型:class [I
    list.get(0) == ints：true
    
从这个运行结果我们可以充分证明 list 里面的元素就是 int 数组。弄清楚这点了，那么修改方法也就一目了然了：将 int 改变为 Integer。
```
public static void main(String[] args) {
        Integer[] ints = {1,2,3,4,5};
        List list = Arrays.asList(ints);
        System.out.println("list'size：" + list.size());
        System.out.println("list.get(0) 的类型:" +  list.get(0).getClass());
        System.out.println("list.get(0) == ints[0]：" + list.get(0).equals(ints[0]));
}
```

        outPut:
        list'size：5
        list.get(0) 的类型:class java.lang.Integer
        list.get(0) == ints[0]：true
        
**在使用 asList 时不要将基本数据类型当做参数。**

## 二、asList 产生的列表不可操作
对于上面的实例我们再做一个小小的修改：
```
public static void main(String[] args) {
        Integer[] ints = {1,2,3,4,5};
        List list = Arrays.asList(ints);
        list.add(6);
}
```
该实例就是讲 ints 通过 asList 转换为 list 类别，然后再通过 add 方法加一个元素，这个实例简单的不能再简单了，但是运行结果呢？打出我们所料：


    Exception in thread "main" java.lang.UnsupportedOperationException
        at java.util.AbstractList.add(Unknown Source)
        at java.util.AbstractList.add(Unknown Source)
        at com.chenssy.test.arrayList.AsListTest.main(AsListTest.java:10)
运行结果尽然抛出 UnsupportedOperationException 异常，该异常表示 list 不支持 add 方法。这就让我们郁闷了，list 怎么可能不支持 add 方法呢？难道 JDK 脑袋堵塞了？我们再看 asList 的源码：
```
public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
}
```
asList 接受参数后，直接 new 一个 ArrayList，到这里看应该是没有错误的啊？别急，再往下看:
```
private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable{
        private static final long serialVersionUID = -2764017481108945198L;
        private final E[] a;

        ArrayList(E[] array) {
            if (array==null)
                throw new NullPointerException();
            a = array;
        }
        //.................
}
```
这是 ArrayList 的源码,从这里我们可以看出,此 ArrayList 不是 java.util.ArrayList，他是 Arrays 的内部类。该内部类提供了 size、toArray、get、set、indexOf、contains 方法，而像 add、remove 等改变 list 结果的方法从 AbstractList 父类继承过来，同时这些方法也比较奇葩，它直接抛出 UnsupportedOperationException 异常：
```
public boolean add(E e) {
    add(size(), e);
    return true;
}

public E set(int index, E element) {
    throw new UnsupportedOperationException();
}

public void add(int index, E element) {
    throw new UnsupportedOperationException();
}

public E remove(int index) {
    throw new UnsupportedOperationException();
}       
```
通过这些代码可以看出 asList 返回的列表只不过是一个披着 list 的外衣，它并没有 list 的基本特性（变长）。该 list 是一个长度不可变的列表，传入参数的数组有多长，其返回的列表就只能是多长。所以：
**不要试图改变 asList 返回的列表，否则你会自食苦果。**

原文：[http://wiki.jikexueyuan.com/project/java-enhancement/java-thirtysix.html][1]


  [1]: http://wiki.jikexueyuan.com/project/java-enhancement/java-thirtysix.html