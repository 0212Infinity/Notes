## 单线程

Redis基于Reactor模式开发了**网络事件处理器**、文件事件处理器file event handler,它是单线程的,所以Redis才叫做单线程的模型,它采用IO多路复用机制来同时监听多个Socket,根据Socket上的事件类型来选择对应的事件处理器来处理这个事件。可以实现高性能的网络通信模型,又可以跟内部其他单线程的模块进行对接,保证了Redis内部的线程模型的简单性

文件事件处理器的结构包含4个部分:**多个Socket, IO多路复用程序、文件事件分派器以及事件处理器**(命令请求处理器、命令回复处理器、连接应答处理器等) 

多个Socket可能并发的产生不同的事件，IO多路复用程序会监听多个Socket,会将Socket放入一个队列中排队,每次从队列中有序、同步取出一个Socket给事件分派器,事件分派器把Socket给对应的事件处理器。然后一个Socket的事件处理完之后, IO多路复用程序才会将队列中的下一个Socket给事件分派器。文件事件分派器会根据每个Socket当前产生的事件,来选择对应的事件处理器来处理。 



1. Redis启动初始化时,将连接应答处理器跟AE READABLE事件关联
2. 若一个客户端发起连接,会产生一个AE-READABLE事件,然后由连接应答处理器负责和客户端建立连接,创建客户端对应的socket,同时将这个socket的AEREADABLE事件和命令请求处理器关联,使得客户端可以向主服务器发送命令请求
3. 当客户端向Redis发请求时(不管读还是写请求) ,客户端socket都会产生一个AEREADABLE事件,触发命令,请求处理器。处理器读取客户端的命令内容,然后传给相关程序执行
4. 当Redis服务器准备好给客户端的响应数据后,会将socket的AEWRITABLE事件和命令回复处理器关联,当客户端准备好读取响应数据时,会在socket产生一个AE-WRITABLE事件,由对应命令回复处理器处理,即将准备好的响应数据写入socket,供客户端读取
5. 命令回复处理器全部写完到socket后,就会删除该socket的AE-WRITABLE事件和命令回复处理器的映射



**单线程快的原因:**

1. 纯内存操作
2. 核心是基于非阻塞的IO多路复用机制
3. 单线程反而避免了多线程的频繁上下文切换带来的性能问题



## 分布式锁

1. 首先利用setnx来保证:如果key不存在才能获取到锁,如果key存在,则获取不到锁
2. 然后还要利用lua脚本来保证多个redis操作的**原子性**
3. 同时还要考虑到**锁过期**,所以需要额外的一个看门狗定时任务来监听锁是否需要续约
4. 同时还要考虑到redis节点挂掉后的情况,所以需要采用**红锁**的方式来同时向N/2+1个节点申请锁,都申请到了才证明获取锁成功，这样就算其中某个redis节点挂掉了,锁也不能被其他客户端获取到 



## 持久化机制

### RDB

**RDB**: Redis DataBase将某一个时刻的**内存快照**(Snapshot) ,以二进制的方式写入磁盘



手动触发

- **save**命令: 使Redis处于阻塞状态,直到RDB持久化完成,才会响应其他客户端发来的命令,所以在生产环境一定要慎用
- **bgsave**命令: fork出一个子进程执行持久化,主进程只在fork过程中有**短暂的阻塞**,子进程创建之后,主进程就可以响应客户端请求了 

fork 是linux中的父子进程, 共享一块内存区域

为了保证bgsave中不出现"脏"数据, 这里使用的是写时拷贝(COW)



自动触发

- **save m n**: 在m秒内,如果有n个键发生改变,则自动触发持久化,通过bgsave执行,如果设置多个、只要满足其一就会触发,配置文件有默认配置(可以注释掉)	例如: save 60 10 | save 600 100
- **flushall**: 用于清空redis所有的数据库, **flushdb**清空当前redis所在库数据(默认是0号数据库),会清空RDB文件,同时也会生成dump.rdb、内容为空
- **主从同步:** 全量同步时会自动触发bgsave命令,生成rdb发送给从节点 



优点

1. 整个Redis数据库将只**包含一个**文件dump.rdb,方便持久化
2. 容灾性好,方便备份(只有一个rdb文件)
3. 性能最大化, fork子进程来完成写操作,让主进程继续处理命令,所以是IO最大化。使用单独子进程来进行持久化,**主进程不会进行任何IO操作**,保证了redis的高性能
4. 相对于数据集大时,比AOF的启动效率更高(RDB的恢复速度要比AOF高)

缺点

1. 数据安全性低。RDB是**间隔一段时间进行持久化**,如果持久化之间redis发生故障,会发生数据丢失。所以这种方式更适合数据要求不严谨的时候)
2. 由于RDB是通过fork子进程来协助完成数据持久化工作的,因此,如果当数据集较大时,可能会导致整个服务器停止服务几百毫秒,甚至是1秒钟。会占用cpu (主进程在fork过程中有短暂的阻塞)



### AOF

> RDB保存的是数据集, AOF保存的是命令集

**AOF**: Append Only File以日志的形式记录服务器所处理的每一个写、删除操作,查询操作不会记录,以文本的万式记录,可以打开文件看到详细的操作记录,调操作系统命令进程刷盘

1. 所有的写命令会追加到AOF缓冲中
2. AOF缓冲区根据对应的**策略**向硬盘进行**同步**操作
3. 随着AOF文件越来越大,需要定期对AOF文件进行重写,达到压缩的目的
4. **当Redis重启时,可以加载AOF文件进行数据恢复**



同步策略

**每秒同步**: 异步完成,效率非常高,一旦系统出现宕机现象,那么这一秒钟之内修改的数据将会丢失

**每修改同步**: 同步持久化,每次发生的数据变化都会被立即记录到磁盘中,最多丢一条

**不同步**: 由操作系统控制,可能丢失较多数据 (这里的不同步不是不进行持久化, 而是redis不管, 让操作系统来控制)



优点

1. 数据安全
2. 通过append模式写文件,即使中途服务器宕机也不会破坏已经存在的内容,可以通过redis-check-aof工具解决数据一致性问题
3. AOF机制的rewrite模式。定期对AOF文件进行重写,以达到压缩的目的 



缺点

1. AOF文件比RDB文件大,且恢复速度慢
2. 数据集大的时候,比rdb启动效率低
3. 运行效率没有RDB高

> AOF文件比RDB更新频率高,优先使用AOF还原数据
>
> AOF比RDB更安全也更大
>
> RDB性能比AОF好
>
> 如果两个都配了优先加载AOF 



## 过期键的删除策略

Redis是key-value数据库,我们可以设置Redis中缓存的key的过期时间。Redis的过期策略就是指当Redis中缓存的key过期了, Redis如何处理

- **惰性过期**: 只有当访问一个key时,才会判断该key是否已过期,过期则清除。该策略可以**最大化地节省CPU资源**,却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问,从而不会被清除,**占用大量内存**。

- **定期过期**: 每隔一定的时间,会扫描一定数量的数据库的expires字典中一定数量的key,并清除其中已过期的key,该策略是前两者(惰性、定时[定时器实时监控])的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时,可以在不同情况下使得CPU和内存资源达到最优的平衡效果。

  

  (expires字典会保存所有设置了过期时间的key的过期时间数据,其中, key是指向键空间中的某个键的指针value是该键的毫秒精度的UNIX时间戳表示的过期时间。键空间是指该Redis集群中保存的所有键。)

  Redis中**同时使用**了惰性过期和定期过期两种过期策略。 



## 缓存

