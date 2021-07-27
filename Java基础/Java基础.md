# Java基础

#### 基本数据类型

Java中基本类型有**八种**  

* byte：8位，最大存储数据量是255，存放的数据范围是-128~127之间。
* short：16位，最大数据存储量是65536，数据范围是-32768~32767之间。
* int：32位，最大数据存储容量是2^32 - 1，数据范围是 -2^31 ~ 2^31 - 1
* long：64位，最大数据存储容量是2的64次方减1，数据范围为 -2^63 ~ 2^63 - 1
* float：32位，数据范围在3.4e(-45) ~ 1.4e38，直接赋值时必须在数字后加上f或F。
* double：64位，数据范围在4.9e-324~1.8e308，赋值时可以加d或D也可以不加。
* boolean：只有true和false两个取值。
* char：16位，存储Unicode码，用单引号赋值。

#### int和Integer的区别，Integer常量池

https://www.cnblogs.com/guodongdidi/p/6953217.html

#### 抽象类和接口的区别？

https://zhuanlan.zhihu.com/p/94770324

#### 继承和接口实现的区别？

1. 语义不同。继承表述从属关系，实现表示某种行为的抽象定义的具体实现。
2. 可以实现多个接口，不能多继承
3. 继承类可以不重写方法，实现接口必须要实现方法。但继承抽象类也必须实现其中的抽象方法。

#### 重写和重载的区别 

https://www.runoob.com/java/java-override-overload.html

原理：重载是静态分派，发生在编译期；重写是动态分派，发生在运行期  

https://blog.csdn.net/dj_dengjian/article/details/80811348. 

博客第二段代码改成这个：

```
public class OverrideTest {
    class Father{
        public void doSomething(){
            System.out.println("Father do something");
        }
    }

    class Sun extends Father {
        public void doSomething(){
            System.out.println("Sun do something");
        }
    }

    public static void main(String [] args){
        OverrideTest overrideTest = new OverrideTest();
        Father sun = overrideTest.new Sun();
        Father father = overrideTest.new Father();
        father.doSomething();
        sun.doSomething();
    }
}

最后会打印：

Father do something

Sun do something
```

#### 形参传基本类型和引用类型的区别

都是值传递，基本类型传值，引用类型传对象的首地址

#### Java异常类的了解？项目中怎么使用的？

![Java异常体系](./Pics/Java异常体系.png)  

* Java把异常作为一种类，当做对象来处理。**所有异常类的基类是Throwable类，两大子类分别是Error和Exception**。
* 系统错误由Java虚拟机抛出，用Error类表示。**Error类描述的是内部系统错误**，例如Java虚拟机崩溃。这种情况仅**凭程序自身是无法处理的，在程序中也不会对Error异常进行捕捉和抛出**。
* 异常（Exception）又分为**RuntimeException(运行时异常)和CheckedException(检查时异常)**，两者区别如下：

区别：

1. RuntimeException：程序运行过程中才可能发生的异常。一般为代码的逻辑错误。例如：类型错误转换，数组下标访问越界，空指针异常、找不到指定类等等。
2. CheckedException：编译期间可以检查到的异常，**必须显式的进行处理（捕获或者抛出到上一层）**。例如：IOException, FileNotFoundException等等。

项目中：

1. 对于运行时异常，尽量做好测试，尽量避免
2. 对于受检异常，需要本地处理的就本地处理，需要抛出的就抛到顶层，Spring项目就写AOP切面统一处理，普通项目就抛到一个顶层口子收起来的地方统一处理。另外有很多需要自定义异常的情况，就继承Exception。

#### 出现了异常怎么保证资源正常关闭？

1. 把异常catch住然后在finally块中关闭资源。
2. 用try with resource的方式

#### try catch finally的执行逻辑和return的逻辑

https://blog.csdn.net/qq_40933663/article/details/88824391

#### Java SPI，Dubbo SPI

https://www.cnblogs.com/jy107600/p/11464985.html

#### equals、hashcode

https://zhuanlan.zhihu.com/p/78249480   

1. 默认equals方法比较内存地址
2. 默认hashcode方法返回对象的内存地址
3. hashcode只有在hashmap算hash位置的时候才会用到
4. 自定义类作为key放入HashMap/HashSet时要重写equals和hashcode，避免出现两个对象内容相同，但内存地址不同，导致hashcode不同，从而在map中put(O1,value)之后，用map.get(O2)获取不到。
5. 重写equals方法是必须重写hashcode方法，以避免在HashMap中出现两个equals比较相同的对象，却被hash到了两个不同的桶。

#### static块，static属性，非static属性，构造函数，其他类实例的初始化顺序？

https://blog.csdn.net/u013728021/article/details/78719504  

#### static关键字作用？局部变量static存储在哪，全局变量加不加static存储情况有什么区别

https://www.cnblogs.com/dolphin0520/p/3799052.html  

https://blog.csdn.net/weixin_39968640/article/details/110614049  

#### Java引用和C++指针的区别（结合对象访问方式一起看）

https://blog.csdn.net/qq_42077125/article/details/80325620

#### Java对象访问方式

https://blog.csdn.net/java2000_wl/article/details/8015105  

#### object类有哪些方法

https://blog.csdn.net/he6687086/article/details/94882098