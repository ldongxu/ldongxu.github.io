---
title: Java 集合细节（一）：请为集合指定初始容量
date: 2017-06-02 18:12:28
categories: Java
tags: Java
---
集合是我们在 Java 编程中使用非常广泛的，它就像大海，海纳百川，像万能容器，盛装万物，而且这个大海，万能容器还可以无限变大（如果条件允许）。当这个海、容器的量变得非常大的时候，它的初始容量就会显得很重要了，因为挖海、扩容是需要消耗大量的人力物力财力的。同样的道理，Collection 的初始容量也显得异常重要。所以：**对于已知的情景，请为集合指定初始容量。**
<!--more-->
```java
public static void main(String[] args) {
    StudentVO student = null;
    long begin1 = System.currentTimeMillis();
    List<StudentVO> list1 = new ArrayList<>();
    for(int i = 0 ; i < 1000000; i++){
        student = new StudentVO(i,"chenssy_"+i,i);
        list1.add(student);
    }
    long end1 = System.currentTimeMillis();
    System.out.println("list1 time：" + (end1 - begin1));

    long begin2 = System.currentTimeMillis();
    List<StudentVO> list2 = new ArrayList<>(1000000);
    for(int i = 0 ; i < 1000000; i++){
        student = new StudentVO(i,"chenssy_"+i,i);
        list2.add(student);
    }
    long end2 = System.currentTimeMillis();
    System.out.println("list2 time：" + (end2 - begin2));
}
```
上面代码两个 list 都是插入 1000000 条数据，只不过 list1 没有没有申请初始化容量，而 list2 初始化容量 1000000。那运行结果如下：

    list1 time：1638
    list2 time：921
    
从上面的运行结果我们可以看出 list2 的速度是 list1 的两倍左右。在前面 LZ 就提过，ArrayList 的扩容机制是比较消耗资源的。我们先看 ArrayList 的 add 方法：
```java
public boolean add(E e) {  
    ensureCapacity(size + 1);   
    elementData[size++] = e;  
    return true;  
}  

public void ensureCapacity(int minCapacity) {  
    modCount++;         //修改计数器
    int oldCapacity = elementData.length;    
    //当前需要的长度超过了数组长度，进行扩容处理
    if (minCapacity > oldCapacity) {  
        Object oldData[] = elementData;  
        //新的容量 = 旧容量 * 1.5 + 1
        int newCapacity = (oldCapacity * 3)/2 + 1;  
            if (newCapacity < minCapacity)  
                newCapacity = minCapacity;  
      //数组拷贝，生成新的数组 
    elementData = Arrays.copyOf(elementData,   newCapacity);  
    }  
}
```
ArrayList 每次新增一个元素，就会检测 ArrayList 的当前容量是否已经到达临界点，如果到达临界点则会扩容 1.5 倍。然而 ArrayList 的扩容以及数组的拷贝生成新的数组是相当耗资源的。所以若我们事先已知集合的使用场景，知道集合的大概范围，我们最好是指定初始化容量，这样对资源的利用会更加好，尤其是大数据量的前提下，效率的提升和资源的利用会显得更加具有优势。

原文：[http://wiki.jikexueyuan.com/project/java-enhancement/java-thirtyfive.html][1]


  [1]: http://wiki.jikexueyuan.com/project/java-enhancement/java-thirtyfive.html
