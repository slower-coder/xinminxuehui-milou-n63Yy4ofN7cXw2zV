
**大纲**


**1\.使用jstat了解线上系统的JVM运行状况**


**2\.使用jmap和jhat了解线上系统的对象分布**


**3\.如何分析JVM运行状况并合理优化**


**4\.使用jstat分析模拟的BI系统JVM运行情况**


**5\.使用jstat分析模拟的计算系统JVM运行情况**


**6\.问题汇总**


 


**1\.使用jstat了解线上系统的JVM运行状况**


**(1\)JVM的整体运行原理简单总结**


**(2\)功能强大的jstat**


**(3\)jstat \-gc PID**


**(4\)其他的jstat命令**


**(5\)到底该如何使用jstat工具**


**(6\)新生代对象增长的速率**


**(7\)Young GC的触发频率和每次耗时**


**(8\)每次Young GC后有多少对象进入老年代**


**(9\)Full GC的触发时机和耗时**


 


**(1\)JVM的整体运行原理简单总结**


一.对象优先在Eden区分配


二.Young GC的触发时机和执行过程


三.对象进入老年代的时机


四.Full GC的触发时机和执行过程


 


接下来介绍如何使用工具分析运行的系统：


一.对象增长的速率


二.Young GC的触发频率


三.Young GC的耗时


四.每次Young GC后有多少对象存活下来


五.每次Young GC后有多少对象进入老年代


六.老年代对象增长的速率


七.Full GC的触发频率


八.Full GC的耗时


 


**(2\)功能强大的jstat**


如果平时要对运行中的系统，检查其JVM的整体运行情况。比较实用的工具之一就是jstat，它可以轻易让我们看到当前JVM内：Eden区、S区、老年代的内存情况，以及YGC和FGC的执行次数和耗时。


 


通过这些指标，我们就可以轻松分析出当前系统的运行情况。从而判断当前系统的内存使用压力和GC压力，以及内存分配是否合理。


 


**(3\)jstat \-gc PID**


首先使用jps命令在生产机器Linux上，找出Java进程的PID。接着就针对我们的Java进程执行：jstat \-gc PID，这样就可以看到这个Java进程的内存和GC情况了。运行这个命令后会看到如下指标的信息：



```
$ jstat -gc 1170
S0C    S1C    S0U    S1U    EC    EU    OC    OU    MC    MU    CCSC    CCSU    YGC    YGCT    FGC    FGCT    GCT    


S0C：这是From Survivor区的大小，C代表的是Capacity
S1C：这是To Survivor区的大小，C代表的是Capacity
S0U：这是From Survivor区当前使用的内存大小，U代表的是Used
S1U：这是To Survivor区当前使用的内存大小，U代表的是Used
EC：这是Eden区的大小，E代表的是Eden，C代表的是Capacity
EU：这是Eden区当前使用的内存大小，E代表的是Eden，U代表的是Used
OC：这是老年代的大小，O代表的是Old，C代表的是Capacity
OU：这是老年代当前使用的内存大小，O代表的是Old，U代表的是Used
MC：这是方法区(永久代、元数据区)的大小，M代表的是Metaspace，C代表的是Capacity
MU：这是方法区(永久代、元数据区)的当前使用的内存大小，M代表的是Metaspace，U代表的是Used
YGC：这是系统运行迄今为止的Young GC次数
YGCT：这是Young GC的耗时，T代表的是Time
FGC：这是系统运行迄今为止的Full GC次数
FGCT：这是Full GC的耗时，T代表的是Time
GCT：这是所有GC的总耗时，T代表的是Time
```

这些指标都是非常实用的JVM GC分析指标。


 


**(4\)其他的jstat命令**


除了上面的jstat \-gc命令是最常用的以外，jstat还有一些命令可以看到更多详细的信息，如下所示：



```
jstat -gccapacity PID：堆内存分析
jstat -gcnew PID：年轻代GC分析，这里的TT和MTT可以看到对象在年轻代存活的年龄和存活的最大年龄
jstat -gcnewcapacity PID：年轻代内存分析
jstat -gcold PID：老年代GC分析
jstat -gcoldcapacity PID：老年代内存分析
jstat -gcmetacapacity PID：元数据区内存分析
```

但最完整、最常用、最实用的还是jstat \-gc命令，jstat \-gc命令基本足够分析JVM的运行情况了。


 


**(5\)到底该如何使用jstat工具**


首先需要明确，分析线上的JVM进程，最想要知道的信息有哪些？最想要知道的信息会包括如下：


一.新生代对象增长的速率


二.Young GC的触发频率


三.Young GC的耗时


四.每次Young GC后有多少对象存活下来


五.每次Young GC后有多少对象进入老年代


六.老年代对象增长的速率


七.Full GC的触发频率


八.Full GC的耗时


 


只要知道了这些信息，我们就可以结合前面介绍的JVM GC优化的方法：合理分配内存，尽量让对象留在年轻代不进入老年代，避免频繁FGC。这就是对JVM最好的性能优化了。


 


**(6\)新生代对象增长的速率**


需要了解JVM的第一个信息就是：随着系统运行，每秒会在新生代的Eden区分配多少对象。


 


要获取该信息，只需要在Linux机器上运行命令：jstat \-gc PID 1000 10，该命令意思：每隔1秒更新最新jstat统计信息，一共执行10次jstat统计。通过这行命令，可以灵活地对线上机器，通过固定频率输出统计信息。从而观察出每隔一段时间，JVM中Eden区的对象占用变化。


 


比如执行这行命令后：第一秒显示出Eden区使用了200M内存，第二秒显示出Eden区使用了205M内存，第三秒显示出Eden区使用了209M内存，以此类推。此时我们就可以推断出来，这个系统大概每秒钟会新增5M左右的对象。


 


而且这里可以根据自己系统的情况灵活多变地使用，如果系统负载很低，则不一定每秒进行统计，可以每分或每10分来统计，以此查看系统每隔1分钟或者10分钟大概增长多少对象。


 


此外，系统一般有高峰和日常两种状态，比如系统高峰期用户很多，我们应该在系统高峰期去用上述命令看看高峰期的对象增长速率，然后再在非高峰的日常时间段内看看对象的增长速率，这样就可以了解清楚系统的高峰和日常时间段内的对象增长速率了。


 


**(7\)Young GC的触发频率和每次耗时**


接着需要了解JVM的第二个信息是：大概多久会触发一次YGC，以及每次YGC的耗时。其实多久触发一次YGC是很容易推测出来的，因为系统高峰和日常时的对象增长速率都知道了，根据对象增长速率和Eden区大小，就可以推测出：高峰期多久发生一次YGC，日常期多久发生一次YGC。


 


比如Eden区有800M内存：如果发现高峰期每秒新增5M对象，那么大概3分钟会触发一次YGC。如果发现日常期每秒新增0\.5M对象，那么大概半小时才触发一次YGC。


 


那么每次Young GC的平均耗时呢？jstat会展示迄今为止系统已经发生了多少次YGC以及这些YGC的总耗时。比如系统运行24小时后共发生260次YGC，总耗时为20s。那么平均每次YGC大概耗时几十毫秒，我们由此可以大概知道每次YGC时会导致系统停顿几十毫秒。


 


**(8\)每次Young GC后有多少对象进入老年代**


接着要了解JVM的第三个信息是：每次YGC后有多少存活对象，即有多少对象会进入老年代。


 


其实每次YGC过后有多少对象会存活下来，只能大致推测出来。假设已经推算出高峰期多久会发生一次YGC，比如3分钟会有一次YGC。那么此时就可以执行下述jstat命令：jstat \-gc PID 180000 10。这就相当于让JVM每隔三分钟执行一次统计，连续执行10次。观察每隔三分钟发生一次YGC时，Eden、Survivor、老年代的对象变化。


 


正常来说：Eden区肯定会在几乎放满后又变得很少对象，比如800M只使用几十M。Survivor区肯定会放入一些存活对象，老年代可能会增长一些对象占用。所以这时观察的关键，就是观察老年代的对象增长速率。


 


正常来说：老年代不太可能不停快速增长的，因为普通系统没那么多长期存活对象。如果每次YGC后，老年代对象都要增长几十M，则可能存活对象太多了。存活对象太多可能会导致放入S区后触发动态年龄判定规则进入老年代，存活对象太多也可能导致S区放不下，大部分存活对象需要进入老年代。


 


如果老年代每次在YGC过后就新增几百K或几M的对象，这个还算正常。但如果老年代对象快速过快增长，那一定是不正常的。


 


所以通过上述观察策略，就可以知道每次YGC后有多少对象是存活的，也就是Survivor区里增长的 \+ 老年代增长的对象，就是存活的对象。


 


通过jstat \-gc也可以知道老年代对象的增长速率，比如每隔3分钟一次YGC，每次会有50M对象进入老年代，于是老年代对象的增长速率就是每隔3分钟增长50M。


 


**(9\)Full GC的触发时机和耗时**


只要知道老年代对象的增长速率，那么Full GC的触发时机就很清晰了。比如老年代有800M，每3分钟新增50M，则每1小时就会触发一次FGC。


 


根据jstat输出的系统运行迄今为止的FGC次数以及总耗时，就能计算出每次FGC耗时。比如一共执行了10次FGC，共耗时30s，那么每次FGC大概耗费3s左右。


 


**2\.使用jmap和jhat了解线上系统的对象分布**


**(1\)jstat总结**


**(2\)使用jmap了解系统运行时的内存区域**


**(3\)使用jmap了解系统运行时的对象分布**


**(4\)使用jmap生成堆内存转储快照**


**(5\)使用jhat在浏览器中分析堆转出快照**


 


**(1\)jstat总结**


通过jstat可以非常轻松便捷的了解到线上系统的运行状况，如新生代对象和老年代对象增速、YGC和FGC触发频率以及耗时等，通过jstat可以完全了解线上系统的JVM运行情况，为优化做准备。


 


接下来介绍两个在工作中非常实用的工具：jmap和jhat。这两个工具可以观察线上JVM中的对象分布，能更加细致了解系统运行。即了解系统运行过程中，哪些对象占据了大部分，占据了多少内存空间。


 


**(2\)使用jmap了解系统运行时的内存区域**


其实如果只是了解JVM运行状况，然后进行GC优化，通常jstat就够用了。但有时会发现JVM新增对象速度很快，想知道什么对象占那么多内存。从而可以优化对象在代码中的创建时机，避免对象占用内存过大。


 


首先看一个命令：jmap \-heap PID。这个命令可以打印出一系列信息，这些信息大概就是：堆内存相关的一些参数设置、当前堆内存里各个区域的情况。比如：Eden区的总容量、已经使用的容量、剩余的空间容量，两个Survivor区的总容量、已经使用的容量、剩余的空间容量，老年代的总容量、已经使用的容量、剩余的容量。


 


但是这些信息其实jstat已经有了，所以一般不会用jmap去获取这些信息。毕竟jmap的这种信息还没jstat全面，比如jmap就没有GC相关的统计。


 


**(3\)使用jmap了解系统运行时的对象分布**


jmap中比较有用的一个命令是：jmap \-histo PID。这个命令会打印出类似下面的信息：按各对象占用内存空间大小降序排列，把占用内存最多的对象放在最上面。



```
num       #instances          #bytes      class name
----------------------------------------------------
1:        46608               1111232     java.lang.String
2:         6919                734516     java.lang.Class
3:         4787                536164     java.net.SocksSocketImpl
4:        15935                497100     java.util.concurrent.ConcurrentHashMap$Node
5:        28561                436016     java.lang.Object
```

所以想简单了解当前JVM中的对象内存占用情况，可用jmap \-histo命令。该命令可快速了解当前内存里到底哪个对象占用了大量的内存空间。


 


**(4\)使用jmap生成堆内存转储快照**


如果想查看对象占用内存的具体情况，那么可以使用jmap命令生成一个堆内存转储快照到文件里：


jmap \-dump:live,format\=b,file\=dump.hprof PID


 


这个命令会在当前目录下生成一个dump.hrpof文件，该文件是二进制的格式，不能直接打开看，这个命令会把这一时刻JVM堆内存里所有对象的快照放到文件里。


 


**(5\)使用jhat在浏览器中分析堆转出快照**


可以使用jhat去分析堆内存快照，jhat内置了web服务器，支持通过web界面来分析堆内存转储快照。


 


使用如下命令即可启动jhat服务器，可以指定HTTP的端口，默认的HTTP端口为7000。


jhat \-port 7000 dump.hprof


 


接着就可以在浏览器上访问当前这台机器的7000端口号，这样就可以通过图形化的方式去分析堆内存里的对象分布情况了。


 


**3\.如何分析JVM运行状况并合理优化**


**(1\)开发好系统后的预估性优化**


**(2\)系统压测时的JVM优化**


**(3\)对线上系统进行JVM监控**


 


**(1\)开发好系统后的预估性优化**


什么叫预估性优化？就是估算系统每秒多少请求、每个请求创建多少对象、占用多少内存。机器应选用什么配置、新生代应给多少内存、老年代应给多少内存。Young GC触发的频率、对象进入老年代的速率、Full GC触发的频率。


 


这些信息其实都是可以根据系统代码，大致合理地进行预估的。在预估完成后，就可以采用前面的优化思路，先设置初始的JVM参数。比如堆内存大小、新生代大小、Eden和Survivor的比例、老年代大小、大对象的阈值、大龄对象进入老年代的阈值等。


 


优化思路就是：尽量让每次YGC后的存活对象小于S区的50%，可以都留在新生代里。尽量别让对象进入老年代，减少FGC的频率、避免频繁FGC影响性能。


 


**(2\)系统压测时的JVM优化**


通常一个新系统开发完毕后，会经过一连串的测试，本地单元测试\-\>系统集成测试\-\>测试环境功能测试\-\>预发环境压力测试。总之要保证系统的功能正常，在一定压力下稳定性和并发能力也都正常，最后才会部署到生产环境运行。


 


这里非常关键的一个环节就是预发布环境的压力测试，通常该环节会使用一些压力测试工具模拟如1000个用户同时访问系统。模拟每秒500个请求压力，然后看系统能否支撑住每秒500请求的压力。同时看系统各接口响应延时是否在比如200ms内，即接口性能不能太慢。或者在数据库中模拟出百万级单表数据，然后看系统是否还能稳定运行。


 


具体如何进行系统压测，可以搜Java压力测试，会有很多开源工具。通过这些工具可以轻松模拟出N个用户同时访问你系统的场景，同时还能生成压力测试报告：每秒可支撑多少请求、接口的响应延时等。


 


在该环节，压测工具会对系统发起持续不断的请求。这些请求通常会持续很长时间，如几小时甚至几天。所以可以在该环节，对测试机器运行的系统，采用jstat工具来进行分析。在模拟真实环境的压力测试下，通过jstat命令获取JVM的整体运行状态。


 


前面已具体介绍了如何使用jstat来分析以下JVM的关键运行指标：新生代对象增长的速率、YGC的触发频率、YGC的耗时、每次YGC后有多少对象存活下来、每次YGC后有多少对象进入老年代、老年代对象增长的速率、FGC的触发频率、FGC的耗时。


 


然后根据压测环境中的JVM运行状况：如果发现对象过快进入老年代，可能是因为年轻代太小导致频繁YGC；也可能因为很多对象存活，而S区太小，导致很多对象频繁进入老年代；此时就需要采用前面介绍的优化思路：合理调整新生代、老年代、Eden、Survivor各个区域的内存大小，保证对象尽量留在年轻代、不要过快进入老年代中。


 


有很多人会胡乱搜索网上JVM优化博客，看人家怎么优化就怎么优化。比如很多博客说新生代和老年代的占比一般是3:8，这其实是很片面的。每个系统都是不一样，特点不同、复杂度不同。真正的优化，一定是在实际观察我们的系统后，合理调整内存分布。真正的优化，并没有固定的JVM优化模板。当优化好压测环境的JVM参数，观察YGC和FGC频率都很低，就可上线。


 


**(3\)对线上系统进行JVM监控**


当系统上线后，就需要对线上系统的JVM进行监控，这个监控通常来说有两种办法。


 


第一种方法：


每天在高峰期和低峰期用jstat、jmap、jhat看线上JVM运行是否正常。有没有频繁FGC问题，如果有就优化，没有就每天定时或每周定时去看。


 


第二种方法：


部署专门的监控系统，常见的有Zabbix、OpenFalcon、Ganglia等。可以将线上系统的JVM统计项发送到这些监控系统里去，这样就可以在这些监控系统可视化界面里，看到需要的所有指标：包括各内存区域的对象占用变化曲线、可直接看到Eden区的对象增速、YGC发生的频率以及耗时、老年代的对象增速、FGC的频率和耗时。而且这些工具通常还允许设置监控告警，比如10分钟之内发生5次以上FGC，就需要发送告警给我们。


 


**4\.使用jstat分析模拟的BI系统JVM运行情况**


**(1\)服务于百万级商家的BI系统是什么**


**(2\)刚开始上线BI系统时的部署架构**


**(3\)实时自动刷新报表 \+ 大数据量报表**


**(4\)没什么大影响的频繁Young GC**


**(5\)模拟程序的JVM参数设置**


**(6\)模拟程序**


**(7\)通过jstat观察程序的运行状态**


 


**(1\)服务于百万级商家的BI系统是什么**


作为一个电商平台，可能会有数十万到百万的商家在平台上做生意。电商平台每天会产生大量数据，需要基于这些数据为商家提供数据报表。比如：每个商家每天有多少访客、有多少交易、付费转化率是多少。


 


BI系统其实就是把商家日常经营的数据收集起来进行分析，然后提供各种数据报表给商家的一套系统。


 


这样的一个BI系统，其运行逻辑如下：首先电商平台会提供一个业务平台给商家进行日常使用交互，该业务平台会采集到商家的很多日常经营数据。根据这些日常经营数据，通过Hadoop、Spark等技术计算各种数据报表，这些数据报表会被放入存储到MySQL、Elastcisearch、HBase中。最后基于MySQL、HBase、ES中存储的数据报表，开发出一个BI系统。通过这个BI系统就能把各种存储好的数据展示给商家进行筛选和分析。


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/70c58e2d009f46a19dd30f16694c23d9~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=3HXgV3iUBe%2FUHCholRpXeqUXyYE%3D)
**(2\)刚开始上线BI系统时的部署架构**


刚开始系统上线时，这个BI系统使用的商家是不多的，比如几千个商家，所以刚开始系统部署得非常简单，就是用几台机器来部署上述的BI系统，机器都是普通的4核8G配置。在这个配置下，会给堆内存新生代分配1\.5G内存，Eden区大概1G左右，如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/55207782438c4f4c8db7161aee824f5d~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=5%2BJGw42A8IqGq3uaqk3ExzAMVG4%3D)
**(3\)实时自动刷新报表 \+ 大数据量报表**


刚开始在少数商家的情况下，这个系统没多大问题，运行得非常良好。但使用系统的商家开始越来越多，商家数量级达到几万时就有问题了。


 


首先说明一下此类BI系统的特点；就是在BI系统中有一种实时数据报表，它支持前端页面有一个JS脚本，该JS脚本每隔几秒就会自动发送请求到后台刷新一下数据。如下图示：


![](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/fdb65af512e6406ba9401d9a9474073b~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=ThHhfsrTfoJseUmHworjeMXZkn0%3D)
虽然只有几万商家使用该系统，但可能同时打开实时报表的商家有几千。每个商家打开报表后，前端都会每隔几秒发送请求到后台加载最新数据。于是出现部署BI系统的每台机器每秒请求达几百个，假设每秒500请求。


 


然后每个请求会加载出一张报表所需要的大量数据，因为BI系统可能还要针对这些数据在内存中进行计算加工，才能返回。根据测算，每个请求大概会从MySQL中加载出100K的数据进行计算。因此每秒500个请求，就要加载50M数据到内存中进行计算。


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/063c7328b02340c283a9352beeafbcaa~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=DDDKXyLxpMyfGkiEGjyb3jpxFh0%3D)
**(4\)没什么大影响的频繁Young GC**


在上述系统运行模型下，由于每秒会加载50M的数据到Eden区中，所以只要200s就会填满Eden区，然后触发YGC对新生代进行垃圾回收。当然1G左右的Eden进行YGC速度是比较快的，可能几十ms就搞定了。所以每200s频繁执行一次YGC其实对系统性能影响并不大，而且上述BI系统场景下，基本上每次YGC后存活对象可能会有几十M。


 


因此可能会看到如下场景：BI系统每运行几分钟就会卡顿10ms，但对用户和系统性能几乎没影响。


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/de11d6527e6e404483623d8d0c933051~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=2c%2F1zw7Te2KgIymjxzJXLVu%2B5%2FA%3D)
**(5\)模拟程序的JVM参数设置**


接下来用一段程序模拟出上述BI系统那种频繁YGC的场景，此时JVM参数如下所示：



```
 -XX:NewSize=104857600 -XX:MaxNewSize=104857600 
 -XX:InitialHeapSize=209715200 -XX:MaxHeapSize=209715200 
 -XX:SurvivorRatio=8  -XX:MaxTenuringThreshold=15 
 -XX:PretenureSizeThreshold=3145728 
 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC 
 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log
```

上述把堆内存设置为了200M，把年轻代设置为100M。然后Eden区是80M，每块Survivor区是10M，老年代也是100M。


 


**(6\)模拟程序**


下面是我们的模拟程序：



```
public class Demo {
    public static void main(String[] args) throws Exception {
        Thread.sleep(30000);
        while(true) {
            loadData();
        }
    }
    
    private static void loadData() throws Exception {
        byte[] data = null;
        for(int i = 0; i < 50; i++) {
            data = new byte[100 * 1024];
        }
        data = null;
        Thread.sleep(1000);
    }
}
```

上述这段模拟程序的一些说明：


一.第一行代码为Thread.sleep(30000\)，为什么刚开始要先休眠30s？因为程序刚启动休眠30s可方便我们找到这个程序的PID，即进程ID，然后再执行jstat命令来观察程序运行时的JVM状态。


 


二.接着loadData()方法内的代码会循环50次，模拟每秒50个请求。然后每次请求会分配一个100K的数组，模拟每次请求从数据存储中加载出100K的数据。接着会休眠1秒，模拟这一切都是发生在1秒内的。其实这些对象都是短生存周期的对象，方法运行结束这些对象都是垃圾。


 


三.然后在main()方法里有一个while(true)循环，模拟系统按照每秒50个请求，每个请求加载100K数据的方式不停运行。除非我们手动终止程序，否则永不停止。


 


**(7\)通过jstat观察程序的运行状态**


**一.接着我们使用预定的JVM参数启动程序**


此时程序会先进入一个30秒的休眠状态，于是尽快执行jps命令，查看启动程序的进程ID。如下所示：



```
$ jps
1169 Launcher
1170 Demo
1171 Jps
517 
```

**二.此时会发现我们运行的Demo这个程序的JVM进程ID是1170**


然后尽快执行下述jstat命令：



```
$ jstat -gc 1170 1000 1000
```

这行命令的意思是：针对1170进程统计JVM运行状态，每隔1秒打印一次统计信息，连续打印1000次。


 


然后执行jstat开始统计，每隔一秒都会打印一行新的统计信息，过了几十秒后可看到如下所示的统计信息：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/d2658d67711145ec9335bca0e4963d05~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=tMTuHDr%2FRcKXDZID1B7vBliEeKI%3D)
**三.接着先看如下图示的一段EU信息**


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/33684ffe3eaf4add94a41cd0d92ab9c9~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=NdCuAg5zHznV1393ZfscjH1Zod8%3D)
这个EU就是Eden区被使用的容量，可发现刚开始是5M左右的使用量。接着程序开始运行，每秒都会有对象增长：从5M到10M，接着15M，20M，25M，每秒都会新增5M左右的对象。这个跟上面的代码是完全吻合的，代码也是每秒会增加5M左右的对象。


 


**四.然后当Eden区使用量达到80M左右时，再要分配5M对象就失败了**


此时就会触发一次Young GC，如下图示：


![](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/1c2712b1a6bc4af1a19a3786d521d915~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=b8r3QVOoxWb1A6pxtBtdvkLhcZM%3D)
上面红圈的内容：Eden区的使用量从将近80M降低为3M多，这是因为一次YGC回收掉了大部分对象。


 


**五.所以针对这个模拟代码，可以清晰的从jstat中看出如下信息：**


对象增速大致为每秒5M左右，大致每十几秒会触发一次YGC。下图可以看到，YGC的触发频率，以及每次YGC的耗时。


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/f63ac70b2d4e420da33853b9a2dab26e~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=X%2FC4sHm5U7%2BmfoO21Ln8D54JEAc%3D)
上图清晰告诉我们：一次YGC回收70多M对象，大概花费1毫秒。所以YGC其实是很快的，即使回收800M的对象，也就10毫秒左右。所以如果是线上系统，Eden区800M的话，每秒新增对象50M。十多秒一次YGC耗时10毫秒左右，系统卡顿10毫秒几乎没什么大影响。


 


在这个模拟代码中：80M的Eden区，每秒新增对象5M。大概十多秒触发一次YGC，每次YGC耗时在1毫秒左右。那么YGC回收1G大小的Eden区，耗时大概会在15毫秒左右，毫秒级别。


 


**六.那么每次YGC过后会存活多少对象**


上图S1U就是Survivor中被使用的内存，S1U之前一直是0，在一次YGC过后变成了633K。所以一次YGC后存活了633K的对象而已，可轻松放入10M的Survivor。


 


而且注意上图的OU，就是老年代被使用的内存量，在YGC前后都是0。这说明这个系统运行良好，YGC不会导致对象进入老年代。所以这个系统就几乎不需要什么优化了，因为老年代对象增速几乎为0，FGC发生频率趋向于0，对系统无影响。


 


因此通过这个模拟程序的运行，我们可以使用jstat分析出以下信息的：新生代对象增长的速率、YGC的触发频率、YGC的耗时、每次YGC后有多少对象是存活的、每次YGC后有多少对象进入了老年代、老年代对象增长的速率、FGC的触发频率、FGC的耗时。


 


**5\.使用jstat分析模拟的计算系统JVM运行情况**


**(1\)一个日处理上亿数据的计算系统**


**(2\)这个系统多久会塞满新生代**


**(3\)触发Young GC时会有多少对象进入老年代**


**(4\)系统运行多久老年代就会被填满**


**(5\)这个系统运行多久老年代会触发1次Full GC**


**(6\)该案例应该如何进行JVM优化**


**(7\)模拟程序用的JVM参数**


**(8\)模拟程序**


**(9\)基于jstat分析程序运行的状态**


**(10\)对JVM性能进行优化**


 


**(1\)一个日处理上亿数据的计算系统**


当时团队里自研的一个数据计算系统，日处理数据量在上亿的规模。这个系统会不停的从MySQL数据库以及其他数据源里提取大量的数据，然后加载到自己的JVM内存里来进行计算处理，如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/936ee109d4ef4fcb8b55c91d2d1c75de~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=hIgcPLnx%2FQp9Wuxka5MB5QmN7Lc%3D)
这个数据计算系统会不停的通过SQL语句和其他方式，从各种数据存储中提取数据到内存中来进行计算，大致当时的生产负载是每分钟需要执行500次数据提取和计算的任务。


 


由于这是一套分布式运行的系统，所以生产环境部署了多台机器。每台机器大概每分钟负责执行100次数据提取和计算的任务(15个线程)，每次会提取大概1万条数据到内存计算，平均每次计算大概耗费10秒。然后每台机器4核8G，新生代和老年代分别是1\.5G和1\.5G的内存空间。如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/26539730f3dc40f3bc8af7be1aef2c48~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=3CdazsOLpD0p6lDPULG9kHsedxE%3D)
**(2\)这个系统多久会塞满新生代**


现在明确了一些核心数据，那么该系统多久会塞满新生代的内存空间。既然每台机器上部署的该系统实例，每分钟会执行100次数据计算任务。每次1万条数据需要计算10秒时间，该台机器大概开启15个线程去执行。


 


那么先来看看每次1万条数据大概会占用多大的内存空间：这里每条数据都是比较大的，每条数据大概包含了20个字段，可认为平均每条数据的大小在1K左右，那么每次计算任务提取的1万条数据就对应10M大小。


 


所以如果新生代按照8:1:1的比例来分配Eden和两块Survivor的区域，按照新生代和老年代分别是1\.5G和1\.5G的内存空间可知，Eden区就是1\.2G，每块Survivor区在100M左右。如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/3556534c378a41f88c369119fea5f780~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=P4Lnd%2FOXcFTEhdKaPFdcrm8Bm%2FU%3D)
由于每次执行一个计算任务，就要提取1万条数据到内存，每条数据1K。所以每次执行一个计算任务，JVM会在Eden区里分配10M的对象。那么由于一分钟需要执行大概100次计算任务，所以新生代里的Eden区，基本上1分钟左右就会被迅速填满。


 


**(3\)触发YGC时会有多少对象进入老年代**


假设新生代的Eden区在1分钟后都塞满对象了，在继续执行计算任务时，必然会导致需要进行YGC回收部分垃圾对象。


 


**一.在执行YGC前会先进行检查**


首先会看老年代的可用内存空间是否大于新生代全部对象。此时老年代是空的，大概有1\.5G的可用内存空间，而新生代的Eden区大概有1\.2G对象。


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/56db0733ae984aeb99d2688babc49edc~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=OPpp%2FoGGMrXTHs70SpFjjALpUKQ%3D)
于是会发现老年代的可用内存空间有1\.5G，新生代的对象总共有1\.2G。即使一次YGC过后，即时全部对象都存活，老年代也能放的下的，所以此时会直接执行YGC。


 


**二.执行YGC后，Eden区里有多少对象是存活的无法被垃圾回收的**


由于新生代的Eden区在1分钟就塞满对象需要YGC了，而1分钟内会执行100次任务，每个计算任务处理1万条数据需要10秒钟。


 


假设执行YGC时，有80个计算任务都执行结束了，但还有20个计算任务共计200M的数据还在计算中。那么此时就有200M的对象是存活的，不能被垃圾回收掉。所以总共有1G对象可以进行垃圾回收，200M对象存活无法被垃圾回收。如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/58072665a6e54c7f9f497f517cd31093~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=n%2F74eAhttzy7VmFK%2B8IQ1t3aaoc%3D)
**三.此时执行一次YGC会回收1G对象，然后出现200M的存活对象**


这200M的存活对象并不能放入S区，因为一块S区就100M大小，此时老年代会通过空间担保机制，让这200M对象直接进入老年代中。于是需要占用老年代里的200M内存空间，然后对Eden区进行清空。


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/468f5dc1af444a05adee675aa7175c9a~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=c6ImRw4tt%2BQnUYsWZM%2Bnz32AO9I%3D)
**(4\)系统运行多久老年代就会被填满**


按照上述计算，每分钟都是一个轮回，大概算下来是每分钟都会把新生代的Eden区填满。然后触发一次YGC，接着大概会有200M左右的数据进入老年代。


 


假设2分钟过去了，老年代已有400M内存被占用，只有1\.1G内存可用，此时老年代的可用内存空间已经少于新生代的内存大小了。所以如果第3分钟运行完毕，又要进行YGC，会做什么检查呢？如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/36ad0556376e410c918440270a5fafd1~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=rxWuV%2BGafcwJMfkrpo%2FJQMWhu1c%3D)
**一.首先会检查老年代可用空间是否大于新生代全部对象**


此时老年代的可用空间是1\.1G，新生代对象的大小有1\.2G，如果这次YGC过后新生代对象全部存活，那么老年代是放不下的。


 


**二.接着就得检查HandlePromotionFailure参数是否打开**


如果"\-XX:\-HandlePromotionFailure"参数被打开了，一般都会打开。此时会进入下一个检查：老年代可用空间是否大于历次YGC过后进入老年代的对象的平均大小。


 


前面已计算出大概每分钟会执行一次YGC，每次200M对象进入老年代。此时老年代可用1\.1G，大于每次YGC进入老年代的对象平均大小200M。所以可推测本次YGC后大概率还是有200M对象进入老年代，1\.1G足够。因此这时就可以放心执行一次YGC，然后又有200M对象进入老年代。


 


**三.转折点大概在运行了7分钟后**


执行了7次YGC后，大概1\.4G对象进入老年代。老年代剩余空间不到100M，几乎满了。如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/5eceaf4442fd45098b2a22251814e497~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=lWL%2FkGavjcpC0IMAGF49zpSd%2FSA%3D)
**(5\)这个系统运行多久老年代会触发1次Full GC**


大概在第8分钟运行结束时，新生代又满了。执行YGC之前进行检查，发现老年代此时只有100M的可用内存空间。小于历次YGC后进入老年代的200M对象，于是就会直接触发一次FGC，FGC会把老年代的垃圾对象都给回收掉。


 


假设此时老年代被占据的1\.4G空间里，全部都是可以回收的对象，那么此时就会一次性把这些对象都给回收掉。如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/fb7fbc1355ab4da5b95be1f375565e06~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=Op4CmthhRhZEcQUlDApDD9tO5tg%3D)
然后执行完FGC后，还会继续执行YGC，又有200M对象进入老年代。之前的FGC就是为这次新生代YGC后要进入老年代的对象准备的。如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/175be653a8f04fe4a13aa7932226a533~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=q%2Fr7Gnp6icy5PXbgHDbnBM7LP14%3D)
所以根据这个运行模型，该系统平均八分钟会发生一次FGC，这个频率就很高了，而每次FGC速度都是很慢的、性能很差。


 


**(6\)该案例应该如何进行JVM优化**


通过上述这个案例，可以清楚看到：新生代和老年代应该如何配合使用，什么情况下会触发Young GC和Full GC，什么情况下会导致频繁的Young GC和Full GC。


 


如果要对这个系统进行优化：由于该系统是数据计算系统，每次YGC时都会有一批数据没计算完毕。所以按现有的内存模型，最大问题就是每次YGC后S区放不下存活对象。


 


所以可以对生产系统进行调整：增加新生代的内存比例，3G堆内存的2G给新生代，1G给老年代。这样S区大概就是200M，每次刚好能放得下YGC过后存活的对象。如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/044372412e82486cb3d3569e4a6d1828~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=PSbS4lswiAG3bVfU1gjL2bu%2BiVg%3D)
只要每次YGC过后200M存活对象可以放进Survivor区域，那么等下次YGC时，这个S区的对象对应的计算任务早就结束可回收了。比如此时Eden区里1\.6G空间被占满了，然后S1区里有200M上一轮YGC后存活的对象。如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/517b01bfb0c84080b87b3a30be7f0611~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=dGbqRW6fbhTmOyVkmDN0lbhZVos%3D)
此时执行YGC后：就会把Eden区里1\.6G对象回收掉，S1区里的200M对象也会回收掉，然后Eden区里剩余的200M存活对象会放入S2区。如下图示：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/d7c2ba6af5d241ce985994b4fdc13f70~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=%2Bz8j%2F586H5QEhXr6Bdi6sP9DEDk%3D)
以此类推，基本就很少有对象进入老年代了，老年代的对象也不会太多。这样就把生产系统老年代FGC的频率从几分钟一次降低到几小时一次，大幅度提升了系统的性能，避免了频繁FGC对系统运行的影响。


 


前面说过一个动态年龄判定升入老年代的规则：如果S区中的同龄对象大小超过S区内存的一半，就要直接升入老年代。


 


所以这里的优化仅仅是做一个示例说明而已，实际S区200M还是不够。但表达的是要增加S区大小，让YGC后的对象进入S区，避免进入老年代。


 


实际上为了避免触发动态年龄判定规则，把S区中的对象直接升入老年代，如果新生代内存有限，那么可以调整"\-XX:SurvivorRatio\=8"参数。比如降低Eden区的比例(默认80%)，给两块S区更多的内存空间。让每次YGC后的对象进入S区，避免触发动态年龄规则把它们升入老年代。


 


**(7\)模拟程序用的JVM参数**


把堆内存设置为200M，把年轻代设置为100M。然后Eden区80M，每块Survivor区10M，老年代100M。接着通过\-XX:PretenureSizeThreshold，把大对象阈值修改为20M，避免模拟程序里分配的大对象直接进入老年代。



```
 -XX:NewSize=104857600 -XX:MaxNewSize=104857600 
 -XX:InitialHeapSize=209715200 -XX:MaxHeapSize=209715200 
 -XX:SurvivorRatio=8  -XX:MaxTenuringThreshold=15 
 -XX:PretenureSizeThreshold=20971520 
 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC 
 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log
```

**(8\)模拟程序**



```
public class Demo {
    public static void main(String[] args) throws Exception {
        Thread.sleep(30000);
        while(true) {
            loadData();
        }
    }
    
    private static void loadData() throws Exception {
        byte[] data = null;
        for(int i = 0; i < 4; i++) {
            data = new byte[10 * 1024 * 1024];
        }
        data = null;
        byte[] data1 = new byte[10 * 1024 * 1024];
        byte[] data2 = new byte[10 * 1024 * 1024];
        byte[] data3 = new byte[10 * 1024 * 1024];
        data3 = new byte[10 * 1024 * 1024];
        Thread.sleep(1000);
    }
}
```

上面模拟程序的含义：


一.每秒钟都会执行一次loadData()方法。


二.loadData()首先会分配4个10M的数组，但都会马上成为垃圾。接着会有两个10M的数组会被变量data1和data2引用，必须存活，此时Eden区已经占用了六七十M的空间了。


三.接着是data3变量会依次指向两个10M的数组，1s内会触发YGC。


 


**(9\)基于jstat分析程序运行的状态**


接下来启动程序后马上采用jstat监控其运行状态：



```
$ jps
517 
652 RemoteMavenServer
1213 Launcher
1214 Demo
1215 Jps
$ jstat -gc 1214 1000 1000
```

可以看到如下的信息：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/a2a69199248e4fcbb7aad6ce8af616fd~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=9zWsKTYiuk3x4aJpRfDqlV7puh4%3D)
下面分析这个JVM的运行状态。


 


一.首先看如下图示


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/c2e9c711ea3142598d4d9c6eeb562a45~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=cZowB%2FodhlGdqp%2Bv%2FsuUKCLrxRo%3D)
在最后一行可清晰看到，程序运行后，突然在一秒内就发生了一次YGC。因为按照上述的模拟代码，它一定会在一秒内触发一次YGC的。


 


YGC后，可以发现S1U中有536K的存活对象，这应该就是那些未知对象。然后明显看到在OU中多出30M左右的对象。因此可以确定，在这次YGC时，有30M的对象存活了。因为此时YGC后的存活对象在Survivor区放不下，所以直接进入老年代。


 


二.接着看如下图示


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/1226c6d0e3b94bc89b36a30042eed596~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=a1WEG5pZfSb60sV0q2uPJdA0o28%3D)
上图中红圈部分：很明显每秒会发生一次YGC，每次会导致10M\~30M的对象进入老年代。因每次YGC都存活这么多对象，但S区放不下，所以才直接进入老年代。


 


此时可以看到老年代的对象占用从30M一路升到60M。然后突然在60M后的下一秒，明显发生了一次FGC，对老年代进行回收，因为此时老年代重新变成30M了。


 


为什么会这样？因为老年代总共就100M左右，已占60M了。此时如果发生一次YGC，有30M存活对象要放入老年代，是明显不够的。此时必须要进行FGC，回收掉之前60M对象，然后再放入30M存活对象。


 


所以可以看到：按照模拟代码，几乎是每秒新增80M对象，每秒触发1次YGC。每次YGC后存活20M\~30M的对象，老年代每秒新增20M\~30M的对象，于是几乎每三秒触发一次老年代FGC。


 


这和上面的实时分析引擎的场景很类似：YGC太频繁，而且每次GC后存活对象太多，频繁进入老年代，从而频繁触发老年代的GC。


 


三.YGC和FGC的耗时如下图示


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/69f47f2fffcf4e559d4571c42c433f70~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=TJaEBLo8uuOr%2FQfjyj0uOpmVIVM%3D)
可以发现：28次YGC，结果耗费了120毫秒，平均下来一次YGC要5毫秒左右。但是14次FGC才耗费24毫秒，平均下来一次FGC才耗费一两毫秒。这是为什么呢，为什么YGC比FGC还久？


 


因为按照上述模拟程序：


每次FGC都是由YGC触发的，所以是可能出现YGC比FGC还慢的情形的。因为YGC后存活对象太多要放入老年代，老年代内存不够才触发FGC。所以必须等FGC执行完毕，YGC才能把存活对象放入老年代，才算结束，从而导致YGC比FGC还慢。


 


**(10\)对JVM性能进行优化**


接着按照前面介绍的思路对JVM进行优化，这次模拟程序最大的问题就是每次YGC过后存活对象太多，导致频繁进入老年代，频繁触发FGC。所以只需要调大新生代的内存空间，增加Survivor区的内存即可。调整为如下JVM参数：



```
 -XX:NewSize=209715200 -XX:MaxNewSize=209715200 
 -XX:InitialHeapSize=314572800 -XX:MaxHeapSize=314572800 
 -XX:SurvivorRatio=2  -XX:MaxTenuringThreshold=15 
 -XX:PretenureSizeThreshold=20971520 
 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC 
 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log
```

把堆大小调大为300M，新生代给200M。同时\-XX:SurvivorRatio\=2表明，Eden:Survivor:Survivor的比例为2:1:1。所以Eden区是100M，每个Survivor区是50M，老年代也是100M。


 


接着用这个JVM参数运行程序，用jstat来监控其运行状态如下：


![](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/b8f91911ee064d02b3013d0208ab9d33~tplv-obj.image?lk3s=ef143cfe&traceid=202501022328125506C06DA7A3BAA46FFC&x-expires=2147483647&x-signature=Uamtyjzdfx577OauCvuVhBLllGc%3D)
从上图可见：每秒的YGC后，都会有20M左右的存活对象进入Survivor区。但由于每个S区都是50M，因此可以轻松容纳且不会触发动态年龄判定。


 


因此可以清晰看到每秒触发YGC后，几乎就没有对象会进入老年代。最终只有493K的对象进入了老年代；同时只有YGC，没有FGC。而且12次YGC才55毫秒，没有FGC干扰后，YGC的性能极高。


 


这样这个模拟程序就被成功优化了，同样的程序只调整了内存比例，就能大幅提升JVM性能，几乎消灭FGC。


 


**6\.问题汇总**


**问题一：**


系统如何尽量减少Full GC？


 


**(1\)什么情况下发生FGC**


一.YGC前：


情形一：新生代对象大小 \> 老年代可用内存 \&\& 没开通内存分配担保。


情形二：新生代对象大小 \> 老年代可用内存 \&\& 开通内存分配担保 \&\& 历次新生代GC进入老年代大小平均值 \> 老年代可用内存大小。


 


二.YGC后：


老年代放不下YGC后存活的对象。


 


**(2\)如何避免FGC**


可以让每次YGC后，存活的对象尽量能放在S区，不要进入老年代。


一.调大Survivor区的大小


二.如果系统运算时间比较长导致对象年龄比较大，那么可以调大\-XX:MaxTenuringThreshold参数，使得对象年龄大一些再进入老年代，这样也可以减少进入老年代的对象。


 


**问题二：**


在Tomcat中启动一个war，这是启动了一个JVM进程吗，还是多个？


 


答：Tomcat自己就是一个JVM进程，开发的war包不过就是一些类而已。Tomcat会将war包加载到自己的JVM进程里去执行类的代码逻辑。


 


**问题三：**


生产服务器的堆大小2G，其他都是默认。jmap看新生代Eden区和Survivor区的比例为8，按这比例，Survivor区应该是新生代的十分之一。但Eden区有680M，Survivor区只有10M，且每次YGC后Eden区和Survivor区的总大小都在变化，为什么呢？


 


答：因为设置了允许堆大小动态调整了，这个需要禁止掉的，就是让\-Xmx和\-Xms需要一样的值。


 


 本博客参考[wgetcloud全球加速器服务](https://wgetcloud6.org)。转载请注明出处！
