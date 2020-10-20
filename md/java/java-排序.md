# java 排序

#### 以前使用 collections.sort();

collections.sort(list) 将list进行排序

或者

collections. sort( list, Comparator ) 

#### 现在可以使用     变量.sort()

变量.sort(String.CASE_INSENSITIVE_ORDER);

里面可以使用比较器

Comparator .comparing ()方法可以接受指定的参数进行排序

如

comparing (对象.get属性()) 即返回的比较器会按照那个属性的正序排序

如果要比较两个以上属性可以使用thenComparing()方法

使用reversed() 方法将比较器前的所有操作取反,即 正序变倒序

#### Comparator   比较器接口

```
Comparator.comparing(属性a).reversed()
//作用按照属性a倒序排列
Comparator.comparing(属性a).reversed().thenComparing(属性b)
//作用在按照属性a倒序排列的前提下岁属性b进行正序排序.
Comparator.comparing(属性a).reversed().thenComparing(属性b).reversed()
//作用按照属性a的正序排序(经过两次reversed变为正序) 在按照属性b进行倒序排序(属性b只经过一次reversed)
```

#### comparing()方法的参数

使用方法引用   `集合中元素对象的类名::get属性名()`

使用lambda表达式   i -> ((对象类型)i ).get属性()     i 为集合中的对象,**但是传入的i是object对象的 所以需要强制转换**`

所以推荐使用方法引用 这样的好处是 使用对象的类名就可以使用实例对象的方法

此外  **java.util.Collections**  工具集还有很多的集合工具  max  min   二分查找  , 动态类型安全视图 ,不接修改视图等.