<p align="center">
  <b>如何保证数据的正确性和一致性？</b>
</p>


- 数据不会被恶意串改和删除（一般不在数据库中考虑）
- 在发生宕机、磁盘损坏或其他意外时，数据完好
- 在程序出错的情况下，数据不被损坏（很难补救）

# 如何衡量？

- 一个CRUD操作成功，将永久性生效（Durability）
  - 除了`read`，其余操会导致状态转移
- 任何一个CRUD操作都将被正确的执行
  - 逻辑上不出错
  - 原子性（Atomicity）
    - 一个操作在瞬间完成
    - 只有完成和未完成两种可能
- 主要关注两个问题
  - 系统故障的干扰
  - 其他CRUD操作的干扰

# 解决宕机问题

- Logging/Journaling

## Undo日志

- immediate modified
- 在写入磁盘前，记录当前状态的值
- 需要在数据之前到达磁盘

### 规则

- 每一次对数据的改动都要记录日志
- 日志记录必须在数据之前到达磁盘（[WAL](https://en.wikipedia.org/wiki/Write-ahead_logging)）
- 操作结束之前，所有的数据和日志必须到达磁盘
  - 在\<commit\>之后表示这次操作成功
- 由于logging始终比真实的CRUD先，总是可以回到上一步操作
  - 从后往前恢复

### 缺点

- 日志读写是顺序的，数据访问是随机的，操作结束之前必须将数据刷盘
- 内存缺失了写缓存的作用

## Redo日志

- deferred modified
- 每次记录这次CRUD操作后的值

### 规则

- 每一次对数据的改动都要记录日志
- 操作结束之前所有的日志必须到达磁盘
  - 没有必要每一次都写进磁盘
- 数据到达磁盘后，需要在日志中记录`<END>`
  - `<END>`操作可以不需要
- 重放日志
  - 从前往后恢复
- 需要设置checkpoint来减少重放日志的工作量
  - 强制将之前的操作落盘
  - 定时checkpoint，减少log的大小
  - 在进行checkpoint时候，相应速度会变慢

### 缺点

- 若一个操作很长，则会占用很大的内存空间
  - 操作结束之前不能将数据刷盘

## Undo/Redo日志

### 规则

- 数据可以在任何时间到达磁盘
  - 操作结束前或操作结束后
- 日志必须在数据之前到达磁盘
- 操作结束之前所有的日志必须到达磁盘

# 不受其他CRUD干扰

## Linearizability

- All function calls have a linearization point at some instant between their invocation and their response.
- all functions appear to occur instantly at their linearization point, behaving as specified by the sequential definition.
- 保证无论谁先做谁后做，一定不会让系统confuse
- 通过加锁实现并发控制
  - 二阶段加锁可以实现线性化
  - 但当数据有多份时，若加锁，开销太大