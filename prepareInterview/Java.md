## 1. Java类加载过程

## 2. Java内存结构

##### 1. 虚拟机栈

​	Java方法执行的内存模型。每个方法执行时会创建一个栈帧，存储局部变量表、操作数栈、动态链接、方法出口等信息。

​	局部变量表中存放的有：编译器可知的各种基本数据类型、对象引用和returnAddress类型。

##### 2. 本地方法栈

​	与虚拟机栈相似，不过这个是为native方法服务的。

##### 3. 程序计数器

​	当前线程所执行的字节码的行号指示器。

##### 4. 堆

所有对象实例和数组都要在堆上分配。

此处还有一个直接内存：通过一个存储在堆中的DirectByteBuffer对象作为这块内存的引用进行操作。

##### 5. 方法区

存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等信息。其中有一个位置叫运行时常量池，存放编译期生成的各种字面量和符号引用。

```java
String str1 = new StringBuilder("计算机").append("软件").toString();
System.out.println(str1 == str1.intern()); // jdk1.6及之前为FALSE，1.7之后为true
String str2 = new StringBuilder("ja").append("va").toString();
System.out.println(str2 == str2.intern()); // 一直为false
```

上面这段代码执行后，运行结果如注释中所说。

**原因：**

jdk1.6及之前，intern()方法会将首次遇到的字符串实例复制到永久代中，返回的也是永久代这个字符串实例的引用，而由StringBuilder创建的字符串实例在堆上，所以必然不是一个同一个引用，返回false。

而jdk1.7及之后的虚拟机，不会再复制实例，而是在常量池里记录首次出现的实例引用，因此intern()返回的引用与StringBuilder创建的那个字符串实例是同一个。但是对str2的判断之所以为false，是因为java这个字符串之前已经出现过了，字符串常量池中已经有它的引用了，不会再记录一次StringBuilder在堆上所创建的实例的引用，不符合首次出现的规则，而“计算机软件”则是首次出现的，因此为true。

**扩展：**

1. java这样一个字符串为何之前就存在？

   java线程初始化完成之后，虚拟机会自动调用System类的initializeSystemClass方法，其中有一行代码为sun.misc.Version.init();，而Version类中有静态变量：

   ```java
   private static final String launcher_name = "java";
   private static final String java_version = "1.8.0_202";
   private static final String java_runtime_name = "Java(TM) SE Runtime Environment";
   private static final String java_profile_name = "";
   private static final String java_runtime_version = "1.8.0_202-b08";
   ```

   因此，这些字符串会自动加载进字符串常量池。

2. 

```
String str1 = new StringBuilder("计算机软件").toString();
System.out.println(str1 == str1.intern()); // 一直为false
```

“计算机软件”这个字符串也是一开始就不存在的字符串，为何也是false呢？因为此处的字符串在编译时期就已经确定了，类加载时就会将这个字符串加载到常量池。

而第一个例子中，“计算机”和“软件”也会一开始加载到堆中，而新出现的“计算机软件”是由stringBuilder这个对象调用toString方法来创建的，因此是新字符串。

## 3. 创建对象的过程：

1. 先去方法区的常量池查看是否有这个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过，没有的话要先进行类加载过程。
2. 通过后，虚拟机将为新生对象分配内存（内存大小在类加载完成后即可完全确定），分配方式有“指针碰撞”和“空闲列表”。分配时的并发安全问题，用CAS(compare and swap)或者TLAB(Thread local allocation buffer)方法保证。

   3. 虚拟机为分配到的内存空间都初始化为零值。
      进行必要的设置：如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希吗、对象的GC分代年龄等信息。
   4. 执行init方法。

## 4. 锁升级过程

无锁态(01) -> 偏向锁(01) -> 轻量级锁(自旋锁)(00) --(自旋 > 10次 / 自旋线程数 > CPU核数/2, 1.6之后，加上了自适应自旋，JVM自己控制，也就是自适应自旋)--> 重量级锁(内核态级别的锁)(10)

## 5. GC相关

### 如何判断对象已死？

1. 引用计数法：给对象添加一个引用计数器，有引用的地方就加一，引用失效时减一
   - 效率高
   - 不能解决循环引用的问题
2. 可达性分析算法：一系列GCRoots对象作为起始点
   - 虚拟机栈中引用的对象
   - 方法区中类的静态属性和常量引用的对象
   - 本地方法栈中JNI（即一般说的Native方法）引用的对象。

### 四种引用关系

1. 强引用：平常代码中类似“Object obj = new Object();”这类的引用都是强引用，只要存在就不会被GC收集器回收。
2. 软引用：描述一些有用但非必须的对象。jdk提供了SoftReference类来实现软引用。在要发生内存溢出之前，会将软引用的对象列入垃圾回收范围之内，如果这次回收后还是没有足够的内存，才会抛出内存溢出的异常。基本上用于缓存。
3. 弱引用：ThreadLocal、WeakHashMap会用到。也是用来描述非必须对象的，GC收集器发现就回收，jdk提供了WeakReference类来实现弱引用。
4. 虚引用：get方法拿不到对象，唯一作用就是管理堆外内存。创建虚引用时需要制定一个队列，当队列中有数据的时候，证明有对象被回收了。

### 回收方法区

1. 回收内容：废弃常量和无用的类
2. 回收常量和回收一个普通的java对象非常类似，会查看是否有地方引用到这个常量
3. 回收类的条件比较苛刻，如下：
   1. 该类所有实例均已回收
   2. 加载该类的classLoader已经被回收
   3. 该类对应的java.lang.Class对象没有在任何地方被引用，且无法在任何地方通过反射访问该类的方法

### 垃圾收集算法

#### 标记-清除算法

先标记，再清除。但这两个过程的效率都不高，且会产生大量不连续的碎片，容易在后续为一个大对象分配内存时提前出发一次垃圾回收。

#### 复制算法(针对新生代)

将堆分为两块，这块用完了就把还存活的对象复制到另一块。

目前大部分的划分方法：

Eden：Survivor = 8 : 1，且Eden空间只有一块，Survivor空间有两块。每次只使用Eden区和其中一块Survivor区。

回收时会将Eden区和Survivor区存活的对象复制到另一块Survivor区，然后清理掉刚刚使用的Eden区和Survivor区。因此每次新生代中可用的内存空间为整个新生代容量的90%。

此处有一个分配担保的问题：如果Eden区和第一块Survivor区存活的对象在第二块Survivor区盛不下，那么会有一部分对象通过分配担保机制被复制到老年代。

#### 标记-整理算法(针对老年代)

与标记-清除算法不同，标记-整理算法标记完成之后，会将所有存活的对象向一端移动，然后直接清理完边界以外的内存。

#### 内存分配、回收策略

1. 对象优先在Eden区分配

2. 大对象直接进入老年代

3. 长期存活的对象将进入老年代

   对象在Survivor空间每熬过一次MinorGc，年龄就+1，当增加到一定程度的时候（默认为15岁，最大其实也就是15岁，因为年龄信息存在于对象头中，只有四位，最大就是15），就会被晋升到老年代中。

4. 动态对象年龄判定

   如果在Survivor区中，相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于等于该年龄的对象就可以直接进入老年代。

5. 空间分配担保

   1. minorGC前，会先判断一下老年代的最大可用连续空间是否大于新生代所有对象总空间

   2. 如果成立则代表MinorGC是安全的，否则就要去看下HandlePromotionFailure设置的值是否为允许担保失败。

   3. 如果允许则会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于则会尝试进行这次不安全的minorGc，如果小于或者不允许担保失败，则会进行一次FullGc。

      此处的冒险指的就是上文提到的，如果Eden区和第一块Survivor区存活的对象在第二块Survivor区盛不下，那么会有一部分对象通过分配担保机制被复制到老年代。

### OOM类型总结

1. OutOfMemoryException
   - 堆：创建对象时内存不够用了
   - 方法区：由于此处主要存放一些类的定义，因此像大量使用JSP、GcLib等动态代理技术时容易出现
   - 运行时常量池：使用String.intern()方法时此区容易溢出
2. StackOverflowException
   - 虚拟机栈：当方法调用栈过深时，会报这个异常
   - 本地方法栈；同上
3. DirectMemoryError：堆外内存溢出，可能是可请求的内存大小超过了设置值，也可能是超出了实体机内存容量。

## 6. 虚拟机性能监控与故障处理工具

- jps：JVM Process Status Tool，显示系统中所有的HotSpot虚拟机进程
- jstat：JVM Statistics Monitoring Tool，收集HotSpot虚拟机各方面的运行数据
- jinfo：Configuration Info for Java，显示虚拟机配置信息
- jmap：生产虚拟机的内存转储快照
- jhat：java heap dump browser，分析heapdump文件
- jstack：stack trace for java，显示虚拟机的线程快照
- 可视化工具：
  - jvisualvm
  - jconsole

## 7. 设计一个线程池
## 8. HashMap数据类型

### 	在多线程环境下会出现的问题

​		由于hashmap底层是用数组+链表/红黑树组成的，而只有当链表长度大于8且总容量大于threshold时才会将链表进化为红黑树，而在此之前是走的扩容来降低哈希冲突的。

​		因此，扩容时涉及到一个rehash的操作，rehash时，需要将原链表中的数据分流到两个链表中，此时涉及到了链表操作，在多线程情况下就会出现问题。下面详细探讨一下：

```java
/**
 * 重新hash的核心逻辑
 */
void transfer(Entry[] newTable)
{
    Entry[] src = table;
    int newCapacity = newTable.length;
    //下面这段代码的意思是：
    //  从OldTable里摘一个元素出来，然后放到NewTable中
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
} 
```

#### 正常的ReHash的过程

画了个图做了个演示。

- 我假设了我们的hash算法就是简单的用key mod 一下表的大小（也就是数组的长度）。

- 最上面的是old hash 表，其中的Hash表的size=2, 所以key = 3, 7, 5，在mod 2以后都冲突在table[1]这里了。

- 接下来的三个步骤是Hash表 resize成4，然后所有的<key,value> 重新rehash的过程

  ![HashMap01](.\img\HashMap01.jpg)

  #### 并发下的Rehash

  **1）假设我们有两个线程。**我用红色和浅蓝色标注了一下。

  我们再回头看一下我们的 transfer代码中的这个细节：

  ```java
  do {
  	Entry<K,V> next = e.next; // <--假设线程一执行到这里就被调度挂起了
  	int i = indexFor(e.hash, newCapacity);
  	e.next = newTable[i];
  	newTable[i] = e;
  	e = next;
  } while (e != null);
  ```

  而我们的线程二执行完成了。于是我们有下面的这个样子。

  ![HashMap02](.\img\HashMap02.jpg)

  注意，**因为Thread1的 e 指向了key(3)，而next指向了key(7)，其在线程二rehash后，指向了线程二重组后的链表**。我们可以看到链表的顺序被反转了。

  **2）线程一被调度回来执行。**

  - **先是执行 newTalbe[i] = e;**

  - **然后是e = next，导致了e指向了key(7)，**

  - **而下一次循环的next = e.next导致了next指向了key(3)**

    ![HashMap03](.\img\HashMap03.jpg)

    **3）一切安好。**

    线程一接着工作。**把key(7)摘下来，放到newTable[i]的第一个，然后把e和next往下移**。

![HashMap04](.\img\HashMap04.jpg)

​			**4）环形链接出现。**

​			**e.next = newTable[i] 导致 key(3).next 指向了 key(7)**

​			**注意：此时的key(7).next 已经指向了key(3)， 环形链表就这样出现了。**

![HashMap05](.\img\HashMap05.jpg)

**于是，当我们的线程一调用到，HashTable.get(11)时，悲剧就出现了——Infinite Loop。**

## 9. Volatile关键字干什么用的和底层原理，CAS干什么用的以及原理

volatile关键字可用来修饰类的属性

## 10. 详解equals()方法和hashCode()方法

1. equals方法在Object类中的具体实现如下：

   ```java
   public boolean equals(Object obj) {
       return (this == obj);
   }
   ```

2. hashCode方法在Object类中的方法声明如下：

   ```java
   public native int hashCode();
   ```

   ​		总的来说，equals方法用来比较两个对象是否相等，而hashCode方法主要在Hash类中起作用，用来定位要放到哪个桶里面。

   ​		当我们向hashMap中put一个数据时，首先会拿到这个key的hashCode，用这个hashCode去直接定位这个object在表中的位置。如果该位置没有对象，则直接插入；否则会调用equals方法比较这些对象与object是否相等，如果相等，则不需要做任何操作，否则需要插入object。

   ​		因此，equals与hashCode的判断结果应该是相等的，否则：

    	1. 如果equals相等而hashCode不等，则会出现一个对象被映射到了多个hash位置
    	2. 如果hashCode相等而equals不等，则会出现一个对象被多次存储的现象

## 11. 原子类的功能，实现原理。
## 12. 设计一个拥有热加载功能的IOC容器
## 13. Java中的Condition类是用来干什么的

