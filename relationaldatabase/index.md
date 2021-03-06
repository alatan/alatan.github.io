# 关系型数据库概览


![](/images/db/db.png "数据库总览")

![](/images/db/rdb/sqlDB.png "SQL数据库原理")

## 数据库组件
![](/images/db/rdb/allComponent.png "SQL数据库概览")

### 核心组件
* 进程管理器（process manager）：很多数据库具备一个需要妥善管理的进程/线程池。再者，为了实现纳秒级操作，一些现代数据库使用自己的线程而不是操作系统线程。 
* 网络管理器（network manager）：网路I/O是个大问题，尤其是对于分布式数据库。所以一些数据库具备自己的网络管理器。 
* 文件系统管理器（File system manager）：磁盘I/O是数据库的首要瓶颈。具备一个文件系统管理器来完美地处理OS文件系统甚至取代OS文件系统，是非常重要的。 
* 内存管理器（memory manager）：为了避免磁盘I/O带来的性能损失，需要大量的内存。但是如果你要处理大容量内存你需要高效的内存管理器，尤其是你有很多查询同时使用内存的时候。 
* 安全管理器（Security Manager）：用于对用户的验证和授权。 
* 客户端管理器（Client manager）：用于管理客户端连接。

### 工具 
* 备份管理器（Backup manager）：用于保存和恢复数据。 
* 恢复管理器（Recovery manager）：用于崩溃后重启数据库到一个一致状态。 
* 监控管理器（Monitor manager）：用于记录数据库活动信息和提供监控数据库的工具。 
* 管理员管理器（Administration manager）：用于保存元数据（比如表的名称和结构），提供管理数据库、模式、表空间的工具。

### 查询管理器 
* 查询解析器（Query parser）：用于检查查询是否合法 
* 查询重写器（Query rewriter）：用于预优化查询 
* 查询优化器（Query optimizer）：用于优化查询 
* 查询执行器（Query executor）：用于编译和执行查询

### 数据管理器： 
* 事务管理器（Transaction manager）：用于处理事务 
* 缓存管理器（Cache manager）：数据被使用之前置于内存，或者数据写入磁盘之前置于内存 
* 数据访问管理器（Data access manager）：访问磁盘中的数据

## 数据查询的流程
本章集中探讨数据库如何通过如下进程管理SQL查询的：
1. 客户端管理器
2. 查询管理器
3. 数据管理器（含恢复管理器）
4. 客户端管理器

### 客户端管理器
> 客户端管理器是处理客户端通信的。客户端可以是一个（网站）服务器或者一个最终用户或最终应用。客户端管理器通过一系列知名的API（JDBC, ODBC, OLE-DB …）提供不同的方式来访问数据库。客户端管理器也提供专有的数据库访问API。

![](/images/db/rdb/ClientManager.png "客户端管理器")
当你连接到数据库时： 
* 管理器首先检查你的验证信息（用户名和密码），然后检查你是否有访问数据库的授权。这些权限由DBA分配。 
* 然后，管理器检查是否有空闲进程（或线程）来处理你对查询。 
* 管理器还会检查数据库是否负载很重。 
* 管理器可能会等待一会儿来获取需要的资源。如果等待时间达到超时时间，它会关闭连接并给出一个可读的错误信息。 
* 然后管理器会把你的查询送给查询管理器来处理。 
* 因为查询处理进程不是『不全则无』的，一旦它从查询管理器得到数据，它会把部分结果保存到一个缓冲区并且开始给你发送。 
* 如果遇到问题，管理器关闭连接，向你发送可读的解释信息，然后释放资源。

### 查询管理器
![](/images/db/rdb/QueryManager.png "查询管理器")
这个多步骤操作过程如下： 
* 查询首先被解析并判断是否合法 
* 然后被重写，去除了无用的操作并且加入预优化部分 
* 接着被优化以便提升性能，并被转换为可执行代码和数据访问计划。 
* 然后计划被编译 
* 最后，被执行

### 数据管理器
![](/images/db/rdb/DataManager.png "查询管理器")
在这一步，查询管理器执行了查询，需要从表和索引获取数据，于是向数据管理器提出请求。
但是有 2 个问题： 
* 关系型数据库使用事务模型，所以，当其他人在同一时刻使用或修改数据时，你无法得到这部分数据。 
* 数据提取是数据库中速度最慢的操作，所以数据管理器需要足够聪明地获得数据并保存在内存缓冲区内。

#### 缓冲区
* 缓冲池(buffer pool)是一种常见的降低磁盘访问的机制；
* 缓冲池通常以页(page)为单位缓存数据；
* 缓冲池的常见管理算法是LRU，memcache，OS，InnoDB都使用了这种算法；
* InnoDB对普通LRU进行了优化：将缓冲池分为老生代和新生代，入缓冲池的页，优先进入老生代，页被访问，才进入新生代，以解决预读失效的问题页被访问，且在老生代停留时间超过配置阈值的，才进入新生代，以解决批量数据访问，大量热数据淘汰的问题

* [缓冲区详解](https://juejin.cn/post/6844903874172551181 "缓冲区详解")

## 三个范式
* 第一范式（1NF）：数据库表中的字段都是单一属性的，不可再分。这个单一属性由基本类型构成，包括整型、实数、字符型、逻辑型、日期型等。
* 第二范式（2NF）：数据库表中不存在非关键字段对任一候选关键字段的部分函数依赖（部分函数依赖指的是存在组合关键字中的某些字段决定非关键字段的情况），也即所有非关键字段都完全依赖于任意一组候选关键字。
* 第三范式（3NF）：在第二范式的基础上，数据表中如果不存在非关键字段对任一候选关键字段的传递函数依赖则符合第三范式。所谓传递函数依赖，指的是如 果存在"A → B → C"的决定关系，则C传递函数依赖于A。因此，满足第三范式的数据库表应该不存在如下依赖关系： 关键字段 → 非关键字段 x → 非关键字段y

## 参考文章
* [关系型数据库是如何工作的](https://www.pdai.tech/md/db/sql/sql-db-howitworks.html "关系型数据库是如何工作")
