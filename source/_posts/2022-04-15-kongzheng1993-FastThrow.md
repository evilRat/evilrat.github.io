---
title: 什么！JVM连报错都优化了？
excerpt: 'fast throw'
tags: [JVM]
categories: [JVM]
comments: true
date: 2022-04-15 18:30:52
---


今天查一个生产问题，发现很奇怪的NPE，没有打印堆栈信息，幸好几天前也发生过类似的问题，找到了堆栈信息，修复了问题。

今天查了一些资料，JVM竟然有一个Fast Throw的机制。为了避免同类异常频繁抛出，造成性能下降。

相关JDK源码如下：

```java
// If this throw happens frequently, an uncommon trap might cause
  // a performance pothole.  If there is a local exception handler,
  // and if this particular bytecode appears to be deoptimizing often,
  // let us handle the throw inline, with a preconstructed instance.
  // Note:   If the deopt count has blown up, the uncommon trap
  // runtime is going to flush this nmethod, not matter what.
  if (treat_throw_as_hot
      && (!StackTraceInThrowable || OmitStackTraceInFastThrow)) {
    // If the throw is local, we use a pre-existing instance and
    // punt on the backtrace.  This would lead to a missing backtrace
    // (a repeat of 4292742) if the backtrace object is ever asked
    // for its backtrace.
    // Fixing this remaining case of 4292742 requires some flavor of
    // escape analysis.  Leave that for the future.
    ciInstance* ex_obj = NULL;
    switch (reason) {
    case Deoptimization::Reason_null_check:
      ex_obj = env()->NullPointerException_instance();
      break;
    case Deoptimization::Reason_div0_check:
      ex_obj = env()->ArithmeticException_instance();
      break;
    case Deoptimization::Reason_range_check:
      ex_obj = env()->ArrayIndexOutOfBoundsException_instance();
      break;
    case Deoptimization::Reason_class_check:
      if (java_bc() == Bytecodes::_aastore) {
        ex_obj = env()->ArrayStoreException_instance();
      } else {
        ex_obj = env()->ClassCastException_instance();
      }
      break;
    default:
      break;
    }
}
```

从上面的代码可以看出，满足以下四种情况，异常会被预定义的无堆栈异常替换：

- tread_throw_as_hot，也就是hot exception，被抛出很多次的异常类型
    - 同样的异常抛出太多
    - 同样的异常抛出次数不为0，以及方法拥有local-exception-handler，那么将被标记为hot
- !StackTraceInThrowable || OmitStackTraceInFastThrow, 由于这两个参数值默认都是true，所以默认整体条件满足。
- 异常是设定内的五种异常
    - NullPointerException
    - ArithmeticException
    - ArrayIndexOutOfBoundsException
    - ArrayStoreException
    - ClassCastException
- 补充：只能是implict exception。stackoverflow上的一个回答说这里的异常要是implict exception, 也就是说代码里面直接throw new NullPointerException() 这种异常是不会被替换的。亲测符合预期。

### 关闭Fast Throw

启动增加-XX:-OmitStackTraceInFastThrow这个参数就可以关闭JIT后特定异常fast throw

### 复现

```java
public class Test{
    public static void main(String... args) {
        String a = null;
        for (int i = 0; i< 700000; i++) {
            try {
                // System.out.println(1111);
                a.toString();
            } catch (Exception e) {
                // System.out.println(2222);
                e.printStackTrace();
            }
        }
    }
}
```

编译后执行：

```shell
java Test
```

结果：

<img src='noParam.png'/>


关闭fast throw执行：

```shell
java -XX:-OmitStackTraceInFastThrow Test
```

结果：
<img src='param.png'>