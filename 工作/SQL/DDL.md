# DDL

DDL(Data Definition Language) 数据定义语言，用于定义数据库的三级结构，包括外模式、概念模式、内模式及其相互之间的映像，定义数据的完整性约束、安全控制等。DDL 不需要 commit，常用关键字有： `CREATE`、`ALTER`、`DROP`、`TRUNCATE`、`COMMENT`、`RENAME`

### 创建数据库

```sql
create database databasename

create database Student
```

### 删除数据库

```sql
drop database databasename

drop database Student
```

### 查看数据库

```sql
# 查看全部数据库
show databases

# 查看当前数据库
# 1
select database()
# 2
status
```

### 连接数据库

```sql
use databasename
```

### 创建数据表

```sql
create table tablename (
  filed1 type1 [is null] [key] [default value] [extra] [comment],
  ...
)[engine] [charset]
```

- 除了表名，字段名和字段类型，其它都是可选参数，可有可无，根据实际情况来定

- `is null` 表示该字段是否允许为空，不指明，默认允许为 `NULL`

- `key` 表示该字段是否是主键，外键，唯一键还是索引

- `default value` 表示该字段在未显示赋值时的默认值

- `extra` 表示其它的一些修饰，比如自增 `auto_increment[=value]`

- `comment` 表示对该字段的说明注释

- `engine` 表示数据库存储引擎，MySQL 支持的常用引擎有 ISAM、MyISAM、Memory、InnoDB 和 BDB，不显示指明默认使用 MyISAM

- `charset` 表示数据表数据存储编码格式，默认为 latin1

### 创建临时表

```sql
create temporary table tablename (
  ...
)
```

- 创建临时表与创建普通表的语句基本是一致的，只是多了一个 `temporary` 关键字
- 临时表的特点是：表结构和表数据都是存储到内存中的，生命周期是当前 MySQL 会话，会话结束后，临时表自动被 drop
- 临时表与 Memory 表的区别是：
  1. Memory 表的表结构存储在磁盘，临时表的表结构存储在内存
  2. `show tables` 看不到临时表，看得到内存表
  3. 内存表的生命周期是服务端 MySQL 进程生命周期，MySQL 重启或者关闭后内存表里的数据会丢失，但是表结构仍然存在，而临时表的生命周期是 MySQL 客户端会话
  4. 内存表支持唯一索引，临时表不支持唯一索引
  5. 在不同会话可以创建同名临时表，不能创建同名内存表

### 创建内存表

```sql
create temporary table tablename (
  ...
) engine=memory
```

### 删除数据表

```sql
drop table tablename
```

### 查看表结构

```sql
desc tablename
```

### 查看建表语句

```sql
show create table tablename
```

### 重命名数据表

```sql
rename table tablename to newtablename
```

### 列操作

- 删除列

```sql
alter table tablename drop column columnname
```

- 增加列

```sql
alter table tablename add column columnname columndefinition

alter table student add column hometown varchar(32)
```

- 重命名列

```sql
alter table tablename change columnname newcolumnname type
```

- 修改列属性

```sql
alter table tablename modify columnname newdefinition
```

`change` 可以更改列名和列类型，但每次都要把新列名和旧列名写上，即使两个列名没有更改

`modify` 只能更改列属性，但只需要写一次列名

### 索引操作

- 增加索引

```sql
alter table tablename add index [indexname](filed1, filed2,...)

alter table student add index index_studentNo(studentNo)
alter table student add index(studentNo)
```

- 查看索引

```sql
show index from tablename
```

- 删除索引

```sql
alter table tablename drop index indexname
```

### 修改存储引擎

```sql
alter table tablename type engine=enginename
```
