## psql
装PostgreSQL的数据库时，会建立一个与初始化数据库时的操作系统用户同名的数据库用户，同时，这个用户是数据库的超级用户，在这个OS用户下，登录数据库时执行的是操作系统认证，所以不需要用户名和密码，当然也可以通过修改`pg_hba.conf`文件来要求输入密码。

### 连接数据库
```bash
psql -h <hostname or ip> -p〈端口〉［数据库名称］［用户名称］
```

可以设置变量
```bash
export PGDATABASE=
export PGHOST=
export PGPORT=
export PGUSER=
```

### 查看数据库
```psql
psql -l 
```

### 连接指定数据库
```psql
\c database_name
```

### 查看表清单
```psql
\d
```

### 查看表的详细信息
```psql
\d table_name
```

### 查看索引的详细信息Postgres-XC
