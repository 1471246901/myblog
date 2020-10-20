# ThreadLocal

---



首先我们来说一下ThreadLocal的含义， Thread线程，Local本地，线程本地到底是什么意思呢?我们来看下面这个小程序。



```java
public class ThreadLocal1 {
  volatile static Person p = new Person();
  
  public static void main(String[] args) {
        
    new Thread(()->{
      try {
        TimeUnit.SECONDS.sleep(2);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      
      System.out.println(p.name);
    }).start();
    
    new Thread(()->{
      try {
        TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      p.name = "lisi";
    }).start();
  }
}

class Person {
  String name = "zhangsan";
}
```



我们可以看到这个小程序里定义了一个类，这个类叫Person,类里面定义了一个 String类型的变量name, name的值为"zhangsan", 在ThreadLocal1这个类里，我们实例化了这个 Person类，然后在main方法里我们创建了两个线程，第一个线程打印了p.name,第二个线程把 p.name的值改为了"Iisi",两个线程访问了同一个对象。



这个小程序想想也知道，最后的结果肯定是打印出了"lisi”而不是"zhangsan",因为原来的值虽然是"zhangsan",但是有一个线程1秒终之后把它变成"lisi"了，另一个线程两秒钟之后才打印出来，那它一定是变成"Iisi”了，所以这件事很正常。



但是有的时候我们想让这个对象每个线程里都做到自己独有的一 份，我在访问这个对象的时候，我一个线程要修改内容的时候要联想另外一个线程，怎么做呢?我们看这个小程序。



```java
public class ThreadLocal2 {
  //volatile static Person p = new Person();
  static ThreadLocal<Person> tl = new ThreadLocal<>();
  
  public static void main(String[] args) {
        
    new Thread(()->{
      try {
        TimeUnit.SECONDS.sleep(2);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      
      System.out.println(tl.get());
    }).start();
    
    new Thread(()->{
      try {
        TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      tl.set(new Person());
    }).start();
  }
  
  static class Person {
    String name = "zhangsan";
  }
}
```



这个小程序中，我们用到了ThreadLocal,我们看main方法中第二个线程，这个线程在1秒终之后往t对象中设置了一个Person对象， 虽然我们访问的仍然是这个tl对象，第一个线程在两秒钟之后回去get获取tl对象里面的值，第二个线程是1秒钟之后往tl对象里set了-个值，从多线程普通的角 度来讲，既然我一个线程往里边set了一 个值， 另外个线程去get这个值的时候应该是能get到才对，但是很不幸的是，我们1秒终的时候set了个值，两秒钟的时候去拿这个值是拿不到的。



这个小程序证明了这一点，这是为什么呢?原因是如果我们用ThreadLocal的时候，里边设置的这个值是线程独有的，线程独有的是什么意思呢?就是说这个线程里用到这个ThreadLocal的时候，只有自己去往里设置，设置的是只有自己线程里才能访问到的Person,而另外一个线程要访问的时候， 设置也是自己线程才能访问到的Person,这就是ThreadLocal的含义。



session 实现就是依靠  ThreadLocal



>   公众号 马士兵