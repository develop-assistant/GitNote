# JVM线上调优



## 1. 调优原则

1. 必须有监控；
2. 调优目标: 
   	- 最短停顿时间? 	CMS
      	- 最大吞吐量(吞吐量 = 运行用户代码时间 /（运行用户代码时间 + 垃圾收集时间))？ Parallel Scavenge
3. 调优手段



### 官方推荐调优流程

![官方推荐jvm调优流程](../assets/官方推荐jvm调优流程.png)



### 1.2. Hotspot虚拟机优化大纲

![Hotspot配置优化表](../assets/Hotspot配置优化表.png)



## 

## 2. 线上问题

- 内存泄露；
- 某个进程突然 `CPU` 飙升；
- 线程死锁；
- 响应变慢(长时间卡顿)。
- FGC频繁



## 3. 调优方向

- 内存调优
- 垃圾回收策略调优



## 4. Arthas线上监控

1. 线上启动arthas后attach到我们需要监控的进程；
2. 帮助文档

```shell
[arthas@1701]$ help
 NAME         DESCRIPTION
 help         Display Arthas Help
 keymap       Display all the available keymap for the specified connection.
 sc           Search all the classes loaded by JVM
 sm           Search the method of classes loaded by JVM
 classloader  Show classloader info
 jad          Decompile class
 getstatic    Show the static field of a class
 monitor      Monitor method execution statistics, e.g. total/success/failure count, average rt, fail rate, etc.
 stack        Display the stack trace for the specified class and method
 thread       Display thread info, thread stack
 trace        Trace the execution time of specified method invocation.
 watch        Display the input/output parameter, return object, and thrown exception of specified method invocation
 tt           Time Tunnel
 jvm          Display the target JVM information
 ognl         Execute ognl expression.
 mc           Memory compiler, compiles java files into bytecode and class files in memory.
 redefine     Redefine classes. @see Instrumentation#redefineClasses(ClassDefinition...)
 dashboard    Overview of target jvm's thread, memory, gc, vm, tomcat info.
 dump         Dump class byte array from JVM
 heapdump     Heap dump
 options      View and change various Arthas options
 cls          Clear the screen
 reset        Reset all the enhanced classes
 version      Display Arthas version
 shutdown     Shutdown Arthas server and exit the console
 stop         Stop/Shutdown Arthas server and exit the console. Alias for shutdown.
 session      Display current session information
 sysprop      Display, and change the system properties.
 sysenv       Display the system env.
 vmoption     Display, and update the vm diagnostic options.
 logger       Print logger info, and update the logger level
 history      Display command history
 cat          Concatenate and print files
 pwd          Return working directory name
 mbean        Display the mbean information
 grep         grep command for pipes.
 profiler     Async Profiler. https://github.com/jvm-profiling-tools/async-profiler
```

![arthas指令表](../assets/arthas指令表.png)



### 1、dashboard

在arthas的命令行界面，输入dashboard，会实时展示当前tomcat的多线程状态、Jvm各区域、GC情况等信息

![arthas-dashboard](../assets/arthas-dashboard.png)



### 2、查看线程监控

常用参数

输入thread会显示所有线程的状态信息

输入thread -n 3会显示当前最忙的3个线程，可以用来排查线程CPU消耗

输入thread -b 会显示当前处于BLOCKED状态的线程，可以排查线程锁的问题

![arthas-thread](../assets/arthas-thread.png)



### 3、jvm监控

输入jvm，查看jvm详细的性能数据

![arthas-jvm](../assets/arthas-jvm-1227024.png)



### 4、函数耗时监控

通常说一个接口性能不好，其实就是接口响应时间比较长造成的，具体代码中哪个函数耗时比较长呢？可以使用trace功能来监控一下

```
trace com.idc.order.modules.sys.controller.AccountController login
```

解释：

-j参数可以过滤掉jdk自身的函数

om.idc.order.modules.sys.controller.AccountController是接口所在的类

Login是接口的入口函数

![arthas-trace](../assets/arthas-trace.png)

通过圈起来的部分可以看到，接口的入口函数login总耗时268ms

其中verify函数耗时85ms

很明显，最慢的函数已经找到了，接下里就要去对代码进行进一步分析，然后再进行优化



## 参考

- https://juejin.im/post/5dd0c0b95188253d73575ca1
- https://juejin.im/post/59e6c1f26fb9a0451c397a8c