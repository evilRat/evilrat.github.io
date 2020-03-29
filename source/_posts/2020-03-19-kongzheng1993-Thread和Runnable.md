# Thread和Runnable

我们都知道Java中创建线程有两种方式，继承Thread类和实现Runnable接口。其实呢，我们看了Thread的源码就能了解到：Thread实现了Runnable，其实也是个Runnable。

Thread的构造方法中有一个是：

![image-20200319172854321](C:\Users\kongz\AppData\Roaming\Typora\typora-user-images\image-20200319172854321.png)

这里传入的是一个Runnable，而继续往下执行，到了最后也是执行的这个target的`run()`方法。

如果我们不传入Runnable呢？

![image-20200319173106843](C:\Users\kongz\AppData\Roaming\Typora\typora-user-images\image-20200319173106843.png)

可以看到这个方法target是null。最后执行的是Thread重写的`run()`方法。

所以看到这里可以了解到：

- Runnable就是个接口，写一个类实现Runnable接口，它并不能执行，它要提交到一个线程（Thread），在Thread实例化调用它的构造方法的时候，才会触发Runnable的run方法。

- 继承Thread来创建线程，重写了`run()`方法，当我们实例化这个类的时候，将会调用到父类，也就是Thread的无参构造函数，也就会执行我们这里重写的`run()`方法了。而这里的Thread实现了Runnable接口，所以它其实也是个Runnable。

## 重要属性

### 优先级

```java
private int  priority;
public final static int MIN_PRIORITY = 1;
public final static int NORM_PRIORITY = 5;
public final static int MAX_PRIORITY = 10;
   public final void setPriority(int newPriority) {
        ThreadGroup g;
        checkAccess();
        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
            throw new IllegalArgumentException();
        }
        if((g = getThreadGroup()) != null) {
            if (newPriority > g.getMaxPriority()) {
                newPriority = g.getMaxPriority();
            }
            setPriority0(priority = newPriority);
        }
    }
    public final int getPriority() {
        return priority;
    }
```

### 线程的状态

```java
public enum State {
        NEW,
        RUNNABLE,
        BLOCKED,
        WAITING,
        TIMED_WAITING,
        TERMINATED;
    }
      public State getState() {
        return sun.misc.VM.toThreadState(threadStatus);
    }
```

网上找到的图：

![image-20200319181144600](C:\Users\kongz\AppData\Roaming\Typora\typora-user-images\image-20200319181144600.png)

### 主要方法

- 本地方法获取当前线程

  ```java
  public static native Thread currentThread();
  ```

- run()

  ```java
  public void run() {
          if (target != null) {
              target.run();
          }
      }
  ```

  这里就是上面说的，如果是传入Runnable的话，这里target就不会是null了，如果没有传入target，就不会执行什么，但是一般我们不传入target，就会重写`run()`方法了。

- start()

  ```java
  public synchronized void start() {
          /**
           * This method is not invoked for the main method thread or "system"
           * group threads created/set up by the VM. Any new functionality added
           * to this method in the future may have to also be added to the VM.
           *
           * A zero status value corresponds to state "NEW".
           */
          if (threadStatus != 0)
              throw new IllegalThreadStateException();
  
          /* Notify the group that this thread is about to be started
           * so that it can be added to the group's list of threads
           * and the group's unstarted count can be decremented. */
          group.add(this);
  
          boolean started = false;
          try {
              start0();
              started = true;
          } finally {
              try {
                  if (!started) {
                      group.threadStartFailed(this);
                  }
              } catch (Throwable ignore) {
                  /* do nothing. If start0 threw a Throwable then
                    it will be passed up the call stack */
              }
          }
      }
  
      private native void start0();
  ```

  这里会调用一个本地方法`start0()`，应该会起一个线程执行`run()`方法。

- boolean interrupted() & void interrupt() & boolean isInterrupted()

  **interrupt()**:中断本线程(将中断状态标记为true)
  **isInterrupted()**:检测本线程是否已经中断 。如果已经中断，则返回true，否则false。中断状态不受该方法的影响。 如果中断调用时线程已经不处于活动状态，则返回false。
  **interrupted()**:检测当前线程是否已经中断 。如果当前线程存在中断，返回true，并且修改标记为false。再调用isIterruoted()会返回false。如果当前线程没有中断标记，返回false，不会修改中断标记。

- void sleep(long millis) & void sleep(long millis, int nanos)

  ```java
  public static native void sleep(long millis) throws InterruptedException;
  
      /**
       * Causes the currently executing thread to sleep (temporarily cease
       * execution) for the specified number of milliseconds plus the specified
       * number of nanoseconds, subject to the precision and accuracy of system
       * timers and schedulers. The thread does not lose ownership of any
       * monitors.
       *
       * @param  millis
       *         the length of time to sleep in milliseconds
       *
       * @param  nanos
       *         {@code 0-999999} additional nanoseconds to sleep
       *
       * @throws  IllegalArgumentException
       *          if the value of {@code millis} is negative, or the value of
       *          {@code nanos} is not in the range {@code 0-999999}
       *
       * @throws  InterruptedException
       *          if any thread has interrupted the current thread. The
       *          <i>interrupted status</i> of the current thread is
       *          cleared when this exception is thrown.
       */
      public static void sleep(long millis, int nanos)
      throws InterruptedException {
          if (millis < 0) {
              throw new IllegalArgumentException("timeout value is negative");
          }
  
          if (nanos < 0 || nanos > 999999) {
              throw new IllegalArgumentException(
                                  "nanosecond timeout value out of range");
          }
  
          if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
              millis++;
          }
  
          sleep(millis);
      }
  ```

  两个参数的sleep只是将nanos转为millis(四舍五入转换)，调用sleep(millis)方法。
  sleep(millis)是本地方法，让当前线程休眠指定时间。
  sleep不释放锁。

- void join() & void join(long millis) & void join(long millis, int nanos)

  将线程加入到当前线程中，使父线程等待子线程执行完毕再继续执行。

  