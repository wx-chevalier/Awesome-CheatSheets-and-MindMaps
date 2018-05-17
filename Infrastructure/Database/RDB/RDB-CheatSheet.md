# Introduction

**数据库的核心组件：**

* 过程管理器(The process manager)：数据库都会有一个过程池/线程池需要进行管理。此外，为了使运行时间更短，现代数据库会使用自己的线程来替代操作系统线程。
* 网络管理器(The network manager)：网络的输入输出是个大问题，特别是对于分布式数据库来说。所以部分数据库针对网络管理打造了自己的管理器。
* 文件系统管理器(File system manager)：磁碟 I/O 是数据库的第一瓶颈。使用管理器进行磁碟文件进行管理是很重要的。
* 内存管理器(Memory manager)：当你需要处理大量内存数据或大量查询，一个高效的内存管理器是必须的。
* 安全管理器(Security manager)：进行认证和用户认证管理。
* 客户端管理器(Client manager)：进行客户端连接管理

![图片描述](http://img.blog.csdn.net/20160209193822791)

**数据库的工具：**

备份管理器：进行数据库的备份与恢复

复原管理器：在数据库崩溃后进行数据库重启

监视管理器：进行数据库活动日志记录，同时进行数据库监视

管理员管理器：进行 metadata 存储，管理数据库，表空间，数据泵等

查询管理器：对查询进行有效性检验，优化，编译和执行

数据管理器：包括事务管理器，缓存管理器，数据访问管理器

## Reference

### Practices & Resources

* [How does a relational database work ](http://coding-geek.com/how-databases-work/)

# 数据库组件

## 客户端管理器

![图片描述](http://img.blog.csdn.net/20160209194102418)

客户端管理器用于处理和管理客户端的通信。客户端可以是一台服务器或是终端应用。客户端管理器透过不同的 API 来提供访问权，例如：JDBC，ODBC，OLE-DB 等。

当你连接到一个数据库时：

* 管理器会对你的身份和授权进行确认。
* 如果验证通过，会对你的查询请求进行处理。
* 管理器同时会检查数据库是否处于满负荷状态。
* 管理器会等待请求资源的返回。如果发生超时，它会关闭连接并返回可读的错误信息。
* 然后会把你的查询发送给查询管理器，而你的查询是被处理状态。
* 管理器会存储部分结果到缓冲区然后开始进行结果返回。
* 如果出现异常，管理器会中断连接，返回相关原因解释并释放资源。

## 查询管理器

查询管理器是数据库的重要组成部分。其工作过程是：

* 查询会被解释以确认有效性
* 然后会被重写以消除不必要的操作并进行预优化处理
* 然后会被优化处理以提高性能并发送到执行和数据访问计划
* 然后改计划会被编译处理
* 最后进行执行查询

![图片描述](http://img.blog.csdn.net/20160209194441184)

### 查询重写器

重写器的目的是：

1.  进行查询预优化处理
2.  避免不必要的操作
3.  帮助优化器找出最佳方案

**常见的重写规则：**

**视图合并：**如果你在查询中使用了视图，那么该视图会被转换层 SQL 视图代码

**子查询扁平化：**子查询使查询优化变得困难，因此重写器会修改含有子查询的查询以消除子查询。

例如：

![图片描述](http://img.blog.csdn.net/20160209205206311)

会被重写为：

![图片描述](http://img.blog.csdn.net/20160209205221493)

**消除不必要的操作符：**例如当你使用了 UNIQUE 唯一约束而同时使用了 DISTINCT 操作符，那么 DISTINCT 将会被消除。

**多于 JOIN 连接清除：**当你 有两次相同条件的 JOIN 连接但是其中一个条件被隐藏了或者是一个多于的 JOIN，那么它会被清除。

**分区处理：**如果你使用了一个分区表，那么重写器会找出那个分区会被使用。

**自定义规则：**如果你有自定义的查询规则，重写器会执行这些规则。

## 数据管理器

查询管理器的作用是执行查询并对资源发出请求，数据管理器会处理这些请求并返回结果。但这里有两个问题：

* 关系数据库使用的是事务模型。所以你有可能得不到数据，因为其他人可能会正同时使用/修改这些数据。
* 数据获取是数据库中最慢的操作，因此数据管理必须要能高效地获取并数据存放在内存缓冲区。

### 缓存管理器

如前所述，数据库的主要瓶颈是磁碟 I/O。所以现代数据库使用了缓存管理器来提高效率。

查询执行器的数据请求对象是缓存管理器而不是直接的文件系统。缓存管理器有一个内存里缓存叫做缓冲池。从内存获取数据会大大提高数据库速度。

![图片描述](http://img.blog.csdn.net/20160209195818502)

#### 缓冲-替换策略

很多主流数据库(如：SQL Server，MySQL，Oracle 等)使用的是 LRU 算法。

LRU 是 Least Recently Used 的简写，意思最近使用。其理念是缓存最近使用的数据以便再次使用时快速读取。

![图片描述](http://img.blog.csdn.net/20160209195917393)

虽然它有很多优点但也存在不足，比方说表/索引的大小超过了缓冲区大小。因此出现了进阶版本的 LRU，这就是 LRU-K，例如在 SQL Server 使用的是 LRU-K，K=2。在 LRU-K 中：首先考虑数据的 K 次最近使用记录；根据数据的使用次数分配权值；如果有新的数组载入缓存，旧的但经常使用的数据不会被移除，但是当旧数据不再使用，将会被移除，所以权值的设立有助于减少多余数据。

### 事务管理器

事务管理器是为了确保每个查询会执行自己的事务。在讲述事务管理期前，我们需要理解 ACID 事务的概念。

ACID 是一个工作单元，它的意思是：

**Atomicity(原子性)：**事务是”全或全不”的，即使是 10 个小时的事务。如果事务崩溃了，会发生状态回滚。

**Isolation(隔离性)：**如果事务 A 和 B 同时运行，那么事务 A 和 B 的结果必须是一致的，不论 A 对于 B 是完成前/完成后/过程中的状态。

**Durability(耐久性)：**一旦事务完成，数据会存放在数据库中而不论发生什么情况(异常或错误)。

**Consistency(一致性)：**只有有效数据被写入数据库。一致性与原子行和隔离性关联。

#### 并发控制

确保隔离性，附着性和原子性的关键是能对同一数据进行正确写操作(添加，更新和删除)：

如果仅仅是数据读取事务，那么它们可以不与其它修改事务发生冲突；

如果一个修改事务处理的数据被其它事务读取，数据库需要找到方法来隐藏这些修改操作。同时，它需要保证这些修改操作不会被清除。

以上问题就是并发控制。最简单的处理方法是逐个执行事务。但是这不利于进行规模扩张，也无法发挥服务器/CPU 的多核性能。理想的处理方式是每当事务新建或取消时：

监视所有事务的全部操作，检查同时读取/修改相同数据的两个(或多个)事务是否发生冲突

，在发生冲突的事务中进行操作记录以减少冲突部分的大小，把冲突部分以其它次序进行处理，判别某事务是否可以取消

更正规的做法是进行冲突日程表管理。但是在企业级数据库中，是很难为每个新事务事件分配足够多的处理时间。所以会使用其它方法来进行处理。

### 锁管理器

为了处理以上问题，多数数据库会采用锁或数据版本来进行处理。但这是个内容丰富的话题，以下会把讨论重点放在锁部分。

什么是锁呢？

* 事务是否需要数据
* 是否锁定了数据
* 另一事务是否需要相同数据
* 是否不得不等待直至第一个事务释放这些数据

这叫做排斥锁。但是排斥锁针对的对象相同数据的读取和等待，这是不利于资源调配的。还有一种锁，叫共享锁。

在共享锁中：

* 一个事务是否只需读取数据 A
* 共享锁对数据锁定并读取数据
* 如果第二个事务也只需要读取数据 A
* 共享锁对数据锁定并读取数据
* 如果第三个事务只需要修改数据 A
* 那么会对数据进行排斥锁锁定，但它必须等待直至事务一，二释放共享锁才对数据 A 进行排斥锁锁定

![图片描述](http://img.blog.csdn.net/20160209200418122)

锁管理器的作用是提供和释放锁。从内部角度看，它把锁存储在一个有关联的 hash 数据表中。

* 哪些事务锁定了数据
* 哪些事务在等待数据

#### 死锁

锁的存在会导致一个问题：两个事务在无限期地等待数据：

![图片描述](http://img.blog.csdn.net/20160209200455051)

在上图中：

* 事务 A 对数据 1 使用了排斥锁，同时在等待获取数据 2
* 事务 B 对数据 2 使用了排斥是，同时在等待获取数据 1

这就是死锁。

遇到死锁后，锁管理器会选择对哪个事务进行撤销(回滚)以消除死锁。但要进行选择，并不是件容易的事。DB2 和 SQL Sever 使用了两段锁协议(Two-Phase Locking Protocol)来进行处理。

* 在增长段，事务会得到锁，但是不能释放锁。
* 在下降段，事务可释放锁，但是不能得到锁。

![图片描述](http://img.blog.csdn.net/20160209200616796)

其核心理念是：

* 释放不再使用的锁以减少其他事务对这些锁的等待时间
* 避免事务开始后对数据进行修改，所以这是非连贯事务

# Partition

## 垂直划分

按照功能划分，把数据分别放到不同的数据库和服务器。当一个网站开始刚刚创建时，可能只是考虑一天只有几十或者几百个人访问，数据库可能就个 db，所有表都放一起，一台普通的服务器可能就够了，而且开发人员也非常高兴，而且信心十足，因为所有的表都在一个库中，这样查询语句就可以随便关联了，多美的一件事情。但是随着访问压力的增加，读写操作不断增加，数据库的压力绝对越来越大，可能接近极限，这时可能人们想到增加从服务器，做什么集群之类的，可是问题又来了，数据量也快速增长。这时可以考虑对读写操作进行分离，按照业务把不同的数据放到不同的库中。其实在一个大型而且臃肿的数据库中表和表之间的数据很多是没有关系的，或者更加不需要(join)操作，理论上就应该把他们分别放到不同的服务器。例如用户的收藏夹的数据和博客的数据库就可以放到两个独立的服务器。这个就叫垂直划分(其实叫什么不重要)。

![](http://7xkt0f.com1.z0.glb.clouddn.com/fdsafas%E5%9B%BE%E7%89%871.png)

## 水平划分

垂直数据划分包括把数据库表分割成在不同服务器上保存的不同数据库实例。每台服务器一般分配完成一个特殊的任务。这样就可以对那些表中的 IO 进行分割。这种类型的分割取决于将系统逻辑地划分成许多部分，以便这些部分能够独立操作。如果实例间需要最少量的交互进行事务处理，这种处理就很有必要。

例如，如果你的数据库系统维护销售、营销和广告数据，最好是把这些表分割成单个的数据库实例，阻止它们共享同一台服务器上的 IO。可能你还需要处理这两个共享一些相同数据(例如客户数据)的系统。能够分割这些商业功能，你就可以在必要时向外扩展数据库环境，提高系统效率。

你可以采取一些措施，如在每一台服务器上使用相互连接的表和视图，以便实例可以从其它实例中查看数据。这样做可以减少应用程序层决定在哪找到它需要的数据时所需的额外计算量。你需要保证应用程序层具有必要的逻辑性，以决定将数据保存在哪台服务器上。

![](http://7xkt0f.com1.z0.glb.clouddn.com/fasdfasd%E5%9B%BE%E7%89%872.png)