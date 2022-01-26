# 19 | TiDB Server 关键性能参数与优化
## CPU
### Dynamic frequency scaling
- 定义：CPU动态节能技术用于降低服务器功耗，通过选择系统空闲状态不同的电源管理策略，可以实现不同程度降低服务器功耗，更低的功耗意味着CPU唤醒更慢，对性能影响更大。
- 5种模式
	- performance
	- userspace
	- powersave
	- ondemand
	- conservative
- 配置命令
```
cpupower frequency-set --governor performance
```

### Numa Binding
- 定义：内存直接绑定在CPU上，CPU只有访问自身管理的内存物理地址时，才会有较短的响应时间。
- NUMA架构的服务器，通过绑定NUMA节点，尽可能的避免跨NUMA访问内存，提升tidb-server的性能。
- 可使用`numactl`命令绑定。

## 内存
### Transparent Huge Page(THP)
- 对于数据库应用，不推荐使用THP，因为数据库往往具有稀疏而不是连续的内存访问模式，且当高阶内存碎片化比较严重时，分配THP页面会出现较大延迟。若开启针对THP的直接内存规整功能，也会出现系统CPU使用率激增的现象，因此建议关闭THP。

### Virtual Memory Parameters
- `dirty_ratio`、`dirty_background_ratio`：通常不需调整。对于高性能SSD，比如NVME设备来说，降低其值有利于提高内存回收时的效率。

## Disk
### I/O scheduler
- I/O调度程序确定I/O操作何时在存储设备上运行以及持续多长时间，也称为I/O升降机。
	- NOOP
	- CFQ
	- DEADLINE

### Mount Parameters
- 默认的方式下linux会把文件访问的时间atime做记录，文件系统在文件被访问、创建、修改等的时候记录下了文件的一些时间戳，比如：文件创建时间、最近一次修改时间和最近一次访问时间；这在绝大部分的场合都是没有必要的。
- 可以尝试使用`noatime`和`nodirtime`禁止记录最近一次访问时间戳。

## TiDB Configuration
### Performance
#### max-proc
- 控制tidb-server使用的CPU核数，在单台机器上部署多个tidb-server设置这个变量可以限制tidb-server使用的资源，避免对其他进程造成影响。

#### token-limit
- 配置可以同时执行请求的session数量，用于流量控制，避免并发请求数过多造成tidb-server资源耗尽，服务无法响应。

#### force-priority
- 控制tidb-server的优先级，设置后，所有请求都会使用该优先级来执行。

#### committer-concurrency
- 控制一个事物commit阶段的最大的并发数量。对于一个大事务来说，提交事务需要向TiKV发送大量写请求。设置更大的并发可能让大事务更快提交完成，但是也可能会造成TiKV瞬时压力过大，请求堆积，无法响应。

### TiKV Client
#### grpc-connection-count
- 设置TiDB和每个TiKV之间的grpc连接数量，当大量并发请求发送到一个grpc连接的时候，单个grpc连接串行发送请求，可能会成为瓶颈，当grpc连接成为瓶颈时，设置更大的grpc连接数可以提升性能，但代价是更多消耗系统资源。

### Prepared Plan Cache
- enabled：开启后减少执行计划造成的计算开销，让同样类型的语句使用相同的执行计划，提升性能，代价是当数据或查询条件变化时，执行计划可能会不是最好的。
- capacity：用来限制cache的大小（缓存条数），避免占用过多内存，如果应用使用大量不同类型的请求，超过capacity上限，plan cache的效果会打折扣。

## TiDB System Variables
### Concurrency
- 系统资源空闲较多时，设置更高并发度让资源利用的更充分，可以提升整体性能，但是更高并发往往消耗更多系统资源，在资源紧张时反而会降低整体性能
|属性名|含义|
|:-:|:-:|
|tidb_distsql_scan_concurrency|控制TableScan和IndexScan算子的并发度|
|tidb_index_lookup_concurrency|控制IndexLookUp算子的并发度|
|tidb_build_stats_concurrency|控制Analyze执行的并发度，可能会影响在线业务的延迟|
|tidb_hash_join_concurrency|控制HashJoin算子的并发度|
|tidb_index_lookup_join_concurrency|控制IndexLookUpJoin算子的并发度|
|tidb_ddl_reorg_worker_cnt|控制DDL加索引的并发度|

### Batch Size
- 一个Chunk包含多行数据，SQL语句执行时，执行器以Chunk为单位来执行获取数据、表达式值和JOIN等操作，当结果集较大时，更大的Chunk size可以提升性能，如果结果集很小，小于Chunk size，申请过大chunk的内存会造成性能的损失。
- tidb_init_chunk_size/tidb_max_chunk_size
- tidb_index_join_batch_size

### Limit
- tidb_store_limit：控制同时发往一个TiKV节点的请求数量，避免单个TiKV节点因为请求量太大，超过处理能力，造成大量请求超时或返回错误。
- tidb_retry_limit：控制乐观事务的重试次数，乐观事务在遇到事务冲突的时候会进行重试，但是重试次数过多会使冲突加剧，造成性能下降，重试次数太少会造成事务执行成功率下降。

### Backoff
- 在请求遇到可重试的错误时，在重试前需要等待一段时间，这个时间设置的过大，会增加延迟，如果设置的过小，会造成很多无谓的重试，消耗过多的系统资源。
- `tidb_backoff_weight`：TiDB backoff最大时间的权重，通过这个变量来调整最大重试时间。
- `tidb_backoff_lock_fast`：读请求遇到的锁的backoff时间。
