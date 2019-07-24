# Oracle Rac基础知识点

#### Oracle Rac的特殊进程：**

##### ①全局缓存服务进程（LMS）：

管理全局数据块的访问，用于不同实例的缓冲区之间的传递block的映射，保证数据库的一致性。

##### ②锁监控进程（LMON）：

全局队列服务监控器，监视全局队列和集群间的资源交互，执行全局队列的恢复操作。
LMON进程会定期通信，使用节点间的网络心跳和控制文件的磁盘心跳来检查节点的健康状态，当某个节点出现故障时负责集群的重构。

##### ③锁管理器守护进程（LMD）:

全局队列服务守护进程，管理全局队列和集群间的资源交互，执行全局队列的恢复操作，并且负责死锁的检查和监控转换超时。

##### ④锁进程（LCD）：

管理非缓存融合，锁请求是本地的资源请求。管理共享资源实例的资源请求和跨实例调用操作。



#### Oracle 11g Rac环境与单实例的区别是：

①每一个Rac的节点的实例都有自己的sga、后台进程、alert，trace日志。
②数据文件：控制文件必须存放在共享存储中，有所有的实例共享。
③联机重做日志文件：在同一时间点，只有一个实例可以写入，但是其他实例可以在恢复和存档期间读取。
④归档日志：属于该实例，其他日志在介质恢复期间访问此归档日志。

#### OracleRac的集群组件：由后台进程、服务、OCR、表决磁盘等构成。

①集群就绪服务CRSd：为Oracle集群提供高可用框架，管理集群的资源的状态，CRS管理Oracle集群注册表（OCR）。CRS需要公共IP、私有IP和虚拟IP才能运行。
					在开始安装CRS之前，公共和私有接口应当处于运行状态，而且能够相互执行ping操作。
②集群同步服务OCSSd：它提供对节点成员关系的访问，包含集群组服务和集群锁定。OCSSd的故障会导致计算机重新启动，以避免“脑裂”。
					 OCSSd以两种心跳机制来提供这些服务：网络心跳和磁盘心跳。
③evmd进程：发布crs产生的时间，控制crs，css等进程通信
④ONS：Onsctl这个命令是用来管理ONS(Oracle Notification Service)是OracleClustser实现FAN Event Push模型的基础。
⑤集群件启动进程OHASd（高可用性服务守护进程）：它来启动所有其他Oracle集群软件守护进程。
                    OHASd对每个集群件守护进程都有一个资源，这些资源存储在Oracle本地注册表中。
					有四种主要代理：oraagent、orarootagent、cssdagent和cssdmonitor。这些代理对各自的集群件守护进程执行启动、停止、检查和清理操作。
⑥集群注册表OCR:用来存储Orcle集群件中所定义的全部集群资源的元数据、配置和状态信息。vote：存放节点成员状态信息。
⑦Oracle本地注册表——OLR：OLR是11gR2引入的，它只存储与本地节点有关的信息，OLR存储OHASd通常需要的信息（集群的版本配置等）。
⑧表决磁盘：必须为共享磁盘，它保存了节点之间的心跳信息。如果有任何节点不能ping表决磁盘，那么集群立即确认通信故障，将该节点从集群组中逐出。
⑨虚拟IP：在公共IP子网中配置这个IP，监听器将会被配置为监听VIP，而不是公共IP。
⑩单一客户端访问名称SCAN：集群可以有1～3个SCAN监听器和SCAN VIP，客户端仅以SCAN VIP进行连接，因此集群节点发生变化时并不需要改变客户端的连接配置。
每个数据库实例都是用数据库初始化参数REMOTE_LISTENER将自己注册到本地监听器和SCAN监听器。

OracleRac的集群组件：由后台进程、服务、OCR、表决磁盘等构成。
CRS-4638: Oracle High Availability Services is online
CRS-4537: Cluster Ready Services is online
CRS-4529: Cluster Synchronization Services is online
CRS-4533: Event Manager is online

#### oracle rac安装步骤：

首先关闭selinux，防火墙，时间同步服务，内核参数，limit
大页内存，共享存储，dns解析，修改hosts
创建目录用户，传软件，解压，
图形化界面安装 grid asm什么磁盘组,oracle 软件，dbca创建数据库
软件安装完成后打上补丁，调整参数

集群资源管理操作：
如集群资源状态检查（如实例，监听，服务，ocr，votedis）
集群集群资源配置（asm状态，lister状态配置，scan，和vip状态检查配置集群时钟同步，实例启停，同步）

ocr磁盘组调整

# ocrconfig -add +NEW_OCRVOTDG     

# ocrconfig -delete +OCRVOTDG

votedisk 磁盘组调整
$ crsctl replace votedisk +NEW_OCRVOTDG
$ crsctl query css votedisk

问题点，怎么找，怎么定位，然后解决，心跳断的可能性。

select 'oradebug setospid '||spid||';'||chr(10)||' oradebug close_trace;'||chr(10)||'oradebug flush;'||chr(10) from v$process where spid is not null;
lsof /u01 | grep del

集群是oracle数据库的一套高可用解决方案，最大特点是使用了共享存储架构，节点之间采用高速互联机制，为应用提供负载均衡之类的功能。这也有一定的弊端，过于依赖存储性能，节点数太多也会增加节点间通信的压力。一套rac集群有它自己相应的集群服务和集群资源。如后台进程，集群服务，ocr，表决盘，磁盘组资源，vip资源，监听资源，GNS等。

RAC的安装部署，我来说一下我的工作中需要注意的点。
1.关闭操作系统透明大业内存配置。
grub.conf transparent_hugepage=never谨慎操作。
开启表准大页内存（根据sga的大小计算大业内存）

2：配置时间同步服务时注意时区
3：根据部署指南设置磁盘调度算法

4：multipath 多路径软件，对比wwid来写入mutipath.conf
udev 规则呢常用的有12的规则，是针对mutipath的多路径软件常用的，99-oracle我个人习惯是写iscsi使用uuid的绑定规则用的，60-raw是我一般测试使用磁盘分区用的。
也可以使用asmlib管理磁盘

5：运行安装前检查脚本，是一个非常好的查漏补缺的过程

6：安装软件安装好后打上补丁，修改参数，和配置

维护操作：
检查集群资源状态，起停集群，启停单个集群资源呀，还有将一些资源加入集群，删除某个集群资源。比如说添加替换ocr和votedisk的检查和调整。

基础的故障定位：比如说集群异常宕机时候，就需要从gird $ORACLE_HOME/log/node下来查看集群日志，排查宕机原因。其中包括crs，css，集群守护进程等待日志。
oracle 12c日志位置发生变化，在$ORACLE_BASE/diag下