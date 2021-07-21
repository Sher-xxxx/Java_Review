# Java复习

## Java程序设计

### JVM

1. ClassLoader**加载**过程：JVM启动，运行bootstrap classloader（启动类加载器），该ClassLoader加载Java核心API（ExtClassLoader和AppClassLoader也被加载），调用ExtClassLoader加载扩展API，最后AppClassLoader加载CLASSPATH目录下定义的Class。

2. ClassLoader（类加载器）是Java运行时唤醒的一部分，负责动态加载Java类到JVM的内存空间中（因为Java是**按需加载**的）。Java自带三个类加载器：

   1. Bootstrap ClassLoader，最顶层的加载类，主要加载核心类库
   2. Extention ClassLoader，扩展的类加载器
   3. Appclass Loader，加载当前应用的classpath的所有类

3. 一个类加载的时候，使用父类委托模式。

   1. **父类委托模式**：类加载器本身不去加载类，而是将加载任务交给父类加载器完成
   2. 目的：**避免重复加载**，当父类已经加载该类的时候，没有必要子ClassLoader再加载一次；**安全因素**，避免自定义类库替代核心类库
   3. 缺点：单向，子加载器去请求父加载器，上层类加载器无法访问下层类加载器所加载的类

4. 几个重要的方法：

   1. loadClass()：

      ```java
      protected Class<?> loadClass(String name, boolean resolve)
              throws ClassNotFoundException{}
      ```

      ClassLoader.loadClass()是ClassLoader的入口点，name是需要的类的名称，resolve告诉方法是否需要解析类。注：并不总是需要类解析，如果JVM只需要知道该类是否存在或找到该类的超类，那么就不需要解析

   2. defineClass()：

      ```java
      protected final Class<?> defineClass(byte[] b, int off, int len)
              throws ClassFormatError{}
      ```

      接收由原始字节组成的数组，并转换成java.lang,Class对象。管理JVM的许多复杂的实现层面，把字节码分析称运行时数据结构、校验有效性等。

   3. findSystemClass():

      ```java
      protected final Class<?> findSystemClass(String name)
          throws ClassNotFoundException{}
      ```

      从本地文件系统装入文件。在本地文件系统中寻找类文件，如果存在就使用defindClass方法将原始字节转换成Class对象，以将该文件转换成类。

   4. resolveClass():

      ```java
      protected final void resolveClass(Class<?> c) {}
      ```

      在编写自己的loadClass时可以调用resolveClass，取决于loadClass的resolve参数值

   5. forname():

      ```java
      public static Class<?> forName(String name, boolean initialize,
                                     ClassLoader loader)
          throws ClassNotFoundException{}
      ```

      Class.forName()和ClassLoader.loadClass()目的一样，都是用来加载Class，但是作用上有区别

      - Class.forName()除了将类的.class文件加载到JVM外，还会对类进行解释，执行类中的static块
      - ClassLoader只会将.class文件加载到JVM中，只有在newInstance才会执行static块

5. JVM加载类过程：装载、连接、初始化

   - 装载：找到对应的.class文件，读入JVM

   - 连接：

     - 验证，验证class是否符合规范

     - 准备，给类的静态变量分配并初始化存储空间

     - 解释，可选，根据类中符号引用查找相应实体，再把符号引用替换成一个实体引用

       ![img](https://upload-images.jianshu.io/upload_images/8576307-dfa6742d52a6c3de.png?imageMogr2/auto-orient/strip|imageView2/2/w/1008/format/webp)

   - 初始化：.class文件初始化
   
6. 所有的I/O可以分为两类：面向**字符**的输入/输出流，面向**字节**的输入/输出流。面向：指的是在哪个意义上保持一致

   1. 面向**字节**：保证文件二进制内容和读入JVM内部的**二进制内容**一致，适合读入视频文件或音频文件，或任何不需要做变换的文件内容。一次读入或读出是8位二进制。字节流能处理所有类型的数据。
   2. 面向**字符**：系统文件中的字符和读入内存的”**字符**“一致。如：有一个”有“字，希望读出来还是”有“，不管他之间的二进制内容是什么。一次读入或读出是16位二进制。字符流只能处理字符类型的数据。
   3. **Tips：**处理纯文本数据就优先考虑使用字符流，除此之外都使用字节流。

7. 面向字符的I/O类，Reader和Writer类，实际上隐式地做了编码转换。Java的I/O系统中能够指定转换编码，即字符与字节相互转换的地方——InputStreamReader，OutputStreamWriter，这两个类是字节流和字符流的适配器类，承担编码转换的任务

### 类型转换

1. Java基本数据类型分为三大类：布尔型boolean、字符型char和数值型（整型byte/short/int/long、浮点型float/double）。

   基本数据类型从低级到高级分别为（byte/short/char—int—long—float—double），从低级到高级可以自动转换，从高级到低级需要强制类型转换，或者包装类过渡类型转换

   ``` java
   char c='c'; int i=c; // 低级到高级，自动转换
   int i=99; char c=(char)i; // 高级到低级，强制类型转换
   ```

   取值范围：

   | 类型    | 取值范围                        | 包装类    | 用途                                                         |
   | ------- | ------------------------------- | --------- | ------------------------------------------------------------ |
   | byte    | [-2^7, 2^7-1]， 有符号8位表示   | Byte      | 最小的整形类型，常用于网络传输、文件或其他I/O数据流，默认值0 |
   | short   | [-2^15, 2^15-1]，有符号16位表示 | Short     | 不常用，默认值0                                              |
   | char    | [0, 2^16-1]，无符号16位         | Character | 使用Unicode表示字符，映射成16位数字                          |
   | int     | [-2^31, 2^32-1]，有符号32位     | Integer   | 默认值0                                                      |
   | long    | [-2^63, 2^63-1]，有符号64位     | Long      | 以l或L结束，long x = 123323l，默认0L                         |
   | float   | 单精度32位                      | Float     | 以f或F结束，float x=0.6f，默认0F                             |
   | double  | 双精度64位                      | Double    | Java基本类型中最高精度，默认0.0                              |
   | boolean | true/false                      | Boolean   |                                                              |

### 程序结构

1. assert(断言)，是一个包含布尔表达式的语句，如果表达式计算为false，则报告Assertionerror，用于调试。默认情况下禁用。一般情况下，assert用于保证程序最基本、最关键的正确性，通常在开发和测试时开启，断言不应该用于验证传递给公有方法的参数，不过可以在公有方法和非公有方法利用断言测试后置条件，以及，断言不应该以任何方式改变程序的状态
2. main方法必须是public的

### 运算符

1. 优先级顺序（高-低）：
   - . （）
   - ++ --
   - new
   - /  * %
   - \+ -
   - \>> << 
   - \> <  >=  <=
   - == != 
   - &
   - ^
   - !
   - &&
   - ||
   - ? :
   - = += -= *= /= %= ^=
   - &= <<= >>=
2.  && （逻辑与）和 &（布尔逻辑与） 区别：&&前后算第一个，如果前面的为false，后面的就不用计算，&需要从头算到尾，|| 和 | 同理，即逻辑与或为短路的，布尔逻辑不是短路的

### 异常

1. Java程序中所有抛出的异常都从Throwable派生而来，Throwable有两个直接子类：Error和Exception
   1. Error类对象由Java虚拟机生成并抛弃，通常Java程序不会对这类错误处理。程序运行时本身无法解决，只能通过其他程序干预。
   2. Exception类对象由Java程序处理或抛弃，有不同的子类对应于不同类型的异常。程序本身可以解决，由异常代码调整程序运行方向。
2. 异常关键字：try，catch，throw，throws，finally
   1. try，catch，finally均不能单独使用，catch可以有一个或多个，finally只能有一个
   2. throw和throws区别：
      - throw用于方法内部，用来抛出异常
      - throws用于方法外部，用来声明方法可能抛出异常
3. Java异常和C++异常的区别：
   - C++可以抛出任何类型的异常，如int，float等，也可以是用户自定义的抽象数据对象。缺乏规范和统一。
   - Java一般不需要重新定义自己的异常对象
4. final、finally、finalize区别
   1. final关键字：如果一个类被声明为final，那么他不能再派生出子类，不能作为父类被继承。一个类不能既被声明为final，又被声明为abstract。将变量声明为final，可以保证他们再使用中不被改变。初始化可以在两个地方：定义处，在final变量定义时就赋值；在构造函数中。被声明为final的函数也只能使用，不能重写
   2. finally：异常处理时使用，如果抛出异常，在有finally块的情况下会进入finally块。
   3. finalize：方法名，Java允许使用finalize()方法在垃圾回收器将对象从内存中清除出去之前做必要的工作。在Object类中定义，所有的类都继承了他。在垃圾收集器删除对象之前对这个对象调用的。

### 反射

1. 反射主要是指程序可以访问、检测和修改他本身的状态或行为的一种能力。Java中的反射设计一种强大的工具，能够创建灵活的代码，可以在运行时装配。无需在组件之间进行链接。Java反射机制主要提供的功能：
   - 在运行时判断任意一个对象所属的类
   - 在运行时构造任意一个类的对象
   - 在运行时判断任意一个类所具有的成员变量和方法
   - 在运行时调用任意一个对象的方法

## 传递与引用

### 传值与传引用

1. 不管Java参数的类型是什么，一律传递参数的副本。对于基本类型变量，Java传递值的副本；对于对象类型变量，Java传递引用的副本

### 静态变量与私有变量

1. 静态对象和非静态对象的区别：

   |          | 静态对象             | 非静态对象             |
   | -------- | -------------------- | ---------------------- |
   | 拥有属性 | 类共同拥有的         | 类各对象独立拥有的     |
   | 内存分配 | 内存空间上独立       | 在各个附属类里分配     |
   | 分配顺序 | 先分配静态对象的空间 | 再对非静态对象分配空间 |

2. 在任意多个类的实例中，一个静态变量的实例只存在一个。

3. 定义在类里面的变量会被赋予一个默认的值。

4. 静态方法不能调用非静态变量，非静态方法可以引用静态变量。原因：静态方法不属于对象，属于类，不需要实例化；而非静态变量属于对象，需要先实例化。

### 输入/输出流

1. I/O流：

   ![IO流](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTA1MTgyMzQyMjI3)

   - 按照数据类型分为：字符流和字节流
   - 按照数据流向分为：输入流和输出流

2. 输入流（InputStream）进行读操作，输出流（OutputStream）进行写操作。输入字节流InputStream，输出字节流OutputStream，都是抽象类。

3. InputStream：

   - ByteArrayInputStream、StringBufferInputStream、FileInputStream 是三种基本的介质流，分别从Byte数组、StringBuffer和本地文件中读取数据
   - PipeInputStream从与其他线程共用的管道中读取数据
   - ObjectInputStream和所有FilterInputStream的子类都是装饰流

4. OutputStream：

   - ByteArrayOutputStream、FileOutputStream是两种基本的介质流，分别向Byte数组和本地文件中写入数据
   - PipedOutputStream是向其他线程共用的管道中写入数据
   - ObjectOutputStream和所有FilterOutputStream的子类都是装饰类

5. 字符串写入文件：

   ```java
   import java.io.*;
   public class Test{
       public static void main(String args[]){
           try{
               FileOutputStream out - new FileOutputStream("file.txt");
               out.write("asdadhoasdhu".getBytes());
               out.close();
           }catch(IOException e){}
       }
   }
   ```

6. 以InputStream（输入流）/OutputStream（输出流）为后缀的是字节流；以Writer（输出流）/Reader（输入流）为后缀的是字符流

### 序列化

1. **序列化**是把Java对象转换成字节序列的过程，**反序列化**就是把字节序列恢复为Java对象的过程。**为什么**需要序列化：在两个Java进程进行通信时进行Java序列化与反序列化，发送方把这个Java对象转换为字节序列，在网络上传送，接收方需要从字节序列上恢复处Java对象。**好处**：实现了数据的持久化，可以把数据永久地保存到硬盘上；利用序列化实现远程通信，在网络上传送对象的字节序列。
2. 如何实现Java的序列化：通过实现Serializable接口或Externalizable接口就可以实现。想要实现序列化对象必须先创建一个OutputStream，然后把他嵌进ObjectOutputStream，这时就能用writeObject()方法把对象写入OutputStream，读的时候需要把InputStream嵌入ObjectInputStream中，再调用readObject。不过这样读出来的只是一个Object的reference，在用之前还得先下传。
