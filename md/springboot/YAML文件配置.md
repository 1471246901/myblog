# YAML

### YAML是以**数据为中心**的语言（结构简单，比xml简单）

spring boot  推荐使用 yml 作为配置文件

#### YAMl语法

1.  ##### 基本语法

2.  1.  **键值对形式存在（空格必须有）**
    2.  **以空格缩进** **表示层级关系只要是左对齐就表示** **同一层级**
    3.  **属性和值大小写敏感**

3.  ##### 值的写法

4.  1.  **字面量** **普通的值** **（数字布尔字符转）**
    2.  字面量直接来写  字符串默认不加引号

    3.  双引号不会转义里面的特殊字符 （\n 表示换行）

    4.  单引号转义特殊字符  （\n 表示\n)

5.  ##### **对象** **map** 集合  **键值对**

    对象和map以K:V的方式表示如：

    ![реор1е :  аде: 20 ](https://raw.githubusercontent.com/1471246901/myblog/master/img/clip_image001.png)

    注意缩进就可以了

    ```yaml
    ### 行内写法 
    people：{name：张三，age 20}
    ```

    

6.  ##### **数组list**

    用-值的方式表示数组内的元素如：

    ![pets:  -cat  -do ](https://raw.githubusercontent.com/1471246901/myblog/master/img/clip_image002.png)

     

    ```yaml
    ### 行内写法 
    pets: {cat,dog,pig}
    ```

     

7.  ##### 组合写法

8.  1.  如配置下面java bean
    2.  ![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/clip_image003.png)

    3.  dog类

    4.  ![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/clip_image004.png)

    5.  YAML 写法(冒号后加空格 , - 后也要加空格)

    6.  ```yaml
        Persion:
        lastName: ZhangSan
        age: 20
        boss:false
        birth: 2017/12/12
        map：{k1: v1,K2: v2}
        Lists:
        - lisi
        - asd
        Dog:
        Name: littledog
        Age: 12
        ```

    7.  