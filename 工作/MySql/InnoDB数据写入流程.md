# InnoDB 数据写入流程

1. 事务开始
2. 生成 Undo Log，记录旧版本数据，用于回滚和 MVCC
3. 修改 Buffer Pool 中的数据页，这是内存操作
4. 写入 Redo Log Buffer，记录数据变更，确保持久性
5. 写入 Binlog，主从复制和逻辑恢复
6. 两阶段提交 2PC，协调 Redo Log 和 Binlog
7. 事务提交，持久化日志，释放锁

## 事务开始

1. 用户执行 `BEGIN` 或隐式开启事务
2. InnoDB 分配 TRX_ID，用于标识事务和 MVCC

## 生成 undo log

记录数据修改前的状态，用于支持事务回滚和 MVCC

## 修改 Buffer Pool

将数据写入 Buffer Pool 中的对应数据页

1. 如果数据页未在 Buffer Pool 中则先将数据从磁盘加载到 Buffer Pool
2. 修改 Buffer Pool 中的数据，生成脏页

数据页不会立即刷盘，由后台线程异步处理

## 写入 Redo Log Buffer

记录数据页的物理修改

事务提交前，Redo Log 先写入内存中的 Log Buffer，根据 `innodb_flush_log_at_trx_commit` 参数决定何时刷盘

- 0：每秒刷盘，可能丢失 1 秒数据
- 1（默认）：事务提交时立即刷盘，确保持久性
- 2：写入 OS 缓存，依赖系统刷盘（如崩溃可能丢失数据）

Redo Log先进入 Prepare状态，标记事务已准备好提交，但未最终确认

## 写入 Binlog

记录逻辑操作

事务提交前，Binlog 写入线程级内存缓存，根据 `sync_binlog` 参数决定刷盘时机

- 0：依赖 OS 刷盘，风险最高
- 1（推荐）：事务提交时强制刷盘
- N：每 N 次事务批量刷盘

## 2PC

确保 Redo Log 和 Binlog 逻辑一致

Prepare 阶段

- InnoDB 将 Redo Log 标记为 `PREPARE` 状态
- 此时 Redo Log 已刷盘（若 `innodb_flush_log_at_trx_commit=1`）

Commit 阶段

- 写入 Binlog 并刷盘（若 `sync_binlog=1`）
- InnoDB 将 Redo Log 标记为 `COMMIT` 状态

## 事务提交

释放行锁，清理 Undo Log，通知用户事务完成

