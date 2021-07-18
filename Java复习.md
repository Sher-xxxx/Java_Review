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

