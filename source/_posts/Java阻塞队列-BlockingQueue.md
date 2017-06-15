---
title: Java阻塞队列-BlockingQueue
date: 2017-06-15 12:30:24
catagories: Java
tags: [Java队列,Queue,Java]
---
队列是一种数据结构。它有两个基本操作：在队列尾部加入一个元素和从队列头部移除一个元素。就是说，队列以一种先进先出的方式管理数据（FIFO (first-in-first-out)），如果你试图向一个已经满了的阻塞队列中添加一个元素或者是从一个空的阻塞队列中移除一个元索，将导致线程阻塞。在多线程进行合作时，阻塞队列是很有用的工具。工作者线程可以定期地把中间结果存到阻塞队列中而其他工作者线线程把中间结果取出并在将来修改它们。队列会自动平衡负载。如果第一个线程集运行得比第二个慢，则第二个线程集在等待结果时就会阻塞。如果第一个线程集运行得快，那么它将等待第二个线程集赶上来。
<!--more-->
Queue接口与List、Set同一级别，都是继承了Collection接口。LinkedList实现了Queue接 口。Queue接口窄化了对LinkedList的方法的访问权限（即在方法中的参数类型如果是Queue时，就完全只能访问Queue接口所定义的方法了，而不能直接访问LinkedList的非Queue的方法），以使得只有恰当的方法才可以使用。


## 队列
首先看一下java.util下的Queue接口定义：
```java
package java.util;

/**
 * <p>This interface is a member of the Java Collections Framework.
 *
 * @see java.util.Collection
 * @see LinkedList
 * @see PriorityQueue
 * @see java.util.concurrent.LinkedBlockingQueue
 * @see java.util.concurrent.BlockingQueue
 * @see java.util.concurrent.ArrayBlockingQueue
 * @see java.util.concurrent.LinkedBlockingQueue
 * @see java.util.concurrent.PriorityBlockingQueue
 * @since 1.5
 * @author Doug Lea
 * @param <E> the type of elements held in this collection
 */
public interface Queue<E> extends Collection<E> {

    boolean add(E e);

    boolean offer(E e);

    E remove();

    E poll();

    E element();

    E peek();
}
```

除了基本的继承了java.util.Collection的操作之外，队列提供额外的插入、提取和查找操作。这些方法中的每一种都有两种形式：一种是如果操作失败则抛出一个异常，另一个返回一个特殊的值（`null`或`false`），具体取决于操作）。后一种形式的插入操作设计，专门用于容量限制实现;（在大多数实现中，插入操作不能失败。）

Queue接口继承了`java.util.Collection`，Queue接口方法概要：
| 方法 |  抛出异常  | 返回特定值 |  说明   |
| --------   | ----- |  ----  | :----: |
| 插入 | add(e) | offer(e) | add(e)如果队列已满，则抛出一个IIIegaISlabEepeplian异常；<br>offer(e)如果队列已满返回false，否则返回true |
| 移除 | remove() | poll() | remove()移除并返回队列头部的元素    如果队列为空，则抛出一个NoSuchElementException；<br>poll()移除并返问队列头部的元素 如果队列为空，则返回null |
| 查找 | element() | peek() | element()返回队列头部的元素             如果队列为空，则抛出一个NoSuchElementException；<br>peek()返回队列头部的元素  如果队列为空，则返回null |

## 阻塞队列

>阻塞队列 (BlockingQueue)是`java.util.concurrent`包下重要的数据结构，BlockingQueue继承了Queue接口。
BlockingQueue提供了线程安全的队列访问方式：当阻塞队列进行插入数据时，如果队列已满，线程将会阻塞等待直到队列非满；从阻塞队列取数据时，如果队列已空，线程将会阻塞等待直到队列非空。并发包下很多高级同步类的实现都是基于BlockingQueue实现的。

阻塞队列 (BlockingQueue接口)在Queue接口的基础上增加了阻塞操作put和take。put方法在队列满时阻塞，take方法在队列空时阻塞。
并且带超时的offer和poll方法变种：
例如，下面的调用：
`boolean success = q.offer(x,100,TimeUnit.MILLISECONDS);`
尝试在100毫秒内向队列尾部插入一个元素。如果成功，立即返回true；否则，当到达超时进，返回false。
同样地，调用：
`Object head = q.poll(100, TimeUnit.MILLISECONDS);`
如果在100毫秒内成功地移除了队列头元素，则立即返回头元素；否则在到达超时时，返回null。



BlockingQueue是个接口，你需要使用它的实现之一来使用BlockingQueue，Java.util.concurrent包下具有以下 BlockingQueue 接口的实现类：

+ **ArrayBlockingQueue**：ArrayBlockingQueue 是一个有界的阻塞队列，其内部实现是将对象放到一个数组里。有界也就意味着，它不能够存储无限多数量的元素。它有一个同一时间能够存储元素数量的上限。你可以在对其初始化的时候设定这个上限，但之后就无法对这个上限进行修改了(译者注：因为它是基于数组实现的，也就具有数组的特性：一旦初始化，大小就无法修改)。

+ **DelayQueue**：DelayQueue 对元素进行持有直到一个特定的延迟到期。注入其中的元素必须实现 java.util.concurrent.Delayed 接口。

+ LinkedBlockingQueue：LinkedBlockingQueue 内部以一个链式结构(链接节点)对其元素进行存储。如果需要的话，这一链式结构可以选择一个上限。如果没有定义上限，将使用 Integer.MAX_VALUE 作为上限。

+ **PriorityBlockingQueue**：PriorityBlockingQueue 是一个无界的并发队列。它使用了和类java.util.PriorityQueue一样的排序规则。你无法向这个队列中插入null值。所有插入到PriorityBlockingQueue 的元素必须实现 java.lang.Comparable 接口。因此该队列中元素的排序就取决于你自己的 Comparable 实现。

+ **SynchronousQueue**：SynchronousQueue 是一个特殊的队列，它的内部同时只能够容纳单个元素。如果该队列已有一元素的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走。同样，如果该队列为空，试图向队列中抽取一个元素的线程将会阻塞，直到另一个线程向队列中插入了一条新的元素。据此，把这个类称作一个队列显然是夸大其词了。它更多像是一个汇合点。