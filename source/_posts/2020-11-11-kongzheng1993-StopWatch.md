---
title: 你不知道StopWatch吗？
excerpt: 'Spring'
tags: [Spring]
categories: [Spring]
comments: true
date: 2020-11-11 10:30:52
---

昨天测试提了一个bug，页面数据导出excel太慢，ontest环境试了一下，导出4k条数据，竟然花了3min！我本地和test环境测试都没问题啊，一时间摸不着头脑。准备给代码关键步骤加日志，发到ontest测试。正准备`long start = System.currentTimeMillis();`，同事来了一句：你不知道StopWatch吗？


Spring提供的计时器StopWatch对于秒、毫秒为单位方便计时的程序，尤其是单线程、顺序执行程序的时间特性的统计输出支持比较好。也就是说假如我们手里面有几个在顺序上前后执行的几个任务，而且我们比较关心几个任务分别执行的时间占用状况，希望能够形成一个不太复杂的日志输出，StopWatch提供了这样的功能。而且Spring的StopWatch基本上也就是仅仅为了这样的功能而实现。


文章开头的场景，下面就是类似的实践。

```java
import org.springframework.util.StopWatch;  
  
public class StopWatchDemo {  
  
    /** 
     * @param args 
     * @throws InterruptedException 
     */  
    public static void main(String[] args) throws InterruptedException {  
        // TODO Auto-generated method stub  
        StopWatch clock = new StopWatch();  
        clock.start("TaskOneName");  
        Thread.sleep(1000 * 3);// 任务一模拟休眠3秒钟  
        clock.stop();  
        clock.start("TaskTwoName");  
        Thread.sleep(1000 * 10);// 任务一模拟休眠10秒钟  
        clock.stop();  
        clock.start("TaskThreeName");  
        Thread.sleep(1000 * 10);// 任务一模拟休眠10秒钟  
        clock.stop();  
  
        System.out.println(clock.prettyPrint());  
    }  
  
}
```
日志输出如下：

```bash
StopWatch '': running time (millis) = 22926
-----------------------------------------
ms     %     Task name
-----------------------------------------
02990  013%  TaskOneName
09968  043%  TaskTwoName
09968  043%  TaskThreeName
```

### 原理：

这个`StopWatch`的原理也很简单，`StopWatch`有个内部类`TaskInfo`，内部有两个属性，taskName和timeMillis。说是叫taskName，不过这里面并没有启动新的线程，只是新建一个TaskInfo对象，在StopWatch的一次启停之后，记录启停之间的时间差。

