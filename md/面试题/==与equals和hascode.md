# **hash code、equals和“==”三者的关系**



1.如果是基本变量，没有hashcode和equals方法，基本变量的比较方式就只有==,；

2.如果是变量，由于在java中所有变量定义都是一个指向实际存储的一个句柄（你可以理解为c++中的指针），在这里==是比较句柄的地址（你可以理解为指针的存储地址），而不是句柄指向的实际内存中的内容，如果要比较实际内存中的内容，那就要用equals方法，但是！！！

如果是你自己定义的一个类，比较自定义类用equals和==是一样的，都是比较句柄地址，因为自定义的类是继承于object，而object中的equals就是用==来实现的，你可以看源码。

那为什么我们用的String等等类型equals是比较实际内容呢，是因为String等常用类已经重写了object中的equals方法，让equals来比较实际内容，你也可以看源码。

\3. hashcode
在一般的应用中你不需要了解hashcode的用法，但当你用到hashmap，hashset等集合类时要注意下hashcode。

你想通过一个object的key来拿hashmap的value，hashmap的工作方法是，通过你传入的object的hashcode在内存中找地址，当找到这个地址后再通过equals方法来比较这个地址中的内容是否和你原来放进去的一样，一样就取出value。

所以这里要匹配2部分，hashcode和equals
但假如说你new一个object作为key去拿value是永远得不到结果的，因为每次new一个object，这个object的hashcode是永远不同的，所以我们要重写hashcode，你可以令你的hashcode是object中的一个恒量，这样永远可以通过你的object的hashcode来找到key的地址，然后你要重写你的equals方法，使内存中的内容也相等。。。





**首先，**从语法角度，也就是从强制性的角度来说，hashCode和equals是两个独立的，互不隶属，互不依赖的方法，equals成立与hashCode相等这两个命题之间，谁也不是谁的充分条件或者必要条件。

但是，从为了让我们的程序正常运行的角度，我们应当向Effective Java中所言

重载equals的时候，一定要（正确）重载hashCode

使得equals成立的时候，hashCode相等，也就是a.equals(b)->a.hashCode() == b.hashCode()，或者说此时，equals是hashCode相等的充分条件，hashCode相等是equals的必要条件（从数学课上我们知道它的逆否命题：hashCode不相等也不会equals），但是它的逆命题，hashCode相等一定equals以及否命题不equals时hashCode不等都不成立。

所以，如果面试的时候，最好把hashCode与equals之间没有强制关系，以及根据（没有语法约束力的）规范的角度，应当做到...这两层意思都说出来:P