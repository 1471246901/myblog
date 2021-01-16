# java容器类有哪些分别介绍一下

![image-20201125104947480](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20201125104947480.png)

## **Set下各种实现类对比**

HashSet基于哈希表实现，有以下特点：

​     1.不允许重复

​      2.允许值为null,但是只能有一个

​      3.无序的。

​      4.没有索引，所以不包含索引操作的方法

LinkedHashSet跟HashSet一样都是基于哈希表实现。只不过linkedHashSet在hashSet的基础上多了一个链表，这个链表就是用来维护容器中每个元素的顺序的。有以下特点：

​      1.不允许重复

​      2.允许值为null,但是只能有一个

​      3.有序的。

​      4.没有索引，所以不包含索引操作的方法

TreeSet是SortedSet接口的唯一实现类，是基于二叉树实现的。TreeSet可以确保集合元素处于排序状态。TreeSet支持两种排序方式，自然排序 和定制排序，其中自然排序为默认的排序方式。向TreeSet中加入的应该是同一个类的对象。有以下特点：

​      1.不允许重复

​      2.不允许null值

​      3.没有索引，所以不包含索引操作的方法

## **List下各种实现类对比。（这几个类都是有序的，允许重复的）**

**ArrayList**是基于数组实现的，其特点是查询快，增删慢。

​    查询快是因为数组的空间是连续的，查询时只要通过首地址和下标很快就能找到元素。

​    增删慢是因为数组是不能扩容的，一旦增加或者删除元素，内部操作就是新开辟一个数组把元素copy到新的数组，老的数  组等待被垃圾回收。

**LinkedList**是基于链表实现的。相比于ArrayList其特点是查询慢，增删快。

查询慢：因为链表在内存中开辟的空间不一定是连续的（基本上不可能是连续的）所以链表实现的方式是每个元素节点都会存放自己的地址，数据以及下一个节点的地址，这样把所有的元素连接起来。所以当要查询元素时只能一个一个的往下找，相比于数组的首地址加下标会慢上不少。

下面是链表的数据存储方式：假设有三个元素

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/20190731135031290.png)

Vector也是基于数组实现的，相比于arrayList它是线程安全的。如果不考虑线程安全它，ArrayList性能更优。

## **Map是双列集合的超类。也就是键值对形式。**

HashMap和Hashtable都实现了Map接口，但决定用哪一个之前先要弄清楚它们之间的分别。主要的区别有：线程安全性，同步(synchronization)，以及速度。

-   HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的，并可以接受null(HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行)。
-   HashMap是非synchronized，而Hashtable是synchronized，这意味着Hashtable是线程安全的，多个线程可以共享一个Hashtable；而如果没有正确的同步的话，多个线程是不能共享HashMap的。Java 5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。
-   另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。
-   由于Hashtable是线程安全的也是synchronized，所以在单线程环境下它比HashMap要慢。如果你不需要同步，只需要单一线程，那么使用HashMap性能要好过Hashtable。
-   HashMap不能保证随着时间的推移Map中的元素次序是不变的。

LinkedHashMap和hashMap的区别在于多维护了一个链表，用来存储每一个元素的顺序，就跟HashSet和LinkedHashSet差不多。

HashMap通常比TreeMap快一点(树和哈希表的数据结构使然)，建议多使用HashMap，在需要排序的Map时候才用TreeMap。