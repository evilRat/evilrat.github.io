---
title: java.lang.IllegalMonitorStateException
excerpt: ''
tags: [多线程, Java]
categories: [Java]
comments: true
date: 2020-04-27 00:30:52
---

## java.lang.IllegalMonitorStateException

从JDK源码开始看：

```java
/**
 * Thrown to indicate that a thread has attempted to wait on an
 * object's monitor or to notify other threads waiting on an object's
 * monitor without owning the specified monitor.
 *
 * @author  unascribed
 * @see     java.lang.Object#notify()
 * @see     java.lang.Object#notifyAll()
 * @see     java.lang.Object#wait()
 * @see     java.lang.Object#wait(long)
 * @see     java.lang.Object#wait(long, int)
 * @since   JDK1.0
 */
public
class IllegalMonitorStateException extends RuntimeException {
    private static final long serialVersionUID = 3713306369498869069L;

    /**
     * Constructs an <code>IllegalMonitorStateException</code> with no
     * detail message.
     */
    public IllegalMonitorStateException() {
        super();
    }

    /**
     * Constructs an <code>IllegalMonitorStateException</code> with the
     * specified detail message.
     *
     * @param   s   the detail message.
     */
    public IllegalMonitorStateException(String s) {
        super(s);
    }
}
```

从注释部分可以了解到：当我们在没有拥有指定对象的监视器时，就去等待这个对象的监视器或者通知其他线程去等待这个对象的监视器，就会抛出`IllegalMonitorStateException`。

换句话说，我们要想调用一个对象的`wait()`、`notify()`等方法来实现线程通信，就要先获取这个对象的监视器。也就是要用`synchronized`修饰这个代码块。

我们知道`synchronized`修饰的代码块编译后会被`monitorenter`和`monitorexit`包围，这两个虚拟机命令就是获取和释放对象的监视器。

所以Object自带的`wait()`、`notify()`等方法，是让我们在使用`synchronized`时进行线程通信用的。

```java
import java.lang.Runnable;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Test {

    private int i = 0;

    Object obj = new Object();

    public void odd() {
        synchronized (obj) {
            while (i < 3) {
                if (i % 2 == 1) {
                    System.out.println(Thread.currentThread().getName() + "打印：" + i);
                    i++;
                    System.out.println("odd before notify");
                    obj.notify();
                    System.out.println("odd after notify");
                } else {
                    try {
                        System.out.println("odd before wait");
                        obj.wait();
                        System.out.println("odd after wait");
                    } catch (Exception e) {

                    }
                }
                System.out.println("odd exit while");
            }
            System.out.println("odd exit synchronized");
        }
    }

    public void even() {
        synchronized (obj) {
            while (i < 3) {
                if (i % 2 == 0) {
                    System.out.println(Thread.currentThread().getName() + "打印：" + i);
                    i++;
                    System.out.println("even before notify");
                    obj.notify();
                    System.out.println("even after notify");
                } else {
                    try {
                        System.out.println("even before wait");
                        obj.wait();
                        System.out.println("even after wait");
                    } catch (Exception e) {

                    }
                }
                System.out.println("even exit while");
            }
            System.out.println("even exit synchronized");
        }
    }

    public static void main(String[] args) {

        Test test = new Test();

        Thread thread1 = new Thread(new Runnable() {

            @Override
            public void run() {
                test.odd();
            }
        }, "奇数");

        Thread thread2 = new Thread(new Runnable() {

            @Override
            public void run() {
                test.even();
            }
        }, "偶数");

        thread1.start();
        thread2.start();
    }
}
```

运行结果：

```
odd before wait
偶数打印：0
even before notify
even after notify
even exit while
even before wait
odd after wait
odd exit while
奇数打印：1
odd before notify
odd after notify
odd exit while
odd before wait
even after wait
even exit while
偶数打印：2
even before notify
even after notify
even exit while
even exit synchronized
odd after wait
odd exit while
odd exit synchronized
```

- **wait()**：使当前执行代码的线程进行等待，wait()方法是Object类的方法，该方法用来将当前线程置入“预执行队列”中，并且在wait()所在的代码行处停止执行，直到接到通知或被中断为止。在调用wait()之前，线程必须获得该对象的对象级别锁，即只能在同步方法或同步块中调用wait()方法。在执行wait()方法后，当前线程释放锁。在从wait()返回前，线程与其他线程竞争重新获得锁。如果调用wait()方法时没有持有适当的锁，则抛出IllegalMonitorStateException异常,它是RuntimeException的一个子类，因此，不需要try-catch语句进行捕捉异常。

- **notify()**：也要在同步方法或同步块中调用，即在调用前，线程也必须获得该对象的对象级别锁。如调用notify()时没有持有适当的锁，也会抛出IllegalMonitorStateException。该方法用来通知那些可能等待该对象的对象锁的其他线程，如果有多个线程等待，则由线程规划器随机挑选出其中一个呈wait状态的线程，对其发出通知notify，并使它等待获取该对象的对象锁。需要说明的是，在执行notify()方法后，当前线程不会马上释放该对象锁，呈wait状态的线程也并不能马上获取该对象锁，到等到执行notify()方法的线程将程序执行完，也就是退出synchronized代码块后，当前线程才会释放锁，而呈wait状态所在的线程才可以获取该对象锁。当第一个获得了该对象锁的wait线程运行完毕以后，它会释放掉该对象锁，此时如果该对象没有再次使用notify语句，则即便该对象已经空闲，其他wait状态等待的线程由于没有得到该对象的通知，还会继续阻塞在wait状态，直到这个对象发出一个notify或notifyAll。

如果我们用了显式锁`Lock`，就不要用Object自带的这套机制了。比如`ReentrantLock`依赖`CAS`和`LockSupport`来实现，`LockSupport`提供了`park`和`unpark`。

- 调用`park`方法会使得当前线程丢失CPU使用权，从Runnable状态转变为Waiting状态。
- 调用`unpark`方法则反过来让Waiting状态的某个线程转变状态为Runnable，等待操作系统调度。

