---
title: Java中类与对象的比较及hashCode和equals方法详解
date: 2017-05-26 15:03:49
categories: Java
tags: Java
---
重新认识理解Java类、对象之间的对比，和equals和hashCode方法重要性。
<!--more-->
### 类与类的比较

类与类的比较，通常有两种方式：

- 直接用 == 比较class：在Java中，类是静态的，唯一的，共享的单例，所以这里用 == 在恰当不过了，使用 == 更能说明实现关系。
```
Integer obj1 = 1;
Integer obj2 = 1;
if(Integer.class == Integer.class) {
  // true
}

if(obj1.getClass() == obj2.getClass()) {
  // true
}
```
- 比较 class.getName()。
```
Integer obj1 = 1;
Integer obj2 = 1;
if(Integer.class.getName() == Integer.class.getName()) {
  // true
}

if(obj1.getClass().getName() == obj2.getClass().getName()) {
  // true
}
```

### 类与对象的比较

判读啊一个对象和一个类是不是具有实现关系，通常有两种方式：

- obj instanceof Class ：
```
// 注意：obj 不能是原始类型，例如 int, long
// obj 只能是引用类型，例如 Integer, Long
Integer obj = 1;
if(obj instanceof Integer) {
  // true
}

if(obj instanceof Object) {
  // true
}
```
实现方式：先判断 obj 是不是 null，随后判断 obj.getClass() 与 Integer.class 是否相等。
因此 instanceof 相等于：
`if(obj != null && obj.getClass() == Integer.class)`

>instanceof 用于检测类与对象的 直接实现关系，检测方式是 对象->类。

- Class<?>.isInstance(obj)
```
// 注意：obj 可以是引用类型，也可以是原始类型，例如 int, long
Integer obj = 1;
if(Integer.class.isInstance(obj)) {
  // true
}

if(Object.class.isInstance(obj)) {
  // true
}
```
>Class<?>.isInstance(obj) 用于检测 非直接实现关系，检测方式是 类->对象。
`isInstance`是用C语言 native 代码实现的。当一个类继承了某个接口，类，或者而是抽象类。那么，`isInstance` 旨在查找出这个对象到底实现了哪些类。
```
interface I {
}

class A implements I {
}

class B extends A {
}

B b = new B();

I.class.isInstance(b); //查询b是不是实现了接口I true
A.class.isInstance(b);//查询b是不实现了A类  true
B.class.isInstance(b);//查询b是不实现了B类  true
```
>通常来说 ,对于一个对象和一个类来说，(obj instanceOf ObjClass) 关系如果为真，那么Class<?.isInstance(obj)关系也为真。
但反过来就不一样了。
Class<?>.isInstance(obj)更加适合 泛类型 的检测（如代理，接口，抽象类等规则）
而 instanceOf符合直接类型的检测。

### 对象与对象的比较

一、基本类型比较
 
Java中，基本类型比较用“==”，只要两个基本类型值相等即返回ture，否则返回false。
 
二、引用类型比较
 
引用类型比较比较变态，可以用“==”，也可以用“equals()”来比较，equals()方法来自于Object类，每个自定义的类都可以重写这个方法。Object类中的equals()方法仅仅通过“==”来比较两个对象是否相等。
 
在用“==”比较引用类型时，仅当两个应用变量的对象指向同一个对象时，才返回ture。言外之意就是要求两个变量所指内存地址相等的时候，才能返回true，每个对象都有自己的一块内存，因此必须指向同一个对象才返回ture。

我们看一个例子，让我们创建一个简单的类Employee：
```
public class Employee
{
    private Integer id;
    private String firstname;
    private String lastName;
    private String department;

    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }
    public String getFirstname() {
        return firstname;
    }
    public void setFirstname(String firstname) {
        this.firstname = firstname;
    }
    public String getLastName() {
        return lastName;
    }
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
    public String getDepartment() {
        return department;
    }
    public void setDepartment(String department) {
        this.department = department;
    }
}
```
```
public class EqualsTest {
    public static void main(String[] args) {
        Employee e1 = new Employee();
        Employee e2 = new Employee();

        e1.setId(100);
        e2.setId(100);
     
        System.out.println(e1.equals(e2)); //Prints false in console
    }
}
```
毫无疑问，上面的程序将输出false。
接下来我们重写一下equals()方法：
```
public boolean equals(Object o) {
        if(o == null)
        {
            return false;
        }
        if (o == this)
        {
           return true;
        }
        if (getClass() != o.getClass())
        {
            return false;
        }
        Employee e = (Employee) o;
        return (this.getId() == e.getId());
}
```
在重写equals方法后，EauqlsTest将会输出true。

但是现在有一种情况，请看下面示例：
```
import java.util.HashSet;
import java.util.Set;

public class EqualsTest
{
    public static void main(String[] args)
    {
        Employee e1 = new Employee();
        Employee e2 = new Employee();

        e1.setId(100);
        e2.setId(100);

        System.out.println(e1.equals(e2)); //Prints 'true'

        Set<Employee> employees = new HashSet<Employee>();
        employees.add(e1);
      
        System.out.println(employees.contains(e2)); // Print ‘false’
    }
}
```
如果两个employee对象equals返回true，Set中应该只存储一个对象`employees.contains(e2)`应该返回true才对，问题在哪里呢？ 
我们忘掉了第二个重要的方法hashCode()。

>就像JDK的Javadoc中所说的一样，如果重写equals()方法必须要重写hashCode()方法。

**需要注意的**

- 尽量保证使用对象的同一个属性来生成hashCode()和equals()两个方法。在我们的案例中,我们使用员工id。
- eqauls方法必须保证一致（如果对象没有被修改，equals应该返回相同的值）
- 任何时候只要a.equals(b),那么a.hashCode()必须和b.hashCode()相等。
- 两者必须同时重写。

**当使用ORM的时候特别要注意的**
- 如果你使用ORM处理一些对象的话，你要确保在hashCode()和equals()对象中使用getter和setter而不是直接引用成员变量。因为在ORM中有的时候成员变量会被延时加载，这些变量只有当getter方法被调用的时候才真正可用。
- 例如在我们的例子中，如果我们使用e1.id == e2.id则可能会出现这个问题，但是我们使用e1.getId() == e2.getId()就不会出现这个问题。


### equals()和hashCode()方法

java.lang.Object类中有两个非常重要的方法：
```
public boolean equals(Object obj)
public int hashCode()
```
Object类是类继承结构的基础，所以是每一个类的父类。所有的对象，包括数组，都实现了在Object类中定义的方法。

#### 一、equals()方法详解

equals()方法是用来判断其他的对象是否和该对象相等.

  equals()方法在object类中定义如下： 
```
public boolean equals(Object obj) {  
    return (this == obj);  
}  
```
很明显是对两个对象的地址值进行的比较（即比较引用是否相同）。但是我们知道，String 、Math、Integer、Double等这些封装类在使用equals()方法时，已经覆盖了object类的equals()方法。

  比如在String类中如下：
```
public boolean equals(Object anObject) {  
    if (this == anObject) {  
        return true;  
    }  
    if (anObject instanceof String) {  
        String anotherString = (String)anObject;  
        int n = count;  
        if (n == anotherString.count) {  
            char v1[] = value;  
            char v2[] = anotherString.value;  
            int i = offset;  
            int j = anotherString.offset;  
            while (n– != 0) {  
                if (v1[i++] != v2[j++])  
                    return false;  
            }  
            return true;  
        }  
    }  
    return false;  
}  
```
很明显，这是进行的内容比较，而已经不再是地址的比较。依次类推Math、Integer、Double等这些类都是重写了equals()方法的，从而进行的是内容的比较。当然，基本类型是进行值的比较。

equals方法的性质有：

- 自反性（reflexive）。对于任意不为null的引用值x，x.equals(x)一定是true。

- 对称性（symmetric）。对于任意不为null的引用值x和y，当且仅当x.equals(y)是true时，y.equals(x)也是true。

- 传递性（transitive）。对于任意不为null的引用值x、y和z，如果x.equals(y)是true，同时y.equals(z)是true，那么x.equals(z)一定是true。

- 一致性（consistent）。对于任意不为null的引用值x和y，如果用于equals比较的对象信息没有被修改的话，多次调用时x.equals(y)要么一致地返回true要么一致地返回false。

- 对于任意不为null的引用值x，x.equals(null)返回false。

对于Object类来说，equals()方法在对象上实现的是差别可能性最大的等价关系，即，对于任意非null的引用值x和y，当且仅当x和y引用的是同一个对象，该方法才会返回true。

***需要注意的是当equals()方法被override时，hashCode()也要被override。按照一般hashCode()方法的实现来说，相等的对象，它们的hash code一定相等。***

#### 二、hashcode()方法详解

hashCode()方法给对象返回一个hash code值。这个方法被用于hash tables，例如HashMap。

它的性质是：

- 在一个Java应用的执行期间，如果一个对象提供给equals做比较的信息没有被修改的话，该对象多次调用hashCode()方法，该方法必须始终如一返回同一个integer。

- 如果两个对象根据equals(Object)方法是相等的，那么调用二者各自的hashCode()方法必须产生同一个integer结果。

- 并不要求根据equals(java.lang.Object)方法不相等的两个对象，调用二者各自的hashCode()方法必须产生不同的integer结果。然而，程序员应该意识到对于不同的对象产生不同的integer结果，有可能会提高hash table的性能。

大量的实践表明，由Object类定义的hashCode()方法对于不同的对象返回不同的integer。

在object类中，hashCode定义如下：
```
public native int hashCode();
```
 说明是一个本地方法，它的实现是根据本地机器相关的。当然我们可以在自己写的类中覆盖hashcode()方法，比如String、Integer、Double等这些类都是覆盖了hashcode()方法的。
例如在String类中定义的hashcode()方法如下：
```
public int hashCode() {  
    int h = hash;  
    if (h == 0) {  
        int off = offset;  
        char val[] = value;  
        int len = count;  
  
        for (int i = 0; i < len; i++) {  
            h = 31 * h + val[off++];  
        }  
        hash = h;  
    }  
    return h;  
}  
```

想要弄明白hashCode的作用，必须要先知道Java中的集合。　　
    总的来说，Java中的集合（Collection）有两类，一类是List，再有一类是Set。前者集合内的元素是有序的，元素可以重复；后者元素无序，但元素不可重复。这里就引出一个问题：要想保证元素不重复，可两个元素是否重复应该依据什么来判断呢？
    这就是Object.equals方法了。但是，如果每增加一个元素就检查一次，那么当元素很多时，后添加到集合中的元素比较的次数就非常多了。也就是说，如果集合中现在已经有1000个元素，那么第1001个元素加入集合时，它就要调用1000次equals方法。这显然会大大降低效率。   
    于是，Java采用了哈希表的原理。哈希（Hash）实际上是个人名，由于他提出一哈希算法的概念，所以就以他的名字命名了。哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上，初学者可以简单理解，hashCode方法实际上返回的就是对象存储的物理地址（实际可能并不是）。  
    这样一来，当集合要添加新的元素时，先调用这个元素的hashCode方法，就一下子能定位到它应该放置的物理位置上。如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了；如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址。所以这里存在一个冲突解决的问题。这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。  

 **简而言之，在集合查找时，hashcode能大大降低对象比较次数，提高查找效率！**

**Java对象的eqauls方法和hashCode方法是这样规定的：**

- 1、相等（相同）的对象必须具有相等的哈希码（或者散列码）。

- 2、如果两个对象的hashCode相同，它们并不一定相同。


以下是Object对象API关于equal方法和hashCode方法的说明(是对之前2点的官方详细说明)：
>- If two objects are equal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce the same integer result.
- It is not required that if two objects are unequal according to the equals(java.lang.Object) method, then calling the hashCode method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.

*如不按要求去做了，会发现相同的对象可以出现在Set集合中，同时，增加新元素的效率会大大下降。*

在object类中，hashcode()方法是本地方法，返回的是对象的地址值，而object类中的equals()方法比较的也是两个对象的地址值，如果equals()相等，说明两个对象地址值也相等，当然hashcode()也就相等了；在String类中，equals()返回的是两个对象内容的比较，当两个对象内容相等时，Hashcode()方法根据String类的重写代码的分析，也可知道hashcode()返回结果也会相等。以此类推，可以知道Integer、Double等封装类中经过重写的equals()和hashcode()方法也同样适合于这个原则。当然没有经过重写的类，在继承了object类的equals()和hashcode()方法后，也会遵守这个原则。

#### 三、示例说明

下面以HashSet为例进行分析，我们都知道：在hashSet中不允许出现重复对象，元素的位置也是不确定的。在hashset中又是怎样判定元素是否重复的呢？

在java的集合中，判断两个对象是否相等的规则是：

+ 1.判断两个对象的hashCode是否相等
    如果不相等，认为两个对象也不相等，完毕；
    如果相等，转入2。
    （这一点只是为了提高存储效率而要求的，其实理论上没有也可以，但如果没有，实际使用时效率会大大降低，所以我们这里将其做为必需的。）

+ 2.判断两个对象用equals运算是否相等
    如果不相等，认为两个对象也不相等;
    如果相等，认为两个对象相等。（equals()是判断两个对象是否相等的关键）

例：
```
package com.bijian.study;

import java.util.HashSet;
import java.util.Iterator;

public class HashSetTest {

    public static void main(String[] args) {
        HashSet hs = new HashSet();
        hs.add(new Student(1, "zhangsan"));
        hs.add(new Student(2, "lisi"));
        hs.add(new Student(3, "wangwu"));
        hs.add(new Student(1, "zhangsan"));

        Iterator it = hs.iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
        }
    }
}

class Student {
    int num;
    String name;

    Student(int num, String name) {
        this.num = num;
        this.name = name;
    }

    public String toString() {
        return num + ":" + name;
    }
}
```
运行结果：

    1:zhangsan  
    3:wangwu  
    2:lisi  
    1:zhangsan 
    
为什么hashSet添加了相等的元素呢，这是不是和hashset的原则违背了呢？
回答是：没有。因为在根据hashcode()对两次建立的new Student(1,“zhangsan”)对象进行比较时，生成的是不同的哈希码值，所以hashset把他当作不同的对象对待了，当然此时的equals()方法返回的值也不等。
  
 怎么解决这个问题呢？答案是：在Student类中重新hashcode()和equals()方法。
 
**重写equals()和hashcode()小结：**

- 1.重点是equals，重写hashCode只是技术要求（为了提高效率）
- 2.为什么要重写equals呢？因为在java的集合框架中，是通过equals来判断两个对象是否相等的
- 3.在hibernate中，经常使用set集合来保存相关对象，而set集合是不允许重复的。在向HashSet集合中添加元素时，其实只要重写equals()这一条也可以。但当hashSet中元素比较多时，或者是重写的equals()方法比较复杂时，我们只用equals()方法进行比较判断，效率也会非常低，所以引入了hashCode()这个方法，只是为了提高效率，且这是非常有必要的。

比如可以这样写：
```
public int hashCode(){  
   return 1; //等价于hashcode无效  
} 
```
这样做的效果就是在比较哈希码的时候不能进行判断，因为每个对象返回的哈希码都是1，每次都必须要经过比较equals()方法后才能进行判断是否重复，但这会引起效率的大大降低。


