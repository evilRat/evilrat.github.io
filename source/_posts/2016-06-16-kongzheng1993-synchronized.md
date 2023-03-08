---
layout: post
title:  "synchronized浅析"
date:   2016-05-20
excerpt: "synchronized"
tag:
comments: true
---

## synchronized是什么？

synchronized是一个关键字，并不是一个“锁”，“加锁”这个操作更符合它的含义。

字面意思它是同步的过去式。
<img src="fanyi.png">

多线程环境：
<img src="1.png">


## 原理

synchronized的底层实现是使用操作系统的mutex lock实现的。JDK5之前被称为重量级锁，JDK6对synchronized内在机制进行了优化，加入了CAS、轻量级锁和偏向锁对功能，性能上已经跟ReentrantLock相差无几，而且在使用上更简单，不易出错。所以如果仅仅要实现互斥效果，不需要基于Lock的复杂操作（中断、条件等），推荐优先使用synchronized。

synchronized用的锁是存在Java对象头里的。JVM基于`进入`和`退出`Monitor对象来实现方法同步和代码块同步。

## synchronized的几种加锁方式以及基础说明

修饰内容|锁类型|示例
-|-|-
没加锁|没加锁|示例1
修饰代码块|任意对象锁|示例2
修饰普通方法|this锁|示例3
修饰静态方法|类锁|示例4

### 示例1:没有synchronized加锁

```java
public class NoSynchronizedDemo {
    public void method() {
        System.out.println("Method 1 start");
    }
}
```

查看核心字节码

```java
  public void method();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Method 1 start
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 5: 0
        line 6: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/lhx/cloud/javathread/NoSynchronizedDemo;
```

### 示例2:同步方法块，锁是括号里面的对象

```java
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("Method 1 start");
        }
    }
}
```

查看字节码

```java
  public void method();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter
         4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #3                  // String Method 1 start
         9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_1
        13: monitorexit
        14: goto          22
        17: astore_2
        18: aload_1
        19: monitorexit
        20: aload_2
        21: athrow
        22: return
```

可以看在加锁的代码块，多了个 `monitorenter` , `monitorexit`

**monitorenter**
每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：
  - 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。
  - 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.
  - 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权

*总结：*
- synchronized是可重入锁，即如果当前线程以获得锁对象，可再次获取该锁对象。即：该锁对象的监视器锁 monitor 具有可重入性，每进入一次，进入次数+1
- 从synchronized使用的语法上，如果修饰代码块，`synchronize (object) {} object` 即为锁对象
- 如果修饰方法，普通方法可认为是 this 锁，即`当前对象锁`
- 静态方法可认为是`类锁`

**monitorexit**

- 执行monitorexit的线程必须是objectref所对应的monitor的所有者。
- 指令执行时，monitor的进入数减1
- 如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者
- 其他被这个monitor阻塞的线程可以尝试去获取这个monitor的所有权

*总结：*
通过以上描述，应该能很清楚的看出Synchronized的实现原理，Synchronized的语义底层是通过一个`monitor`的对象来完成，其实wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。

### 示例3:普通同步方法，锁是当前实例对象

```java
public class SynchronizedDemo2 {
    public synchronized void method() {
        System.out.println("Method 1 start");
    }
}
```

查看字节码

```java
  public synchronized void method();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Method 1 start
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
```

**注意:** 在flags上增加了ACC_SYNCHRONIZED

### 示例4:静态同步方法，锁是当前类的class对象

```java
public class SynchronizedDemoStatic {
    public static synchronized void method() {
        System.out.println("Method 1 start");
    }
}
```

查看字节码

```java
  public static synchronized void method();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=0, args_size=0
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Method 1 start
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
```

**注意：** 在flags上增加了ACC_STATIC, ACC_SYNCHRONIZED

*总结：*

针对示例3、示例四，在flags上均增加了`ACC_SYNCHRONIZED`

从反编译的结果来看，方法的同步并没有通过指令`monitorenter`和`monitorexit`来完成（理论上其实也可以通过这两条指令来实现），不过相对于普通方法（没加synchronized的），其常量池中多了`ACC_SYNCHRONIZED`标示符。**JVM就是根据该标示符来实现方法的同步的**：当方法调用时，调用指令将会检查方法的`ACC_SYNCHRONIZED`访问标志是否被设置，如果设置了，执行线程将先获取`monitor`，获取成功之后才能执行方法体，方法执行完后再释放`monitor`。在方法执行期间，其他任何线程都无法再获得同一个`monitor`对象。 其实本质上没有区别，只是方法的同步是一种**隐式**的方式来实现，无需通过字节码来完成。