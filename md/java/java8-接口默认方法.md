# java8接口默认方法

#### 含义

默认方法是指接口的默认方法，它是java8的新特性之一。顾名思义，默认方法就是接口提供一个默认实现，且不强制实现类去覆写的方法。默认方法用default关键字来修饰。



#### 使用

```java
public interface TestInterface1
{
    default void test()
    {
        System.out.println("TestInterface1");
    }   
}
public class Test implements TestInterface1
{
    //实现类可以不去实现接口的默认方法
}
```

#### 注意

当一个类继承了两个接口,但是两个接口有同名同参的default方法时必须重写该方法

可以自己实现

可以去使用两个接口的任意一个

使用  TestInterface1.super 去调用自己的父接口

```java
public interface TestInterface1
{
    default void test()
    {
        System.out.println("TestInterface1");
    }   
}

public interface TestInterface2
{
    default void test()
    {
        System.out.println("TestInterface2");
    }
}

public class Test implements TestInterface1, TestInterface2
{
    @Override
    public void test()
    {
        // 调用TestInterface1接口的默认test()方法
        TestInterface1.super.test();
    }
}
```

#### 接口的静态方法

```java
public interface TestInterface2
{
    static void test()   //将default 改为 static 即可
    {
        System.out.println("TestInterface2");
    }
}

```

使用时候使用父类名去调用静态方法

但是对于类继承来说子类名可以调用父类静态方法

#### 总结

解决了接口新增加方法后影响了已经现有的继承该接口的类,使得这些现有的类不用去实现该方法.