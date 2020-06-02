# JVM初步理解

## 1. 内存模型

![](http://img.doze9097.top//20200602123523.png)

关注点：

​	一个方法区、一个栈、一个堆

## 2. 实例化顺序

​	若 `Objecto = new Object();`被执行

1. JMP启动，代码编译生成一个`.class`文件

2. `.class`文件被类加载器（Class Loader）加载入JVM的内存中

3. 类Object被加载入**方法区**；在堆中创建Object类的**类类型对象**（非实例对象），作为方法区的数据结构的接口。

   > JVM创建对象之前，会通过寻找**类类型对象**是否已经加载类，若加载好，则为你的对象分配内存(new Object())

## 3. 理解

程序在运行时，需要动态加载一些类，这些类可能之前用不到所以不用加载到JVM，而是在运行时根据需要加载。

举个例子我们的项目底层有时是用mysql，有时用oracle，需要动态地根据实际情况加载驱动类，这个时候反射就有用了，假设 `com.java.dbtest.myqlConnection`，`com.java.dbtest.oracleConnection`这两个类我们要用，这时候我们的程序就写得比较动态化，通过`Class tc = Class.forName("com.java.dbtest.TestConnection");`通过类的全类名让jvm在服务器中找到并加载这个类，而如果是oracle则传入的参数就变成另一个了。



## 4. 参考链接

https://www.zhihu.com/question/24304289