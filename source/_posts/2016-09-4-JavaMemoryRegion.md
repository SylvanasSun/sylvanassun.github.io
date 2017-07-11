---
layout:     post
title:      "Java虚拟机中的内存区域"
subtitle:   "JVM MemoryRegion"
date:       2016-09-04 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Java
    - JVM
tags:
    - JVM
    - Java
---

### 概述


----------


&nbsp;&nbsp;对于C/C++程序员来说需要自己负责每一个对象生命开始到终结的维护.而对于Java程序员来说,则可以在Java虚拟机自动内存管理机制的帮助下,不需要为每一个new的对象去写delete/free代码,由虚拟机管理内存可以让我们把注意力放在实现业务逻辑上.

&nbsp;&nbsp;但也正是因为Java虚拟机接管了内存控制的权力,所以一旦出现内存泄漏或溢出方面的异常问题,就需要了解虚拟机如何使用与维护内存,这样才能更好的排查错误解决异常.

### 运行时数据区域


----------


&nbsp;&nbsp;Java虚拟机在执行Java程序时会把它所管理的内存划分为若干个不同的数据区域.这些区域都有着各自的用途,以及创建和销毁的时间,有的区域则会随着虚拟机进程的启动而存在,有些区域则会依赖于用户线程的启动和结束而创建和销毁.

&nbsp;&nbsp;根据<<Java 虚拟机规范>>的规定,Java虚拟机管理的内存将会包括以下图所示的几个运行时数据区域.

![运行时数据区域](http://ww2.sinaimg.cn/mw690/63503acbjw1f7j0psfbuaj20l90degne.jpg)

#### 程序计数器

&nbsp;&nbsp;程序计数器(Program Counter Register)可以看作为当前线程所执行的字节码的行号指示器,它占用的内存很小.

&nbsp;&nbsp;字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令,分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖程序计数器来完成.

&nbsp;&nbsp;在Java虚拟机中,多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的.在任何一个确定的时刻,一个处理器(或多核CPU中的一个内核)都只会执行一条线程中的指令.为了线程切换后能够恢复到正确的执行位置,每一条线程都需要有一个独立的程序计数器,各条线程之间计数器互不影响,独立存储,这种类型的内存区域为"线程私有"的内存.

&nbsp;&nbsp;如果线程正在执行的是一个Java方法,程序计数器记录的是正在执行的虚拟机字节码指令的地址;如果正在执行的是一个Native方法,程序计数器值则为空(Undefined).

&nbsp;&nbsp;程序计数器内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError异常情况的区域.

#### 虚拟机栈

&nbsp;&nbsp;Java虚拟机栈(Java Virtual Machine Stacks)与程序计数器一样,是线程私有的内存,它的生命周期与线程相同.

&nbsp;&nbsp;虚拟机栈描述的是Java方法执行的内存模型:即每个Java方法在执行的同时都会创建一个栈帧(Stack Frame)用于存储局部变量表、操作数栈、动态链接、方法出口等信息.栈帧是方法运行时的基础数据结构.

&nbsp;&nbsp;每一个方法从调用直至执行完成的过程,就对应着一个栈帧在虚拟机栈中入栈到出栈的过程.

&nbsp;&nbsp;栈帧中的局部变量表存放了编译期可知的各种基本数据类型(boolean、byte、char、short、int、float、long、double)、对象引用类型(reference类型,它不等同于对象本身,可能是一个指向对象起始地址的引用指针,也可能是指向一个代表对象的句柄或其他与此对象相关的位置)和returnAddress类型(指向了一条字节码指令的地址).

&nbsp;&nbsp;64位长度的long和double类型的数据会占用2个局部变量空间(Slot),其他的数据类型只占用1个.局部变量表所需的内存空间是在编译期间完成分配的.当进入一个方法时,这个方法需要在栈帧中分配多大的局部变量空间是完全确定的,在方法运行期间不会改变局部变量表的大小.

&nbsp;&nbsp;Java虚拟机规范对虚拟机栈区域规定了两种异常情况:

 1. 如果线程请求的栈深度大于虚拟机所允许的深度,将会抛出StackOverflowError异常.

 2. 如果虚拟机可以动态扩展,并在扩展时无法申请到足够的内存时,将会抛出OutOfMemoryError异常.

#### 本地方法栈

&nbsp;&nbsp;本地方法栈(Native Method Stack)与虚拟机栈基本相似.它们之间的区别只不过是虚拟机栈为虚拟机执行Java方法(也就是字节码)服务,而本地方法栈则为虚拟机使用到的Native方法服务.

&nbsp;&nbsp;Java虚拟机规范对本地方法栈没有强制的规定,具体的虚拟机可以自由实现它.例如Sun HotSpot虚拟机甚至将本地方法栈与虚拟机栈合二为一.

&nbsp;&nbsp;与虚拟机栈相同,本地方法栈也会抛出StackOverflowError和OutOfMemoryError异常.

#### Java堆

&nbsp;&nbsp;在大多数情况下,Java堆(Java Heap)是Java虚拟机所管理的内存中最大的一块.它是被所有线程共享的一块内存区域,在虚拟机启动时创建.Java堆的唯一目的就是存放对象实例,几乎所有的对象实例都在这里分配内存.

&nbsp;&nbsp;但随着JIT编译器的发展与逃逸分析技术逐渐成熟,栈上分配、标量替换优化技术将会导致一些微妙的变化发生,所有对象都分配在堆上也渐渐变得不是那么"绝对"了.

&nbsp;&nbsp;Java堆也是垃圾收集器管理的主要区域,也可以称为GC堆(Garbage Collected Heap).由于现在垃圾收集器基本都采用分代收集算法,所以Java堆中可以细分为新生代和老年代(再细化的有Eden空间、Form Survivor空间、To Survivor空间等).

&nbsp;&nbsp;从内存分配的角度来看,线程共享的Java堆中可能划分出多个线程私有的分配缓冲区(Thread Local Allocation Buffer TLAB).

&nbsp;&nbsp;Java虚拟机规范规定,Java堆可以处于物理上不连续的内存空间中,只要逻辑上是连续的即可.在实现时,既可以实现是固定大小的,也可以是可扩展的,如果在堆中没有内存完成实例分配,并且堆也无法再扩展时,将会抛出OutOfMemoryError异常.

#### 方法区

&nbsp;&nbsp;方法区(Method Area)与Java堆一样是各个线程共享的内存区域,它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据.Java虚拟机规范中把方法区描述为堆的一个逻辑部分,但它有一个别名叫Non-Heap(非堆),用于与Java堆区分开来.

&nbsp;&nbsp;在HotSpot虚拟机中,方法区被称作为"永久代(Permanent Generation)".因为HotSpot将GC分代收集扩展至了方法区,或者说是使用永久代来实现方法区.这样HotSpot的垃圾收集器可以像管理Java堆一样管理这部分内存,能够省去编写方法区的内存管理代码工作.但这样做法会更容易遇到内存溢出问题.所以HotSpot官方决定放弃永久代并逐步改为采用Native Memory来实现方法区的规划.在JDK1.7的HotSpot中,已经把原本放在永久代的字符串常量池移出.

&nbsp;&nbsp;Java虚拟机规范规定,当方法区无法满足内存分配需求时,将抛出OutOfMemoryError异常.

#### 运行时常量池

&nbsp;&nbsp;运行时常量池(Runtime Constant Pool)是方法区的一部分.Class文件中除了有类的版本、字段、方法、接口等描述信息外,还有一项信息是常量池(Constant Pool Table),它用于存放编译期生成的各种字面量和符号引用,这部分内容将在类加载后进入方法区的运行时常量池中存放.一般来说,除了保存Class文件中描述的符号引用外,还会把翻译出来的直接引用也存储在运行时常量池中.

&nbsp;&nbsp;运行时常量池相对于Class文件中常量池的另外一个重要特征是具备动态性.Java语言并不要求常量一定只有编译期才能产生,运行期间也可能将新的常量放入池中,这种特性利用得比较多的便是String类的intern()方法.

&nbsp;&nbsp;当运行时常量池无法申请到内存时会抛出OutOfMemoryError异常.

#### 直接内存

&nbsp;&nbsp;直接内存(Direct Memory)并不是虚拟机运行时数据区域的一部分,也不是Java虚拟机规范中定义的内存区域.但是部分内存也被频繁地使用,而且也可能导致OutOfMemoryError异常.

&nbsp;&nbsp;在JDK1.4中新加入了NIO(New Input/Output)类,引入了一种基于通道(Channel)与缓冲区(Buffer)的I/O方式,它可以使用Native函数库直接分配堆外内存,然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作.这样避免了在Java堆中和Native堆中来回复制数据,提高了性能.

&nbsp;&nbsp;由于本机直接内存不会受到Java堆大小的限制,但它还是会受到本机总内存大小以及处理器寻址空间的限制.当我们配置虚拟机参数时,会根据实际内存设置-Xmx等参数信息,但经常会忽略直接内存,导致各个内存区域总和大于物理内存限制,从而导致动态扩展时出现OutOfMemoryError异常.

### OutOfMemoryError异常案例


----------


#### Java堆溢出

```java

/**
 * Java堆内存溢出,Java堆内存的OOM异常是实际应用中常见的内存溢出异常情况.
 *
 * 堆内存用于存储对象实例,只要不断地创建对象,并且保证GC Roots到对象之间有可达路径来避免
 * 垃圾回收机制清除这些对象,这样就可以在对象数量到达最大堆的容量限制后产生内存溢出异常.
 *
 * 以下启动参数中,-Xms20 -Xmx20m 将堆的最小值与最大值设置为一样的,既可避免自动扩展.
 * -XX:+HeapDumpOnOutOfMemoryError可以让虚拟机在出现内存溢出异常时Dump出当前的内存堆转储快照以便分析.
 *
 * VM Args: -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 * <p>
 * Created by sylvanasp on 2016/9/4.
 */
public class HeapOOM {

    static class OOMObject {

    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<OOMObject>();

        while (true) {
            list.add(new OOMObject());
        }
    }

}

```

#### 虚拟机栈溢出

```java

/**
 * 在单线程中,以下两种方法均无法让虚拟机产生OutOfMemoryError异常,尝试的结果均为StackOverflowError异常.
 *
 * 1.使用-Xss参数减少栈内存容量.结果为StackOverflowError异常,出现时输出的堆栈深度相应缩小.
 *
 * VM Args: -Xss128k
 *
 * Created by sylvanasp on 2016/9/4.
 */
public class JavaVMStackSOF {

    private int stackLength = 1;

    /**
     * 定义大量的本地变量,增大此方法帧中本地变量表的长度.
     * 结果:抛出StackOverflowError异常时输出的堆栈深度相应缩小.
     */
    public void stackLeak() {
        stackLength++;
        stackLeak();
    }

    public static void main(String[] args) throws Throwable {
        /**
         * 在单线程下,无论是栈帧太大还是虚拟机栈容量太小,当内存无法分配时,
         * 虚拟机抛出的都是StackOverflowError异常.
         */
        JavaVMStackSOF javaVMStackSOF = new JavaVMStackSOF();
        try {
            javaVMStackSOF.stackLeak();
        } catch (Throwable e) {
            System.out.println("stack length:" + javaVMStackSOF.stackLength);
            throw e;
        }
    }

}

```

```java

/**
 * 通过不断建立线程的方式产生内存溢出异常.
 * 但是这样产生的内存溢出异常与栈空间是否足够大并不存在任何联系.
 * 在这种情况下,为每个线程的栈分配的内存越大,反而越容易产生内存溢出异常.
 * 因为内存最后才由虚拟机栈和本地方法栈"瓜分".
 * 所以每个线程分配到的栈容量越大,可以建立的线程数量自然就越少,建立线程时就越容易把剩下的内存耗尽。
 *
 * 如果是建立过多线程导致的内存溢出,在不能减少线程数或者更换64位虚拟机的情况下.
 * 就只能通过减少最大堆和减少栈容量来换取更多的线程.
 *
 * VM Args: -Xss2M
 * <p>
 * Created by sylvanasp on 2016/9/4.
 */
public class JavaVMStackOOM {

    private void dontStop() {
        while (true) {

        }
    }

    public void stackLeakByThread() {
        while (true) {
            Thread thread = new Thread(new Runnable() {
                public void run() {
                    dontStop();
                }
            });
            thread.start();
        }
    }

    public static void main(String[] args) {
        JavaVMStackOOM javaVMStackOOM = new JavaVMStackOOM();
        javaVMStackOOM.stackLeakByThread();
    }

}

```

#### 方法区溢出

```java

/**
 * 方法区内存溢出
 * 方法区是用于存放Class的相关信息的,所以要让方法区产生内存溢出的基本思路为:
 * 在运行时产生大量的类去填满方法区,直到溢出.
 * 所以可以使用CGLib直接操作字节码运行时生成大量的动态类.
 *
 * VM Args : -XX:PermSize=10M -XX:MaxPermSize=10M
 *
 * Created by sylvanasp on 2016/9/4.
 */
public class JavaMethodAreaOOM {

    static class OOMObject {

    }

    public static void main(final String[] args) {
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setUseCache(false);
            enhancer.setCallback(new MethodInterceptor() {
                public Object intercept(Object o, Method method, Object[] objects,
                                        MethodProxy methodProxy) throws Throwable {
                    return methodProxy.invokeSuper(o,objects);
                }
            });
            enhancer.create();
        }
    }

}

```

#### 运行时常量池溢出

```java

/**
 * 方法区和运行时常量池溢出
 * 在JDK1.6及之前的版本中,由于常量池分配在永久代内.
 * 所以可以通过 -XX:PermSize和-XX:MaxPermSize限制方法区大小.
 * 从而间接限制其中常量池的容量.
 *
 * 而使用JDK1.7运行这段程序则不会得到相同的结果,while循环将一直进行下去.
 *
 * VM Args : -XX:PermSize=10M -XX:MaxPermSize=10M
 *
 * Created by sylvanasp on 2016/9/4.
 */
public class RuntimeConstantPoolOOM {

    public static void main(String[] args) {
        // 使用List保持常量池的引用,避免Full GC回收常量池行为.
        List<String> list = new ArrayList<String>();
        int i = 0;
        while (true) {
            /**
             * String.intern()是一个Native方法.
             * 它的作用是:
             * 如果字符串常量池中已经包含了一个等于此String对象的字符串,则返回代表池中这个字符串的String对象.
             * 否则,将此String对象包含的字符串添加到常量池中,并且返回此String对象的引用.
             */
            list.add(String.valueOf(i++).intern());
        }
    }

    /**
     * 这段代码在JDK1.6中运行,会得到两个false,而在JDK1.7中运行,会得到一个true和一个false.
     * 产生差异的原因为:
     * 在JDK1.6中,intern()方法会把首次遇到的字符串实例复制到永久代中,返回的也是永久代中这个字符串实例的引用.
     * 而由StringBuilder创建的字符串实例在Java堆上,所以必然不是同一个引用.所以返回false.
     *
     * 在JDK1.7中(或部分其他虚拟机,如JRockit),intern()实现不会再复制实例.
     * 只是在常量池中记录首次出现的实例引用,因此intern()返回的引用和由StringBuilder创建的字符串实例为同一个.
     * 对str2比较返回false是因为"java"这个字符串在执行StringBuilder.toString()之前已经出现过,
     * 字符串常量池中已经有它的引用了,不符合"首次出现"的原则.
     *
     */
    private void stringPool() {
        String str1 = new StringBuilder("Hello").append("World哈哈").toString();
        System.out.println(str1.intern() == str1);

        String str2 = new StringBuilder("ja").append("va").toString();
        System.out.println(str2.intern() == str2);
    }

}

```

#### 直接内存溢出

```java

/**
 * 直接内存溢出
 * DirectMemory容量可以通过 -XX:MaxDirectMemorySize指定.
 * 如果不指定,则默认与Java堆最大值(-Xmx)一致.
 *
 * VM Args : -Xmx20M -XX:MaxDirectMemorySize=10M
 *
 * Created by sylvanasp on 2016/9/4.
 */
public class DirectMemoryOOM {

    private static final int _1MB = 1024 * 1024;

    /**
     * 越过DirectByteBuffer类,直接通过反射获取Unsafe实例进行内存分配.
     * Unsafe.getUnsafe()方法限制了只有引导类加载器才会返回实例,
     * 即只有rt.jar中的类才能使用Unsafe的功能.
     *
     * 使用DirectByteBuffer类分配内存虽然也会抛出内存溢出的异常,
     * 但它抛出异常时并没有真正的向操作系统申请分配内存,而是通过计算得知内存无法分配,于是手动抛出异常.
     *
     * 而unsafe.allocateMemory()则可以真正申请分配内存.
     *
     */
    public static void main(String[] args) throws IllegalAccessException {

        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe) unsafeField.get(null);
        while (true) {
            unsafe.allocateMemory(_1MB);
        }
    }

}

```

### End


----------


> 资料参考于 <<深入理解 Java虚拟机 第二版>>.