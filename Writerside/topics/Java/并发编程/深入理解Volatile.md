---
title: 深入理解Volatile
cover: false
toc: true
mathjax: false
tags:
  - 并发
  - Java
categories: 并发编程
url: 165052629
sort: 8
date: 2022-04-21 15:31:28
keywords:
password:
summary:
---

在{% post_link 'Java/并发编程/并发理论基础：Java 内存模型JMM' %}一文中讲到 JMM 模型是 Java 抽象出来的一个概念，用来屏蔽不同硬件和操作系统对于内存访问的差异，让 Java 程序能在不同平台上达到一致的运行效果。规范了 JVM 与计算机内存如何系统工作，规定了一个线程如何和何时可以看到其他线程修改后的共享变量的值，以及如何将共享变量的值写入主存。JMM 围绕原子性，可见性和有序性展开的，其实现的核心主要包括`volatile`,`synchronized`和`final`关键字以及几项`happen-before`原则。本文就`volatile`如何在 JMM 中的作用以及实现原理展开讨论。

## volatile的特性

JMM是围绕原子性、可见性、有序性展开的。`volatile` 是 JMM 实现的一个核心之一，讨论volatile 在 JMM 中的作用也需要围绕原子性、可见性、有序性这三个方面来展开。

### 可见性

根据[JSR-133 FAQ](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)中的说明，volatile字段是用于在线程之间传递状态的特殊字段。**每次读取volatile时，都会看到任意一个线程对该volatile的最后一次写入**。 实际上，程序员将volatile字段指定为不能接受由于缓存或重排序而导致的“过时”值的字段。禁止编译器和运行时在寄存器中分配它们。它们还必须确保在写入后将其从高速缓存（cache）中刷新到主存(memory)，以便它们可以立即对其他线程可见。 同样，在读取volatile字段之前，必须使高速缓存无效，以便可以看到主内存中的值而不是本地处理器高速缓存中的值。

同时[The Java Language Specification](https://docs.oracle.com/javase/specs/index.html)也规定了对于`volatile`修饰的共享变量，在多个线程中的可见性[volatile Fields](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.3.1.4)。

>A field may be declared `volatile`, in which case the Java Memory Model ensures that all threads see a consistent value for the variable ([§17.4](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4)).

**volatile内存读写语义**

* 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。

* 当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效，线程接下来将从主内存中 读取共享变量。

**也就是说每次读取volatile都是从主存读取，写入也会刷新到主存，因而保证了不同线程拿到的都是最新值，即保证了共享资源对各个CPU上的线程的可见性，这其实就是保证了缓存一致性。**

> 关于本地内存、主存、JVM 内存结构和计算硬件内存的关系可参考{% post_link 'Java/并发编程/并发理论基础：Java 内存模型JMM' %}

### 原子性

**对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性**(基于这点，我们通过会认为volatile不具备原子性)。volatile仅仅保证对单个volatile变量的读/写具有原子性，而锁的互斥执行的特性可以确保对整个临界区代码的执行具有原子性。

JVM 在 64 位`long`和`double`的读写操作具体是原子性实现还是分段读（效率原因）写没有做明确的规定，但是建议在定义全局的 long 或者 double 变量时声明为`volatile`来避免并发问题。具体内容可查看[Non-atomic Treatment of double and long](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.7)。

### 有序性

`volatile`是通过禁止指令重排序来保证有序性的。指令重排序有两种：

* 编译器重排序
* CPU 指令集重排序

关于禁止编译器重排还是以一段代码来进行对比：

```java
public class CompilerOutOrdering {
    private static int field1;
    private static int field2;
    private static int field3;
    private static int field4;
    private static int field5;
    private static int field6;

    private static void assign(int i) {
        field1 = i << 1;
        field2 = i << 2;
        field3 = i << 3;
        field4 = i << 4;
        field5 = i << 5;
        field6 = i << 6;
    }

    public static void main(String[] args) throws Exception {
        for (int i = 0; i < 10000; i++) {
            assign(i);
        }
        Thread.sleep(1000);
    }
```

上面代码经过 JIT 编译后的汇编如下：

![汇编代码](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/2022/04/image-20220420205006346.png)

可以看出源码和字节码顺序保持一致，而汇编赋值变成了 1，6，5，4，3，2。

把 1 和 6 加上 volatile 关键字修饰：

![volatile 修饰后汇编](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/2022/04/image-20220420205248483.png)

汇编变成了 1 始终第一个赋值，6 始终最后一个赋值，但是 1 和 6 之间的顺序不保证。

## volatile 实现

### JMM内存交互层面实现

volatile修饰的变量的read、load、use操作和assign、store、write必须是连续的，即修改后必须立 即同步回主内存，使用时必须从主内存刷新，由此保证volatile变量操作对多线程的可见性。

### 硬件层面实现

通过lock前缀指令，会锁定变量缓存行区域并写回主内存，这个操作称为“缓存锁定”，缓存一致性机制会阻止同时修改被两个以上处理器缓存的内存区域数据。一个处理器的缓存回写到内存会导致其他处理器的缓存无效。关于 lock 指令也可以通过上图的汇编代码可以看到。

关于缓存锁定可以查看{% post_link 'Java/并发编程/并发理论基础：缓存可见性、MESI' %}。

关于 lock 指令可以查看{% post_link 'Java/并发编程/并发理论接触：内存屏障' %}。

### volatile在hotspot的实现

#### 字节码解释器实现

JVM中的字节码解释器(bytecodeInterpreter)，用C++实现了JVM指令，其优点是实现相对简单且容易理解，缺点是执行慢。

`bytecodeInterpreter.cpp`

```c++
          int field_offset = cache->f2_as_index();
          if (cache->is_volatile()) {
            if (tos_type == itos) {
              obj->release_int_field_put(field_offset, STACK_INT(-1));
            } else if (tos_type == atos) {
              VERIFY_OOP(STACK_OBJECT(-1));
              obj->release_obj_field_put(field_offset, STACK_OBJECT(-1));
            } else if (tos_type == btos) {
              obj->release_byte_field_put(field_offset, STACK_INT(-1));
            } else if (tos_type == ztos) {
              int bool_field = STACK_INT(-1);  // only store LSB
              obj->release_byte_field_put(field_offset, (bool_field & 1));
            } else if (tos_type == ltos) {
              obj->release_long_field_put(field_offset, STACK_LONG(-1));
            } else if (tos_type == ctos) {
              obj->release_char_field_put(field_offset, STACK_INT(-1));
            } else if (tos_type == stos) {
              obj->release_short_field_put(field_offset, STACK_INT(-1));
            } else if (tos_type == ftos) {
              obj->release_float_field_put(field_offset, STACK_FLOAT(-1));
            } else {
              obj->release_double_field_put(field_offset, STACK_DOUBLE(-1));
            }
            OrderAccess::storeload();
          } else {
            .......
          }  
```

可以看到主要是通过 `storeload` 屏障实现的。而在不同的处理器中 `storeload` 实现的指令不同：

例如在 linux_x8下 `orderAccess_linux_x86.hpp`

```c++
inline void OrderAccess::storeload()  { fence();            }
......

inline void OrderAccess::fence() {
   // always use locked addl since mfence is sometimes expensive
#ifdef AMD64
  __asm__ volatile ("lock; addl $0,0(%%rsp)" : : : "cc", "memory");
#else
  __asm__ volatile ("lock; addl $0,0(%%esp)" : : : "cc", "memory");
#endif
  compiler_barrier();
}

......
// A compiler barrier, forcing the C++ compiler to invalidate all memory assumptions
static inline void compiler_barrier() {
  __asm__ volatile ("" : : : "memory");
}
```

通过 lock 指令以及 `compiler_barrier`禁止 CPU 和编译器指令重排。

#### 模板解释器实现

模板解释器(templateInterpreter)，其对每个指令都写了一段对应的汇编代码，启动时将每个指令与对应 汇编代码入口绑定，可以说是效率做到了极致。

`templateTable_x86.cpp`

```c++
// 负责执行putfield或putstatic指令
void TemplateTable::putfield_or_static(int byte_no, bool is_static, RewriteControl rc) {
  transition(vtos, vtos);

  ......
  volatile_barrier(Assembler::Membar_mask_bits(Assembler::StoreLoad |
                                               Assembler::StoreStore));
  ......
}

// ----------------------------------------------------------------------------
// Volatile variables demand their effects be made known to all CPU's
// in order.  Store buffers on most chips allow reads & writes to
// reorder; the JMM's ReadAfterWrite.java test fails in -Xint mode
// without some kind of memory barrier (i.e., it's not sufficient that
// the interpreter does not reorder volatile references, the hardware
// also must not reorder them).
//
// According to the new Java Memory Model (JMM):
// (1) All volatiles are serialized wrt to each other.  ALSO reads &
//     writes act as aquire & release, so:
// (2) A read cannot let unrelated NON-volatile memory refs that
//     happen after the read float up to before the read.  It's OK for
//     non-volatile memory refs that happen before the volatile read to
//     float down below it.
// (3) Similar a volatile write cannot let unrelated NON-volatile
//     memory refs that happen BEFORE the write float down to after the
//     write.  It's OK for non-volatile memory refs that happen after the
//     volatile write to float up before it.
//
// We only put in barriers around volatile refs (they are expensive),
// not _between_ memory refs (that would require us to track the
// flavor of the previous memory refs).  Requirements (2) and (3)
// require some barriers before volatile stores and after volatile
// loads.  These nearly cover requirement (1) but miss the
// volatile-store-volatile-load case.  This final case is placed after
// volatile-stores although it could just as well go before
// volatile-loads.

void TemplateTable::volatile_barrier(Assembler::Membar_mask_bits order_constraint ) {
  // Helper function to insert a is-volatile test and memory barrier
  __ membar(order_constraint);
}
```

`assembler_x86.hpp`

```c++
 void  (Membar_mask_bits order_constraint) {
    // We only have to handle StoreLoad x86平台只需要处理StoreLoad
    if (order_constraint & StoreLoad) {
      // All usable chips support "locked" instructions which suffice
      // as barriers, and are much faster than the alternative of
      // using cpuid instruction. We use here a locked add [esp-C],0.
      // This is conveniently otherwise a no-op except for blowing
      // flags, and introducing a false dependency on target memory
      // location. We can't do anything with flags, but we can avoid
      // memory dependencies in the current method by locked-adding
      // somewhere else on the stack. Doing [esp+C] will collide with
      // something on stack in current method, hence we go for [esp-C].
      // It is convenient since it is almost always in data cache, for
      // any small C.  We need to step back from SP to avoid data
      // dependencies with other things on below SP (callee-saves, for
      // example). Without a clear way to figure out the minimal safe
      // distance from SP, it makes sense to step back the complete
      // cache line, as this will also avoid possible second-order effects
      // with locked ops against the cache line. Our choice of offset
      // is bounded by x86 operand encoding, which should stay within
      // [-128; +127] to have the 8-byte displacement encoding.
      //
      // Any change to this code may need to revisit other places in
      // the code where this idiom is used, in particular the
      // orderAccess code.

      int offset = -VM_Version::L1_line_size();
      if (offset < -128) {
        offset = -128;
      }
			
     // 下面这两句插入了一条lock前缀指令: lock addl $0, $0(%rsp)
      lock();  // lock前缀指令
      addl(Address(rsp, offset), 0);// Assert the lock# signal here  // addl $0, $0(%rsp)
    }
  }
```