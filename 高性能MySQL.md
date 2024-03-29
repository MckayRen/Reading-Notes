### 高性能MySQL

#### 1.3.3 事务日志

- 使用事务日志可以提高事务的效率
- 存储引擎在修改表数据时只需要修改其内存拷贝，再把修改行为记录到持久在硬盘上的事务日志中，不需要每次都将修改的数据持久到磁盘
- 事务日志采用追加的方式。事务日志持久后，内存中被修改的数据可以在后台慢慢刷回到磁盘中。称之为预写式日志（Write-Ahead Logging），修改数据需要写两次磁盘
- 系统崩溃时，会在重启时自动恢复已经被修改并持久化到事务日志的数据

#### 1.3.4 MySQL中的事务，自动提交

- 默认采用自动提交（AUTOCOMMIT），如果不是显式的开启一个事务，每一个查询都会被当作一个事务执行提交操作

#### 1.3.4 多版本并发控制

- 根据事务的开始时间不同，每个事务对同一张表、同一个时刻所看到的数据有可能不同
- InnoDB的MVCC，通过在每行记录的后面保存两个隐藏的列来实现的，一列是保存行的创建时间的，一列是保存行的过期时间（或删除时间），存储的值是系统版本号，递增的
- SELECT：InnoDB会根据下面两个条件检查每行记录 
  1. InnoDB只会查找早于当前事务版本号的数据行
  2. 行的删除版本要么未定义，要么大于当前事务版本号
- INSERT：为每条新插入的每一行保存当前系统版本号作为行版本号
- DELETE：为删除的每一行保存当前系统版本号作为行删除标示
- UPDATE：InnoDB是新插入一条数据，保存当前系统版本号作为行版本号，同时保存当前系统版本号到原来的行作为行删除标示
- MVCC只在REPEATABLE READ和READ COMMITED两个隔离级别下工作
- InnoDB采用两阶段锁协议，事务执行过程中，随时都可以锁定，锁只有在COMMIT或者ROLLBACK的时候才会释放，并且所以的锁都是在同一个时间释放的

#### 4.3.1 范式的优点和缺点

- 范式化的更新操作通常比反范式化更快
- 当数据较好的范式化时，就只有很少或者没有重复数据，所以只需要修改更少的数据
- 范式化的表通常更小，所以可以放进内存里，所以执行操作会很快
- 很少有多余的数据，意味着查询时更少需要 distinct 或者 order by

#### 5.3 高性能的索引策略

##### 5.3.1 独立的列

##### 5.3.2 前缀索引和索引选择性

- 需要索引很长的字符列时，可以选择只索引开始的部分字符 ``` ALTER TABLE my_table ADD KEY(city(7)) ```
- 前缀索引是一种使索引更小更快的方法，但是无法做 order by 和 group by，也无法做覆盖扫描

##### 5.3.3 多列索引

##### 5.3.4 选择合适的索引列顺序

##### 5.3.5 聚簇索引

##### 5.3.6 覆盖索引

###### 如果一个索引包含（或者覆盖）所有要查询的字段的值，则成为覆盖索引

- 索引条目远小于数据行大小
- 因为索引是按照列值顺序存储的，对于范围查询要比从磁盘随机读取数据 I/O 要快得多
- 一些存储引擎如 MyISAM 在内存中只缓存索引，数据则依赖操作系统缓存，因此访问数据需要系统调用，影响性能
- InnoDB 的聚簇索引，二级索引（非聚簇索引）在叶子节点保存了行的主键值，如果能够覆盖查询，则减少了一次对聚簇索引的二次查询

##### 5.3.7 使用索引扫描来做排序

##### 5.3.8 压缩（前缀索引）索引

##### 5.3.9 避免冗余和重复索引

##### 5.3.10 避免未使用的索引

##### 5.3.11 索引和锁

#### 6.4 查询执行的基础

查询执行路径：

1. 客户端发送一条查询给服务器
2. 服务器如果命中缓存则直接返回
3. 服务器进行 SQL 解析、预处理，再由优化器生成对应的执行计划
4. MySQL 根据优化器生成对应的执行计划，调用存储引擎的 API 来执行查询
5. 将结果返回给客户端

查询状态：

- Sleep 等待客户端发送新的请求
- Query 执行查询或者正在将结果发送给客户端
- Locked 在MySQL服务器层，等待表锁
- Analyzing and statistics 收集存储引擎的统计信息，并生成查询的执行计划
- Copy to tmp table [on disk] 执行查询，并将结果集复制到一个临时表中。一般是 group by，要么是文件排序，要么是 union 操作。如果后面有 on disk，代表正在将内存临时表放到磁盘上
- Sorting result 对结果集进行排序
- Sending data 线程可能在多个状态之间传送数据，或者在生成结果集，或者在向客户端发送数据

##### 6.4.2 查询缓存

在解析一个查询语句之前，如果查询缓存是打开的，那么 MySQL 会优先检查这个查询是否命中查询缓存中的数据。这个检查是通过一个对大小写敏感的哈希查找实现的。

如果当前查询恰好命中了查询缓存，那么在返回结果之前 MySQL 会检查一次用户权限，如果权限没有问题，MySQL 会跳过其他所有阶段，直接从缓存中拿到结果直接返回给客户端。这种情况，查询不会被解析，不用生成执行计划，不会被执行。











