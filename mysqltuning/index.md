# MySQL调优


## 影响mysql的性能因素
* 业务需求对MySQL的影响
* 存储定位对MySQL的影响
    * 不适合放进MySQL的数据
        * 二进制多媒体数据
        * 流水队列数据
        * 超大文本数据
    * 需要放进缓存的数据
        * 系统各种配置及规则数据
        * 活跃用户的基本信息数据
        * 活跃用户的个性化定制信息数据
        * 准实时的统计信息数据
        * 其他一些访问频繁但变更较少的数据
* Schema设计对系统的性能影响
    * 尽量减少对数据库访问的请求
    * 尽量减少无用数据的查询请求
* 硬件环境对系统性能的影响
## 性能分析
### MySQL常见瓶颈
* CPU：CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时候
* IO：磁盘I/O瓶颈发生在装入数据远大于内存容量的时候
* 服务器硬件的性能瓶颈：top，free，iostat 和 vmstat来查看系统的性能状态

### 性能下降SQL慢 执行时间长 等待时间长 原因分析
* 查询语句写的烂
* 索引失效（单值、复合）
* 关联查询太多join（设计缺陷或不得已的需求）
* 服务器调优及各个参数设置（缓冲、线程数等）

### MySQL常见性能分析手段
> 在优化MySQL时，通常需要对数据库进行分析，常见的分析手段有慢查询日志，EXPLAIN 分析查询，profiling分析以及show命令查询系统状态及系统变量，通过定位分析性能的瓶颈，才能更好的优化数据库系统的性能。

#### 性能瓶颈定位
我们可以通过 show 命令查看 MySQL 状态及变量，找到系统的瓶颈：

```sql
Mysql> show status ——显示状态信息（扩展show status like ‘XXX’）
Mysql> show variables ——显示系统变量（扩展show variables like ‘XXX’）
Mysql> show innodb status ——显示InnoDB存储引擎的状态
Mysql> show processlist ——查看当前SQL执行，包括执行状态、是否锁表等
Shell> mysqladmin variables -u username -p password——显示系统变量
Shell> mysqladmin extended-status -u username -p password——显示状态信息
```
#### Explain(执行计划)
> 使用 Explain 关键字可以模拟优化器执行SQL查询语句，从而知道 MySQL 是如何处理你的 SQL 语句的。分析你的查询语句或是表结构的性能瓶颈

#### 慢查询日志
MySQL 的慢查询日志是 MySQL 提供的一种日志记录，它用来记录在 MySQL 中响应时间超过阈值的语句，具体指运行时间超过 long_query_time 值的 SQL，则会被记录到慢查询日志中。

#### Show Profile 分析查询
通过慢日志查询可以知道哪些 SQL 语句执行效率低下，通过 explain 我们可以得知 SQL 语句的具体执行情况，索引使用等，还可以结合Show Profile命令查看执行状态。

## 性能优化
### 索引优化
* 全值匹配我最爱
* 最佳左前缀法则，比如建立了一个联合索引(a,b,c)，那么其实我们可利用的索引就有(a), (a,b), (a,b,c)
* 不在索引列上做任何操作（计算、函数、(自动or手动)类型转换），会导致索引失效而转向全表扫描
* 存储引擎不能使用索引中范围条件右边的列
* 尽量使用覆盖索引(只访问索引的查询(索引列和查询列一致))，减少select
* is null ,is not null 也无法使用索引
* like "xxxx%" 是可以用到索引的，like "%xxxx" 则不行(like "%xxx%" 同理)。like以通配符开头('%abc...')索引失效会变成全表扫描的操作，
* 字符串不加单引号索引失效
* 少用or，用它来连接时会索引失效
* <，<=，=，>，>=，BETWEEN，IN 可用到索引，<>，not in ，!= 则不行，会导致全表扫描
#### 一般性建议
* 对于单键索引，尽量选择针对当前query过滤性更好的索引
* 在选择组合索引的时候，当前Query中过滤性最好的字段在索引字段顺序中，位置越靠前越好。
* 在选择组合索引的时候，尽量选择可以能够包含当前query中的where字句中更多字段的索引
* 尽可能通过分析统计信息和调整query的写法来达到选择合适索引的目的
* 少用Hint强制索引

### 查询优化
* 永远小表驱动大表（小的数据集驱动大的数据集）
* ORDER BY关键字优化
    * ORDER BY子句，尽量使用 Index 方式排序，避免使用 FileSort 方式排序
    * MySQL 支持两种方式的排序，FileSort 和 Index，Index效率高，它指 MySQL 扫描索引本身完成排序，FileSort 效率较低；
    * ORDER BY 满足两种情况，会使用Index方式排序；①ORDER BY语句使用索引最左前列 ②使用where子句与ORDER BY子句条件列组合满足索引最左前列
    * 尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀
    * 如果不在索引列上，filesort 有两种算法，mysql就要启动双路排序和单路排序
    * 双路排序：MySQL 4.1之前是使用双路排序,字面意思就是两次扫描磁盘，最终得到数据
    * 单路排序：从磁盘读取查询需要的所有列，按照ORDER BY 列在 buffer对它们进行排序，然后扫描排序后的列表进行输出，效率高于双路排序
    * 优化策略
        * 增大sort_buffer_size参数的设置
        * 增大max_lencth_for_sort_data参数的设置
* GROUP BY关键字优化
    * group by实质是先排序后进行分组，遵照索引建的最佳左前缀
    * 当无法使用索引列，增大 max_length_for_sort_data 参数的设置，增大sort_buffer_size参数的设置
    * where高于having，能写在where限定的条件就不要去having限定了

### 数据类型优化
MySQL 支持的数据类型非常多，选择正确的数据类型对于获取高性能至关重要。不管存储哪种类型的数据，下面几个简单的原则都有助于做出更好的选择。
* 更小的通常更好：一般情况下，应该尽量使用可以正确存储数据的最小数据类型。
* 简单就好：简单的数据类型通常需要更少的CPU周期。例如，整数比字符操作代价更低，因为字符集和校对规则（排序规则）使字符比较比整型比较复杂。
* 尽量避免NULL：通常情况下最好指定列为NOT NULL

## 参考文章
* [MySQL调优](https://juejin.cn/post/6850037271233331208#heading-54 "MySQL调优")
