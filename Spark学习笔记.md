# spark笔记

零零散散学了一些spark，记个笔记吧

废话不多说，上图，看了一下官网第一张图，挺nb的

![image-20210912230710487](C:\Users\fanyuping\AppData\Roaming\Typora\typora-user-images\image-20210912230710487.png)

##### 一、spark和Hadoop的根本区别是什么？

spark多个作业之间的数据通信是基于内存，而Hadoop是基于磁盘

##### 二、了解spark的架构

spark core ：spark最核心的东西

spark sql ，spark streaming 等等都是基于spark core进行扩展的

##### 三、spark运行环境

###### 3.1 local模式

1. 他就相当于是Hadoop的本地模式，有Hadoop基础的人大家应该都能理解

2. 本地模式提交应用

   ```bash
   bin/spark-submit \
   --class org.apache.spark.examples.SparkPi \
   --master local[2] \
   ./examples/jars/spark-examples_2.12-3.0.0.jar \
   10
   ```

   --master local[2] 部署模式，默认为本地模式，2表示虚拟cpu核数量

###### 3.2 standalone 模式

1. 翻译一下就是“独立部署”，也就是集群模式，master-slave

2. 启动集群命令：sbin/start-all.sh

3. standalone模式提交应用

   ```bash
   bin/spark-submit \
   --class org.apache.spark.examples.SparkPi \
   --master spark://linux1:7077 \
   ./examples/jars/spark-examples_2.12-3.0.0.jar \
   10
   ```

   --master spark://linux1:7077 连接到spark集群

   数字10表示程序的入口参数，用于设定当前应用的任务数量

4. 提交参数说明

   | 参数                     | 解释                                                         | 可选值(例)                                                   |
   | ------------------------ | ------------------------------------------------------------ | :----------------------------------------------------------- |
   | --class                  | Spark 程序中包含主函数的类                                   |                                                              |
   | --master                 | Spark 程序运行的模式(环境)                                   | 本地：local[*]、集群：spark://linux1:7077、运行与 Yarn：Yarn |
   | -executor-memory 1G      | 指定每个 executor 可用内存为 1G                              |                                                              |
   | --total-executor-cores 2 | 指定所有executor使用的cpu核数 。 为 2 个                     |                                                              |
   | --executor-cores         | 指定每个executor使用的cpu核数                                |                                                              |
   | application-jar          | 打包好的应用 jar，包含依赖。这 个 URL 在集群中全局可见。 比 如 hdfs:// 共享存储系统，如果是file:// path，那么所有的节点的 path 都包含同样的 jar |                                                              |
   | application-arguments    | 传给 main()方法的参数                                        |                                                              |

###### 3.3 Yarn 模式

1. standalone模式有spark自身提供计算资源，无需其他框架提供资源，但是spark主要是计算框架，而并非资源调度框架，国内spark一般都是运行在Yarn上的。其实现在不必要纠结是怎样结合在一起的，循序渐进嘛

###### 3.4 K8s 模式

1. 大佬们经常说的容器内spark
2. 顺便说说k8s，主要是做容器编排的，核心就是它的pod，可以先了解一下

##### 四、spark运行架构

之前也说过，spark是标准的master-slave结构

master：在spark里一般叫Driver

slave：在spark里一般叫Executor

Driver 负责分发计算逻辑等等，

executor 主要是负责接收driver发来的计算逻辑，计算，可以说是苦力哈哈哈

下面官方一点的说说吧

###### 4.1 Driver

1. spark驱动器节点，用于执行spark任务的main方法，下面说说dirver在一次spark作业中主要负责的是什么工作
   1. 将用户程序转为job
   2. 在Executor之间调度task
   3. 跟踪Executor的执行情况
   4. web ui 展示

###### 4.2 Executor

1. 其实每个Executor就是集群一个节点的java进程，负责接收task，执行task
2. 同样如果有executor节点出错，dirver会调度到别的节点继续执行
3. 我的工作
   1. 负责运行task，结果返回给dirver
   2. 通过自身的块管理器，为用户程序中要求缓存的RDD提供内存存储，RDD是缓存在executor进程内的，任务可以充分利用缓存数据加速运算

###### 4.3 master 和 worker

1. 相当于Yarn 的 RM 和 NM ，刚刚不是说他有standalone模式嘛，这时候它靠的就是自己的资源调度。

###### 4.4 ApplicationMaster

1. Hadoop用户向yarn集群提交应用程序的时候，提交程序中应该还包含AM，用于向资源调度器申请执行任务的资源容器 container，运行用户自己的job，监考任务执行，跟踪任务的状态处理任务失败等异常情况

   相当于是说，rm和driver之间的解耦合，就是靠am

##### 五、并行度

​	集群同时运行的任务数量，并行的运行。

##### 六、有向无环图--DAG

​	DAG（Directed Acyclic Graph）有向无环图是由点和线组成的拓扑图形，该图形具有方 向，不会闭环。

```
大数据计算引擎框架我们根据使用方式的不同一般会分为四类，其中第一类就是 Hadoop 所承载的 MapReduce,它将计算分为两个阶段，分别为 Map 阶段 和 Reduce 阶段。 对于上层应用来说，就不得不想方设法去拆分算法，甚至于不得不在上层应用实现多个 Job  的串联，以完成一个完整的算法，例如迭代计算。 由于这样的弊端，催生了支持 DAG 框 架的产生。因此，支持 DAG 的框架被划分为第二代计算引擎。如 Tez 以及更上层的 Oozie。这里我们不去细究各种 DAG 实现之间的区别，不过对于当时的 Tez 和 Oozie 来 说，大多还是批处理的任务。接下来就是以 Spark 为代表的第三代的计算引擎。第三代计 算引擎的特点主要是 Job 内部的 DAG 支持（不跨越 Job），以及实时计算。 这里所谓的有向无环图，并不是真正意义的图形，而是由 Spark 程序直接映射成的数据 流的高级抽象模型。简单理解就是将整个程序计算的执行过程用图形表示出来,这样更直观， 更便于理解，可以用于表示程序的拓扑结构。
```

##### 七、提交流程

spark应用程序提交到yarn去执行的时候，有两种部署执行的方式，client和cluster

###### 7.1 client

1. 类似本地local模式

###### 7.2 cluster模式

Cluster 模式将用于监控和调度的 Driver 模块启动在 Yarn 集群资源中执行。一般应用于 实际生产环境。

 在 YARN Cluster 模式下，任务提交后会和 ResourceManager 通讯申请启动 ApplicationMaster，

 随后 ResourceManager 分配 container，在合适的 NodeManager 上启动 ApplicationMaster， 此时的 ApplicationMaster 就是 Driver。 

Driver 启动后向 ResourceManager 申请 Executor 内存，ResourceManager 接到 ApplicationMaster 的资源申请后会分配 container，然后在合适的 NodeManager 上启动 Executor 进程 

Executor 进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行 main 函数， 

之后执行到 Action 算子时，触发一个 Job，并根据宽依赖开始划分 stage，每个 stage 生 成对应的 TaskSet，之后将 task 分发到各个 Executor 上执行。

