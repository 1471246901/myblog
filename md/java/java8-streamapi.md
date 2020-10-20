# java8StreamAPI

#### 简介

java8新添加了一个特性:流stream . stream让开发者能够以一种声明的方式处理数据源(集合,数组,行文件等),他专注于对数据源进行各种高效的聚合操作和大批量数据的操作.

stream API 将处理的数据源看做一种stream(流),流在管道中传输和运算,支持的运算包括筛选,排序,聚合等,到达终点后便得到最终的处理结果.

#### 概念

##### **元素** 

Stream是一个来自数据源的元素队列，Stream本身并不存储元素。

##### **数据源**

（即Stream的来源）包含集合、数组、I/O channel、generator（发生器）等。

##### **聚合操作** 

类似SQL中的filter、map、find、match、sorted等操作

##### **管道运算** 

Stream在Pipeline中运算后返回Stream对象本身，这样多个操作串联成一个Pipeline，并形成fluent风格的代码。这种方式可以优化操作，如延迟执行(laziness)和短路( short-circuiting)。

##### **内部迭代** 

不同于java8以前对集合的遍历方式（外部迭代），Stream API采用访问者模式（Visitor）实现了内部迭代。

##### **并行运算** 

Stream API支持串行（stream() ）或并行（parallelStream() ）的两种操作方式。

#### stream特点

使用stream和lambda表达式可以大幅度提高编码效率和代码可读性,在数据量巨大时并行计算可以提供很好的性能

stream默认串行操作,也可以进行并行运算

对于数组,arraylist等通过索引的流速度提升很快,对于treeset,链表等,并行可能增加运行的时间

对于数组来说stream的速度和迭代器速度差不多,对于对象stream的速度很快

#### 举例

java8之前 使用for来计算大于0的数的和

```java
public static void main(String[] args)
    {  
        List<Integer> numbers = Arrays.asList(-1, -2, 0, 4, 5);
        
        long count = 0;
        
        for(Integer number: numbers)
        {
            if(number > 0)
            {
                count++;
            }
        }
        
        System.out.println("Positive count: " + count);
    }
```

java8之后可以这样使用

```java
public static void main(String[] args)
    {  
        List<Integer> numbers = Arrays.asList(-1, -2, 0, 4, 5);
        
        long count = 0;
        
        count = numbers.Stream().filter(i->i>0).count();
        
        System.out.println("Positive count: " + count);
    }
```

#### Stream 流的生成

##### Collection 接口的方法

```
default Stream<E> stream()返回一个顺序Stream与此集合作为其来源。 
当此方法应该重写spliterator()方法不能返回spliterator是IMMUTABLE ， CONCURRENT ，或后期绑定 。 （详见spliterator() ） 

实现要求： 
默认的实现创建顺序 Stream从收集的 Spliterator 。 
结果 
连续 Stream在这个集合中的元素 
从以下版本开始： 
1.8 
```

```java
/**
     * @return a sequential {@code Stream} over the elements in this collection
     * @since 1.8
     */
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }

    /**
     * @return a possibly parallel {@code Stream} over the elements in this collection
     * @since 1.8
     */
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
```

##### 行文件转化成流

```java
Stream<String> stream = Files.lines(Paths.get("文件路径"));

	stream.filter(i->i.endsWith("ii"))
				.map(i->i+"大大大")
				.forEach(System.out::println);
```

#### 函数介绍

| 返回值类型               | 方法介绍和参数                                               |
| ------------------------ | ------------------------------------------------------------ |
| `boolean`                | `allMatch(predicate)`  返回此流的所有元素是否与提供的谓词匹配。 |
| `boolean`                | `anyMatch(Predicate predicate)`  返回此流的任何元素是否与提供的谓词匹配。 |
| `static  Stream.Builder` | **`builder()`  返回一个 `Stream`的构建器。**                 |
| ` R`                     | **`collect(Collector collector)`  使用 [Collector](package-summary.html#MutableReduction)对此流的元素执行 [mutable reduction](package-summary.html#MutableReduction)  `Collector` 。** |
| ` R`                     | `collect(Supplier supplier,  BiConsumer accumulator, BiConsumer combiner)`  对此流的元素执行 [mutable reduction](package-summary.html#MutableReduction)操作。 |
| `static  Stream`         | `concat(Stream a, Stream b)`  创建一个懒惰连接的流，其元素是第一个流的所有元素，后跟第二个流的所有元素。 |
| `long**`**               | **`count()`  返回此流中的元素数。**                          |
| `Stream`                 | **`distinct()`  返回由该流的不同元素（根据 [`Object.equals(Object)`](../../../java/lang/Object.html#equals-java.lang.Object-)  ）组成的流。** |
| `static  Stream`         | `empty()`  返回一个空的顺序 `Stream` 。                      |
| `Stream`                 | `filter(Predicate predicate)`  返回由与此给定谓词匹配的此流的元素组成的流。 |
| `Optional`               | `findAny()`  返回[描述](../../../java/util/Optional.html)流的一些元素的`Optional`如果流为空，则返回一个空的`Optional`  。 |
| `Optional`               | `findFirst()`  返回[描述](../../../java/util/Optional.html)此流的第一个元素的`Optional`如果流为空，则返回一个空的`Optional`  。 |
| ` Stream`                | **`flatMap(Function> mapper)`  返回由通过将提供的映射函数应用于每个元素而产生的映射流的内容来替换该流的每个元素的结果的流。** |
| `DoubleStream`           | `flatMapToDouble(Function mapper)`  返回一个 `DoubleStream`  ，其中包含将该流的每个元素替换为通过将提供的映射函数应用于每个元素而产生的映射流的内容的结果。 |
| `IntStream`              | `flatMapToInt(Function mapper)`  返回一个 `IntStream`  ，其中包含将该流的每个元素替换为通过将提供的映射函数应用于每个元素而产生的映射流的内容的结果。 |
| `LongStream`             | `flatMapToLong(Function mapper)`  返回一个 `LongStream`  ，其中包含将该流的每个元素替换为通过将提供的映射函数应用于每个元素而产生的映射流的内容的结果。 |
| `void`                   | **`forEach(Consumer action)`  对此流的每个元素执行操作。**   |
| `void`                   | `forEachOrdered(Consumer action)`  如果流具有定义的遇到顺序，则以流的遇到顺序对该流的每个元素执行操作。 |
| `static  Stream`         | `generate(Supplier s)`  返回无限顺序无序流，其中每个元素由提供的 `Supplier` 。 |
| `static  Stream`         | `iterate(T seed,  UnaryOperator f)`  返回有序无限连续 `Stream`由函数的迭代应用产生 `f`至初始元素  `seed` ，产生 `Stream`包括 `seed` ，  `f(seed)` ， `f(f(seed))` ，等 |
| `Stream`                 | `limit(long maxSize)`  获取指定数量的流,截短长度不能超过 `maxSize` 。 |
| ` Stream`                | **`map(Function mapper)`  返回由给定函数应用于此流的元素的结果组成的流。** |
| `DoubleStream`           | `mapToDouble(ToDoubleFunction mapper)`  返回一个 `DoubleStream` ，其中包含将给定函数应用于此流的元素的结果。 |
| `IntStream`              | `mapToInt(ToIntFunction mapper)`  返回一个 `IntStream` ，其中包含将给定函数应用于此流的元素的结果。 |
| `LongStream`             | `mapToLong(ToLongFunction mapper)`  返回一个 `LongStream` ，其中包含将给定函数应用于此流的元素的结果。 |
| `Optional`               | `max(Comparator comparator)`  根据提供的 `Comparator`返回此流的最大元素。 |
| `Optional`               | `min(Comparator comparator)`  根据提供的 `Comparator`返回此流的最小元素。 |
| `boolean`                | `noneMatch(Predicate predicate)`  返回此流的元素是否与提供的谓词匹配。 |
| `static  Stream`         | `of(T... values)`  返回其元素是指定值的顺序排序流。          |
| `static  Stream`         | `of(T t)`  返回包含单个元素的顺序 `Stream` 。                |
| `Stream`                 | `peek(Consumer action)`  返回由该流的元素组成的流，另外在从生成的流中消耗元素时对每个元素执行提供的操作。 |
| `Optional`               | `reduce(BinaryOperator accumulator)`  使用 [associative](package-summary.html#Associativity)累积函数对此流的元素执行 [reduction](package-summary.html#Reduction) ，并返回描述减小值的  `Optional` （如果有）。 |
| `T`                      | `reduce(T identity, BinaryOperator accumulator)`  使用提供的身份值和 [associative](package-summary.html#Associativity)累积功能对此流的元素执行 [reduction](package-summary.html#Reduction) ，并返回减小的值。 |
| ` U`                     | `reduce(U identity,  BiFunction accumulator, BinaryOperator combiner)`  执行 [reduction](package-summary.html#Reduction)在此流中的元素，使用所提供的身份，积累和组合功能。 |
| `Stream`                 | **`skip(long n)` 跳过指定数量的元素**                        |
| `Stream`                 | **`sorted()`  返回由此流的元素组成的流，根据自然顺序排序。** |
| `Stream`                 | **`sorted(Comparator comparator)`  返回由该流的元素组成的流，根据提供的 `Comparator`进行排序。** |

#### 中间操作和终端操作

Stream中的操作从概念上讲分为**中间操作和终端操作**：

-   中间操作：例如peek(),map(),filter() 等方法提供Consumer（消费）函数，但执行peek()方法时不会执行Consumer函数，而是等到流真正被消费时（终端操作时才进行消费）才会执行，这种操作为中间操作；
-   终端操作：例如forEach()、collect()、count()等方法会对流中的元素进行消费，并执行指定的消费函数（peek方法提供的消费函数在此时执行），这种操作为终端操作。