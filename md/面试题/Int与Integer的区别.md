# int和Integer的区别 

 int是常量，Integer是int的包装类。int不能为null,Integer可以为null,说明Integer 是对象。

无论怎么比较 通过  对象.equals(对象)  还是 对象.equals(int值)   只要数值相同,结果都为true

下面讨论以下== 的情况 , ==是判断数值类型的值或引用类型的地址的方法

## Integer == Integer

### new Integer == new Integer

由于Integer实际是对一个Integer对象的引用，所以两个通过new生成的Integer变量永远是不相同的(`Integer i = new Integer(100),Integer j= new Integer(100)`)，因为New生成的是两个不同的对象，其内存地址不同。

### new Integer == 直接赋值的 Integer

非new生成的Integer变量和new Integer生成的Integer变量比较的时候，结果为false(因为非new生成的Integer变量指向的是Java常量池中的对象，而new出来的对象指向的是堆中新建的对象，两者内存地址不同)，下面返回的是false

### 直接赋值的 Integer == 直接赋值的 Integer

Integer 内缓存-128 到+127 的数值,所以导致两个对象相比较(Integer 与Integer这两个都是直接赋值而不是new  `Integer i = 100,Integer j=100`) 结果 -128 到+127  会出现相等情况

## Integer == int

Integer变量和int变量进行比较时，只要两个变量的值相等，则结果就为True，(因为包装类Integer和基本数据类型比较的时候，java会自动拆箱为int，然后进行比较，实际上就是两个int变量进行比较)，下面运行的结果为true

## int == int

两数相等绝对为true



