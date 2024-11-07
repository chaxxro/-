# TCL

TCL(Transaction Control Language) 事务控制语言，又名 TPL(Transaction Process Language) 事务处理语言，它能确保被 DML 语句影响的表的所有行及时得以更新。常用关键字有：`START TRANSACTION`、`BEGIN`、`SAVEPOINT`、`ROLLBACK`、`COMMIT`、`SET TRANSACTION`

### 查看事务模式

MySQL 默认操作模式就是 autocommit 自动提交模式

自动提交事务由环境变量 `autocommit` 来控制，该变量只对当前会话有效

```sql
select @@global.autocommit

show variables like "%autocommit%"
```

`autocommit` 默认值是 1，表示在命令行模式下每条增删改语句在键入回车后，都会立即生效，而不需要手动 commit

### 关闭/开启自动提交事务

```sql
# 开启当前会话的自动提交事务
set autocommit = 1
# 关闭当前会话的自动提交事务
set autocommit = 0

# 永久关闭
# 通过修改配置文件 my.cnf 文件，添加配置项
autocommit=0
```

### 事务控制

- 开启事务

```sql
# 1
start transaction

# 2
begin
```

- 提交事务

```sql
commit
```

- 回滚

```sql
rollback
```

- 保存点

```sql
# 创建一个保存点
savepoint pointname

# 删除一个保存点
release savepoint pointname

# 回滚保存点
rollback to pointname
```

### 事务等级

- 查看事务等级

```sql
# 查看全局事务等级
select @@global.tx_isolation

# 查看当前会话事务等级
select @@session.tx_isolation
select @@tx_isolation
show variables like 'tx_isolation'
```

- 设置事务等级

```sql
set scopename transaction isolation level txlevel
# scopename 包括 global 和 session，默认值 session
# txlevel 包括 read uncommitted、read committed、repeatable read、serializable
# InnoDB 默认事务等级是 repeatable read
```

