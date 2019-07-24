# Oracle体系结构

**1）oracle 体系结构：**
内存结构（sga<6大池子>+pga）：
share pool(库高速缓存：存放数据字典，共享sql区，数据字典高速缓存：以行记录的模式）SQL、PL/SQL程序进行语法分析、编译、执行的内存区域。
db buffer cache(数据文件中读取的数据块的副本，由所有并行用户所共享，LRU列表，keep pool，recycle pool，nK buffer cache)
log buffer cache（循环缓冲区，对数据库所做更改的信息，重做条目ddl dml）
large pool（I/O服务器进程，备份还原操作）
java pool   stream pool
pga（堆栈空间，用户全局区：游标状态，用户会话数据，排序，散列，位图区）

**2）进程结构：**
用户进程：连接到oracle 应用程序，或者工具。
服务器进程：连接到oracle实例，创建会话时启动。
dbwn：将db buffer cache 中的脏数据写入磁盘上的数据库文件中。（从检查点队列的尾部写入，推进检查点）db_writer_process
lgwr:将重做日志缓冲区中的脏数据写入到磁盘上的数据文件中：（commit，3分之1满。dbwn写进程，每隔3秒）
ckpt：DBwn执行，将检查点信息写入数据文件头，控制文件头
smon：instance启动时进行恢复，清除不必要的临时段
pmon：恢复失败的用户进程（清除db buffer cache，是否用户进程资源），监控会话当会话超时时断开，监听的动态注册。
reco：解决有问题的事物处理，删除用问题的事物
arcn：日志切换，当online redo放到指定位置，收集事物处理重做数据后传输到指定位置

**3）存储结构：**
参数文件，控制文件，数据文件，联机重做日志文件，口令文件，备份文件，归档文件，预警日志文件。
逻辑存储结构（数据库，表空间，段，区，块），物理存储结构（文件）：由数据文件关联。

**4）数据库的启动状态：**
nomount：参数文件，分配内存，打开后台进程。
mount：控制文件，（读取控制文件中的数据库文件，online logfile 的名称和状态）
open：数据文件，打开数据文件，online logfile。
shutdown：允许新的连接，等待当前会话结束，等待当前事物结束，强制检查点并关闭文件。（abort，immdiate，transactional，nomal）

**5）Oracle数据库锁可以分为以下几大类：**
DML锁（data locks，数据锁），用于保护数据的完整性；（表锁，行几锁，共享锁，排他锁）
DDL锁（dictionary locks，字典锁），用于保护数据库对象的结构，如表、索引等的结构定义；
内部锁和闩（internal locks and latches），保护 数据库的内部结构。

**6）还原数据（还原段，undo_retention还原保留时间）：**

修改前的数据块的副本，保留到事物结束。
作用：读取一致性，oracle闪回（查询，闪回事物处理，闪回表，），从失败的事物进行恢复。



