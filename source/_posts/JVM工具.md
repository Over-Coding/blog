---
title: JVM工具
date: 2020-01-20 10:02:18
categories:
	- JVM
tags: 
	- JVM
	- 工具
toc: true
typora-root-url: JVM工具
---



### 1、概述

JDK为开发人员提供一些工具，可以查看异常堆栈、虚拟机运行日志、垃圾收集器日志、线程快照、堆转储快照等。当我们遇到一些问题时，通过这些工具定位问题，解决问题。下面介绍几个常用的工具。



### 2、基础工具

#### 2.1、jps

jps（JVM Process Status Tool）列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class，main() 函数所在的类）名称以及这些进程的本地虚拟机唯一ID(LVMID，Local Virtual Machine Identifier)。



**命令格式：**

```shell
jps [options] [hostid]
```



**参数解释：**

* options：可以使用如下参数。

  * -q：显示进程ID。
  * -m：显示进程ID，主类名称，以及传入main方法的参数。
  *  -l：显示进程ID，主类全名。
  * -v：显示进程ID，主类名称，以及传入JVM的参数。
  * -V：显示进程ID，主类名称。

  注意：其中 -mlv 可以连用。

* hostid：主机或者服务器的ip，如果不指定，默认为当前的主机或者是服务器。



**示例：**列出所有的虚拟机进程。

```shell
jps -l
```



#### 2.2、jstat

jstat（JVM Statistics Monitoring Tool）是用于监视虚拟机各种运行状态信息的命令行工具。可以显示本地或者远程虚拟机进程中的类加载、内存、垃圾收集、即时编译等运行时数据。



**命令格式：**

```shell
jstat [option vmid [interval[s|ms] [count]]] <pid>
```



**参数解释：**

* option：指定要查询的虚拟机信息，主要分为3类：类加载、内存、垃圾收集和即时编译状况，具体选项及作用如下：
  * -class：监视类加载、卸载数量、总空间以及类装载所耗费的时间。
  * -compiler：输出即时编译器编译过的方法、耗时等信息。
  * -gc：监视Java 堆状况，包括 Eden 区、2个 Survivor区、老年代、永久代等的容量，已用空间，垃圾收集时间合计等信息。
  * -gccapacity：监视内容与 gc 基本相同，但输出主要关注 Java 堆各个区域使用到的最大、最小空间。
  * -gccause：显示有关垃圾收集统计信息（同-gcutil），以及上一次和当前（如果适用）垃圾收集事件的原因。
  * -gcnew：显示新生代行为的统计信息。
  * -gcnewcapacity：显示有关新生代大小及其相应空间的统计信息。
  * -gcold：显示有关老年代行为的统计信息。
  * -gcoldcapacity：显示有关老年代大小及其相应空间的统计信息。
  * -gcmetacapacity：显示有关元空间大小的统计信息。
  * -gcutil：显示有关垃圾收集统计信息。
  * -printcompilation：显示Java HotSpot VM编译方法统计信息。



* vmid：如果是本地虚拟机进程，vmid和本地虚拟机唯一ID是一致的；如果是远程虚拟机进程，那么vmid的格式应当是：protocol:lvmid[@hostname[:port]/servername]

* interval：采样间隔，单位为秒(s)或毫秒(ms)，默认单位是毫秒。必须为正整数。指定后，该jstat命令将在每个间隔产生其输出。

* count：要显示的样本数。



**示例：**

1. 展示类加载器统计信息。

```shell
jstat -class 6240
```

结果：

Loaded  Bytes  Unloaded  Bytes     Time
   610  1232.9        0     0.0       0.08

* Loaded：已加载的类总数。
* Bytes：加载的KB数。
* Unloaded：卸载的类总数。
* Bytes：卸载的KB数。
* Time：执行类加载和卸载操作所花费的时间。



2. 展示Java HotSpot VM 即时编译器统计信息。

```shell
jstat -compiler 6240
```

结果：

Compiled Failed Invalid   Time   FailedType FailedMethod
      74      0       0     0.02          0

* Compiled：执行的编译任务数。
* Failed：失败的编译任务数。
* Invalid：无效的编译任务数。
* Time：执行编译任务所花费的时间。
* FailedType：上次失败的编译的编译类型。
* FailedMethod：上次失败的编译的类名和方法。



3. 展示垃圾收集的堆统计信息。

```shell
jstat -gc 6240
```

结果：

 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT    GCT
1024.0 1024.0  0.0    0.0    8192.0   5049.1   10240.0      0.0     4480.0 776.8  384.0   76.6       0    0.000   0      0.000  0.000

* S0C：当前幸存者空间0容量（KB）。
* S1C：当前幸存者空间1容量（KB）。
* S0U：幸存者空间0使用大小（KB）。
* S1U：幸存者空间1使用大小（KB）。
* EC：当前伊甸园空间容量（KB）。
* EU：伊甸园空间使用大小（KB）。
* OC：当前老年代容量（KB）。
* OU：老年代使用大小（KB）。
* MC：元空间容量（KB）。
* MU：元空间使用大小（KB）。
* CCSC：压缩的类空间容量（KB）。
* CCSU：使用的压缩类空间（KB）。
* YGC：新生代垃圾收集事件的数量。
* YGCT：新生代垃圾收集时间。
* FGC：完整GC垃圾收集事件的数量。
* FGCT：完整GC垃圾收集时间。
* GCT：总垃圾收集时间。



#### 2.3、jinfo

jinfo（Configuration Info for Java）的作用是实时查看和调整虚拟机各项参数。



**命令格式：**

```shell
jinfo [option] <pid>
```



**参数解释：**

* option
  * no option：输出全部的参数和系统属性。
  * -flag name：输出对应名称的参数。
  * -flag[+|-]name：开启或者关闭对应名称的参数。
  * -flag name=value：设定对应名称的参数。
  * -flags：输出全部的参数。
  * -sysprops：输出系统属性。



**示例：**输出当前进程的jvm全部参数。

```shell
jinfo -flags 6240
```



#### 2.4、jmap

jmap（Memory Map for Java）可以生成 java 程序的 dump 文件，也可以查看堆内对象信息、查看ClassLoader 的信息以及 finalizer 队列。



**命令格式：**

```shell
jmap [option] <pid>
```



**参数解释：**

* option
  * no option：查看进程的内存映像信息，类似 Solaris pmap命令。
  * -heap：显示Java堆详细信息。
  * -histo[:live]：显示堆中对象的统计信息。
  * -clstats：打印类加载器信息。
  * -finalizerinfo：显示在F-Queue队列等待Finalize线程执行finalizer方法的对象。
  * -dump:<dump-options>：生成堆转储快照。



**示例：**生成堆转储快照。

```shell
jmap -dump:live,format=b,file=E:\jmap.bin 34152
```



#### 2.5、jstack

jstack（Stack Trace for Java）命令用于查看或导出虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、长时间等待外部资源等。线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么 事情，或者等待什么资源。如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息。



**命令格式：**

```shell
jstack [option] <pid>
```



**参数解释：**

* option
  * -F：当线程挂起时，使用jstack -l pid 请求不被响应时，强制输出线程堆栈。
  * -l：除堆栈外，显示关于锁的附加信息，例如ownable synchronizers
  * -m：可以同时输出java以及C/C++的堆栈信息。



**示例：** 输出堆栈信息，并显示关于锁的附加信息。

```shell
jstack -l 34692
```



### 3、可视化工具

#### 3.1、JConsole

JConsole（Java Monitoring and Management Console）是一款基于JMX的可视化监视、管理工具。它的主要功能是通过JMX的MBean 对系统进行信息收集和参数动态调整。

**连接：**

![](console连接.PNG)

可以连接本地进程，也支持远程进程。



**主界面：**

![](console主界面.PNG)

概览中展示了堆内存、线程、类、CPU的使用信息。上面还可以查看每个模块详细的信息。这里没什么难度，就不详细介绍了。



#### 3.2、JVisualVM

VisualVM（All-in-One Java Troubleshooting Tool）是功能最强大的运行监视和故障处理程序之一，曾经在很长一段时间内是Oracle官方主力发展的虚拟机故障处理工具。Oracle曾在VisualVM的软件说明中写上了“All-in-One”的字样，预示着它除了常规的运行监视、故障处理外，还将提供其他方面的能力，譬如性能分析（Profiling）。VisualVM的性能分析功能比起JProfiler、YourKit等专业且收费的Profiling工具都不遑多让。而且相比这些第三方工具，VisualVM还有一个很大的优点：不需要被监视的程序基于特殊Agent去运行，因此它的通用性很强，对应用程序实际性能的影响也较小，使得它可以直接应用在生产环境中。这个优点是JProfiler、YourKit等工具无法与之媲美的。

![](visualVM首页.PNG)