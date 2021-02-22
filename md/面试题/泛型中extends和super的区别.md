# 泛型中extends和super的区别

`<? extends T>`和`<? super T>`是Java泛型中的“通配符（Wildcards）”和“边界（Bounds）”的概念。

-   <? extends Fruit>：是指 “上界通配符（Upper Bounds Wildcards）”

    意思是Fruit  和 Fruit 的子类都可以

    ![img](https://images2015.cnblogs.com/blog/820480/201611/820480-20161125004120143-1731938777.png)

-   <? super Fruit>：是指 “下界通配符（Lower Bounds Wildcards）”

    意思是 Fruit 和  Fruit 的父类都可以

    ![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/820480-20161125004216471-1377946016.png)

>   https://blog.csdn.net/qq_36898043/article/details/79655309