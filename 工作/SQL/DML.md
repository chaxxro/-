# DML

DML(Data Manipulation Language) 数据操作语言，供用户对数据库中数据的操作，包括数据的增加、删除、更新，载入等操作。常用关键字有：`UPDATE`、`DELETE`、`INSERT INTO`、`LOAD`

### 插入数据

```sql
insert into tablename(column1,column2,...) values(value1,value2,...)

# 如果插入值刚好与数据表的所有列一一对应，那么可以省略书写插入的指定列
insert into tablename values(value1,value2,...)
```

### 删除数据

```sql
delete from tablename where condition
```

### 修改数据

```sql
update tablename set columnname=newvalue,... where condition
```

### 备份数据

- 导出数据库

```sql
mysqldump -u username -p databasename [tablename] > output.sql
```

- 还原数据库

```sql
source file.sql
```

- 导出 csv 文件

```sql
select * from tablename into outfile "file.csv"
select * from tablename into outfile "file.csv" fields terminated BY ',' optionally enclosed by '"' lines terminated by '\n'
```

- 导入 csv 文件

```sql
load data infile "/path/to/file.csv" into table tablename
```
