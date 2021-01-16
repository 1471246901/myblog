# jmap工具使用

命令jmap是一个多功能的命令。它可以生成 java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列

```shell
PS C:\Users\14712> jmap -help
Usage:
    jmap -clstats <pid>
        to connect to running process and print class loader statistics
    jmap -finalizerinfo <pid>
        to connect to running process and print information on objects awaiting finalization
    jmap -histo[:live] <pid>
        to connect to running process and print histogram of java object heap
        if the "live" suboption is specified, only count live objects
    jmap -dump:<dump-options> <pid>
        to connect to running process and dump java heap
    jmap -? -h --help
        to print this help message

    dump-options:
      live         dump only live objects; if not specified,
                   all objects in the heap are dumped.
      format=b     binary format
      file=<file>  dump heap to <file>

    Example: jmap -dump:live,format=b,file=heap.bin <pid>
```



## 参数：

-   **option：** 选项参数。
-   **pid：** 需要打印配置信息的进程ID。
-   **executable：** 产生核心dump的Java可执行文件。
-   **core：** 需要打印配置信息的核心文件。
-   **server-id** 可选的唯一id，如果相同的远程主机上运行了多台调试服务器，用此选项参数标识服务器。
-   **remote server IP or hostname** 远程调试服务器的IP地址或主机名。

## option

-   **no option：** 查看进程的内存映像信息,类似 Solaris pmap 命令。
-   **heap：** 显示Java堆详细信息
-   **histo[:live]：** 显示堆中对象的统计信息
-   **clstats：**打印类加载器信息
-   **finalizerinfo：** 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
-   **dump:<dump-options>：**生成堆转储快照
-   **F：** 当-dump没有响应时，使用-dump或者-histo参数. 在这个模式下,live子参数无效.
-   **help：**打印帮助信息
-   **J<flag>：**指定传递给运行jmap的JVM的参数

## 打印类加载器信息

`jmap -clstats 4312` 



```shell
 6632    15         0        736           0     3064          30       617      6224     4184     6352    10536 sun.util.logging.PlatformLogger
 6633    15         0        512           0      696          18        21      2672     1352     2832     4184 sun.util.logging.PlatformLogger$Bridge
 6634    15         0        512           0      416           2        21       352      280     1176     1456 sun.util.logging.PlatformLogger$ConfigurableBridge
 6635    15         0        536           0      416           3         5       456      304     1288     1592 sun.util.logging.PlatformLogger$ConfigurableBridge$LoggerConfiguration
 6636    15         0        512           0      232           1         0       144      120      920     1040 sun.util.logging.internal.LoggingProviderImpl$LogManagerAccess
 6637    15         0        520           0     5056          11       811      3192     2848     6312     9160 sun.util.resources.Bundles
 6638    15         0        568           0     1312           3       103       680      688     2152     2840 sun.util.resources.Bundles$2
 6639    15         0        512           0      256           1         0       144      128      944     1072 sun.util.resources.Bundles$CacheKeyReference
 6640    15         0        512           0      296           3         0       448      272     1152     1424 sun.util.resources.Bundles$Strategy
 6641    15         0        568           0      824           3        35       536      496     1696     2192 sun.util.resources.LocaleData$1
 6642  6643         0        560           0      336           1         5       168      192     1064     1256 sun.util.resources.LocaleData$CommonResourceBundleProvider
 6643    15         0        560           0      544           2        14       368      344     1344     1688 sun.util.resources.LocaleData$LocaleDataResourceBundleProvider
 6644  4096         0        632           0     1832          10       285      2120     1496     3328     4824 sun.util.resources.OpenListResourceBundle
 6645  6644         0        632           0      800           5        71       960      640     1912     2552 sun.util.resources.TimeZoneNamesBundle
 6646  6642         0        560           0     1056           3        60       640      552     1904     2456 sun.util.resources.provider.LocaleDataProvider
 6647    15         0        600           0     1248           5       159      1000      856     2256     3112 sun.util.resources.provider.NonBaseLocaleDataMetaInfo
 6648  4250         0        544           0      304           2         5       312      192     1104     1296 sun.util.spi.CalendarProvider
             12376272    4216016       75576 12129872       62338   2488246  14441856 11090464 21744488 32834952 Total
                37.7%      12.8%        0.2%    36.9%           -      7.6%     44.0%    33.8%    66.2%   100.0%
Index Super InstBytes KlassBytes annotations    CpAll MethodCount Bytecodes MethodAll    ROAll    RWAll    Total ClassName
```

## 生成堆转储快照dump文件

`jmap  -dump:file=C:\Users\14712\Desktop\q.dump 4312`

```shell
Heap dump file created
```

