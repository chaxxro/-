# DCL

DCL(Data Control Language) 数据控制语言，用于对数据库，数据表的访问角色和权限的控制等。常用关键字有：`GRANT`、`REVOKE`、`DENY`

### 新建用户

```sql
create user username@host [identified by password]

create user xx@localhost identified by 'xx'
create user xx@192.168.1.1 identified by 'xx'
create user xx@"%" identified by 'xx'
CREATE USER xx@"%"
```

- `username` 用户名

- `host` 指定该用户可以在哪台主机登陆，本地可以使用 `localhost`，任意主机可以使用 `%`

- `password` 表示用户密码

### 删除用户

```sql
drop user username@host

drop user xx@localhost
```

### 用户授权

```sql
grant privileges on databasename.tablename to username@host

grant select on *.* to xx@'%'
grant all on *.* TO xx@'%'

# 刷新一下权限
flush privileges
```

- `privileges` 是一个用逗号分隔的赋予 MySQL 用户的权限列表，如 `select`、`insert`、`update` 等，如果要授予所有的权限则使用 `all`

- `databasename` 数据库名，`tablename` 表名，如果要授予该用户对所有数据库和表的相应操作权限则可用 `*` 表示，如 `*.*`

- 使用 `grant` 为用户授权时，如果指定的用户不存在，则会新建该用户并授权

设置允许用户远程访问 MySQL 服务器时，一般使用该命令，并指定密码

```sql
grant select on *.* to xx@'%' identified by '123'
```

### 撤销权限

```sql
revoke privileges on databasename.tableanme from username@host

revoke select on *.* from xx@'%'
revoke all on *.* from xx@'%'
```

### 查看权限

```sql
# 从 mysql.user 表中查看
select * from mysql.user where user='username'

# 从授权信息中查看
show grants for username@host
```

### 修改密码

```sql
set password for username@host= password(newpassword)

set password for xx@localhost=password("1234")
# 修改当前用户密码
set password = password("1234")
```
