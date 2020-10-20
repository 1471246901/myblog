# synchronized

----

当synchronized关键字，有不少同学已经耳熟能详了，作为作为复习还是要巩固一下，因为面试常问这个东西。



第一个是多个线程去访问同一个资源的时候，对这个资源要上锁。为什么要上锁呢？访问某一段代码或者某临界资源的时候是需要有一把锁的概念的。



比如：我们对一个数字做递增，两个程序对它一块儿来做递增，递增就是把一个程序往上加1，如果两个线程共同访问的时候，第一个线程读它是0，然后把它加1，在自己线程内部内存里面算还没有写回去的时候，第二个线程读到了它还是0，加1再写回去，本来加了两次，但是还是1。



那么我们在对这个数字递增的过程中上把锁，就是说第一个线程对这个数字访问的时候是独占的，不允许别的线程来访问，不允许别的线程来对它进行计算，我必须加完1再释放锁，其他线程才能对它继续加。



实质上，这把锁并不是对数字进行锁定的，你可以任意指定，想锁谁就锁谁。



我第一个程序是这么写的，如果说你想上了把锁之后才能对count进行减减访问，你可以new一个Object，所以这里锁定就是0，当我拿到这把锁的时候才能执行这段代码。



```
/**
 * synchronized关键字
 * 对某个对象加锁
 * @author mashibing
 */

package com.mashibing.juc.c_001;

public class T {
  
  private int count = 10;
  private Object o = new Object();
  
  public void m() {
    synchronized(o) { //任何线程要执行下面的代码，必须先拿到o的锁
      count--;
      System.out.println(Thread.currentThread().getName() + " count = " + count);
    }
  }
  
}
```



我们来谈一下synchronized的一些特性。如果说你每次都定义一个锁的对象Object o，把它new出来，那加锁的时候太麻烦，每次都要new一个新的对象出来。所以呢，最简单的方式就是synchronized(this)，锁定当前对象就行。



```
/**
 * synchronized关键字
 * 对某个对象加锁
 * @author mashibing
 */

package com.mashibing.juc.c_002;

public class T {
  
  private int count = 10;
  
  public void m() {
    synchronized(this) { //任何线程要执行下面的代码，必须先拿到this的锁
      count--;
      System.out.println(Thread.currentThread().getName() + " count = " + count);
    }
  }
  
}
```



如果你要是锁定当前对象，你也可以写成synchronized方法，这个和synchronized(this)是等值的。



```
/**
 * synchronized关键字
 * 对某个对象加锁
 * @author mashibing
 */

package com.mashibing.juc.c_003;

public class T {

  private int count = 10;
  
  public synchronized void m() { //等同于在方法的代码执行时要synchronized(this)
    count--;
    System.out.println(Thread.currentThread().getName() + " count = " + count);
  }
}
```



我明知道静态方法static是没有this对象的，你不需要new出一个对象来就能执行这个方法，但是如果这个上面加一个synchronized的话就表示synchronized(T.class)，这里这个synchronized(T.class)锁的就是T类的对象。



```java
/**
 * synchronized关键字
 * 对某个对象加锁
 * @author mashibing
 */

package com.mashibing.juc.c_004;

public class T {

  private static int count = 10;
  
  public synchronized static void m() { //这里等同于synchronized(FineCoarseLock.class)
    count--;
    System.out.println(Thread.currentThread().getName() + " count = " + count);
  }
  
  public static void mm() {
    synchronized(T.class) { 
        //考虑一下这里写synchronized(this)是否可以？   不可以,static方法访问不到this
      count --;
    }
  }

}
```

