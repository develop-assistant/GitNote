# JVM调优指令

## jdk命令行工具

```text
jps: 显示系统内所有hotspot虚拟机进程
jstat: 用于收集Hotspot虚拟机各方面的运行参数。
jinfo: 显示虚拟机配置信息
jmap: 生成虚拟机内存转储快照
jhat: 用于分析heapdump文件，它会建立一个http/html服务器，让用户可以在浏览器上查看分析结果
jstack: 显示虚拟机的线程快照
```



## 1. jps

`jps`(JVM Process Status) 命令类似 UNIX 的 `ps` 命令。

### 命令帮助

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jps -help
usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
```

### 参数说明

```
-l : 输出主类全名或jar路径
-q : 只输出LVMID
-m : 输出JVM启动时传递给main()的参数
-v : 输出JVM启动时显示指定的JVM参数
```

### 示例详解

```bash
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jps
25378 Jps
1701 idc-order-boot.jar
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jps -l
1701 target/idc-order-boot.jar
25391 sun.tools.jps.Jps
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jps -q
25410
1701
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jps -m
25425 Jps -m
1701 idc-order-boot.jar --spring.profiles.active=dev --server.port=8080
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jps -v
25440 Jps -Denv.class.path=.:/usr/java/jdk1.8.0_201/jre/lib/rt.jar:/usr/java/jdk1.8.0_201/lib/dt.jar:/usr/java/jdk1.8.0_201/lib/tools.jar -Dapplication.home=/usr/java/jdk1.8.0_201 -Xms8m
1701 idc-order-boot.jar
```



## 2. jstat

jstat（JVM Statistics Monitoring Tool） 使用于监视虚拟机各种运行状态信息的命令行工具。 它可以显示本地或者远程（需要远程主机提供 RMI 支持）虚拟机进程中的类信息、内存、垃圾收集、JIT 编译等运行数据，在没有 GUI，只提供了纯文本控制台环境的服务器上，它将是运行期间定位虚拟机性能问题的首选工具。

### 命令帮助

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -help
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -options
-class
-compiler
-gc
-gccapacity
-gccause
-gcmetacapacity
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcutil
-printcompilation
```

### 参数说明

```
[option] : 操作参数
LVMID : 本地虚拟机进程ID
[interval] : 连续输出的时间间隔
[count] : 连续输出的次数
```



### option参数说明

| 选项              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| -class            | 监视类装载、卸载数量、总空间以及类装载所耗费的时间           |
| -gc               | 监视java堆状况，包括Eden区，2个servivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息 |
| -gccapacity       | 监视内容与-gc基本相同，但输出主要关注java堆各个区域使用到的最大、最小空间 |
| -gcutil           | 监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比 |
| -gccause          | 与-gcutil功能一样，但是会额外输出导致上一次gc产生的原因      |
| -gcnew            | 监视新生代GC状况                                             |
| -gcnewcapacity    | 监视内容与-gcnew基本相同，输出主要关注使用到的最大最小空间   |
| -gcold            | 监视GC老年代状况                                             |
| -gcoldcapacity    | 监视内容与-gcold基本相同，输出主要关注使用到的最大最小空间   |
| -gcpermcapacity   | 输出永久代使用到的最大、最小空间                             |
| -compiler         | 输出JIT编译器编译过的方法、耗时等信息                        |
| -printcompilation | 输出已经被JIT编译的方法                                      |



### 示例详解

#### -class

监视类装载、卸载数量、总空间以及耗费的时间

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -class 1701
Loaded  Bytes  Unloaded  Bytes     Time
 14317 27168.2      140   209.1      25.95
```

说明

```
Loaded : 加载class的数量
Bytes : class字节大小
Unloaded : 未加载class的数量
Bytes : 未加载class的字节大小
Time : 加载时间
```



#### -compiler

输出JIT编译过的方法数量耗时等

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -compiler 1701
Compiled Failed Invalid   Time   FailedType FailedMethod
   13515      1       0   134.67          1 org/springframework/boot/loader/jar/Handler openConnection
```

说明

```
Compiled : 编译数量
Failed : 编译失败数量
Invalid : 无效数量
Time : 编译耗时
FailedType : 失败类型
FailedMethod : 失败方法的全限定名
```



#### -gc

监视java堆状况，包括Eden区，2个servivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -gc 1701 1000 5
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
5888.0 5888.0  0.0    0.0   47168.0   4656.0   117740.0   48648.4   86952.0 82657.6 10672.0 9866.7    306    6.441  14      2.882    9.323
5888.0 5888.0  0.0    0.0   47168.0   4656.0   117740.0   48648.4   86952.0 82657.6 10672.0 9866.7    306    6.441  14      2.882    9.323
5888.0 5888.0  0.0    0.0   47168.0   4656.0   117740.0   48648.4   86952.0 82657.6 10672.0 9866.7    306    6.441  14      2.882    9.323
5888.0 5888.0  0.0    0.0   47168.0   4656.0   117740.0   48648.4   86952.0 82657.6 10672.0 9866.7    306    6.441  14      2.882    9.323
5888.0 5888.0  0.0    0.0   47168.0   4656.0   117740.0   48648.4   86952.0 82657.6 10672.0 9866.7    306    6.441  14      2.882    9.323
```

上述指令的意思是1701进程每隔1000ms输出一次gc情况，一共5次。



说明

C即Capacity 总容量，U即Used 已使用的容量

```
S0C : survivor0区的总容量
S1C : survivor1区的总容量
S0U : survivor0区已使用的容量
S1C : survivor1区已使用的容量
EC : Eden区的总容量
EU : Eden区已使用的容量
OC : Old区的总容量
OU : Old区已使用的容量
MC : 当前元空间的容量 (KB)
MU : Metaspace的使用 (KB)
CCSC: 压缩类空间大小
CCSU: 压缩类空间使用大小
YGC : 新生代垃圾回收次数
YGCT : 新生代垃圾回收时间
FGC : 老年代垃圾回收次数
FGCT : 老年代垃圾回收时间
GCT : 垃圾回收总消耗时间
```



#### -gccapacity

同-gc，不过还会输出Java堆各区域使用到的最大、最小空间

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -gccapacity 1701
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
 10240.0 156992.0  58944.0 5888.0 5888.0  47168.0    20480.0   314048.0   117740.0   117740.0      0.0 1126400.0  86952.0      0.0 1048576.0  10672.0    306    14
```

说明

```
NGCMN : 新生代占用的最小空间
NGCMX : 新生代占用的最大空间
OGCMN : 老年代占用的最小空间
OGCMX : 老年代占用的最大空间
OGC：当前年老代的容量 (KB)
OC：当前年老代的空间 (KB)
PGCMN : perm占用的最小空间
PGCMX : perm占用的最大空间
```



#### -gcutils

监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -gcutil 1701
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  11.78  41.32  95.06  92.45    306    6.441    14    2.882    9.323
```



#### -gccause

与-gcutil功能一样，但是会额外输出导致上一次gc产生的原因

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -gccause 1701
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC
  0.00   0.00  11.95  41.32  95.06  92.45    306    6.441    14    2.882    9.323 Heap Inspection Initiated GC No GC
```

说明

```
LGCC：最近垃圾回收的原因
GCC：当前垃圾回收的原因
```



#### -gcnew

统计新生代的行为

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -gcnew 1701
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
5888.0 5888.0    0.0    0.0  1  15 2080.0  47168.0   5656.6    306    6.441
```

说明

```
TT：Tenuring threshold(提升阈值)
MTT：最大的tenuring threshold
DSS：survivor区域大小 (KB)
```



#### -gcnewcapacity

新生代与其相应的内存空间的统计

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -gcnewcapacity 1701
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC      YGC   FGC
   10240.0   156992.0    58944.0  15680.0   5888.0  15680.0   5888.0   125632.0    47168.0   306    14
```

说明

```
NGC:当前年轻代的容量 (KB)
S0CMX:最大的S0空间 (KB)
S0C:当前S0空间 (KB)
ECMX:最大eden空间 (KB)
EC:当前eden空间 (KB)
```



#### -gcold

统计旧生代的行为

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -gcold 1701
   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT
 86952.0  82657.6  10672.0   9866.7    117740.0     48648.4    306    14    2.882    9.323
```



#### -gcoldcapacity

统计旧生代的大小和空间

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -gcoldcapacity 1701
   OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     GCT
    20480.0    314048.0    117740.0    117740.0   306    14    2.882    9.323
```



#### -gcmetacapacity

元空间统计

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -gcmetacapacity 1701
   MCMN       MCMX        MC       CCSMN      CCSMX       CCSC     YGC   FGC    FGCT     GCT
       0.0  1126400.0    86952.0        0.0  1048576.0    10672.0   306    14    2.882    9.323
```



#### -printcompilation

hotspot编译方法统计

```shell
[root@iz2ze2e5wmatyx36v5jw5lz ~]# jstat -printcompilation 1701
Compiled  Size  Type Method
   13515     22    1 java/util/ResourceBundle$CacheKey getLoader
```

说明

```
Compiled：被执行的编译任务的数量
Size：方法字节码的字节数
Type：编译类型
Method：编译方法的类名和方法名。类名使用"/" 代替 "." 作为空间分隔符. 方法名是给出类的方法名. 格式是一致于HotSpot - XX:+PrintComplation 选项
```

