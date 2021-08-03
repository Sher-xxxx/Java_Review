

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

## 递归 条件 概率

### 典型递归问题

1. 斐波那契数列 

### 循环与条件

1. for循环可以不使用"{}"，但是**仅限于执行语句**，变量声明语句必须使用"{}"

2. 例题：查找100以内所有的素数

   方法：筛选法，从头到尾把当前数的倍数筛选掉，比如2的倍数4,6,8,10.....都筛选掉，剩余的即为素数
   
   ```java
   public void sushu(){
       int[] a = new int[101];
       int i, j = 2;
       while (j < 101) {
           if (a[j] == 0) {
               for (i = j + 1; i < 101; i++) {
                   if (i % j == 0) {
                       a[i] = 1;
                   }
               }
           }
           j++;
       }
   
       for (int k = 0; k < 101; k++) {
           if (k >= 2 && a[k] == 0) {
               System.out.println(k);
           }
       }
   }
   ```
   



## Java内存管理

### 垃圾收集

1. **Java垃圾回收机制：**在Java中，程序员**不需要显示地释放一个对象的内存**，而是由虚拟机自动执行（优点：避免引起内存泄漏）。在JVM中，有一个垃圾回收线程，它是低优先级的，在正常情况下不会执行，只有在虚拟机空闲或当前堆内存不足时才会触发执行，扫描哪些没有被任何引用的对象，并将他们添加到要回收的集合中进行回收。
2. **GC**：垃圾收集（Gabage Collection），Java中使用GC来监视Java程序的运行，当对象不再使用时会自动释放后对象所使用的内存，Java使用一系列软指针来跟踪对象的各个引用，并用一个对象表将这些软指针映射为对象的引用。**软指针**不直接指向对象，而是指向对象的引用。使用软指针，GC能够以单独的线程在后台运行，并依次检查每个对象，通过更改对象表项，GC可以标记对象、移除对象、移动对象或检查对象。
3. 调用System类中的静态gc()方法可以运行垃圾收集器，但这样并不能保证立即回收指定对象。
4. Java的垃圾回收机制是为所有的Java应用进程服务的，任何一个进程都不能命令垃圾回收机制做什么、怎么做或做多少。在JVM垃圾收集器收集一个对象之前，一般要求程序调用适当的方法释放资源，但在没有明确释放资源的情况下，Java的finalize()默认机制可以终止该对象来释放资源。
5. 在Java语言中，判断一块内存空间是否符合垃圾收集器收集标准的标准只有两个：
   1. 给对象赋予了空值null，以后再也没有调用过
   2. 给对象赋予了新值，即重新分配了内存空间

### 内存管理

1. Java所有对象都在堆中，对象内存的分配是由程序完成的，内存的释放是由GC完成的。GC为了能够正确地释放对象，必须监控每个对象的运行状态，包括申请、引用、被引用、赋值等。释放对象的根本原则是该对象不再被引用。
2. Java内存泄漏：①对象是可达的，在有向图中，存在通路可以于其相连；②对象是无用的，即程序以后不会再使用这些对象。
3. 内存泄漏原因：
   1. 全局集合：必须注意管理存储库的大小，必须有某种机制使得存储库中移除不再需要的数据。
   2. 缓存：缓存是一种用于快速查找已经执行的操作结果的数据结构，如果一个操作执行起来很慢，对于常用的输入数据就可以将操作的结果缓存，并在下次调用操作时调用缓存的数据。缓存是以动态方法实现的，方法如下：
      - 检查结果是否在缓存中，在就返回结果
      - 不在缓存中就进行计算
      - *将计算出来的结果添加到缓存中*，会有潜在的内存泄漏→如果缓存所占的空间过大，就移除缓存最久的结果，将计算的结果添加到缓存中
   3. ClassLoader：ClassLoader本身可以关联许多类及其静态字段，所以就有许多内存被泄漏

### clone

1. Cloneable和Serializable一样都是标记型接口，内部没有方法和属性，implements Cloneable表示该对象能被克隆，能够使用Object.clone()方法，如果没有引用该接口就会报错CloneNotSupportedException。

2. Object类的clone()是protected的，不能直接调用，可以被子类调用。Object默认实现的是一个浅复制，

   ```java
   @HotSpotIntrinsicCandidate
   protected native Object clone() throws CloneNotSupportedException;
   ```

## 面向对象

### 基础知识

1. 对象是同类事物的一种抽象表现形式，而实例时对象的具体化，一个对象可以实例化很多实例。 
2. 类方法：
   - clone(): 创建并返回此对象的副本
   - equals(Object obj): 其他对象与此对象是否相等
   - finalize()：当垃圾回收器确定不存在该对象的更多引用时，由对象 的垃圾回收器调用此方法
   - getClass(): 返回对象的类
   - hashCode(): 返回该对象的哈希值
   - notify(): 唤醒在等待的单个线程
   - notifyAll(): 唤醒所有等待的线程
   - toString(): 返回该对象的字符串表示
   - wait(): 线程等待，直到调用notify()或notifyAll()
   - wait(long timeout):  线程等待，直到调用notify()或超过timeout
   - wait(long timeout, int nanos): 线程等待，直到...或其他线程中断当前线程
3. Java创建对象的方式：
   1. new语句创建对象
   2. 反射手段，调用java.lang.Class或java.lang.reflect.Constructor类的newInstance()实例方法
   3. 调用对象的clone方法
   4. 使用反序列化手段，调用java.io.ObjectInputStream对象的readObject()方法
4. 内部类：静态内部类意味着创建一个static内部类的对象，不需要一个外部类对象，不能从static内部类的一个对象访问一个外部类对象

### 集合类

1. Java容器类库主要有两种类型：Collection和Map

   ![img](https://pic3.zhimg.com/80/v2-37a924e5830ed38d9c76296314cb4baa_1440w.png)

2. Collection类型是单个元素的集合，Map持有key-value关联。所有的Java容器类都可以自动调整自己的尺寸。

3. List、Set、Map将所有的对象一律视为Object类型。

4. Collection、List、Set、Map都是接口，不能被实例化。

5. Vector容器确切地知道它所持有的对象是什么类别，不进行边界检查。

6. HashMap和Hashtable的区别：

   1. HashMap没有分类或排序，允许一个null键和多个null值。
   
   2. Hashtable类似于HashMap，但是不允许null键和null值，比HashMap慢，因为它是同步的。
   
   3. Hashtable继承自Dirtionary类，HashMap是Map接口的一个实现。
   
      ![image-20210803113028457](C:\Users\86132\AppData\Roaming\Typora\typora-user-images\image-20210803113028457.png)
   
      ![image-20210803112926417](C:\Users\86132\AppData\Roaming\Typora\typora-user-images\image-20210803112926417.png)
   
   4. Hashtable的方法是Synchronize的，在多个线程访问Hashtable时，不需要为自己的方法实现同步，而HashMap需要。

#### List

1. List所有的元素**可以重复**且有序，主要分为**ArrayList**和**LinkedList**，前者底层使用数组实现的List，后者使用链表实现的List
2. Vector是一个已经被弃用的类，因为他是线程同步的，访问速度慢。Stack满足后进先出，LinkedList可以实现所有的栈功能。

![image-20210726200701529](C:\Users\86132\AppData\Roaming\Typora\typora-user-images\image-20210726200701529.png)

#### Queue

1. 队列是一个先进先出的数据结构，LinkedList提供了方法支持队列操作，并且实现了Queue接口，可以通过LinkedList向上转型为Queue

#### Set

1. set中的元素不可重复且无序
2. HashSet底层使用散列函数，在查询方面有优化
3. TreeSet底层使用红黑树

#### Map

1. Map是一种使用键值对存储的数据结构，Map可以多维扩展，key唯一
2. HashMap更适合查找、删除、插入
3. TreeMap更适合遍历

线程安全的集合类：Vector，Stack，Hashable，java.util.concurrent下的所有集合类

### 多态的概念

1. 什么是多态：多态可以认为是”一个接口，多种方法“，在程序运行中才决定调用哪个函数。多态性是允许将父对象设置成为和它的一个或多个子对象相等的技术，赋值之后父对象就可以根据当前赋值给它的子对象的特性以不用的方式运作。
2. 重载和覆盖的区别：覆盖（override）是指派生类重写基类的函数，重写的函数必须有一致的参数表和返回值；重载（overload）是指编写一个与已有函数同名但参数表不同的函数。

## 继承与接口

### 不能继承的情况

1. 匿名内部类是没有名字的内部类，不能继承其他类，但一个内部类可以作为一个接口，由另一个内部类实现
2. final类不能被继承，一个final类中的所有方法都默认为final。

### 抽象类和接口

1. 抽象类只能作为其他类的基类，不能直接被实例化，对抽象类不能使用new操作符，抽象类如果含有抽象的变量或值，他们要么是null类型，要么包含对非抽象类的实例的引用。

2. 抽象类允许包含抽象成员，但不是必须的，抽象类也可以有非抽象方法。

3. 抽象类不允许是final的。

4. 如果一个非抽象类从抽象类中派生，则必须通过覆盖来实现所有继承来的抽象成员。

5. AbstractList继承图

   ![image-20210727155259846](C:\Users\86132\AppData\Roaming\Typora\typora-user-images\image-20210727155259846.png)

   

6. HashSet继承图

   ![image-20210727155426697](C:\Users\86132\AppData\Roaming\Typora\typora-user-images\image-20210727155426697.png)

7. AbstractSet继承图

   ![image-20210727155713785](C:\Users\86132\AppData\Roaming\Typora\typora-user-images\image-20210727155713785.png)

8. 接口用于描述系统对外提供的所有服务，因此接口中的成员常量和方法必须是public公开类型的。接口中的方法仅仅描述能做什么，但不指明如何去做，因此接口中的方法都是抽象的。接口只有静态（static）变量。接口中的变量是final类型。接口的**方法**默认是**public abstract**，**属性**默认是**public static final**常量，且必须**赋初值**。



