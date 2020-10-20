# Serializable

----

一般情况下，我们在定义实体类时会继承Serializable接口，类似这样：

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/6609c93d70cf3bc72df42e93b89375a5cc112a75.jpeg)

我们在实体类中引用了Serializable这个接口，那么这个接口到底有什么？细心的你会发现我们还定义了个serialVersionUID变量。这个变量到底有什么作用？

**什么是Serializable接口**

一个对象序列化的接口，一个类只有实现了Serializable接口，它的对象才能被序列化。

**什么是序列化？**

序列化是将对象状态转换为可保持或传输的格式的过程。与序列化相对的是反序列化，它将流转换为对象。这两个过程结合起来，可以轻松地存储和传输数据。

**为什么要序列化对象**

把对象转换为字节序列的过程称为对象的序列化把字节序列恢复为对象的过程称为对象的反序列化**什么情况下需要序列化？**

当我们需要把对象的状态信息通过网络进行传输，或者需要将对象的状态信息持久化，以便将来使用时都需要把对象进行序列化

那为什么还要继承Serializable。那是存储对象在存储介质中，以便在下次使用的时候，可以很快捷的重建一个副本。

或许你会问，我在开发过程中，实体并没有实现序列化，但我同样可以将数据保存到mysql、Oracle数据库中，为什么非要序列化才能存储呢？

我们来看看Serializable到底是什么，跟进去看一下，我们发现Serializable接口里面竟然什么都没有，只是个空接口

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/d53f8794a4c27d1e4daefeb16f46626adcc43857.jpeg)

一个接口里面什么内容都没有，我们可以将它理解成一个标识接口。

比如在课堂上有位学生遇到一个问题，于是举手向老师请教，这时老师帮他解答，那么这位学生的举手其实就是一个标识，自己解决不了问题请教老师帮忙解决。在Java中的这个Serializable接口其实是给jvm看的，通知jvm，我不对这个类做序列化了，你(jvm)帮我序列化就好了。

Serializable接口就是Java提供用来进行高效率的异地共享实例对象的机制，实现这个接口即可。

什么是JVM？

JVM是Java Virtual Machine（Java虚拟机）的缩写，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。

**为什么要定义serialversionUID变量**

简单看一下 Serializable接口的说明

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/3812b31bb051f819b3bca6b0ad2785e92c73e7c3.jpeg)

从说明中我们可以看到，如果我们没有自己声明一个serialVersionUID变量,接口会默认生成一个serialVersionUID

However, it is <em>strongly recommended</em> that all serializable classes explicitly declare serialVersionUID values, since the default serialVersionUID computation is highly sensitive to class details that may vary depending on compiler implementations, and can thus result in unexpected<code>InvalidClassException</code>s during deserialization.

但是强烈建议用户自定义一个serialVersionUID,因为默认的serialVersinUID对于class的细节非常敏感，反序列化时可能会导致InvalidClassException这个异常。

在前面我们已经新建了一个实体类User实现Serializable接口，并且定义了serialVersionUID变量。

我们把User写到文件，然后读取出来。

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/738b4710b912c8fc194a77a490905d41d78821b2.jpeg)

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/eac4b74543a982262e36f935e21176054b90eb42.jpeg)

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/472309f790529822721c0b30bf59b4cf0b46d4a9.jpeg)

是的，你没有看错，序列化与反序列化操作过程就是这么的简单。只需要将User写入到文件中，然后再从文件中进行恢复，恢复后得到的内容与之前完全一样，但是两者是不同的对象。前面提到过一个问题，如果将serialVersionUID变量去掉，我们来看看，会发生什么事情。

刚开始提到了，serialVersionUID要不要指定呢？如果不指定会出现什么样的后果？如果指定了以后后边的值又代表着什么意思呢？既然系统指定了这个字段，那么肯定是有它的作用的。

这个serialVersionUID是用来辅助对象的序列化与反序列化的，原则上序列化后的数据当中的serialVersionUID与当前类当中的serialVersionUID一致，那么该对象才能被反序列化成功。这个serialVersionUID的详细的工作机制是：在序列化的时候系统将serialVersionUID写入到序列化的文件中去，当反序列化的时候系统会先去检测文件中的serialVersionUID是否跟当前的文件的serialVersionUID是否一致，如果一直则反序列化成功，否则就说明当前类跟序列化后的类发生了变化，比如是成员变量的数量或者是类型发生了变化，那么在反序列化时就会发生crash，并且回报出错误：

```
java.io.InvalidClassException: User; local class incompatible: stream classdesc serialVersionUID = -1451587475819212328, local class serialVersionUID = -3946714849072033140at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:699)at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1885)at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1751)at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2042)at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1573)at java.io.ObjectInputStream.readObject(ObjectInputStream.java:431)at Main.readUser(Main.java:32)at Main.main(Main.java:10)
```

![img](D:/Typora/data/image/8ad4b31c8701a18b8a85f73af6bcc80c2938fef5.jpeg)





>   百家号: 洪生鹏