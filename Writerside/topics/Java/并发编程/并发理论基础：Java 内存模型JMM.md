# 并发理论基础：Java 内存模型JMM

在{% post_link 'Java/并发编程/并发理论基础：缓存可见性、MESI' %}一文中分析了缓存可见性导致的并发问题。为了解决缓存可见性问题所以有了缓存一致性协议 `MESI`，但是`MESI`的同步等待机制会影响性能，所以用了 storeBuffer 来优化，但是优化过后还是会在一些场景存在可见性问题。这个时候又不能单方面的放弃优化，所以就提供一套内存屏障些指令来让我们的开发人员可以根据自己的场景来决定什么是需要禁用CPU缓存优化来避免可见性问题。

在{% post_link 'Java/并发编程/并发理论基础：指令重排序现象' %}一文中分析了指令重排序现象引发的并发问题。虽然 CPU 层面通过`as-if-serial`原则来保证指令集重排乱序执行的结果在单线程场景下结果的正确性。但是`as-if-serial`原则在多线程场景是行不通的，因为CPU没办法通过指令来辨别多线程中的指令逻辑依赖，所以这个时候CPU和编译器它们也提供了屏障指令来让我们的开发人员可以根据自己的场景来决定什么是需禁止重排序来避免重排序可能导致的并发问题。

以上我们说的都是计算机硬件方面的实现,但是 JVM 是运行在操作系统层面的，不同的硬件架构的缓存体系不一样，指令重排序的策略不一样，所提供的内存屏障指令也就有差异。而不同的操作系统对于硬件封装的指令集也不同。所以在Java中为了简化开发人员的工作，避免开发人员需要对底层的系统原理理解过分的依赖，所以封装了一套规范，把这些复杂的指令操作与开发人员编码隔离开来，这套规范就是”Java内存模型“。



## Java内存模型

JVM 在软件层面上有着和计算机硬件层面上的诸多异曲同工之妙，诸如 CPU 和 JVM 中的程序计数器、JIT 编译器与 CPU 中的指令乱序执行优化等。对于缓存一致性协议[《Java虚拟机规范》](http://www.cs.umd.edu/~pugh/java/memoryModel/)中试图定义一种 **Java 内存模型（Java Memory Model，JMM）** 来屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。

JMM规范了Java虚拟机与计算机内存是如何协同工作的:**规定了一个线程如何和何时可 以看到由其他线程修改过后的共享变量的值，以及在必须时如何同步的访问共享变量**。JMM 描述的是一种抽象的概念，一组规则，通过这组规则控制程序中各个变量在共享数据区域和私 有数据区域的访问方式，JMM是围绕原子性、有序性、可见性展开的。

>此处的变量（Variables）与 Java 编程中所说的变量有所区别，它包括了**实例字段**、**静态字段**和**构成数组对象的元素**，但不包括局部变量与方法参数，因为后者是虚拟机栈中线程私有的，不会被共享，自然就不会存在竞争问题。

Java内存模型实际上就是规范了 JVM 如何提供按需禁用缓存和重排序优化的方法。其核心就包括`volatile`、`synchronized` 和 `final` 三个关键字，以及几项`Happens-Before` 规则。有了JMM 作为Java的开发人员只需要使用几个关键字(`sychronized`，`volatile`，`final`) ,并且理解几个`happens before`规则，就能根据自己的需要来禁用缓存优化和指令重排序，从而避免并发问题。

### JMM内存模型和JVM内存结构之前的区别

- **JVM 内存结构**：是指 JVM 运行时将数据分区域存储，强调对内存空间的划分。

- **JMM 内存模型**：是指线程和主内存之间的抽象关系，即 JMM 中定义了线程在 JVM 主内存中的工作方式，如果要想深入了解 Java 并发编程，就要先理解好 JMM。

### JVM中的工作内存和主内存

- **主内存（Main Memory）**：类比的物理硬件为主内存，实则是指 JVM 内存的一部分。

- **工作内存（Working Memory）**：类比的物理硬件为 CPU 高速缓存，实则是指每条线程都有自己的工作内存，其中保存了被该线程使用到的变量在主内存中的拷贝副本（对于对象实例并不是完全拷贝，而是用到了哪个字段就拷贝哪个）。

线程对变量的所有操作（读取、赋值等）都**必须在工作内存中进行**，而不能直接读写主内存中的变量。不同的线程之间也**无法直接访问对方工作内存中的变量**，线程间变量值的传递**均需要通过主内存来完成**。

![JMM控制流程](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/2022/04/image-20220414235858873.png)

## JMM与硬件内存架构的关系

Java内存模型与硬件内存架构之间存在差异。硬件内存架构没有区分线程栈和堆。对于硬件，所有的线程栈和堆都分布在主内存中。部分线程栈和堆可能有时候会出现在CPU缓存中和CPU内部的寄存器中。如下图所示，Java内存模型和计算机硬件内存架构是一个交叉关系:

![JMM 与硬件内存架构关系](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/2022/04/image-20220415000708910.png)

从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：

- 线程之间的共享变量存储在主内存（Main Memory）中
- 每个线程都有一个私有的本地内存（Local Memory），本地内存是JMM的一个抽象概念，并不真实存在，它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。本地内存中存储了该线程以读/写共享变量的拷贝副本。
- 从更低的层次来说，主内存就是硬件的内存，而为了获取更好的运行速度，虚拟机及硬件系统可能会让工作内存优先存储于寄存器和高速缓存中。
- Java内存模型中的线程的工作内存（working memory）是cpu的寄存器和高速缓存的抽象描述。而JVM的内存结构只是一种对内存的物理划分而已，它只局限在内存，而且只局限在JVM的内存。

## JMM模型下的线程间通信

线程间通信必须要经过主内存。

如下，如果线程A与线程B之间要通信的话，必须要经历下面2个步骤：

1）线程A把本地内存A中更新过的共享变量刷新到主内存中去。

2）线程B到主内存中去读取线程A之前已更新过的共享变量。

![线程间的通讯](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/2022/04/v2-8750cb14ecaa93509e3f1981563513e1_r.jpg)



关于主内存与工作内存之间的具体交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步到主内存之间的实现细节，Java内存模型定义了以下八种操作来完成：

- **lock（锁定）**：作用于主内存的变量，把一个变量标识为一条线程独占状态。
- **unlock（解锁）**：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- **read（读取）**：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
- **load（载入）**：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- **use（使用）**：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
- **assign（赋值）**：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- **store（存储）**：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。
- **write（写入）**：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。

![内存模型的八种操作](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/2022/04/image-20220415001420504.png)

除此之外，Java内存模型还规定了在执行上述8种基本操作时必须满足如下规则：

* 不允许read和load、store和write操作之一单独出现，即不允许一个变量从主内存读取了但工作内存不接受，或者工作内存发起回写了但主内存不接受的情况出现。

  >如果要把一个变量从主内存拷贝到工作内存，那就要按顺序执行read和load操作，如果要把变量从工作内存同步回主内存，就要按顺序执行store和write操作。注意，Java内存模型只要求上述两个操作必须按顺序执行，但不要求是连续执行。也就是说read与load之间、store与write之间是可插入其他指令的，如对主内存中的变量a、b进行访问时，一种可能出现的顺序是`read a`、`read b`、`load b`、`load a`。

* 不允许一个线程丢弃它最近的assign操作，即变量在工作内存中改变了之后必须把该变化同步回主内存。

* 不允许一个线程无原因地（没有发生过任何assign操作）把数据从线程的工作内存同步回主内存中。

* 一个新的变量只能在主内存中“诞生”，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量，换句话说就是对一个变量实施use、store操作之前，必须先执行assign和load操作。

* 一个变量在同一个时刻只允许一条线程对其进行lock操作，但lock操作可以被同一条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。

* 如果对一个变量执行lock操作，那将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行load或assign操作以初始化变量的值。

* 如果一个变量事先没有被lock操作锁定，那就不允许对它执行unlock操作，也不允许去unlock一个被其他线程锁定的变量。

* 对一个变量执行unlock操作之前，必须先把此变量同步回主内存中（执行store、write操作）。

这 8 种操作和规则在 [JSR-133](https://gitee.com/yuanjianchen/programming-resources/blob/master/Java/JSR133/JSR133%E4%B8%AD%E6%96%87%E7%89%88.pdf) 中并没有被提及，已经放弃采用这 8 种描述去片面地定义 JMM 的访问协议（仅是描述方式改变了），而是由更完备的**先行发生原则（Happens-Before）**和**因果关系**来进行约束。

## Happens Before 原则

**先行发生原则（Happens-Before）**是判断数据是否存在竞争、线程是否安全的主要依据，依靠这个原则，我们可以解决并发环境下两个操作之间是否可能存在冲突的所有问题。

先行发生原则是 JMM 中定义的两项操作之间的偏序关系，例如：操作 A 先行发生于操作 B（即发生在操作 B 之前），则操作 A 产生的影响（包括修改了内存中共享变量的值、发送了消息、调用了方法等）就能被操作B观察到。

```java
// 在线程A中执行
i=1;

// 在线程B中执行
j=i;

// 在线程C中执行
i=2;
```

假设线程 A 先行发生于线程 B（A、B 有先行发生关系），那么在线程 B 执行后变量 `j` 的值一定等于 `1`。

假设线程 C 出现在了线程 A 和线程 B 的操作之间（A、B有先行发生关系），但线程 C 与线程 B 又没先行发生关系的情况下，那变量 `j` 的值是一个不确定的数，线程 B 就存在读取到过期数据的风险，不具备线程安全性。



作为开发人员来说，如果不想太深入底层去了解计算机底层的原理，又想编写出正确的并发程序，那么就必须对Happens Before原则加以理解，理解这些原则才能帮助我们避免并发程序的BUG，在出现并发问题后也能马上发现问题的所在。

### 程序次序规则（Program Order Rule）

定义：在一个线程中，按照程序代码的执行流顺序，先执行的操作happen—before后执行的操作。

说明：这个规则的意思就是，在同一个线程中前面的写操作对于后面的读操作来说是可见的。按下面的代码来说，x=1的写入对于flag=true是可见的。

```java
     int x;
     boolean flag;
 
     void write(){
         x=1;       //这里的值对于后续操作可见
         flag=true;
 
     }
```

### 管程锁定规则（Monitor Lock Rule）

定义：一个锁的unlock(解锁)操作happen—before后面对该锁的lock(加锁)操作。

说明：如果线程1解锁了A对象，然后线程2对A进行了加锁操作，那么线程1对共享变量的所有写操作对于线程2是可见的。

这个逻辑对应到代码里面就如下, write()方法 synchronized 代码块执行完之后（也就是对this对象的解锁操作，synchronized代码执行完自动解锁）的结果，对于 read()方法 进入synchronized 代码块(也就是对this对象的加锁操作)是可见的，也就是 代码2 会看到 x=x+1 的结果。

```java
int x; //共享变量
 
 void write(){
     synchronized(this){
         x=x+1;  //代码1
     }
 }
 
 void read(){
       synchronized(this){
          Systemt.out.println(x) //代码2
     }
     
 }
```

### Volatile 变量规则（Volatile Variable Rule）

定义：对一个 `volatile` 变量的 write 操作先行发生（happen—before）于后面（时间上）对这个变量的 read 操作。

说明：这个规则的是说，如果一个线程先修改了volatile的变量，那么这个操作对于后续其他线程对这个volatile变量读操作是可见的。 如下代码，线程1调用write() 修改了共享变量 x，然后线程2调用了read() 读取x，这个时候线程1 操作 x=1 对于线程2是可见的。

```java
volatile  int x; //共享变量
     
//线程1调用write()
void write(){
    x=1;
}

//线程2调用 read()
void read(){
    System.out.println(x); 

}
```

### 线程启动规则（Thread Start Rule）

定义：`Thread` 对象的 `start()` 方法先行发生（happen—before）于此线程的每一个动作。

说明：如果A线程调用 B线程的start()方法，那么A线程 在调用B.start()之前对共享变量的所有写操作对于B线程来说都是可见的。

如下代码，当先运行的线程为线程A，A线程先对共享变量v进行赋值，然后A线程调用B线程start()方法，那么B线程是可以看到v=10的这个操作的。

```java
 Thread threadB=new Thread(()->{
             System.out.println(v);
         });
 
 v=10;   //此操作对于线程B来说是可见的
 threadB.start(); //当前线程调用线程B的start()方法
 
```

### 线程终止规则（Thread Termination Rule）

定义：线程中的所有操作都先行发生（happen-before）于对此线程的终止检测（可以通过 `Thread.join()` 方法结束、`Thread.isAlive()` 的返回值等手段检测到线程已经终止执行）。

说明：以join()为例，如果A线程调用B线程的Join()方法，那么当B线程的Join方法返回后，A线程可以看到B线程对共享变量的所有写操作。 以下面代码为例，当前线程调用了threadB的join()方法并返回后，线程设置x=1的操作对于当前线程是可见的。

```java
 int x; //共享变量
 
 public void test() throws  Exception{
 
     Thread threadB=new Thread(()->{
         x=1;
     });
 
     threadB.start();
     threadB.join();//这个操作返回之后，threadB操作 x=1 对于当前线程是可见的
 
 }
```

### 线程中断规则（Thread Interruption Rule）

定义：对线程 `interrupt()` 方法的调用先行发生（ happen—before）于被中断线程的代码检测到中断事件的发生（可以通过 `Thread.interrupted()` 方法检测到是否有中断发生）。

说明：线程A调用了线程B的interrupt()方法，那么当线程B触发interrupt之后，线程A对所有共享变量的写操作对于线程B来说都是可见的。

```java
     int x; //共享变量
 
     public void test() throws  Exception{
 
         Thread threadB=new Thread(()->{
             try {
                 this.wait();
             } catch (InterruptedException e) {
                 System.out.println(1);//此处是可以看到 x=1的操作的
             }
         });
 
         threadB.start();
         x=1;//修改共享变量
         threadB.interrupt();//调用threadB中断方法
 
     }
 
```

### 对象终结规则（Finalizer Rule）

一个对象的初始化完成（构造函数执行结束）先行发生（happen—before）于它的 `finalize()` 方法的开始。

说明：调用对象finalize()方法时，对象初始化完成的任意操作，对于调用finalize()线程来说都是可见的。

```java
 public class Test {
 
     int x; //共享变量
 
     public Test() {
         this.x = 8;
     }
 
     public void test() throws  Exception,Throwable{
        Test test=new Test();
         test.finalize(); //x = 8操作对于此线程可见
 
     }
 
 }
```

### 传递性（Transitivity）

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作C，那就可以得出操作 A 先行发生于操作 C 的结论。

传递性规则要与其他规则组合理解，以volatile规则+传递性规则为例，下面代码中

1、因为 y=1 happen-before x=2（顺序性原则）

2、而 x = 2 happen-before x = 3（volatile变量 规则）;

3、 而x = 3又 happen-before z=4（顺序性原则）。

4、所以最终得出 y=1 happen-before z=4(传递性原则);

```java
              int x;
     volatile  int y; 
               int z;
 
 
     public void  test1() {
         y=1;
         x = 2; //因为 y=1 happen-before x=2（顺序性原则） 所以y=1 对于x=2可见
 
     }
 
     public void  test2() {
         x = 3;  //因为  x = 2 happen-before x=3（volatile 规则） 所以x = 2 对于x=3可见
         z=4;  //因为  y=1 happen-before  x=2，而 x = 2 happen-before x = 3;  而x = 3又 happen-before  z=4（顺序性原则），所以 y=1 happen-before z=4(传递性原则);
 
     }
```

JVM 在进行代码优化时需要保证这些先行发生关系，先行发生原则不依赖任何同步器协助就可以在编码中直接利用。如果两个操作之间的关系不在此列，并且无法从这 8 条规则推导出来的话，它们就没有顺序性保障，JVM 可以对它们**随意地进行重排序**。

### 利用先行发生原则分析程序代码

```java
class MyClass {

    private int value = 0;

    public void setValue(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

这是一组 getter/setter 方法，假设存在线程 A 和 B，线程 A 先调用了（时间上的先后）`setValue(1)` 方法，然后线程 B 调用了同一个对象的 `getValue()`，那么线程 B 收到的返回值是什么？利用先行发生原则进行分析：

1. 两个方法分别由线程 A 和线程 B 调用，不在同一线程中，所以**程序次序规则**在这里不适用。

1. 由于没有同步块（`synchronized` 或 `java.util.concurrent.ReentrantLock`），自然不会发生 lock 和 unlock 操作，所以**管程锁定规则**不适用。

1. 由于 `value` 变量没有被 `volatile` 关键字修饰，所以 **Volatile 变量规则**不适用。

1. **线程启动、终止、中断规则**和**对象终结规则**也和这里完全没有关系。

因为没有一个适用的先行发生规则，所以最后一条**传递性**也无从谈起，因此我们可以判定尽管线程 A 在操作时间上先于线程 B，但是无法确定线程 B 中 `getValue()` 方法的返回结果，换句话说，这里面的操作**不是线程安全**的。

修复这个问题：

- 要么把 `getter()`、`setter()` 方法都定义为 `synchronized` 方法，这样就可以套用**管程锁定规则**来满足先行发生关系。

- 要么把 `value` 定义为 `volatile` 变量，由于 `setter()` 方法对 `value` 的修改不依赖 `value` 的原值，满足 `volatile` 关键字使用场景，这样就可以套用 **Volatile 变量规则**来满足先行发生关系。

## 因果关系（Causality）

先行发生原则仅仅是 JMM 的一个必要非充分的规则约束集，早期的 JMM 规范在不引入**因果关系（Causality）**这一条件约束的情况下，会出现一个看似更加诡异的问题——无中生有（out of thin air）。

**场景 A**

```java
int x = 0, y = 0;

// 线程1 执行 foo 方法
void foo(){
  r1 = x;
  y=r1;
}

// 线程2 执行 bar 方法
void bar(){
  r2 = y;
  x=r2;
}

// 有可能出现 r1 == r2 == 42
```

场景 A 如果以顺序一致性方式执行，是完全不会有任何同步问题的，但 JMM 并没有选择顺序一致性。而单纯地根据先行发生原则约束来看，这里首先不构成任何先行发生关系，其次由于 `x`、`y` 变量均在多线程环境下读写，这里也必然存在访问冲突，两个条件综合来看，`x` 和 `y` 势必出现数据竞争。

**为什么值有可能为 42**

> However, in a future aggressive system, Thread 1 could speculatively write the value 42 to y, which would allow Thread 2 to read 42 for y and write it out to x, which would allow Thread 1 to read 42 for x, and justify its original speculative write of 42 for y.

当 `x` 依赖 `y` 的值，`y` 也依赖 `x` 的值，构成了循环依赖，成为了**因果循环（causal cycle）**的场景。

当一个相当激进的编译器遇到这样的场景，可能就会做出不可思议的优化手段，它有可能推测 `y` 为任何值，当然可能就是 42，因为无论什么值，这个循环场景都能够自圆其说。

在这种自圆其说（self-justifying）的场景下，配合这套先行发生原则来约束，就会出现严重问题，说到底还是先行发生原则太弱，除非刻意给变量加上 `volatile` 关键字，进而构成先行发生关系，但是使用 `volatile` 是有代价的，似乎没必要，我们只是需要采取某种措施防止编译器过于激进的优化即可。

------

**场景 B**

```java
int x = 0, y = 0;

// 线程1 执行 foo 方法
void foo(){
  r1 = x;
  if(r1 != 0)
    y = 42;
}

// 线程2 执行 bar 方法
void bar(){
  r2 = y;
  if(r2 != 0)
     x = 42;
}

// 有可能出现 r1 == r2 == 42
```

场景 B 同样不构成任何先行发生关系，也没有额外的同步动作，却可能出现直接把 `x`、`y` 的写入操作越过条件语句，不分青红皂白地提前执行。

但场景 B 更令人费解的点在于，按照逻辑来看，由于 `r1` 和 `r2` 均为 0，应该不存在任何 x，y 的写入操作，也就不会出现数据竞争，原则上不需要任何同步手段也不会有同步问题。而事实上，上面两个场景恰恰说明，单凭这套先行发生原则的约束也没办法保证场景 B 正确执行。

-----

庆幸的是，无论场景 A 还是场景 B ，在 JMM 里是**绝对禁止**的，而先行发生原则又无法对这两个场景做出约束，JMM 就借助了**因果关系（Causality）**来保证。

简单说，因果关系核心在于对数据流（数据赋值关系）和控制流（条件控制语句，如 `if`）的依赖关系的分析，比如场景 B，`x` 和 `y` 的赋值操作前提是 `r1` 和 `r2` 不为 0，这种就是基于控制流的因果关系分析。

那么既然场景 B 出现了 `r1 == r2 == 42` 的情况，就说明编译器破坏了这层因果关系，这显然不能被 JMM 所接受。基于因果关系，场景 A 也不会凭空推测出 42，因为程序中并没有 42 写入的行为。

----

**因果关系的破坏**

```java
int a = 0, b = 1;

// 线程 1 执行 foo
void foo(){
  r1 = a;
  r2 = a;
  if(r1 == r2)
    b = 2;
} 

// 线程 2 执行 bar
void bar(){
  r3 = b;
  a = r3;
}

// 有可能出现 r1 == r2 == r3 == 2
```

场景 C 有可能出现 `r1 == r2 == r3 == 2` 的结果是被「允许」的。基于编译器优化策略，场景 C 被编译器优化后的代码：

```java
// 线程 1 执行 foo
void foo(){
  b = 2;
  r1 = a;
  r2 = r1; 
}

// 线程 2 执行 bar
void bar(){
  r3 = b;
  a = r3;
}
```

- `foo()` 方法中变量 `a` 重复读被清理，`r2 = a` 被 `r2 = r1` 取代；

- `if(r1 == r2)` 必然为 `true`，那么条件判断多余，可以被清理；

- `b = 2` 此时就有可能被重排序到 `foo()` 方法的开始位置。

这就是基于控制流依赖的因果关系被打破的典型例子，从 `foo()` 方法执行语句来看，重复读消除是合理的，避免读到不同的值，这直接导致 `if` 判断确实无论什么情况下都是 `true`，这样一来，这一层因果关系被打破了，不需要因为满足 `if` 条件 `b = 2` 才会被执行，这种打破是**由开发者的编码逻辑**决定的，合情合理。反观场景 B 才是由编译器自身激进优化的问题才被因果关系「拦下」的。



## JMM规范下对并发三大特征的实现

JMM 就是是围绕着在并发过程中如何处理**原子性**、**可见性**和**有序性**这 3 个特征来建立的。

#### 原子性（Atomicity）

**原子性（Atomicity）** 操作是指不可再被划分（不会被分不同时间片中）**的操作指令，JMM 要求在没有任何的同步手段的前提下，Java 基本数据类型变量的读写必须具备原子性。

但是允许了两个特例存在，非 `volatile` 修饰的 64位 `double` 和 `long` 类型，这就是它们的**非原子性协定**。

不同版本的 Java 可能有不同的处理方式，一般对 `double` 和 `long` 的处理方式是，将一个 64 位拆分成两个 32 位分别原子性读写，这样一来多线程环境下就有问题了，很可能高 32 位和低 32 位分别被两个不同线程读写，可能出现读取到「半个变量」的诡异问题。

> 「这种情况非常罕见，在目前商用 JVM 中不会出现，虽然 JMM 有非原子性协定，但还是强烈建议 JVM 将 `double` 和 `long` 类型的读写操作实现为原子操作。目前各种平台下的商用 JVM 几乎都选择把 64 位数据的读写操作作为原子操作来对待。读者只要知道这件事情就可以了，无须太过在意这些几乎不会发生的例外情况。」——《深入理解 Java 虚拟机》
>
> [Non-Atomic Treatment of `double` and `long`](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.7)

**Java 实现原子性**

- `synchronized` 关键字

- `sun.misc.Unsafe` 类中的一系列 `compareAndSwap()` 方法

**`synchronized` 关键字实现原子性**

JMM 提供了 lock 和 unlock 操作来满足这种需求，尽管 JVM 未把 lock 和 unlock 操作直接开放给用户使用，但是却依托于管程（monitor）提供了更高层次的字节码指令 `monitorenter` 和 `monitorexit` 来隐式地使用这两个操作，这两个字节码指令反映到 Java 代码中就是同步块——`synchronized` 关键字。

Java 中每个对象实例都关联着一个 monitor，这个 monitor 维护着一个计数器，`monitorenter` 加锁时计数器值 + 1，`monitorexit` 解锁时 - 1，当计数器为 0 表示没有任何线程占用该锁。计数器的数值表示该锁被加锁的次数，这是为了实现可以被同个线程多次加锁，即**可重入**特性，所以 `synchronized` 是不会把自己死锁的，因此在 `synchronized` 块之间的操作也具备原子性。

**`compareAndSwap*()`** **方法实现原子性**

`sun.misc.Unsafe` 类中的一系列 `compareAndSwap*()` 方法也可以实现原子性操作，这些都是 native 方法，会调用 CPU 提供的原子汇编指令来实现。

#### 可见性（Visibility）

**可见性（Visibility）** 是指当一个线程修改了共享变量的值，其他线程能够立即得知这个修改。JMM 通过在变量读取前从主内存刷新变量值这种依赖主内存作为传递媒介的方式来实现可见性，反映到 Java 代码中就是 `volatile` 关键字。



**Java 实现可见性**

- `volatile` 关键字

- `synchronized` 关键字

- `final` 关键字

**`volatile`** **关键字实现可见性**

`volatile` 关键字的语义：

- **保证此变量对所有线程的可见性**：当一条线程修改了这个变量的值会立刻回写到主内存，新值对于其他线程来说是可以立即得知的，所有写操作都能立刻反应到其他线程中。而普通变量被修改后，什么时候被回写到主内存是不定的，另外一条线程B在线程A回写完成了之后再从主内存进行读取操作，新变量值才会对线程B可见。

- **禁止此变量赋值操作指令重排序优化**：普通的变量**仅仅**会保证在该方法的执行过程中所有依赖赋值结果的地方都能获取到正确的结果，这就是 Java 程序中**线程内表现为串行的语义（Within-Thread As-If-Serial Semantics）**，但普通的变量并不能保证赋值操作的顺序与程序代码中的执行顺序一致，所以在当前线程里看程序代码像是有序的，但在其他线程里看来实则是无序的。

`volatile` 关键字可以说是 JVM 提供的最轻量级的控制并发机制，下面这类场景就很适合使用 `volatile` 关键字来控制并发，当 `shutdown()` 方法被调用时，能保证所有线程中执行的 `doWork()` 方法都能停止下来。

```java
volatile boolean shutdownRequested;

public void shutdown() {
    shutdownRequested = true;
}

public void doWork() {
    while (!shutdownRequested) {
        // do something...
    }
}
```



但 `volatile` 仅仅只能够保证变量的可见性，在不符合以下两条规则的运算场景中，仍需要通过加锁来保证原子性：

- 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值。

- 变量不需要与其他的状态变量共同参与不变约束。

JMM 对 `volatile` 关键字专门定义了一些特殊的访问规则：

- 要求在工作内存中，每次使用变量前都必须先从主内存刷新最新的值（固定的 load -> use 顺序），用于保证能看见其他线程对变量所做的修改后的值。

- 要求在工作内存中，每次修改变量后都必须立刻同步回主内存中（固定的 assign -> store 顺序），用于保证其他线程可以看到当前线程对变量所做的修改。

在 `synchronized` 同步块中，对一个变量执行 unlock 操作前，必须先把该变量同步回主内存中（执行 store、write 操作），所以 `synchronized` 关键字也保证了该变量的可见性。

被 `final` 关键字修饰的字段在构造器中一旦初始化完成，**并且构造器没有把** **`this`** **的引用传递出去**，那再其他线程中就能看见 `final` 字段的值，且 `final` 字段的值不能再被修改，也算保证了该变量的可见性。

`this` 引用逃逸是一件很危险的事情，其他线程有可能通过这个引用访问到「初始化了一半」的对象，例如《Java 并发编程实践》中的例子：

```java
public class ThisEscape {
　　public ThisEscape(EventSource source) {
　　　　source.registerListener(new EventListener() {
　　　　　　public void onEvent(Event e) {
　　　　　　　　doSomething(e);
　　　　　　}
　　　　});
　　}
 
　　void doSomething(Event e) {
　　}
 
　　interface EventSource {
　　　　void registerListener(EventListener e);
　　}
 
　　interface EventListener {
　　　　void onEvent(Event e);
　　}
 
　　interface Event {
　　}
}
```

这将导致 `this` 引用逸出，所谓逸出，就是在不该发布的时候发布了一个引用。在这个例子里面，当我们实例化 `ThisEscape` 对象时，会调用 `source` 的 `registerListener()` 方法，这时便启动了一个线程，而且这个线程持有了 `ThisEscape` 对象（调用了对象的 `doSomething()` 方法），但此时 `ThisEscape` 对象却没有实例化完成（还没有返回一个引用），造成了 `this` 引用逸出，即还没有完成的实例化 `ThisEscape` 对象的动作，却已经暴露了对象的引用。其他线程访问还没有构造好的对象，可能会造成意料不到的问题。只有当构造函数返回时，`this` 引用才应该从线程中逸出。构造函数可以将 `this` 引用保存到某个地方，只要其他线程不会在构造函数完成之前使用它。



#### 有序性（Ordering）

**有序性（Ordering）** 在 `volatile` 关键字的禁止指令重排序语义中是也有提到，编译器和 CPU 会进行代码优化，打乱原本的代码顺序，进行一些执行预测分析等操作，三种重排序类型：

- **编译器优化的重排序**：编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。

- **指令级并行的重排序**：现代 CPU 采用了指令级并行技术（Instruction-Level Parallelism，ILP）来将多条指令重叠执行。如果不存在数据依赖性，CPU 可以改变语句对应机器指令的执行顺序。

- **内存系统的重排序**：由于 CPU 使用缓存和读写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

**Java 实现有序性**

在 Java 程序中就天然存在的有序性：

- **线程内表现为串行的语义**：如果在本线程内观察，所有的操作都是有序的。

- **指令重排序和工作内存与主内存同步延迟现象**：如果在一个线程中观察另一个线程，所有的操作都是无序的。

Java 语言也提供了 `volatile` 和 `synchronized` 两个关键字来保证线程间操作的有序性，`volatile` 关键字本身就包含了禁止指令重排序的语义，而被 `synchronized` 关键字锁住的变量在同一时刻只允许一条线程对其进行 lock 操作，这决定了持有同一个锁的两个同步块只能串行地进入，所以也保证了有序性。



**参考资料**

[The Java Memory Model](http://www.cs.umd.edu/users/pugh/java/memoryModel/)

[JSR 133 (Java Memory Model) FAQ](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)

[Chapter 17. Threads and Locks](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4)

[JMM model of concurrent programming & underlying principle of Volatile](https://programs.team/jmm-model-of-concurrent-programming-underlying-principle-of-volatile.html)

