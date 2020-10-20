# JVM栈帧中的局部变量表

## 栈帧的组成

栈中最基本的单位就是栈帧，那么栈帧内部由什么组成呢？一共有五部分:

1.  局部变量表(Local variables)
2.  操作数栈(operand Stack) (或称做：表达式栈)
3.  动态链接(Dynamic Linking) (或称做：指向运行时常量池的方法引用)
4.  方法返回地址(Return Address) (或称做：方法正常退出或者异常退出的定义)
5.  一些附加信息

<img src="https://raw.githubusercontent.com/1471246901/myblog/master/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20200724093358.png" alt="img" style="zoom:50%;" />

## 局部变量表

局部变量表也被称之为本地变量表,也成为:局部变量数组,这所以这么叫,因为其实这个表就是由数组组成的。

定义为一个数字数组，局部变量数组-->数组的每一个空间(格子),也是最基本的存储单元--->我们叫Slot (变量槽)。当一个实例方法被调用的时候，它的方法参数和方法体内部定义的局部变量将会按照顺序被复制到局部变量表中的每一个slot上。

Slot (变量槽)中存放的是:方法参数和定义在方法体内的局部变量，这些数据类型包括各类基本数据类型、对象引用，以及返回值类型。

32位以内的类型只栈用1个slot (包括byte,short,int,float,char,boolean,returnAddress类型).(byte 、short、char 在存储前被转换为int， boolean 也被转换为int，0表示false，非0表示true。)

64位的类型(long和double)占用两个slot。

数组的长度: 局部变量表所需的容量大小是在编译期确定下来的，并保存在方法的Code属性的maximum local variables数据项中。在方法运行期间是不会改变局部变量表的大小的。